---
title: "迁移博客到hugo"
date: 2018-10-14T12:53:58+08:00
description: "迁移博客到hugo"
categories: ["other"]
tag: ["other"]
---

这两天把blog从[jekyll](https://jekyllrb.com/)迁移到了[hugo](https://gohugo.io/)。下面是hugo对自已的简介：

> The world’s fastest framework for building websites.

> Hugo is one of the most popular open-source static site generators. With its amazing speed and flexibility, Hugo makes building websites fun again.

### 这里记录下期间遇到的问题

- 代码高亮  
这里花了一点时间，按照文档进行高亮风格设置之后发现显示有问题，最后发现是选用主题之后高亮风格只能随主题走。这个坑是最后查看主题的repo才发现的，建议大家遇到问题的时候查看选用的主题文档和repo。

- 历史链接  
尝试将文章链接调整为原blog一样，这样外链和搜索引擎还可以正常工作，结果不幸的是[hugo](https://gohugo.com/)的`Permalinks`支持不子之前的url模式，查看文档发现可以用`aliases`对老的url进行重定向。这种方式比较繁琐需要每个页面都去设置一遍。
