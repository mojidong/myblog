---
title: "shell中判断变量为null"
date: 2012-09-08
description: "shell中判断变量为null"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/2012/09/08/shell-determine-variable-is-null/]
---

* 方法一

	```bash
	if [ $x ];then  
   		echo 'not null'  
	else  
   		echo 'is null'  
	fi  
	```
* 方法二

	```bash
	if [ -z "$x" ];  
   		echo 'is null'  
	else  
   		echo 'not null'  
	fi  
	```
