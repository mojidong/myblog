---
title: "python多版本管理工具pythonbrew"
date: 2013-01-06
description: ""
categories: [ "python" ]
tags: ["python"]
aliases: [/python/2013/01/06/python-pythonbrew-multi-version-management-tools/]
---

当你需要在一台机器上同时安装多个不同版本的python的时候，你可能就需要使用[pythonbrew](https://github.com/utahta/pythonbrew)。

**pythonbrew**可以帮你下载安装不同版本的python并且可以自由的在多个版本间进行切换,它和ruby的rvm类似。
### 安装
---
* 终端输入`curl -kL http://xrl.us/pythonbrewinstall | bash`
	> 执行完成之后它会安装在` ~/.pythonbrew.`目录下 
	
	> 如果你想将pythonbrew安装到指定的位置你需要这样做
	
	>```bash
	export PYTHONBREW_ROOT=/path/to/pythonbrew
	curl -kLO http://xrl.us/pythonbrewinstall
	chmod +x pythonbrewinstall
	./pythonbrewinstall
	```

* 在`~/.bashrc`添加`[[ -s $HOME/.pythonbrew/etc/bashrc ]] && source $HOME/.pythonbrew/etc/bashrc`

### 使用
---
安装python

```bash
pythonbrew install 2.7.2
pythonbrew install --verbose 2.7.2
pythonbrew install --test 2.7.2
pythonbrew install --test --force 2.7.2
pythonbrew install --configure="CC=gcc_4.1" 2.7.2
pythonbrew install --no-setuptools 2.7.2
pythonbrew install http://www.python.org/ftp/python/2.7/Python-2.7.2.tgz
pythonbrew install /path/to/Python-2.7.2.tgz
pythonbrew install /path/to/Python-2.7.2
pythonbrew install 2.7.2 3.2
```

临时切换到指定版本的(当前shell)

```bash
pythonbrew use 2.7.2
```

永久切换到指定版本的

```bash
pythonbrew switch 2.7.2
pythonbrew switch 3.2
```

列出以安装的版本

```bash
pythonbrew list
```

列出所有可安装的版本

```bash
pythonbrew list -k
```

卸载

```bash
pythonbrew uninstall 2.7.2
pythonbrew uninstall 2.7.2 3.2
```

清理源文件和安装包

```bash
pythonbrew cleanup
```

更新pythonbrew

```bash
pythonbrew update
pythonbrew update --master
pythonbrew update --develop
```

停用pythonbrew

```bash
pythonbrew off
```

### 更多
---
更多使用方法请参考<https://github.com/utahta/pythonbrew>
