---
title: "python的web框架webpy(二)"
date: 2012-08-08
description: "python的web框架webpy(二)"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/08/python-web-framework-webpy-2/]
---

之前介绍了webpy，还写了一个自己的web程序，下面我们就来分析下代码。

```python
# coding:utf-8  
import web  
  
urls=(  
      '/','index'  
)  
  
app=web.application(urls,globals())  
  
class index:  
    def GET(self):  
        return 'hello webpy!'  
  
if __name__=='__main__':  
    app.run()  
```

---
```python
import web #导入webpy的模块
```

```python
#这里声明了路由规则，它是一个tuple,由url匹配规则和处理类组成  
#url匹配规则是用正则表达式书写的  
urls=(  
      '/','index' #所有匹配上/的url都交由index类来处理  
)  
#可以声明多条路由规则，每一条都是由url匹配规则和处理类组成  
urls=(  
      '/','index',  
      '/user','user',  
      '/topic','topic'  
)  
```

```python
#初始化webapp  
app=web.application(urls,globals())  
```

```python
#处理类index，它需要实现GET或POST方法  
#return 可以返回你的处理结果到用户浏览器  
class index:  
    def GET(self):  
        return 'hello webpy!'  
```

```python
#启动webapp  
if __name__=='__main__':  
    app.run()  
```
