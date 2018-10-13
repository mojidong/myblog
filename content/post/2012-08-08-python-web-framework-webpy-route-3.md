---
title: "python的web框架webpy【路由规则】（三）"
date: 2012-08-08
description: "今天重点说下webpy的路由规则。"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/08/python-web-framework-webpy-route-3/]
---

今天重点说下webpy的路由规则。

```python
#这里声明了三条路由规则，它是一个tuple,由url匹配规则和处理类组成  
#url匹配规则是用正则表达式书写的  
#可以声明多条路由规则，每一条都是由url匹配规则和处理类组成  
urls=(  
      '/','index',  
      '/user','user',  
      '/topic','topic'  
)  
```

既然url匹配规则是正则表达式那我们就可以灵活的写出各种表达式

```python
urls=(  
      '/','index',  
      '/(user|member)','user', #匹配 http://example.com/user和http://example.com/member  
      '/topic/?','topic',#匹配 http://example.com/topic和http://example.com/topic/  
      '/blog/(\d+)','blog',#匹配 http://example.com/blog/123  
      '/news/(\d+)/(\w+)','news'#匹配 http://example.com/news/123/abc  
)  
```

正则表达式里的分组可以在后面的处理类中使用，例如

```python
'/news/(\d+)','news'  
#如果url为http://example.com/news/123456  
  
class news:  
    #这里的id就是上面分组里匹配上的值(123456)  
    def GET(self,id):  
        return 'id:%s' % id  
```

>注意：url匹配只匹配url路径不包括参数，例如：

>'/news/create?title=(.+)'

>它不会匹配上http://example.com/news/create?title=hello
