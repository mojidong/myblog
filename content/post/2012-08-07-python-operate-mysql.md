---
title: "python操作mysql数据库"
date: 2012-08-07
description: "python操作mysql数据库"
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2012/08/07/python-operate-mysql/]
---

**python操作mysql需要用到[MySQLdb](http://mysql-python.sourceforge.net/)这个库。**

1. 下载MySQLdb，点击[这里下载](http://sourceforge.net/projects/mysql-python/)
2. 使用

	```python
	import MySQLdb  
	try:  
    	con = MySQLdb.connect(host='192.168.1.100', user='myuser', passwd='123456', port='3306', db='mydb',charset='utf8')  
    	c = con.cursor()  
    	c.execute('select * from user');  
    	rows=c.fetchall() #一次读取所有数据,返回的数据结构[(?,?,...),(?,?,..),..]  
    	for row in rows:  
        	for column in row:  
            	print column  
	except Exception,e:  
    	print e  
	finally:  
    	c.close()  
    	con.close()  
    ```
    
    ```python
    import MySQLdb  
	try:  
    	con = MySQLdb.connect(host='192.168.1.100', user='myuser', passwd='123456', port='3306', db='mydb',charset='utf8')  
    	c = con.cursor()  
    	#使用占位符传递参数，参数是一个tuple  
    	c.execute('select * from user where user=%s',('root'));  
    	rows=c.fetchall() #一次读取所有数据,返回的数据结构[(?,?,...),(?,?,..),..]  
    	for row in rows:  
        	for column in row:  
            	print column  
	except Exception,e:  
    	print e  
	finally:  
		c.close()  
    	con.close()  
    ```
    
    ```python
    import MySQLdb  
	try:  
    	con = MySQLdb.connect(host='192.168.1.100', user='myuser', passwd='123456', port='3306', db='mydb',charset='utf8')  
    	c = con.cursor()  
    	#使用占位符传递参数，参数是一个tuple  
    	c.execute('update user set host=%s where user=%s',('root','192.168.1.200'));  
    	con.commit() #更新操作记得提交事物，否则更改不会生效  
	except Exception,e:  
    	print e  
	finally:  
    	c.close()  
    	con.close()  
    ```
    
    ```python
    import MySQLdb  
	try:  
    	con = MySQLdb.connect(host='192.168.1.100', user='myuser', passwd='123456', port='3306', db='mydb',charset='utf8')  
    	c = con.cursor()  
    	vs=[]  
    	vs.append(('root','192.168.1.100'))  
    	vs.append(('db','192.168.1.101'))  
    	vs.append(('pdb','192.168.1.101'))  
    	#使用占位符传递参数，参数是一个list tuple  
    	c.executemany('insert into user(user,host)values(%s,%s)',vs);  
    	con.commit() #更新操作记得提交事物，否则更改不会生效  
	except Exception,e:  
    	print e  
	finally:  
    	c.close()  
    	con.close()  
    ```
3. **注意事项**

>咋看一下其中占位符和字符串格式化差不多，天真的以为数字用%d,浮点数用%f,字符串用%s,那你可以就要悲剧了

>事实上这里的占位符只能是%s
