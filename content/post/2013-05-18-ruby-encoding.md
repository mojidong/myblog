---
title: "ruby编码详解"
date: 2013-05-18
description: "ruby编码详解"
categories: [ "ruby" ]
tags: ["ruby"]
aliases: [/ruby/2013/05/18/ruby-encoding/]
---

今天主要说下ruby编码，主要针对ruby1.9及以上版本。
> ruby1.8对编码的支持是比较弱的

### ruby中的编码
ruby中编码大致分为两类:

1. source encoding
2. io encoding,其中还分internal encoding和external encoding

### source encoding
我们一般会在ruby源码文件头部声明编码格式(magic comment)类似下面这样

```ruby
# encoding: utf-8

# some logic
```

或者

```ruby
# coding: utf-8

# some logic
```

或者

```ruby
# -*- coding: utf-8 -*-

# some logic
```

以上三种方式效果是一样的，都告诉ruby解析器此源码文件格式为utf-8。

> 注意：这里声明只是告诉ruby解析器源码文件格式，并不是设置文件格式。

> 比方说你声明`# encoding: gbk`，然而文件格式却是`utf-8`，运行可是会出错的，

> 因为ruby解析器会用你告诉它的`gbk`编码解析文件，显然这个肯定是要乱码的，
出错是必然的。

### io encodings
* external encoding

	默认的情况下，当创建的字符串对象来自以下位置时使用`default external encoding`：
    
    * CSV

    * File data read from disk
    
    * SDBM
    
    * StringIO
    
    * Zlib::GzipReader
    
    * Zlib::GzipWriter
    
    * String#inspect
    
    * Regexp#inspect  

* internal encoding

	如果`default internal encoding`不为`nil`,则从以下来源的字符串将被转码为`default internal encoding`
    
    * CSV

    * Etc.sysconfdir and Etc.systmpdir
    
    * File data read from disk
    
    * File names from Dir
    
    * Integer#chr
    
    * String#inspect and Regexp#inspect
    
    * Strings returned from Curses
    
    * Strings returned from Readline
    
    * Strings returned from SDBM
    
    * Time#zone
    
    * Values from ENV
    
    * Values in ARGV including $PROGRAM_NAME
    
    * \__FILE__
    
我们有三种方式可以设置`internal encoding`和`external encoding`。

1. 通过`Encoding.default_internal`和`Encoding.default_external`设置
      
  xx.rb
  
  ```ruby
  # encoding: utf-8
  
  Encoding.default_external = 'utf-8'
  Encoding.default_internal = 'gbk'
  
  x='我还是不懂'
  puts x.encoding
  
  open('xx.rb') do |f|
      f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }
  end
  ```
  结果
  
  ```sh
  ➜  ~  ruby xx.rb
  UTF-8
  [GBK,# encoding: utf-8]
  [GBK,x='我还是不懂']
  [GBK,puts x.encoding]
  [GBK,open('xx.rb') do |f|]
  [GBK,f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }]
  [GBK,end]
  ```
2. 通过命令行`ruby -E external:internal` 设置
      
  xx.rb
  
  ```ruby
  # encoding: utf-8
  
  x='我还是不懂'
  puts x.encoding
  
  open('xx.rb') do |f|
      f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }
  end
  ```
  结果
  
  ```sh
  ➜  ~  ruby -E utf-8:gbk xx.rb
  UTF-8
  [GBK,# encoding: utf-8]
  [GBK,x='我还是不懂']
  [GBK,puts x.encoding]
  [GBK,open('xx.rb') do |f|]
  [GBK,f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }]
  [GBK,end]
  ```
3. [io encoding](http://www.ruby-doc.org/core-2.0/IO.html#method-c-new)
      
  xx.rb
  
  ```ruby
  # encoding: utf-8
  
  x='我还是不懂'
  puts x.encoding
  
  open('xx.rb','r:utf-8:gbk') do |f|
      f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }
  end
  ```
  结果
  
  ```sh
  ➜  ~  ruby xx.rb
  UTF-8
  [GBK,# encoding: utf-8]
  [GBK,x='我还是不懂']
  [GBK,puts x.encoding]
  [GBK,open('xx.rb') do |f|]
  [GBK,f.each { |line| puts "[#{line.encoding},#{line.strip}]" if line.strip.size > 0 }]
  [GBK,end]
  ```

> 其中第一种和第二种方式是全局的，第三种是局部的，一般推荐使用第三种。

> 默认情况下`external encoding`会读取当前操作系统的语言环境，`internal encoding`则为nil。
