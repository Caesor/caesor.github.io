---
layout: blog
title: 2015前端面试题汇总
categories: font-end
tag: javascript
---

##apply() 和 call() 的区别
**apply，call的作用就是借用别人的方法来调用，就像调用自己的一样**

```javascript
foo.call(this, arg1, arg2, arg3) == foo.apply(this, arguments) == this.foo(arg1, arg2, arg3)
```
**相同点：**两个方法产生的作用完全一样
**不同点：**方法传递的参数不同

```javascript
//例子一：
function print(a,b,c,d){
	alert(a+b+c+d);
}
function example(a,b,c,d){
	//用call方式借用print，参数显式打散传递
	print.call(this, a, b, c, d);
	//用apply的方式借用print，在参数作为一个数组传递
	//arguments数组是javascript方法内本身自带的
	print.apply(this, arguments);
	//或者封装成数组
	print.apply(this, [a,b,c,d]);
}
//例子二：
function Thing() {}
Thing.prototype.foo = "bar";
function logFoo(aStr) {
    console.log(aStr, this.foo);
}
var thing = new Thing();
//thing 可以使用 logFoo 的方法
logFoo.bind(thing)("using bind"); //logs "using bind bar"
logFoo.apply(thing, ["using apply"]); //logs "using apply bar"
logFoo.call(thing, "using call"); //logs "using call bar"
logFoo("using nothing"); //logs "using nothing undefined"
```

注：javascript对象所有属性都是公开的（public）， 没有私有（private）。 

##作用域
1、内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数。

2、try-catch的catch块、with语句会延长作用域链

3、if、for 中声明的变量会被自动添加到当前执行环境

4、如果不使用 var 声明变量，该变量就会自动被添加到全局环境

5、查询标示符：从作用域链的前段开始，向上逐级查询与给定名字匹配的标示符，如果在局部环境中找到，搜索过程停止。

6、访问全局变量—— window.color

7、垃圾收集：
	a)标记清除
	b)赋值 null，切断变量与此前引用值之间的连接，“解除引用”

##闭包
**闭包：**是指有权访问另一个函数作用域中变量的函数。
常见创建方式：在一个函数内部创建另个一个函数

一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域。但是，闭包的情况又有所不同。在另一个函数内部定义的函数会将包含函数的活动对象添加到他的作用域链中。

函数执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。 该函数的执行环境的作用域链会被销毁，但他的活动对象仍然会留在内存中；直到匿名函数被销毁后，函数的活动对象才会被销毁。

例子：
```javascript
function Thing(){}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function(){
	function doIt(){
		console.log(this.foo);
	}
}
var thing = new Thing();
thing.logFoo();	//undefined
```
**内层函数通过闭包获取外层函数里定义的变量值，不是直接继承自 this;**

```javascript
function Thing(){}
Thing.prototype.foo = "bar";
Thing.prototype.logFoo = function(){
	console.log(this.foo);
}
function doIt(method){
	method();
}
var thing = new Thing();
thing.logFoo();	// logs "bar"
doIt(thing.logFoo);	// undefined
//使用 bind 显示指明上下文
doIt(thing.logFoo.bind(thing));
```

###模块模式——块级作用域
使用闭包来定义公共函数，可以访问私有函数和变量
```javascript
var Counter = (function(){
	var privateCounter = 0;
	function changeBy(val){
		privateCounter += val;
	}
	return {
		increment: function(){
			changeBy(1);
		},
		decrement: function(){
			changeBy(-1);
		},
		value: function(){
			return privateCounter;
		}
	}
})()
alert(Counter.value());	// 0
Counter.increment();
Counter.increment();
alert(Counter.value());	// 2
Counter.decrement();
alert(Counter.value());	// 1
```



##类和继承
注意： `new`会调用一个构造函数， `Object.create`没有调用构造函数

###类
构造函数模式用于定义实例属性，原型模式用于定义方法和共享属性。
```javascript
function Thing(){
	//在这里面定义属性每个实例互不影响
	this.things = [];
}
```

###继承

```javascript
function Super(name){
	this.name = name;
	this.colors = ["red","green","blue"];
}
Super.prototype.sayName = function(){
	alert(this.name);
}
function Sub(name, age){
	//继承属性
	Super.call(this, name);
	this.age = age;
}
Sub.prototype = new Super();
Sub.prototype.constructor = SubType;
Sub.prototype.sayAge = function(){
	alert(this.age);
};
```

##事件

###跨浏览器事件处理
```
var EventUtil = {
	addHandler: function(element, type, handler){
		if( element.addEventListener){	// 非IE DOM 2级
			element.addEventLister(type, handler, false);
		}else{	// IE DOM 2级
			element.attachEvent("on" + type, handler);
		}else{	// DOM 0级
			element["on" + type] = handler; // btn.onclick
		}
	},
	removeHandler: function(element, type, handler){
		if(element.removeEventListener){
			element.removeEventListener(type, handler, false);
		}else if(element.detachEvent){
			element.detachEvent("on" + type, handler);
		}else{
			element["on" + type] = null;
		}
	}	
}
```

###事件委托（事件代理）delegate	
在javascript中，添加到页面上的函数都是对象，都会占用内存，内存中的对象越多，性能就越差。

对事件处理程序过多问题的解决方案就是**事件委托**。事件委托利用率事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。

利用事件委托，只需在DOM书中尽量最高的层次上添加一个事件处理程序。
```javascript
var list = document.getElementById("myLinks");
EventUtil.addHandler(list, "click", function(event){
	event = EventUtil.getEvent(event);
	var target = EventUtil.getTarget(event);
	switch(target.id){
		case "doSomething":
			document.title = "I changed the document's title";
			break;
		case "goSomentWhere":
			location.href = "http://ww.wrox.com"
			break;
		case "sayHi":
			alert("hi");
			break;
	}
})
```

这段代码消耗很低，因为只取得一个DOM元素，只添加一个事件处理程序，占用内存更至少。

##Ajax

###Ajax的原理
Ajax是Asynchronous JavaScript and XML的缩写。

在非Ajax请求中，用户触发一个HTTP请求到服务器,服务器对其进行处理后再返回一个新的HTML页到客户端, 每当服务器处理客户端提交的请求时,客户都只能**空闲等待**,并且哪怕只是一次很小的交互、只需从服务器端得到很简单的一个数据,都要返回一个完整的HTML页,而用户每次都要浪费时间和带宽去重新读取整个页面.

Ajax,通过XmlHttpRequest对象来向服务器发异步请求,从服务器获得数据，然后用javascript来操作DOM而更新页面。
```
var xmlhttp;
if(window.XMLHttpRequest){
	xmlhttp = new XMLHttpRequest();
}else{
	xmlhttp = new ActiveXObject("Microsoft.XMLHTTP");	// IE 5、6
}
//监听
xmlhttp.onreadystatechange = function(){
	if(xmlhttp.readyState == 4 && xmlhttp.status == 200){
		// do something
	}
}
//method url async
xmlhttp.open("GET", "text1.txt", true);	// true：异步，false：同步
xmlhttp.send();
```

###Ajax的优势
1、减轻服务器的负担。因为Ajax的根本理念是“按需取数据”，所以最大可能在减少了冗余请求和响影对服务器造成的负担；

2、无刷新更新页面，减少用户实际和心理等待时间；

3、也可以把以前的一些服务器负担的工作转嫁到客户端，利于客户端闲置的处理能力来处理，减轻服务器和带宽的负担，节约空间和带宽租用成本；

###Ajax跨域的方式
JavaScript出于安全方面的考虑，不允许跨域调用其他页面的对象。但在安全限制的同时也给注入iframe或是ajax应用上带来了不少麻烦。

首先什么是跨域，简单地理解就是因为JavaScript同源策略的限制，a.com 域名下的js无法操作b.com或是c.a.com域名下的对象。更详细的说明可以看下表：
![picture1]({{site.blogimgurl}}/2015-03-10-01.png "ajax跨域")

**1、document.domain+iframe的设置**

对于主域相同而子域不同的例子，可以通过设置document.domain的办法来解决。具体的做法是可以在http://www.a.com/a.html和http://script.a.com/b.html两个文件中分别加上`document.domain = ‘a.com’`；然后通过a.html文件中创建一个iframe，去控制iframe的contentDocument，这样两个js文件之间就可以“交互”了。当然这种办法只能解决主域相同而二级域名不同的情况，如果你异想天开的把script.a.com的domian设为alibaba.com那显然是会报错地！代码如下：

www.a.com上的a.html:
```
document.domain = 'a.com';
var ifr = document.createElement('iframe');
ifr.src = 'http://script.a.com/b.html';
ifr.style.display = 'none';
document.body.appendChild(ifr);
ifr.onload = function(){
    var doc = ifr.contentDocument || ifr.contentWindow.document;
    // 在这里操纵b.html
    alert(doc.getElementsByTagName("h1")[0].childNodes[0].nodeValue);
};
```

script.a.com上的b.html
```
document.domain = 'a.com';
```
这种方式适用于{www.kuqin.com, kuqin.com, script.kuqin.com, css.kuqin.com}中的任何页面相互通信。

备注：某一页面的domain默认等于window.location.hostname。主域名是不带www的域名，例如a.com，主域名前面带前缀的通常都为二级域名或多级域名，例如www.a.com其实是二级域名。 domain只能设置为主域名，不可以在b.a.com中将domain设置为c.a.com。

问题：
1、安全性，当一个站点（b.a.com）被攻击后，另一个站点（c.a.com）会引起安全漏洞。
2、如果一个页面中引入多个iframe，要想能够操作所有iframe，必须都得设置相同domain。

**2、动态创建script**

**3、利用iframe和location.hash**

**4、window.name实现的跨域数据传输**

**5、使用HTML5 postMessage**

参见文章：[JavaScript跨域总结与解决办法](http://www.cnblogs.com/rainman/archive/2011/02/20/1959325.html)

##详解 this
看文章 [all this](www.cnblogs.com/Wayou/p/all-this.html)


##常用状态码
**2xx （成功)——表示成功处理了请求的状态代码。**

200   OK（成功）  服务器已成功处理了请求。 通常，这表示服务器提供了请求的网页。 

201   Created（已创建）  请求成功并且服务器创建了新的资源。 

202   Accepted（已接受）  服务器已接受请求，但尚未处理。 

**3xx （重定向)**

**4xx（请求错误）**

400   Bad Request（错误请求）

404   (请求失败)，请求所希望得到的资源未被在服务器上发现。

408   Request Timeout（请求超时）

**5xx（服务器错误）**

503   (服务不可用)

##CSS

###position
**static**:元素框正常生成

**fixed**:生成绝对定位的元素，相对于浏览器窗口进行定位。

**relative**:可以通过设置垂直或水平位置，让这个元素“相对于”它的起点进行移动。

**absulote**：绝对定位的元素的位置相对于最近的已定位祖先元素(relative / absulote)，如果元素没有已定位的祖先元素，那么它的位置相对于最初的包含块(body)。

###超越行内 style 样式
```css
//具有最高优先级
box{color:red !important;} 	// ie 7/8/FF
```