---
layout: blog
title: Angular 中遇到的一些问题
categories: font-end
tags: angular
---
##在 Angular 中注册全局函数
最近遇到一个很蛋疼的问题，因为页面需要调用 Flash实现一个小游戏，Flash 游戏结束之后需要调用页面的一个全局函数`showResult()`，由于Angular controller 作用域的问题，这个稍稍让人有点难过，但是还是有一种解决方案

```
window['showResut'] = function(s){console.log(s);}
```

即可注册一个全局函数

##请慎用 jquery
今天发现我在一个 ng-click 事件中是用了一个 jquery 的 $.ajax({}) 方法，并在其内部是用了 $compile 服务，结果令人意外，不可预测。

解决方法依然很简单，就是是用 angular 原生的 $http 方法实现异步调用。

##在 Angular 中使用 JSONP
废话少说，直接上代码：

```
$http.jsonp(hosturl + 'web_code?callback=JSON_CALLBACK')
	.success(function(data){
		// code
	});
```

**注意点：**

1、angularJS 中使用 $http.jsonp 函数

2、指定callback和回调函数名，函数名为**JSON_CALLBACK**时，会调用success 回调函数， 必须全为大写。

3、也可以指定其他回调函数，但必须是定义在 window 下的全局函数

4、URL 中必须加上 callback

5、当callback为 JSON_CALLBACK 时，只会调用 success， 既是window中又 全局函数，也不会调用该函数。