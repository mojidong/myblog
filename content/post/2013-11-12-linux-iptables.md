---
title: "linux防火墙iptable使用简介"
date: 2013-11-12
description: "linux防火墙iptable使用简介"
categories: [ "linux" ]
tags: ["linux", "iptables"]
aliases: [/linux/2013/11/12/linux-iptables/]
---

iptables是Linux上常用的防火墙软件。这里简单介绍下怎么使用，依葫芦画瓢就可以了，对于懒人来说足够了。如果你想更详细的了解iptables,以后会详细的讲解。

> iptables命令要在root下执行

###初始化防火墙

```bash
iptables -P INPUT ACCEPT    # 设置INPUT chain默认行为为接受
iptables -P OUTPUT ACCEPT   # 设置OUTPUT chain默认行为为接受
iptables -P FORWARD ACCEPT  # 设置FORWARD chain默认行为为接受
iptables -F                 # 删除所有chain的rule
iptables -Z                 # 封包计数清零
iptables -X					# 删除所有自定义的chain
```

经过上述操作之后你的防火墙现在没有任何规则，也就说你的机器完全暴露在了网络之中，跟没穿衣服没两样。

###新增规则

对于进来的封包过滤的规则都是在INPUT chain上设置的。

* 基本配置

	```bash
	iptables -A INPUT -i lo -j ACCEPT # 接受本地会回环（127.0.0.1）
	iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT # 接受已建立的	联建和与已建立的链接有关系的链接
	iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT # 接受回显请求的ping
	iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT # 开放ssh端口
	```
	> 如果你的ssh端口不是22，请修改为你相应的端口。
	> ssh端口是一定要开放的，不然你就会把自己关在外面，到时候欲哭无泪哦。
* 端口配置

	这里是重点了，开放端口的语法如下
	
	```bash
	iptables -A INPUT -p tcp -m tcp --dport 端口 -j ACCEPT
	iptables -A INPUT -p tcp -m udp --dport 端口 -j ACCEPT
	```
	
	例如

	```bash
	iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
	iptables -A INPUT -p tcp -m tcp --dport 3306 -j ACCEPT
	iptables -A INPUT -p tcp -m udp --dport 3721 -j ACCEPT
	```
	
	要开放什么端口按这样加就好了

###切换默认策略

把衣服穿起来


```bash
iptables -P INPUT DROP    # 设置INPUT chain默认行为为丢弃
```

如果之前的规则一个都没匹配上最终这里的默认行为会触发。

###查看规则

```bash
iptables -L -n
或者
iptables-save
```

个人比较喜欢用`iptables-save`

###保存规则

如果你是redhat或centos

```bash
service iptables save #保存规则
```

如果是其他系统你可以利用`iptables-save`保存规则，`iptables-restore`恢复规则

```bash
iptables-save > /path/to/iptables   #将规则保存到指定文件中
iptables-restore /path/to/iptables  #将规则从之前的文件中恢复
```
