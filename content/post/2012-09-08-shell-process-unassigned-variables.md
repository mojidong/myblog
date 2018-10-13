---
title: "处理shell中未赋值变量造成的灾难"
date: 2012-09-08
description: "在编写shell的时候我们都需要万分小心，尤其是给root执行的脚本"
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/09/08/shell-process-unassigned-variables/]
---

在编写shell的时候我们都需要万分小心，尤其是给root执行的脚本

```bash
#!/bin/bash  
....  
rm -rf $1/$2/bin/  
.... 
```

假设上述脚本我执行的时候没有传递参数，造成的后果是相当恐怖的。
 
 
有没有好的办法解决这个问题，答案是有的

```bash
#!/bin/bash  
....  
if [ ! $1 ];then
   echo '$1 is null'  
   exit 1  
fi  
if [ ! $2 ];then 
   echo '$1 is null'  
   exit 1  
fi 
rm -rf $1/$2/bin  
....  
```

难道我们在使用变量之前，都要判断是否为null，这样太繁琐了，有没有更好的解决办法，答案依然是有的

```bash
#!/bin/bash  
set -u  
....  
rm -rf $1/$2/bin  
....  
```

>当设置set -u 后，使用未赋值的变量时shell将自动退出
