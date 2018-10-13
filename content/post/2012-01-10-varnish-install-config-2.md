---
title: "varnish安装及配置（二）"
date: 2012-01-10
description: "Varish Configuration Language 简称VCL，通过它我们可以完成一些复杂的逻辑处理。下面将详细介绍："
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2012/01/10/varnish-install-config-2/]
---

Varish Configuration Language 简称VCL，通过它我们可以完成一些复杂的逻辑处理。下面将详细介绍：

**Backend declarations:**

```
backend www {  
  .host = "www.example.com";  
  .port = "80";  
  .connect_timeout = 1s;  
  .first_byte_timeout = 5s;  
  .between_bytes_timeout = 2s;  
}
```

或者

```
backend www {  
  .host = "www.example.com";  
  .port = "80";  
}
```

*.host和.port为必填项*

**Directors:**

varnish支持负载均衡，其中有random,client.hash,round-robin,dns,fallback等算法

```
director b2 random {//设置调度算法为random  
  .retries = 5;  
  {  
    // 这里可以引用之前定义的backend  
    .backend = b1;  
    .weight  = 7;  
  }  
  {  
    // 也可以直接定义backend  
    .backend  = {  
      .host = "www.example.com";  
      .port="80";  
    }  
   .weight  = 3;  
  }  
}
```

```
director b2 round-robin {//设置调度算法为round-robin  
  {  
    .backend = b1;//引用之前定义的backend  
  }  
  {  
    .backend = b2;//引用之前定义的backend  
  }  
}
```

其他调度算法的设置都大同小异具体可参考官方文档

**ACLs：**

```
acl local {  
  "localhost";           
  "192.0.2.0"/24;       
  ! "192.0.2.23";       
}
```

在vcl子程序中使用(子程序将在下面讲到)

```
if (client.ip ~ local) {//来源ip符合上面定义则直接访问后端服务器，不缓存  
  return (pass);  
}
```

**Regular Expressions：**

vcl中还支持正则表达式

```
if (req.http.host ~ "(?i)example.com$") {  
        //一些逻辑  
}
```

**Functions：**

hash_data(str) //主要用来生成缓存对象的hash key
 
regsub(str, regex, sub) //替换字符换str中匹配上正则表达式的内容为sub字符串，只替换一次
 
regsuball(str, regex, sub) //同上，不过这个方法是替换所有匹配的
 
ban(ban expression) //使符合表达式的对象不被缓存
 
ban_url(regex) //使符合的url的对象不被缓存

**Subroutines：**

子程序这个说白了就是方法。
我们可以把一个请求分为多个阶段，每个阶段都会调用不同的方法，这样我们只要写出相应阶段的方法，我们的方法就会在相应的阶段被执行，想想这有多么美妙，我们可以干很多事情，我们对请求完全可控！

说白就是方法名称别人定义好了返回类型别人也定义好了你只需要填写你的逻辑就OK了！
下面介绍各个阶段的方法

* vcl_init
	
	当vcl被load的时候被调用，一般情况下你基本不需要更改这里，当然你有这个需求那就另说
	
* vcl_recv

	当请求到达的时候被调用，我们大部分的逻辑都在这里，访问控制，是否需要缓存等
	
* vcl_pipe
	
	管道模式下被调用
	
* vcl_pass
	
	请求直接到后端时被调用
	
* vcl_hash

	缓存对象生成hash key时被调用
	
* vcl_hit

	缓存命中时调用

* vcl_miss

	缓存未命中时调用
	
* vcl_fetch

	后端请求成功返回时调用
	
* vcl_deliver

	缓存对象被传递到客户端前被调用
	
* vcl_error

	当其他方法但会error时被调用
	
* vcl_fini

	请求结束时调用
	
---
从上面几个方法大家可以看出一个请求需要经过好几个阶段，下面的图清晰的说明一个请求在不同的条件下所要经过的阶段，每个阶段都有不同的内置变量可以使用，所有精华都在这张图中！

![image](http://lh5.googleusercontent.com/-yIUFZCAN7Xo/UMCsBvVi0KI/AAAAAAAAAEs/fT65P-D2TP0/s1600/084ffd42-3795-380d-92eb-bc902d034373.png)

---
下面贴几个官方的例子

Example

```
backend default {  
 .host = "www.example.com";  
 .port = "80";  
}  
  
sub vcl_recv {  
    if (req.restarts == 0) {  
        if (req.http.x-forwarded-for) {  
            set req.http.X-Forwarded-For =  
                req.http.X-Forwarded-For + ", " + client.ip;  
        } else {  
            set req.http.X-Forwarded-For = client.ip;  
        }  
    }  
    if (req.request != "GET" &&  
      req.request != "HEAD" &&  
      req.request != "PUT" &&  
      req.request != "POST" &&  
      req.request != "TRACE" &&  
      req.request != "OPTIONS" &&  
      req.request != "DELETE") {  
        /* Non-RFC2616 or CONNECT which is weird. */  
        return (pipe);  
    }  
    if (req.request != "GET" && req.request != "HEAD") {  
        /* We only deal with GET and HEAD by default */  
        return (pass);  
    }  
    if (req.http.Authorization || req.http.Cookie) {  
        /* Not cacheable by default */  
        return (pass);  
    }  
    return (lookup);  
}  
  
sub vcl_pipe {  
    # Note that only the first request to the backend will have  
    # X-Forwarded-For set.  If you use X-Forwarded-For and want to  
    # have it set for all requests, make sure to have:  
    # set bereq.http.connection = "close";  
    # here.  It is not set by default as it might break some broken web  
    # applications, like IIS with NTLM authentication.  
    return (pipe);  
}  
  
sub vcl_pass {  
    return (pass);  
}  
  
sub vcl_hash {  
    hash_data(req.url);  
    if (req.http.host) {  
        hash_data(req.http.host);  
    } else {  
        hash_data(server.ip);  
    }  
    return (hash);  
}  
  
sub vcl_hit {  
    return (deliver);  
}  
  
sub vcl_miss {  
    return (fetch);  
}  
  
sub vcl_fetch {  
    if (beresp.ttl <= 0s ||  
        beresp.http.Set-Cookie ||  
        beresp.http.Vary == "*") {  
                /* 
                 * Mark as "Hit-For-Pass" for the next 2 minutes 
                 */  
                set beresp.ttl = 120 s;  
                return (hit_for_pass);  
    }  
    return (deliver);  
}  
  
sub vcl_deliver {  
    return (deliver);  
}  
  
sub vcl_error {  
    set obj.http.Content-Type = "text/html; charset=utf-8";  
    set obj.http.Retry-After = "5";  
    synthetic {"  
<?xml version="1.0" encoding="utf-8"?>  
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"  
 "http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">  
<html>  
  <head>  
    <title>"} + obj.status + " " + obj.response + {"</title>  
  </head>  
  <body>  
    <h1>Error "} + obj.status + " " + obj.response + {"</h1>  
    <p>"} + obj.response + {"</p>  
    <h3>Guru Meditation:</h3>  
    <p>XID: "} + req.xid + {"</p>  
    <hr>  
    <p>Varnish cache server</p>  
  </body>  
</html>  
"};  
    return (deliver);  
}  
  
sub vcl_init {  
        return (ok);  
}  
  
sub vcl_fini {  
        return (ok);  
}
```

Example

```
backend www {  
  .host = "www.example.com";  
  .port = "80";  
}  
  
backend images {  
  .host = "images.example.com";  
  .port = "80";  
}  
  
sub vcl_recv {  
  if (req.http.host ~ "(?i)^(www.)?example.com$") {  
    set req.http.host = "www.example.com";  
    set req.backend = www;  
  } elsif (req.http.host ~ "(?i)^images.example.com$") {  
    set req.backend = images;  
  } else {  
    error 404 "Unknown virtual host";  
  }  
}  
  
The following snippet demonstrates how to force a minimum TTL for  
all documents.  Note that this is not the same as setting the  
default_ttl run-time parameter, as that only affects document for  
which the backend did not specify a TTL:::  
  
import std; # needed for std.log  
  
sub vcl_fetch {  
  if (beresp.ttl < 120s) {  
    std.log("Adjusting TTL");  
    set beresp.ttl = 120s;  
  }  
}
```

Example

```
acl purge {  
  "localhost";  
  "192.0.2.1"/24;  
}  
  
sub vcl_recv {  
  if (req.request == "PURGE") {  
    if (!client.ip ~ purge) {  
      error 405 "Not allowed.";  
    }  
    return(lookup);  
  }  
}  
  
sub vcl_hit {  
  if (req.request == "PURGE") {  
    purge;  
    error 200 "Purged.";  
  }  
}  
  
sub vcl_miss {  
  if (req.request == "PURGE") {  
    purge;  
    error 200 "Purged.";  
  }  
}
```

官方参考文档：<https://www.varnish-cache.org/docs/3.0/reference/vcl.html>
