---
title: "nginx的location匹配规则详解"
date: 2013-04-15
description: "nginx的location匹配规则详解"
categories: [ "nginx" ]
tags: ["nginx"]
aliases: [/nginx/2013/04/15/nginx-location-rule/]
---

nginx中我们会用location指令来匹配指定的uri进行一些逻辑处理。

location指令语法

```nginx
location [ = | ~ | ~* | ^~ ] uri { ... }
```
location 支持常规字符串和正则表达式两种匹配模式。

* `=` 提高常规字符串匹配优先级
* `~` 正则匹配，大小写敏感
* `~*` 正则匹配，大小写不敏感 
* `^~` 提高常规字符串匹配优先级

### 匹配顺序

1. 首先匹配`=`前缀的常规字符串，如果匹配成功则停止后续匹配。
2. 其次匹配`^~`前缀的常规的字符串，如果匹配成功则停止后续匹配。
3. 再次匹配`~`或`~*`前缀的正则表达式，如果匹配成功则停止后续匹配。
4. 最后匹配不带任何前缀的常规字符串，如果匹配成功则停止后续匹配。
5. 上述都未匹配上则返回404。
 
> 多个正则表达式的匹配优先级是由在文件中的物理顺序决定的。

下面举例说明

```nginx
location / {
	# ...
}

location =/index.html {
	# ...
}

location ~* .*\.html {
	# ...
}

location ^~ /list.html {
	# ...
}

location /about.html {
	# ...
}
```

如果请求的uri是`/index.html`，根据之前的规则`=`前缀的常规字符串被优先匹配（第二个location配置）。

如果请求的uri是`/list.html`，根据之前的规则`^~`前缀的常规字符串被优先匹配（第三个location配置）。

如果请求的uri是`/about.html`，根据之前的规则`~`或`~*`前缀正则表达式被优先匹配（第四个location配置），
第五个location配置实际上不会起作用,因为正则匹配的优先级高于它。

如果请求的uri是`/robots.txt`，根据之前的规则常规字符串会被匹配（第一个location配置）。

### 总结
**`=`总是被优先匹配，其次是`^~`，再次是`~`和`~*`,最后才是没有任何前缀的常规字符串。**
