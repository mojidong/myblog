---
title: "页面回到顶部的实现及原理"
date: 2013-03-03
description: "页面回到顶部的实现及原理"
categories: [ "html/css/js" ]
tags: ["html/css/js"]
aliases: [/html/css/js/2013/03/03/page-top-to-achieve/]
---

现在很多网页都有一个回到顶部的功能，当你拖动滚动条往下拉到一定程度就会在右下方显示一个回到顶部的按钮，点击则可以立即回到页面顶部，非常贴心的设计。

今天打算自己做一个，分析了一下，要点如下

* 监听当前页面滚动事件
* 当滚动到一定像素显示回到顶部的按钮
* 给回到顶部按钮绑定点击事件，点击按钮让滚动条回到顶部并隐藏按钮

核心点主要在js控制按钮的显示和隐藏及点击按钮回到顶部及如何控制回到顶部按钮不随文档滚动

js伪代码如下

```js
//绑定页面滚动事件
$(document).bind('scroll',function(){
	var len=$(this).scrollTop()
	if(len>=100){
		//显示回到顶部按钮
    }else{
    	//隐藏回到顶部按钮
    }
})
//绑定回到顶部按钮的点击事件
$('#回到顶部').click(function(){
	//动画效果，平滑滚动回到顶部
	$(document).animate({ scrollTop: 0 });
	//不需要动画则直接回到顶部
	//$(document).scrollTop(0)
})
```

让按钮不随文档滚动需要使用到css属性position
>static：无特殊定位，对象遵循正常文档流。top，right，bottom，left等属性不会被应用。

>relative：对象遵循正常文档流，但将依据top，right，bottom，left等属性在正常文档流中偏移位置。

>absolute：对象脱离正常文档流，使用top，right，bottom，left等属性进行绝对定位。而其层叠通过z-index属性定义。

>fixed：对象脱离正常文档流，使用top，right，bottom，left等属性以窗口为参考点进行定位，当出现滚动条时，对象不会随着滚动。IE6及以下不支持此参数值

很明显我们需要`fixed`(ie6真是蛋疼)

---

思路有了就写测试代码

```html
<!DOCTYPE HTML>
<html>
	<head>
		<meta http-equiv="content-type" content="text/html; charset=utf-8" />
		<title>回到顶部demo</title>
		<style type="text/css">
			.up{
				position:fixed;
				bottom:20px;
				right:20px;
				width:10px;
				display:none;
			}
		</style>
	</head>
	<body>
		<div style="background:rgb(200,200,200);height:2000px;">我很长</div>
		<a id="up" href="javascript:;" class="up">回到顶部</a>
	</body>
	<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
	<script type="text/javascript">
		//绑定页面滚动事件
		$(document).bind('scroll',function(){
			var len=$(this).scrollTop()
			if(len>=100){
				//显示回到顶部按钮
				$('#up').show();
			}else{
				//影藏回到顶部按钮
				$('#up').hide();
			}
		})
		//绑定回到顶部按钮的点击事件
		$('#up').click(function(){
			//动画效果，平滑滚动回到顶部
			$(document).animate({ scrollTop: 0 });
			//不需要动画则直接
			//$(document).scrollTop(0)
		})
	</script>
</html>
```
测试结果不是很令人满意，拖动滚动条按钮是显示出来了，但是点击却没有回到顶部。

原来问题出在滚动操作的对象上，将`document`换成`body`

```js
//绑定回到顶部按钮的点击事件
$('#up').click(function(){
	//动画效果，平滑滚动回到顶部
	$('body').animate({ scrollTop: 0 });
	//不需要动画则直接
	//$('body').scrollTop(0)
})
```
测试一切ok

### 以为这样就完了，你太天真了少年

浏览器兼容，攻城师心中永远的痛~

代码在ie上不给力，滚动显示不了按钮，滚到底都显示不出来，崩溃。

原来事件要绑在`window`上

```js
//绑定页面滚动事件
$(window).bind('scroll',function(){
	var len=$(this).scrollTop()
	if(len>=100){
		//显示回到顶部按钮
		$('#up').show();
	}else{
		//影藏回到顶部按钮
		$('#up').hide();
	}
})
```

上最终代码

```html
<!DOCTYPE HTML>
<html>
	<head>
		<meta http-equiv="content-type" content="text/html; charset=utf-8" />
		<title>回到顶部demo</title>
		<style type="text/css">
			.up{
				position:fixed;
				bottom:20px;
				right:20px;
				width:10px;
				display:none;
			}
		</style>
	</head>
	<body>
		<div style="background:rgb(200,200,200);height:2000px;">我很长</div>
		<a id="up" href="javascript:;" class="up">回到顶部</a>
	</body>
	<script src="http://ajax.googleapis.com/ajax/libs/jquery/1.9.1/jquery.min.js"></script>
	<script type="text/javascript">
		//绑定页面滚动事件
		$(window).bind('scroll',function(){
			var len=$(this).scrollTop()
			if(len>=100){
				//显示回到顶部按钮
				$('#up').show();
			}else{
				//影藏回到顶部按钮
				$('#up').hide();
			}
		})
		//绑定回到顶部按钮的点击事件
		$('#up').click(function(){
			//动画效果，平滑滚动回到顶部
			$(document).animate({ scrollTop: 0 });
			//不需要动画则直接
			//$(document).scrollTop(0)
		})
	</script>
</html>
```

---

神马还有ie6，哥哥早点洗洗睡吧~
