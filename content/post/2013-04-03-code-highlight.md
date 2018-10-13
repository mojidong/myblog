---
title: "轻量级代码语法高亮插件Highlight.js"
date: 2013-04-03
description: "轻量级代码语法高亮插件Highlight.js"
categories:  [ "html/css/js" ]
tags: ["html/css/js"]
aliases: [/html/css/js/2013/04/03/code-highlight/]
---

[Highlight.js](http://softwaremaniacs.org/soft/highlight/en/)是一款轻量级的语法高亮插件,
支持54种语言和26种主题，详情见[这里](http://softwaremaniacs.org/media/soft/highlight/test.html)


本站使用就是该插件。

### 简单使用

```js
<link rel="stylesheet" href="styles/default.css">
<script src="highlight.pack.js"></script>
<script>hljs.initHighlightingOnLoad();</script>
```
>默认高亮`<pre><code></code></pre>`块包裹的代码

### 定制

你也可以有选择的高亮代码块

```js
$(document).ready(function() {
  $('pre code').each(function(i, e) {hljs.highlightBlock(e)});
});
```
如果你的代码不在`pre`中，则你可能需要使用`<br>`换行，传递参数`true`告诉`Highlight`用`<br>`换行

```js
$('div.code').each(function(i, e) {hljs.highlightBlock(e, null, true)});
```
> 更多api详情请点[这里](http://highlightjs.readthedocs.org/en/latest/api.html)

有时你可能需要替换`TAB`为你想要的任意字符,便于排版

```js
<script type="text/javascript">
  //替换TAB为4个空白
  hljs.tabReplace = '    '; 
  //替换为span \t
  hljs.tabReplace = '<span class="indent">\t</span>';
  
  hljs.initHighlightingOnLoad();
</script>
```

### 语法识别

Highlight是根据`code`标签上的`class`识别语言的

```html
<pre><code class="java">...</code></pre>
<pre><code class="python">...</code></pre>
<pre><code class="ruby">...</code></pre>
<pre><code class="erlang">...</code></pre>
```

有些情况下可能需要禁止高亮

```html
<pre><code class="no-highlight">...</code></pre>
```

### node.js

Highlight.js 也能够在`node.js`中使用

```sh
npm install highlight.js
```

```js
var hljs = require('highlight.js');

// 设置具体的语言
hljs.highlight(lang, code).value;

// 自动识别语言
hljs.highlightAuto(code).value;
```

如果你有兴趣深入研究请参见[源码](https://github.com/isagalaev/highlight.js)
