---
title: "urllib2上传文件"
date: 2013-01-03
description: "urllib2上传文件"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2013/01/03/urllib2-upload-file/]
---

[httplib2](https://code.google.com/p/httplib2/)好像不支持文件上传,无奈不得不回去使用urllib2。

urllib2本身是不支持文件上传的，但是其良好的扩展性使得其实现这一功能非常简单。

>urllib2所有功能都是由一个个handle实现的。

urllib2加入[poster](http://atlee.ca/software/poster/)这个handle将具备上传功能。

```sh
$ pip install poster
```

使用非常简单

```python
from poster.encode import multipart_encode
from poster.streaminghttp import register_openers
import urllib2

#注册poster这个handle
register_openers()

#编码数据
datagen, headers = multipart_encode({"image": open("explame.jpg", "rb")})

request = urllib2.Request("http://www.explame.org/upload", datagen, headers)
#上传文件并获得响应信息
print urllib2.urlopen(request).read()
```
