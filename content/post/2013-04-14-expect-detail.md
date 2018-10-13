---
title: "expect自动交互详解"
date: 2013-04-14
description: ""
categories: [ "linux" ]
tags: ["linux"]
aliases: [/linux/2013/04/14/expect-detail/]
---

[expect](http://expect.sourceforge.net/)是Unix系统中用来进行自动化控制和测试的软件工具，应用在交互式软件中如telnet，ftp，Passwd，fsck，rlogin，tip，ssh等等。

### 简介

linux下我们经常会写一些脚本来完成一系列繁琐的工作,不幸的是很多交互式的命令，我们不得不人肉去响应。

expect出现就是为了解决这个问题。

```sh
#!/usr/bin/env expect
# 假设用户名和密码都是admin
spawn ssh admin@10.0.0.10
expect {
    password: {
        send admin
    } connecting {
        send yes\n
        expect {
            password: {
                send admin
            }
        }
    }
}
expect eof
```

上面模拟了ssh登陆，无需我们手动输入密码。

### 详解

expect实际就是监听我们目标进程的输出(`spawn`)，根据期望输出(`expect`)，进行响应(`send`)。

expect的核心指令分别是`spawn`,`expect`,`send`。

* spawn

  ```sh
  spawn [args] program [args]
  ```
  创建一个新的进程，并执行给定的程序,后面就可以使用expect监听程序的输出。
  (详细的args请自行man)

* expect

  ```sh
  expect [[-opts] pattern body1] ... [-opts] pattern [bodyn]
  ```
  expect指令的参数是一连串的`opts,pattern,body`，其中`opts`是可选的，最后一个`pattern`的`body`也是可选的。
  
  `pattern`就是用来匹配期望的输出，一旦匹配上就执行后面的`body`
  
  ```sh
  expect {
      busy               {puts busy\n}
      -re "failed|invalid password" {puts failed]\n}
      timeout            {puts timeout\n}
      connected
  }
  ```
  `-re`选项开启正则匹配(更多选项请自行man)。
  
  >expect会等待目标进程的输出，然后进行匹配，这个等待的时间默认是10秒，一旦超过这个时间就直接执行下一条指令，我们可以通过设置timeout来更改这个时间
  
* send

  ```sh
  send [-flags] string
  ```
  
  将字符串传递给当前进程，这里就是模拟人的输入。(详细的flags请自行man)
  
### 关于 expect eof
好奇你的肯定会问最后那个`expect eof`是干嘛的。

这是因为没有`expect eof`程序就直接退出了，因为没什么可干的了。显然这样是有问题的，因为我们的程序实际上是没执行完的。

由`spawn`启动的程序在结束的时候会产生一个`eof`标示，`expect eof`会等待程序的退出标示，一旦匹配就什么也不做，是的什么也不做，没什么可做也就退出了。

**这样就完了吗？不，还有很大的问题**，还记得之前说的expect的超时时间么，是的是10秒，
如果程序执行时间超过10秒或更久，显然`expect eof`会超时，程序会直接退出，解决办法就是设置`timeout`

下面给出一个远程备份文件的例子，来说明问题

```sh
#!/usr/bin/env expect
# 假设用户名和密码都是admin
spawn rsync admin@10.0.0.10:~/data .
set timeout 600 #设置expect超时时间为10分钟,如果不设置同步则无法正确完成
expect {
    password: {
        send admin
    } connecting {
        send yes\n
        expect {
            password: {
                send admin
            }
        }
    }
}
expect eof
```

### 更多
expect使用[Tcl](http://www.tcl.tk/)脚本语言作流程控制，这样就可以实现更为复杂的业务逻辑,详情请点[这里](http://www.tcl.tk/)
