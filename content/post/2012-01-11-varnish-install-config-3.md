---
title: "varnish安装及配置（三）"
date: 2012-01-11
description: "varnish通过几个辅助命令行工具观察varnish的工作情况"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/01/11/varnish-install-config-3/]
---

varnish通过几个辅助命令行工具观察varnish的工作情况

**varnishlog:**

varnish的日志是写入共享内存的，可以使用varnishlog命令行工具读取

`[admin@localhost ~]$ varnishlog -c //上面的命令将输出客户端的请求信息`

```
195 RxRequest    c GET  
195 RxURL        c /BD/310x85-2.jpg  
195 RxProtocol   c HTTP/1.1  
195 RxHeader     c Accept: */*  
195 RxHeader     c Referer: http://www.example.com/index.html  
195 RxHeader     c Accept-Language: zh-cn  
195 RxHeader     c Accept-Encoding: gzip, deflate  
195 RxHeader     c User-Agent: Mozilla/4.0 (compatible; MSIE 6.0; Windows NT 5.1; SV1; QQDownload 679)  
195 RxHeader     c Host: www.example.com  
195 RxHeader     c Connection: Keep-Alive  
195 VCL_call     c recv lookup  
195 VCL_call     c hash  
195 Hash         c /BD/310x85-2.jpg  
195 Hash         c www.explame.com  
195 VCL_return   c hash  
195 Hit          c 1071138920  
195 VCL_call     c hit deliver  
195 VCL_call     c deliver deliver  
195 TxProtocol   c HTTP/1.1  
195 TxStatus     c 200  
195 TxResponse c OK  
195 TxHeader     c Content-Type: image/jpeg  
195 TxHeader     c Last-Modified: Sat, 31 Dec 2011 09:26:44 GMT  
195 TxHeader     c Expires: Wed, 11 Jan 2012 14:19:50 GMT  
195 TxHeader     c Cache-Control: max-age=3600  
195 TxHeader     c Server: lighttpd/1.4.20  
195 TxHeader     c Content-Length: 22214  
195 TxHeader     c Accept-Ranges: bytes  
195 TxHeader     c Date: Wed, 11 Jan 2012 14:02:55 GMT  
195 TxHeader     c Age: 2585  
195 TxHeader     c Connection: keep-alive  
195 TxHeader     c X-Cache: 27238  
195 Length       c 22214  
195 ReqEnd       c 1071326704 1326290575.867818117 1326290575.867902040 1.986548185 0.000028849 0.000055075
```

`[admin@localhost ~]$ varnishlog -b //显示varnish请求后端的信息，和上面的现实相似`

`[admin@localhost ~]$ varnishlog -i RxURL //显示出所有请求的url的信息，-i 参数指定了具体要显示的项目，项目名称就是上面显示信息中`

还有很多有用的参数，大家可以通过varnishlog -h 查看其他参数的作用，或者参考：<https://www.varnish-cache.org/docs/3.0/reference/varnishlog.html>

**varnishncsa:**

此命令输出的日志类似apache 形式的日志，大部分参数和varnishlog 类似，其中有好多参数使用时会提示`-x is not valid for varnishncsa //x代表具体参数`估计是bug

**varnishstat:**

这个命令可能是我们用的最多的也是最有用的命令，它可以统计varnish的很多信息，包括缓存命中次数，未命中次数，请求数，缓存大小等。

`[admin@localhost ~]$ varnishstat `

![image](http://lh5.googleusercontent.com/-JO4bLN7QqwI/UMCzqiTaQaI/AAAAAAAAAFA/U-QVjBp4LoI/s1600/b83740c1-9411-3f89-b131-e2c062a643ab.png)

**下面介绍各个数据的含义：**

* 第一行显示的是varnish自启动到现在运行了多长时间，上面显示的27天7小时30分17秒
* 第二行显示的是启动这个命令的时间，这个三数字最终会变为10，100，1000；分别代表10秒，100秒，1000秒
* 第三行显示的是命中率，分别对象上面的时间，分别是10秒内的命中率，100秒内的命中率，1000秒内的命中率


从第四行开始下面的数据就分为4列

第一列为总数值，第二列为每秒中的数值，第三列自命令（varnishstat）启动以来的平均值，第四列是描述

**其中几个比较重要的数据是**

* cache-hit ：代表缓存命中次数
* miss-hit   ：代表未命中次数
* worker threads ：代表当前工作线程的数量
* expired objects ：代表过期对象的个数
* LRU nuked objects ：代表缓存可使用的内存以达上线而不得不移除的对象个数
* LRU moved objects ：代表LRU策略被移动的对象个数
* Total header bytes ：代表缓存的请求头对象的大小
* Total body bytes	：代表缓存的请求体对象大小

`[admin@localhost ~]$ varnishstat -1 //将显示所有的统计数据 `

还有一些其他的辅助命令行工具请参考：<https://www.varnish-cache.org/docs/3.0/reference/index.html>

