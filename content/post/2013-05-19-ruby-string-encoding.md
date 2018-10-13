---
title: "ruby字符串的encoding,force_encoding,encode,encode!"
date: 2013-05-19
description: "ruby字符串的encoding,force_encoding,encode,encode"
categories: [ "ruby" ]
tags: ["ruby"]
aliases: [/ruby/2013/05/19/ruby-string-encoding/]
---

ruby1.9开始对字符串编码支持已经比较完善，我们可以直接通过使用String类的实例方法 
`encoding`,`force_encoding`,`encode`,`encode!`进行相关的编码操作。

### encoding
ruby1.9中为每个字符串对象增加了encoding信息

```bash
1.9.3p392 :001 > '我还是不懂'.encoding
 => #<Encoding:UTF-8> 
1.9.3p392 :002 > 
```

### force_encoding

某些情况下这个附加编码信息可能不正确我们可以修正它

```bash
1.9.3p392 :011 > x='我还是不懂'
 => "我还是不懂" 
1.9.3p392 :012 > x.encoding
 => #<Encoding:UTF-8> 
1.9.3p392 :013 > x.bytes.to_a
 => [230, 136, 145, 232, 191, 152, 230, 152, 175, 228, 184, 141, 230, 135, 130] 
1.9.3p392 :014 > x.force_encoding 'gbk'
 => "\x{E688}\x{91E8}\x{BF98}\x{E698}\x{AFE4}\x{B88D}\x{E687}\x82" 
1.9.3p392 :015 > x.encoding
 => #<Encoding:GBK> 
1.9.3p392 :016 > x.bytes.to_a
 => [230, 136, 145, 232, 191, 152, 230, 152, 175, 228, 184, 141, 230, 135, 130] 
1.9.3p392 :017 >
```

> 注意：`force_encoding` 方法只是改变了字符串对象的编码信息，并没有改变字符串对象实际存储的内容。

### encode、encode!
在ruby1.9之前如我我们需要编码转换则需要使用一些外部库，
现在我们可以直接使用String对象的实例方法`encode`,`encode!`进行操作

```
encode(encoding [, options] ) → str click to toggle source
encode(dst_encoding, src_encoding [, options] ) → str
encode([options]) → str
encode!(encoding [, options] ) → str click to toggle source
encode!(dst_encoding, src_encoding [, options] ) → str
```

> 详细的api请参考[这里](http://www.ruby-doc.org/core-1.9.3/String.html#method-i-encode)


```bash
1.9.3p392 :009 > x='我还是不懂'
 => "我还是不懂" 
1.9.3p392 :010 > x.encoding
 => #<Encoding:UTF-8> 
1.9.3p392 :011 > x.bytes.to_a
 => [230, 136, 145, 232, 191, 152, 230, 152, 175, 228, 184, 141, 230, 135, 130] 
1.9.3p392 :012 > y=x.encode 'gbk','utf-8'
 => "\x{CED2}\x{BBB9}\x{CAC7}\x{B2BB}\x{B6AE}" 
1.9.3p392 :013 > y.encoding
 => #<Encoding:GBK> 
1.9.3p392 :014 > y.bytes.to_a
 => [206, 210, 187, 185, 202, 199, 178, 187, 182, 174] 
1.9.3p392 :015 > x.encode! 'gbk','utf-8'
 => "\x{CED2}\x{BBB9}\x{CAC7}\x{B2BB}\x{B6AE}" 
1.9.3p392 :016 > x.encoding
 => #<Encoding:GBK> 
1.9.3p392 :017 > x.bytes.to_a
 => [206, 210, 187, 185, 202, 199, 178, 187, 182, 174] 
1.9.3p392 :018 >
```

> 可以看到`encode`改变了编码信息同时也改变了字符串对象存储的内容

### 总结

* `encdoing` 用来查看字符串的编码信息。
* `force_encoding` 用来修正字符串编码信息，注意是修正。
* `encode`,`encode!` 用来转码字符串。
