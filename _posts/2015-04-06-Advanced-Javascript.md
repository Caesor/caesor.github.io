---
layout: blog
title: 高性能javascript
categories: font-end
tags: javascript
---
##DOM

###减少页面重排和重绘
1.将多次改变样式属性的操作合并成一次操作。

2.将需要多次重排的元素，position属性设为absolute或fixed，这样此元素就脱离了文档流，它的变化不会影响到其他元素。例如有动画效果的元素就最好设置为绝对定位。

3.在内存中多次操作节点，完成后再添加到文档中去。（createDocumentFragment）例如要异步获取表格数据，渲染到页面。可以**先取得数据后在内存中构建整个表格的html片段，再一次性添加到文档中去，而不是循环添加每一行。**

4.由于display属性为none的元素不在渲染树中，对隐藏的元素操作不会引发其他元素的重排。如果要对一个元素进行复杂的操作时，可以先隐藏它，操作完成后再显示。这样只在隐藏和显示时触发2次重排。

5.在需要经常取那些引起浏览器重排的属性值时，要缓存到变量。
在最近几次面试中比较常问的一个问题：在前端如何实现一个表格的排序。如果应聘者的方案中考虑到了如何减少重绘和重排的影响，将是使人满意的方案。

###缓存选择器查询结果
每次DOM选择都要计算，缓存它

	//这样的写法就是糟糕的写法
	jQuery('#top').find('p.classA');
	jQuery('#top').find('p.classB');
	//更好的写法是
	var cached = jQuery('#top'); 
	cached.find('p.classA'); 
	cached.find('p.classB');

###缓存列表.length
每次.length都要计算，用一个变量保存这个值

##Javascript

###关于script
1、将js放置到 body 的最底端。现代浏览器虽然允许同时并行下载js文件，但是js下载会阻塞其他资源的下载，如图片。还是会阻止浏览器去加载和渲染html。

2、限制script标签的数量。

每次遇到script都会阻塞浏览器加载和渲染HTML。这个对于内联和外联js都是一样的。

每个外联js，每个http请求都要和服务器建立以此连接，时间开销不小。所以可以将js文件合并减少http请求的数量提高性能

3、非阻塞脚本在html加载完后进行javascript源码的下载。

1）添加defer属性。defer属性告诉浏览器该javascript代码不会影响DOM树，所以等DOM加载完成后执行。当遇到这个script的时候开始下载但是并不立即执行，当**DOM树加载完成在onload事件触发之前**执行，且不会影响到其他资源的下载。

2）动态执行将javascript代码插入文档

	function loadScript(url ,callback){
		var script = document.createElement("script");
		script.type = "text/javascript";
		if( script.readyState ){
			scrpt.onreadystatechange = function(){
				if( script.readyState == "loaded" || script.readyState == "complete" ){
					script.onreadystagechange = null;
					callback();
				}
			};
		}else{
			script.onload = function(){
				callback();
			};
		}	
		script.src = url;
		document.getElementsByTageName_r("head")[0].appendChild(script);
	}

3）通过XMLHttpRequest从服务端获取js动态插入到文档中

##数据访问

###作用域链和原型链
1、使用局部变量。全局变量的返回所需要的时间是最长的，应该用局部变量存起来减少作用域链的查询次数。

2、避免使用`with`和`try-catch`语句。with语句和try-catch会改变执行函数的作用域链。会在作用域链前加上with或catch紫的活跃对象。这就导致本来可以自己访问的局部变量需要在作用域链的后面一层被访问到了。增加了变量访问的时间。

3、javascript中存储数据有四个地方：

1）直接量（字符串、数字、布尔型、Object、Array、Function、正则表达式、NULL以及undefined）

2）变量

3）数组

4）对象成员。

其中直接量和变量是最快的，两者差别不大。但是对于数组和对象来说相对慢些。因为访问对象中的一些属性和方法涉及到原型链的查找。所以需要存储这些需要**遍历原型链的方法或者属性**提高性能。

###事件托管（delegate）
当一个页面存在大量的元素，并且很多元素都有一个或多个事件绑定的时候，会影响页面的性能，这时候使用事件托管监听最近接这些元素的父级即可。

原理：事件冒泡

##相关文章
[移动HTML5前端性能优化指南](http://segmentfault.com/a/1190000002511921)

[高性能HTML](http://www.alloyteam.com/2012/10/high-performance-html/)
