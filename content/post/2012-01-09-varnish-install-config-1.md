---
title: "varnish安装及配置（一）"
date: 2012-01-09
description: "varnish是一款高性能的开源HTTP加速器，用来缓存静态文件（图片,js,css）以减少后端服务器压力,性能要比squid高很多。"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/01/09/varnish-install-config-1/]
---

varnish是一款高性能的开源HTTP加速器，用来缓存静态文件（图片,js,css）以减少后端服务器压力,性能要比squid高很多。

1. 下载varnish,https://www.varnish-cache.org/  #建议下载最新稳定版
2. 编译安装

```
tar xzvf varnish-3.0.2.tar.gz
cd varnish
sh autogen.sh
sh configure  #一些可选参数
make
make install
```

3. 启动varnish,通过varnishd命令来启动varnish服务

`varnishd -a 0.0.0.0:80 -s malloc,10g -t 2592000 -w 200,4000,300 -h classic,300000 -p thread_pools=10 -p session_linger=100 -p listen_depth=4096 -p lru_interval=3600 -p sess_workspace=9437184 -p http_resp_size=4194304 -p thread_pool_workspace=9437184 -u admin -g admin  -f /path/varnish.vcl`

下面分析下上面主要的几个参数：

```
-a 0.0.0.0:80	#设置varnish监听本机80端口的请求

-s malloc,10g	#分配10G的内存用于缓存

-t 2592000	#设置缓存对象过期时间为30天

-w 200,4000,300	#设置线程池中最小线程和最大线程数及线程空闲时间

-h classic,300000	#设置hash算法classic,bucket推荐为缓存对象数的10倍默认为16383,simple_list算法不推荐生产环境使用,critbit算法是一个几乎无锁的树

-p thread_pools=10	#设置线程池大小

-p session_linger=100	#让session重用的时间,重用session可提高性能，这个值设的太大如果没有重用就浪费资源，如果太小重用率太低（大家自己权衡设置）

-p listen_depth=4096	#监听队列的深度

-p sess_workspace=9437184	#session工作内存的大小，vcl操作中需要用到这些内存

-p http_resp_size=4194304	#后端请响应允许的最大字节数，这个的内存就是从上面的sess_workspace分配的

-p thread_pool_workspace=9437184	#设置线程池内存大小，vcl中处理请求和响应将用到

-u admin	#以admin用户运行varnish服务

-g admin	#以admin组运行varnish服务

-f /path/varnish.vcl	#指定vcl配置
```

上述的性能参数大家可根据自己的实际情况调整，性能相关参数请参考

<https://www.varnish-cache.org/docs/3.0/reference/varnishd.html>
