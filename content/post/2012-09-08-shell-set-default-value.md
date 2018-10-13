---
title: "shell中给变量设置默认值"
date: 2012-09-08
description: "shell中给变量设置默认值"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/09/08/shell-set-default-value/]
---

通常shell中我们需要给变量设置默认值，可能会写出如下代码

```bash
#!/bin/bash  
if [ ! $1 ]; then  
       $1='default'  
fi  
```

显然这种方式在变量少的时候没啥问题，一旦变量多起来，我们可能就有大量的重复劳动(if判断)
 
有没有比较优雅的方式，不用写一大堆if判断，显然答案是有的

1. 变量为null时

	```bash
	#当变量a为null时则var=b  
	var=${a-b}  
	```
2. 变量为null且为空字符串的时候

	```bash
	#当变量a为null或为空字符串时则var=b  
	var=${a:-b}  
	```
