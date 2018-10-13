---
title: "python的web框架webpy(一) "
date: 2012-08-08
description: "python的web框架webpy(一) "
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/08/python-web-framework-webpy-1/]
---

python的web框架是一个百花齐放的世界，完全列表请看[这里](http://wiki.python.org/moin/WebFrameworks)。
 
面对如此众多的框架我们要如何选择！它们都有各自的优缺点，你不可能找到一款完美的，其实只需要选择适合的即可！
 
这里介绍[webpy](http://webpy.org/),进入官方首页你可以看到右边有一个hello word的例子


```python
import web  
          
urls = (  
    '/(.*)', 'hello'  
)  
app = web.application(urls, globals())  
  
class hello:          
    def GET(self, name):  
        if not name:   
            name = 'World'  
        return 'Hello, ' + name + '!'  
  
if __name__ == "__main__":  
    app.run()  
```

没错就是这么简单，相信你也会喜欢上他

1. 下载安装包，点击[这里](http://webpy.org/static/web.py-0.37.tar.gz)下载（我这里下载的是web.py-0.37.tar.gz）。
2. 安装webpy

	```
	# tar xzvf web.py-0.37.tar.gz  
	# cd web.py-0.37  
	# python setup.py install 
	```
	
3. 写一个自己的程序

	创建一名为server.py的文件，内容如下
	
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
    
    在终端运行
    
    `# python server.py`
    
    浏览器打开http://localhost:8080,你可以在页面上看到hello webpy!
    
    >你可以在启动的时候设置监听的端口，改变默认的8080端口
    
    >`# python server.py 80`
    
