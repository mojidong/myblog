---
title: "chrome extension show ip"
date: 2013-11-05
description: "chrome extension show ip"
categories: [ "html/css/js" ]
tags: ["chrome"]
aliases: [/html/css/js/2013/11/05/chrome-extension-show-ip/]
---

最近开发了一个chrome插件[show ip](https://chrome.google.com/webstore/detail/show-ip/nfdipejbaaclaioanjipeblnbplaeabl)用来显示请求页面的ip地址。

### 起因

最近经常会在测试环境和线上环境切换（修改host），由于chrome有dns缓存，以至于不确定是否已经切换到相应环境，chrome的开发工具网络里没有显示ip，又不想换firefox，于是到chrome store里寻找下看有没有相应的扩展，找了下还比较多，试用了几款感觉多多少少有点小问题，于是决定自己搞一个。

### 结果

经过一天的折腾就有了现在的[show
ip](https://chrome.google.com/webstore/detail/show-ip/nfdipejbaaclaioanjipeblnbplaeabl)

当然源码是必须得[猛戳这里](https://github.com/mojidong/show_ip)
