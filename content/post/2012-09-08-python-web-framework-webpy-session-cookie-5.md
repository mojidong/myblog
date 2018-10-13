---
title: "python的web框架webpy【session 和 cookie】五"
date: 2012-09-08
description: "webpty中使用session非常简单"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/09/08/python-web-framework-webpy-session-cookie-5/]
---

webpty中使用session非常简单

```python
import web  
web.config.debug = False  
urls = (  
    "/count", "count",  
    "/reset", "reset"  
)  
app = web.application(urls, locals())  
session = web.session.Session(app, web.session.DiskStore('sessions'), initializer={'count': 0})  
  
class count:  
    def GET(self):  
        session.count += 1  
        return str(session.count)  
  
class reset:  
    def GET(self):  
        session.kill()  
        return ""  
  
if __name__ == "__main__":  
    app.run()  
```

initializer 指定session的初始化值

```python
web.session.DiskStore('sessions') #设置session的存储方式为磁盘  
```

我们也可以指定session存储在数据库中

```python
db = web.database(dbn='postgres', db='mydatabase', user='myname', pw='')  
store = web.session.DBStore(db, 'sessions')  
session = web.session.Session(app, store, initializer={'count': 0})  
```

表结构


```sql
create table sessions (  
    session_id char(128) UNIQUE NOT NULL,  
    atime timestamp NOT NULL default current_timestamp,  
    data text  
);  
```

我们可以通过web.config对session进行一些可选设置

```python
web.config.session_parameters['cookie_name'] = 'webpy_session_id'  
web.config.session_parameters['cookie_domain'] = None  
web.config.session_parameters['timeout'] = 86400, #24 * 60 * 60, # 24 hours   in seconds  
web.config.session_parameters['ignore_expiry'] = True  
web.config.session_parameters['ignore_change_ip'] = True  
web.config.session_parameters['secret_key'] = 'fLjUfxqXtfNoIldA0A0J'  
web.config.session_parameters['expired_message'] = 'Session expired'
```

webpy中使用cookie

```python
setcookie(name, value, expires="", domain=None, secure=False):   
  
cookie_name - session id 存储在cookie中的名称  
cookie_domain - cookie的domain  
timeout - session 过期时间，单位为秒  
ignore_expiry -如果设置为True则忽略过期时间  
ignore_change_ip - 如果为False则来自同一ip则session才有效  
secret_key - session id的hash值  
expired_message - session 失效后显示的信息  
```

设置cookie

```python
web.setcookie('age', i.age, 3600) 
```

读取cookie

```python
web.cookies().get(cookieName)
```

