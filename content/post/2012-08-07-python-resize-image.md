---
title: "python修改图片尺寸"
date: 2012-08-07
description: "python对图片的操作需要使用到(Python Imaging Library)PIL库，PIL可以处理几乎所有的图片类型，了解更多请点击[这里](http://www.pythonware.com/products/pil/)。
"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/07/python-resize-image/]
---

>python对图片的操作需要使用到(Python Imaging Library)PIL库，PIL可以处理几乎所有的图片类型，了解更多请点击[这里](http://www.pythonware.com/products/pil/)。

1. 下载PIL并安装
	>windows：直接下载对应的安装包
	
	>linux：
	
	```bash
	$ yum install python-imaging  
	或  
	$ sudo apt-get install python-imaging  
	或  
	直接通过源码安装
	```
2. 使用

	```python
	import Image  
	img = Image.open('example.jpg')  
	new_img= img.resize((w, h),Image.ANTIALIAS) # w代表宽度，h代表高度，最后一个参数指	定采用的算法  
	nwe_img.save('example-new.jpg',quality=100)
	```
    
