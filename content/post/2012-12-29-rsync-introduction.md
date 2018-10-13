---
title: "rsync命令使用介绍"
date: 2012-12-29
description: "rsync命令使用介绍"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/12/29/rsync-introduction/]
---

rsync是Unix下的一款应用软件，它能同步更新两处计算机的文件与目录，并适当利用差分编码以减少数据传输。rsync中一项与其他大部分类似程序或协定中所未见的重要特性是镜像对每个目标只需要一次传送。rsync可拷贝／显示目录属性，以及拷贝文件，并可选择性的压缩以及递归拷贝。

我们先来看看rsync的用法

```bash
Usage: rsync [OPTION]... SRC [SRC]... DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST:DEST
  or   rsync [OPTION]... SRC [SRC]... [USER@]HOST::DEST
  or   rsync [OPTION]... SRC [SRC]... rsync://[USER@]HOST[:PORT]/DEST
  or   rsync [OPTION]... [USER@]HOST:SRC [DEST]
  or   rsync [OPTION]... [USER@]HOST::SRC [DEST]
  or   rsync [OPTION]... rsync://[USER@]HOST[:PORT]/SRC [DEST]
The ':' usages connect via remote shell, while '::' & 'rsync://' usages connect
to an rsync daemon, and require SRC or DEST to start with a module name.
```

>rsync可以使用自己原生的rsync协议通讯，也可以使用ssh协议通讯。

>如果使用原生协议通讯需要需要启动rsync server,默认监听TCP端口873,我们可以修改server的配置文件调整监听端口和加入认证功能等。

鉴于配置原生服务比较繁琐（可能是我太懒），一般情况下都会直接使用ssh协议进行通讯。

原生协议

```bash
#将本机的me目录同步到192.168.1.100的me目录
rsync -avu /me rsync://admin@192.168.1.100:573/me
#如果是默认端口（573）可以省略不写
rsync -avu /me rsync://admin@192.168.1.100:/me
#另一种写法
rsync -avu /me admin@192.168.1.100::/me
```

ssh协议

```bash
#与原生协议的区别仅在于使用一个冒号
rsync -avu /me admin@192.168.1.100:/me
#如果ssh的端口默认不是22,使用 -e 选项进行设置
rsync -avu -e "ssh -p 8000" /me admin@192.168.1.100:/me
```

### rsync常见用法
---
备份

```bash
rsync -avu /me admin@192.168.2.100:/me
#-a选项等价于 -rlptgoD
#-v显示详细信息
#-u如果目标文件比要备份的文件新则不更新
```

一般同步大的目录时间会比较长，如果网络断掉那就需要断点续传

```bash
rsync -avP /me admin@192.168.2.100:/me
#注意-u选项不能与-P选项共存
```

更多选项请使用`rsync -h`或`man rsync`查看
