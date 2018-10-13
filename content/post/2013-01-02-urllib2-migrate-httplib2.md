---
title: "由urlib2转向使用httplib2"
date: 2013-01-02
description: "由urlib2转向使用httplib2"
category: [ "python" ]
tags: ["python"]
aliases: [/python/2013/01/02/urllib2-migrate-httplib2/]
---

提到python下http相关的库一般都会想到urllib、urllib2、httplib。

*  httplib实现了HTTP和HTTPS的客户端协议，一般不直接使用
*  urllib则是封装了httplib
*  urllib2则可以认为是urllib的升级版

>urllib和urllib2经常一起用

使用非常简单

```python
import urllib2
resp=urllib2.urlopen('http://www.explame.com/')
print resp.read()
```

>更多信息请参考官方文档<http://docs.python.org/2/library/urllib2.html>

---

一切都很顺利，但是最近使用的时候发现抓取部分站点页面的时候偶尔会抛出`connection reset by peer`异常。

开始以为是没有加`user-agent`被目标站点屏蔽了，加上之后问题依旧，又尝试加上`Referer`问题依然存在，最后把cookie也加上了依然无果。

google发现很多人也遇到类似问题，说是网络问题造成的。

我可以肯定网络是没有问题的，浏览器能够打开，程序就会报错（偶尔），百思不得其解。

最后我开始怀疑类库有问题，转而使用[httplib2](https://code.google.com/p/httplib2/)。

**httplib2**是一个第三方的类库,支持python2.3之后的版本,0.5.0之后的版本支持python3,官方首页上列举了诸多特性。

使用也非常简单

```python
import httplib2

h = httplib2.Http()
resp, content = h.request("http://example.org/", "GET")
print content
```

提交表单

```python
import httplib2
import urllib

url = 'http://www.example.com/login'   
body = {'USERNAME': 'foo', 'PASSWORD': 'bar'}
headers = {'Content-type': 'application/x-www-form-urlencoded'}
http = httplib2.Http()
response, content = http.request(url, 'POST', headers=headers, body=urllib.urlencode(body))
```

使用cookie

```python
import urllib
import httplib2

http = httplib2.Http()
url = 'http://www.example.com/login'   
body = {'USERNAME': 'foo', 'PASSWORD': 'bar'}
headers = {'Content-type': 'application/x-www-form-urlencoded'}
response, content = http.request(url, 'POST', headers=headers, body=urllib.urlencode(body))

headers = {'Cookie': response['set-cookie']}

url = 'http://www.example.com/home'   
response, content = http.request(url, 'GET', headers=headers)
```

当我切换到httplib2上，之前的问题再也没有出现过。
