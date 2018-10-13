---
title: "python的web框架webpy【Templetor】（四）"
date: 2012-08-08
description: "之前我们讲的都是简单的返回文本到浏览器，例如下面将会返回hello word到浏览器"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/08/python-web-framework-webpy-template-4/]
---

之前我们讲的都是简单的返回文本到浏览器，例如下面将会返回hello word到浏览器

```python
# coding:utf-8  
import web  
  
urls=(  
      '/','index'  
)  
  
app=web.application(urls,globals())  
  
class index:  
    def GET(self):  
        return 'hello word!'  
  
if __name__ == '__main__':  
    app.run()  
```

当然我们也可以返回html到浏览器，例如


```python
# coding:utf-8  
import web  
  
urls=(  
      '/','index'  
)  
  
app=web.application(urls,globals())  
  
class index:  
    def GET(self):  
        return ''''' 
            <html> 
                <head> 
                    <title>hello word</title> 
                </head> 
                <body> 
                    <p>hello word!</p> 
                </body> 
            </html> 
        '''  
  
if __name__ == '__main__':  
    app.run()  
```

>显然我们不可能用上述方法来开发web应用，那有没有更加优雅的方法。

>所有的web框架都提供有template技术来实现复杂的web页面webpy也不例外。

在工程目录下建立templates目录（你也可以叫其他名字），这个目录用于存储我们所有的模板文件，
我们在下面建立一个叫做index.html文件内容如下

```html
$def with (name) #这里定义传递进来的参数，后面会提到  
<!DOCTYPE html>  
<html>  
    <head>  
        <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">  
        <title>hello word</title>  
    </head>  
    <body>  
        <p>hello word!$name</p>  
    </body>  
</html>  
```

现在我们就可以使用这个模板了

```python
# coding:utf-8  
import web  
  
urls=(  
      '/','index'  
)  
  
app=web.application(urls,globals())  
render=web.template.render('templates')#模板目录  
  
class index:  
    def GET(self):  
        return render.index('webpy')#使用index.html模板，传递参数'webpy'（这里会查找templates下第一个匹配上index.*的文件，所以我们的模板文件其实可以使用任何扩展名结尾）  
  
if __name__ == '__main__':  
    app.run()  
```

我们也可以指定具体的模板文件

```python
index = web.template.frender('templates/index.html')  
return index('webpy') 
```

我们也可以直接使用字符串为模板

```python
template = ''''' 
    $def with (name) 
    <!DOCTYPE html> 
    <html> 
        <head> 
            <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"> 
            <title>hello word</title> 
        </head> 
        <body> 
            <p>hello word!$name</p> 
        </body> 
    </html> 
'''  
index = web.template.Template(template)  
return index('webpy') 
```

模板种允许使用python表达式,所有表达式都要以$符号开头

```python
Look, a $string.   
Hark, an ${arbitrary + expression}.   
Gawk, a $dictionary[key].function('argument').   
Cool, a $(limit)ing.  
```

还可以声明变量

```python
$ bug = get_bug(id)  
<h1>$bug.title</h1>  
<div>  
    $bug.description  
<div>  
```

若想不被转义则需要在$后面加行：号

```python
$str='1 < 2'  
<p>$:str</p>  
输出则为：  
<p>1 < 2</p>  
```

所有表达式都要以$开头那么要出$就需要用$$

```python
$$50  
输出为  
$50  
```

模板种注释为$#开头

```python
$#我是注释  
<p>hello word</p>  
```

控制结构

```python
$for i in range(10):   
    I like $i  
  
$for i in range(10): I like $i  
  
$while a:  
    hello $a.pop()  
  
$if times > max:   
    Stop! In the name of love.   
$else:   
    Keep on, you can do it.  
```

在循环中可以访问的一些内置变量

```python
loop.index: the iteration of the loop (1-indexed)  
loop.index0: the iteration of the loop (0-indexed)  
loop.first: True if first iteration  
loop.last: True if last iteration  
loop.odd: True if an odd iteration  
loop.even: True if an even iteration  
loop.parity: "odd" or "even" depending on which is true  
loop.parent: the loop above this in nested loops  
```

例如下面

```python
<table>  
$for c in ["a", "b", "c", "d"]:  
    <tr class="$loop.parity">  
        <td>$loop.index</td>  
        <td>$c</td>  
    </tr>  
</table>  
```

你还可以定义方法

```python
$def say_hello(name='world'):  
    Hello $name!  
  
$say_hello('web.py')  
$say_hello()  
```

还可以植入python代码，使用$code

```python
$code:  
    x = "you can write any python code here"  
    y = x.title()  
    z = len(x + y)  
  
    def limit(s, width=10):  
        """limits a string to the given width"""  
        if len(s) >= width:  
            return s[:width] + "..."  
        else:  
            return s  
  
And we are back to template.  
The variables defined in the code block can be used here.  
For example, $limit(x)  
```

当然也可以使用layout

```python
render = web.template.render('templates/', base='layout')  
```

上面会使用 templates/layout.html 作为 layout

layout.html内容：

```html
$def with (content)  
<html>  
<head>  
    <title>Foo</title>  
</head>  
<body>  
$:content  
</body>  
</html>  
```
