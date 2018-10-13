---
title: "zsh autosuggestions在tmux环境下高亮问题处理"
date: "2017-05-14"
description: "zsh autosuggestions在tmux下高亮问题处理"
categories: [ "linux" ]
tags: ["zsh","oh-my-zsh","zsh-autosuggestions"]
aliases: [/linux/2017/05/14/zsh-autosuggestions/]
---

### 问题
最近在尝试使用[zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)插件，在tmux中发现提示的高亮效果无效这里记录下解决方法。

现象如下：  

1. 在终端下zsh_ autosuggestions 正常工作，提标高亮正常。
2. 在tmux下zsh autosuggestions
   正常工作，提标高亮不正常（*提示为白色，不是正常的浅灰色*）。

### 处理
首先在网上查了一下好像没有对应的处理方法，官方的[issue](https://github.com/zsh-users/zsh-autosuggestions/issues/229)也有人提到了相同的问题但是也没有给出相应的解决方案。无耐只好自已动手解决。

首先想到的就是zsh_ autosuggestions 的提标色在tmux环境可能没有生效，查了一下zsh
autosuggestions
的文档发现`ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE`可以控制高亮顔色。直接在tmux环境设置该配置

```bash
$ ZSH_AUTOSUGGEST_HIGHLIGHT_STYLE="fg=1"
```

发现没有效果。难道是配置不正确，直接打开新的终端进行验证，发现顔色可改变配置应该是没有问题。

这时突然相到是不是终端的类型导致的，直接查看tmux环境的终端类型

```bash
$ env | grep TERM
TERM=screen
```

果然是这样。在看一下终端环境下的类型

```bash
$ env | grep TERM
TERM=xterm-256color
```

到这里基本就说明了为什么终端环境下高亮提示正常而在tmux环境不正常了。在tmux环境更改终端类型

```bash
export TERM=xterm-256color
```

果然高亮提示正常了。

### 方案

直接在`.zshrc`中加`TERM`配即可

```bash
$ echo "export TERM=xterm-256color" >> ~/.zshrc
```

> 注：MacOS iTerm2 可能还须要修改iTerm2配置`Preferences -> Profile -> Terminal -> Report Terminal Type :  xterm-256color`
