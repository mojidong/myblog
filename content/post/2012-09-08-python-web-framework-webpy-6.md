---
title: "python的web框架webpy 六 "
date: 2012-09-08
description: "webpy中获取get和post参数的方式非常简单"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/09/08/python-web-framework-webpy-6/]
---

webpy中获取get和post参数的方式非常简单

```python
args=web.input()  
name=args.get('name')  
age=args.get('age')  
sex=args.get('sex')  
```

>需要注意的是上面方法获得的参数的类型全是unicode,所以对于数值需要作相应的类型转换才能使用
 
获得上传的文件

```python
args=web.input()  
file=args.get('file')#获得文件  
filename=file.filename#获得文件名称  
content=file.read()#读取文件内容  
```
