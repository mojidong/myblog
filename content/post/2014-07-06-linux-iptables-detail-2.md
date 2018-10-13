---
title: "linux防火墙iptables详解(二)"
date: 2014-07-06
description: "linux防火墙iptables详解"
categories: ["linux"]
tags: ["linux"]
aliases: [/linux/2014/07/06/linux-iptables-detail-2/]
---

之前讲解了iptables基本语法和用法，今天详细讲下iptables规则链的整个执行流程。

##规则链执行流程

先上一张图（借用下[鸟哥](http://linux.vbird.org/)的图）

![iptables规则执行流程](/assets/img/iptables.gif)

从上图可以很清晰的看到数据包在规则链中的流动路径:

    数据包先经过mangle表的PREROUTING链,然后进入nat表的PREROUTING链,然后路由进行判定:
    1. 如果目标是本机则进入mangle表的INPUT链,在进入filter表的INPUT链到达本机（应用层接受数据), 最后数据经由mangle表的OUTPUT链,nat表OUTPUT链,filter表OUTPUT链
    2. 如果目标不是本机且开启了forward则数据包进入mangle表FORWARD链,然后到filter表FORWARD链
    最终都到达mangle表POSTROUTING链,经过nat表POSTROUTING链传出

根据上图的数据包的流动情况现在我们可以很容易的添加规则到对应的链路来满足我们的需求,下面是我们常用的一些场景:

* 修改数据包的来源ip地址及端口可以在nat表的PREROUTING链添加规则
* 过滤未授权及非法ip来源及端口的数据包到达本机可以在filter表的INPUT链添加规则
* 过滤未授权及非法ip来源及端口的数据包forward可以在filter表的FORWARD链添加规则
* 修改出口数据包的ip地址及端口可以在nat表的POSTROUTING表添加规则(充当代理服务器)

> 一般来说我们接触并使用最多的是filter表的INPUT链和nat表的PREROUTING,POSTROUTING链

下面是一些简单的列子

```bash
iptables -i lo -j ACCEPT #开放loopback (就是对127.0.0.1的访问)
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT #对新的链接或与现有链接有关联，已经建立的链接的数据包开放
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT # 开放icmp type 为8的数据包可以到达本机
iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT # 开放本机22端口
iptables -A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -j ACCEPT #对来源ip在192.168.1.1~254范围开放22端口
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT # 开放本机的80端口
iptables -t nat -A POSTROUTING -j MASQUERADE # 从本机出去的数据包源ip更改为本机（一般是作为路由）
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 6000 # 把访问本机80端口数据包重定向到6000端口
```

## 实战
需求：

1. 开放本机的22端口只允许ip为172.30.22.11的ip可以访问
2. http服务器只监听了6000端口,要求通过80端口可以访问到
3. 禁止在192.168.2.1~192.168.2.254范围内的ip访问80端口
4. 可以从内部访问外网
5. 本机可以作路由使用使用

根据上述要求结合前面学到的只是我们马上写下了如下code

```bash
iptables -P INPUT ACCEPT   #设置filter表INPUT链的默认行为为ACCEPT
iptables -P OUPUT ACCEPT   #设置filter表OUTPUT链的默认行为为ACCEPT
iptables -F                #删除filter表所有规则
iptables -X                #删除filter表所有自定义的链
iptables -Z                #清空包计数器
iptables -A INPUT -s 172.30.22.11 -p tcp -m tcp --dport 22 -j ACCEPT
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 6000 # 把访问本机80端口数据包重定向到6000端口
```

最后更改默认规则为DROP

```bash
iptables -P INPUT DROP
```

测试发现需求1满足了，需求2没有，访问80端口无响应。这是为什么呢？

回头看看图,原来后面还有filter的INPUT链没作处理,加入如下规则

```bash
iptables -A INPUT -p tcp -m tcp --dport 80 -J ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 6000 -J ACCEPT
```

测试ok

要满足3我们需要改下之前的规则,如下

```bash
iptables -A INPUT -s ! 192.168.1.0/24 -p tcp -m tcp --dport 80 -j ACCEPT
```

测试ok

在服务器上访问外网发现不行,显然4没有满足,回头看图发现OUTPUT这一条路是通的没有问题,数据包是传出去了(可以通过tcpdump抓包验证),问题发生在回来的时候数据包被挡在了外面。

分析发现服务器访问外网发起请求会随机分配一个端口(tcp特性)，外网服务回传的的数据包会回应到这个端口,而我们前面就开了80和6000其他端口都是被拒绝的

原因知道了但是发现不好改啊，端口是随机的。看来只能换一个思路,我们可以根据链接状态来判定

```bash
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT #对新的链接或与现有链接有关联，已经建立的链接的数据包开放
```

测试ok

现在整个规则如下

```bash
iptables -P INPUT ACCEPT   #设置filter表INPUT链的默认行为为ACCEPT
iptables -P OUPUT ACCEPT   #设置filter表OUTPUT链的默认行为为ACCEPT
iptables -F                #删除filter表所有规则
iptables -X                #删除filter表所有自定义的链
iptables -Z                #清空包计数器
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -s 172.30.22.11 -p tcp -m tcp --dport 22 -j ACCEPT
iptables -A INPUT -s ! 192.168.1.0/24 -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -s -p tcp -m tcp --dport 6000 -j ACCEPT
iptables -t nat -A PREROUTING -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 6000 # 把访问本机80端口数据包重定向到6000端口
iptables -P INPUT DROP
```

需求5嘛就留个坑吧!
