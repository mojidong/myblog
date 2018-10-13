---
title: "linux下安装mysql"
date: 2012-08-07
description: "linux下安装mysql"
categories: [ "linux" ]
tags: ["linux", "mysql"]
aliases: [/linux/2012/08/07/linux-install-mysql/]
---

1. 首先下载mysql源码包，我这里下载[MySQL Community Server 5.5](http://dev.mysql.com/downloads/mysql/5.5.html)。

2. 检查机器是否装有cmake,直接输入cmake显示帮助信息表示已安装
>现在mysql编译都需要用cmake生成makefile文件，关于cmake更多可以点击[这里](http://www.cmake.org/)。如果你的机器上没有cmake请先行安装cmake。

	```bash
	$ yum install cmake  
	或者  
	$ sudo apt-get install cmake  
	```

3. 解压mysql并编译

	```bash
	$ tar xzvf mysql-5.5.27.tar.gz  
	$ cd  mysql-5.5.27  
	$ cmake -DCMAKE_INSTALL_PREFIX=/path/to/mysql  
	$ make -j4  
	$ make install  
	```

	> cmake这里可能出错，一般都是没有安装gcc或gcc-c++  
	cmake的时候可以设置许多编译参数，cmake -L  查看可以设置的参数  
	注意:如果cmake出错一定要删除目录下生成的CMakeCache.txt文件才能再次cmake

4. 创建mysql用户

	`$ useradd -r  mysql`

	> 上面命令将创建一个名为mysql的系统账号

5. 将mysql安装目录所有者更改为mysql用户并初始化数据库

	```bash
	$ chown -R mysql:mysql mysql  
	$ cd mysql  
	$ scripts/mysql_install_db --user=mysql 
	```

6. 拷贝mysql配置文件到/etc下

	`$ cp support-files/my-innodb-heavy-4G.cnf /etc/my.cnf`

	>support-files下有针对不同需求的配置文件，你可以根据自己的实际情况选取

7. 设置mysql服务(开机自启动)

	```bash
	$ cp support-files/mysql.server /etc/init.d/mysql  
	$ chkconfig --add mysql 
	```

	> 如果你的机器上没有chkconfig命令那么你可能需要安装

	```bash
	$ yum install chkconfig  
	或者  
	$ sudo apt-get install chkconfig 
	```

	你就可以通过

	```bash
	service mysql start  #启动mysql
	service mysql stop	#停止mysql
	service mysql restart #重启mysql
	```
