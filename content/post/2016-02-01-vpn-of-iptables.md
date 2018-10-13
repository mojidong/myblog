---
title: "vpn的iptables配置"
date: "2016-02-01"
description: "vpn的iptables配置"
categories: [ "linux" ]
tags: ["linux","iptables"]
aliases: [/linux/2016/02/01/vpn-of-iptables/]
---

之前配置vpn在配置iptables这一块遇到一些问题这里总结一下。 

我的iptables配置如下   

```bash
iptables -A INPUT -i lo -j ACCEPT
iptables -A INPUT -m state --state RELATED,ESTABLISHED -j ACCEPT
iptables -A INPUT -p icmp -m icmp --icmp-type 8 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 80 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 1723 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 6000 -j ACCEPT
iptables -A INPUT -p tcp -m tcp --dport 61234 -j ACCEPT
iptables -t nat -A PREROUTING ! -s 192.168.10.0/24 -p tcp -m tcp --dport 80 -j REDIRECT --to-ports 6000
iptables -t nat -A POSTROUTING -s 192.168.10.0/24 -j MASQUERADE
```

配置完成后vpn连接成功。
