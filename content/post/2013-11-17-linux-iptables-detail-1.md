---
title: "linux防火墙iptables详解(一)"
date: 2013-11-17
description: "linux防火墙iptables详解"
categories: ["linux"]
tags: [linux]
aliases: [/linux/2013/11/17/linux-iptables-detail-1/]
---

今天详细讲下linux防火墙iptables。

## 1.表和链

iptables有如下几个表：


* mangle表主要功能是修改数据包
* nat表主要用来修改包的源地址和目标地址
* filter表主要用来对包进行过滤

> 对于mangle一般不建议使用，会出现一些意想不到的问题，如果你愿意完全可以忘掉mangle的存在，后面主要围绕nat和filter讲解。

#### 1.1 mangle
说了忘掉的。

#### 1.2 nat
nat有如下几条链

* PREROUTING 对进来的包进行处理
* POSTROUTING 对出去的包进行处理
* OUTPUT 对本地产生的出去的包进行处理
	
#### 1.3 filter
filter有如下几条链

* INPUT 对进来得包进行处理
* OUTPUT 对出去的包进行处理
* FORWARD 对转发的包进行处理

## 2.规则

一条规则一定是挂在某条链上的，一条链一定是挂在某张表上的，所以一条规则一定是挂在某张表的某条链上的。

```
table
  |
  ---chain
    	|
  		---rule
```
#### 2.1 语法

新增一条规则需要使用`iptables`命令（需要root权限）。

我们先来看下`iptables`的基本语法

```bash
iptables [-t table] {-A|-D} chain rule-specification           #添加或删除规则

iptables [-t table] -I chain [rulenum] rule-specification      #插入规则

iptables [-t table] -R chain rulenum rule-specification        #替换规则

iptables [-t table] -D chain rulenum                           #删除规则

iptables [-t table] -S [chain [rulenum]]                       #列出某条链下的规则

iptables [-t table] {-F|-L|-Z} [chain [rulenum]] [options...]  #-F删除规则，-L列出规则，-Z清空数据包计数

iptables [-t table] -N chain                                   #创建链，一般用于自定义链

iptables [-t table] -X [chain]					                #删除链

iptables [-t table] -P chain target			                #设置链的默认行为

iptables [-t table] -E old-chain-name new-chain-name          #编辑规则

rule-specification = [matches...] [target]                    #规则语法

match = -m matchname [per-match-options]                      #规则语法matches部分

target = -j targetname [per-target-options]                   #规则语法target部分
```

> 参数table是可选的，如果没有设置table默认则是filter。

#### 2.2 初始化

在添加规则之前我们一般会初始化防火墙

```bash
iptables -P INPUT ACCEPT   #设置filter表INPUT链的默认行为为ACCEPT
iptables -P OUPUT ACCEPT   #设置filter表OUTPUT链的默认行为为ACCEPT
iptables -F                #删除filter表所有规则
iptables -X                #删除filter表所有自定义的链
iptables -Z                #清空包计数器
```

> 这里有个值得注意的地方就是一定要记得设置`filter`的`INPUT`,`OUTPUT`的默认行为为`ACCEPT`,因为如果不设置，后面的命令把所有规则删除了，这个时候就会走到链的默认行为，如果之前的默认行为是`DROP`或`REJECT`那就会把自己挡在外面（假定你是ssh登录），到时候真是欲哭无泪。

#### 2.3 添加规则

```bash
iptables [-t table] -A chain rule-specification 
rule-specification = [matches...] [target] 
target = -j targetname [per-target-options]
```

根据上面的语法我们得知规则就是`matches`和`target`。

>一旦`matches`成功就执行后面的`target`，如果不成功则继续下一条规则。

例如开放22端口

```bash
iptables -A INPUT -p tcp -m tcp --dport 22 -j ACCEPT
```

这里的`matches`就是`-p tcp -m tcp --dport 22`，`target`就是`-j ACCEPT`

> target比较常用的有`ACCEPT`,`DROP`,`REJECT`

我们还可以修改`matches`以达到更精确的控制

```bash
iptables -A INPUT -s 192.168.1.0/24 -p tcp -m tcp --dport 22 -j ACCEPT
```

上面命令仅当网段是`192.168.1.1-225`的ip访问22端口时才会匹配，所以你可以不断调整你的`matches`来达到你的目的。

---
说到这里你可能会有疑问了，如果所有的规则都没匹配上怎么办。还记得之前说的链的默认规则吗？

我们之前把`INPUT`设置为了`ACCEPT`，之所以把它设置为`ACCEPT`是为了避免我们设置规则的时候把自己关在外面，当你把所有规则设置好之后确认没有任何问题之后记得把它设置为`DROP`，这个时候你的防火墙才真真正正有效了。

```bash
iptables -P INPUT DROP
```






