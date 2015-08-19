---
layout: blog
title: Angular中动态插入directive
categories: font-end
tags: angular
---

最近正在践行javascript设计模式，想在正在写的登陆弹窗中应用刚刚学习的**惰性单例模式**(根据需要动态插入登陆框)。

正好手头有一个使用Angular的项目需要登陆模块，于是想小试牛刀一下。结果发现动态插入的**directives**
在页面中没有被解析，显示如下：
`<login-layer></login-layer>`
于是折腾了一番终于在万能的**stackoverflow**上找到解决方案：
动态插入页面的directive需要先编译，然后插入，解决方案如下，直接上代码：

```javascript
angular.module("exampleApp", [])
	.controller('defaultCtrl', function($scope, $compile){
		var res = $compile("<login-layer />")($scope);
		$('body').append(res); // 我已经引入了jquery
	});
```

追其原因，因为页面在最初加载的时候已经将整个页面编译解析了一遍，之后插入的 **directive** 页面不会去实时动态解析，需要我们手动解析，然后插入。