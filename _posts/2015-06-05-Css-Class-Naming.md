---
layout: blog
title: 再探CSS 中 class 命名规范
categories: font-end
tags: html css
---
一直以来我的CSS 的 class命名都是比较随意，有时采用驼峰式、有时采用下划线，好像没有什么统一的标准，想到什么英文单词就拿过来用，这对于自己瞎写的小项目无伤大雅，遇到冲突的问题可稍加调整改变即可。

今天，我学习并掌握了一种新的class命名规范，觉得它更加科学更加稳健。在此总结。

##关注分离
class 的命名要遵循关注分离、松耦合的原则，同时注重易于理解

下面的代码展示了一些简单的 class 命名规范
```html
<div class="news">
	<div class="news-list">
		<h2 class="news-list-title"></h2>
		<p class="news-list-description"></p>
		<ariticle class="news-list-detail"></ariticle>
	</div>
</div>
```

```css
.news {}
.news-list {}
.news-list-title {}
.news-list-description {}
.news-list-detail {}
```
class的层次分级一般不超过三级较好！

##减少“多类症”
当 HTML 源代码满眼望去都是 class 时，你会完全抓狂！为了实现代码复用，减少重复冗余，我们需要把代码拆分在不同的 class 下面，并且注重寻找平衡点，减少不必要的 class

##减少嵌套层级
使用 LESS 编写样式可以很方便的嵌套，但是不能过度嵌套选择符

下面这样是很low的
```
.news .news-list .news-list-share ul li h5 a:hover{...}
```
所以选择符嵌套在必要的情况下一般不超过三层；选择符叠加一般不多于两个。

##公用样式
对于整个设计搞，CSS 的重用非常重要，一定要在拿到设计稿后对一些公有样式进行选择性抽取。
如：
```
breadcrumb {}
pages {}
btn-default {}
btn-default_type1 {}
btn-default_type2 {}
```
在这里我们看到了 “-” 和 “_”，我是这样区分的：短线“-”表示层级关系，用“ _” 表示表示同一类对象的不同表现方式。

对于一些在公有样式的基础上有些私有特性的目标，我们选择**先继承，后微调**

```html
<form class="login">
	<button class="btn-default"></button>	
</form>
```
```css
.login btn-default {}
```
公有样式我们可以嵌套用，堆叠用，但是不要单独用！

##关于命名空间
如果我们采用了多个库，我们会发现命名空间的好处。
比如YUI中的 yui, Pure 中的 pure, Amaze 中的 am, 都是为了

1、多个框架共存

2、多人协作开发命名冲突

3、使用第三方服务插件时产生干扰

4、自己编写的CSS意外覆盖框架

##扯点别的
对于一个项目，我们可以将样式分别存储在多个文件中。多个文件能够更好的管理公用样式和组件。
比如：
```
reset.css 	/* 默认基础样式，或使用 normalize.css */
icon.css 	/* 所有的图标URL和基础样式 */
skin.css 	/* 页面所有预定义颜色，便于更改色调主题 */
component.css 	/* 公用组件样式，比如：弹窗、页码、导航、按钮等 */
main.css 	/* All your self */
```

##参考文章
[https://en.bem.info/method/definitions](https://en.bem.info/method/definitions)

[http://amazeui.org/css](http://amazeui.org/css)

[normalize.css](http://necolas.github.io/normalize.css/)