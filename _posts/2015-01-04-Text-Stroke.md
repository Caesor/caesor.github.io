---
layout: blog
title: CSS3——文字镂空效果
categories: font-end
tags: css
---
今天看到QQ交流群中有人提问说怎么做出文字镂空的效果？我印象中见过几个网站的 Logo 部分使用镂空效果，可以通过文字看到背景，心想肯定有解决方案，而且肯定没那么麻烦。

果然，使用css3的 text-fill-color 属性可以完美解决这个问题！下面给出几个例子：

**例一：字体镂空**

	<p class="eg1">这是一个晴朗的早晨</p>
	.eg1{
		height:100px;
		line-height: 100px;
		font-size: 70px;
		font-family:georgia;
		text-align: center;
		-webkit-text-fill-color:transparent;
		-webkit-text-stroke:1px #fff;
		background-image: url(../resource/image/sky5.jpg);
	}

<p class="eg1">这是一个晴朗的早晨</p>
<style>
	.eg1{
		height:100px;
		line-height: 100px;
		font-size: 70px;
		font-family:georgia;
		text-align: center;
		-webkit-text-fill-color:transparent;
		-webkit-text-stroke:1px #fff;
		background-image: url(../resource/image/sky5.jpg);
	}
</style>

**例二：利用字体镂空给字体添加背景**

	<p class="eg1">这是一个晴朗的早晨</p>
	.eg2{
		height:100px;
		line-height: 100px;
		font-size: 70px;
		font-family:georgia;
		text-align: center;
		text-transform: uppercase;
		-webkit-text-fill-color:transparent;
		background:-webkit-linear-gradient(top, #eee, #aaa 30%, #333 61%, #000);
		-webkit-background-clip:text;
	}

<p class="eg2">text-fill-color</p>
<style>
	.eg2{
		height:100px;
		line-height: 100px;
		font-size: 70px;
		font-family:georgia;
		text-align: center;
		text-transform: uppercase;
		-webkit-text-fill-color:transparent;
		background:-webkit-linear-gradient(top, #eee, #aaa 30%, #333 61%, #000);
		-webkit-background-clip:text;
	}
</style>

其中，background-clip 表示背景所覆盖的范围； 可以有四个值： text、content－box、padding-box、border-box