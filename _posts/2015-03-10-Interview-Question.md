---
layout: blog
title: 2015前端面试题汇总（持续更新中）
categories: font-end
tag: javascript
---
##HTML

###标签语义化
语义化是指用合理HTML标记以及其特有的属性去格式化文档内容。

语义化的(X)HTML文档有助于提升你的网站对访客的易用性，比如使用PDA、文字浏览器以及残障人士将从中受益。对于搜索引擎或者爬虫软件来说，则有助于它们建立索引，并可能给予一个较高的权值。

###语义化的意义

1、和搜索引擎建立良好沟通，有助于爬虫抓取更多的有效信息，比如h1~h6、strong用于不同权重的标题；隐藏文本等等

2、提升用户体验 例如title、alt用于解释名词或解释图片信息、例如label标签的活用；

3、代码可读、便于维护、提高开发效率、快速达成共识，

4、行业机构对语义化标签的扩展和浏览器厂商在技术上的支持力度逐渐提升，例如web标准化组织刚刚开始推广的html5，其中新增了许多语义化的标签，例如header、aside、nav、section等等，在Chrome、opera、safari、firefox等浏览器中均得到很好支持。

5、对浏览器的语义化——script、style

参见文章：新浪UED——[语义化标签的实战意义](http://ued.sina.com.cn/?p=157)

##浏览器

###URL

http://video.google.co.uk:80/videoplay?docid=-7246927612831078230&hl=en#00h02m30s

**location.href**： 返回当前页面的完整URL，location.toString()也返回这个值

**location.protocl-传输协议**：http，类似的协议还包含：https，ftp， etc。

**location.host-主机或主机名**：video.google.co.uk 。

注：host包括端口号、hostname不带端口号的服务器名称

**子域名**：video。

**域名**：google.co.uk。

**顶级域名**（TLD）是：uk。uk 

指的是国家顶级域名（ccTLD）。比如：google.com 

其中顶级域名是 com。

二级域名（SLD）是：com.uk。

**location.port-端口**：80，这是 web 服务器的默认端口。

**location.pathname-路径**：/videoplay。路径通常指一个文件或 web 服务器上的一个位置。如：/directory/file.html 。

**location.search-URL查询字符串**：URLs 可以有很多参数。参数以问号（?）开始，以（&）隔开。

**location.hash-散列**：看到了“#00h02m30s”了吗？这称为一个字符串或一个命名锚链。字符串过去通常指网页文件中的一个内部片段。

###从输入 URL 到页面加载完成的过程中都发生了什么事情？

1、browser checks cache; if requested object is in cache and is fresh, skip to #9

2、browser asks OS for server's IP address

3、OS makes a DNS lookup and replies the IP address to the browser

4、browser opens a TCP connection to server (this step is much more complex with HTTPS)

5、browser sends the HTTP request through TCP connection

5、browser receives HTTP response and may close the TCP connection, or reuse it for another request

6、browser checks if the response is a redirect (3xx result status codes), authorization request (401), error (4xx and 5xx), etc.; these are handled differently from normal responses (2xx)

7、if cacheable, response is stored in cache

8、browser decodes response (e.g. if it's gzipped)
browser determines what to do with response (e.g. is it a HTML page, is it an image, is it a sound clip?)

9、browser renders response, or offers a download dialog for unrecognized types

参见 ：

百度FEX——[从输入 URL 到页面加载完成的过程中都发生了什么事情？](http://fex.baidu.com/blog/2014/05/what-happen/)

StackOverflow——[what happens when you type in a URL in browser [closed]](http://stackoverflow.com/questions/2092527/what-happens-when-you-type-in-a-url-in-browser)

##apply、call 和 bind

###apply，call的作用
1、就是借用别人的方法来调用，就像调用自己的一样

2、将一个函数的上下文改为由第一个参数所指定对象的上下文，如果没有指定，那么Global将被指定。

###bind
bind()是把该函数绑定到指定的context上下文执行环境中，简单讲就是把该函数的this对象指向传到bind()中的参数context上，并**返回一个函数实例**。

###区别

    foo.call(this, arg1, arg2, arg3) == foo.apply(this, arguments) == this.foo(arg1, arg2, arg3)


call() 和 apply() 的第一个实参是要调用函数的母对象，它是调用上下文，在函数体内通过 this 来获得对它的引用。在 ECMAScript 5 严格模式中， call() 和 apply() 的第一个实参都会变为 this 的值，哪怕 传入的实参是原始值甚至是 null 或 undefined 。在 ECMAScript 和非严格模式中，传入的 null 和 underfined 都会被全局对象代替。而其他原始值则会被相应的包装对象（wrapper object）所代替。

**相同点：**两个方法产生的作用完全一样
**不同点：**方法传递的参数不同

例子一：

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

例子二：

    function sayHi(name,greeting){
    	var name = name||this.name,
    	    greeting = greeting||this.greeting;
    	console.log("Hello "+name+","+greeting);
    }
    var name = "Terry",
        greeting = "how are you?",
        me = {
        	name : "John",
    		greeting : "你好！"
    	};
    sayHi.call(this,name,greeting);	// Hello Terry, how are you?
    sayHi.call(this);	// Hello Terry, how are you?
    sayHi.apply(this,[name,greeting]);	// Hello Terry, how are you?
    sayHi.apply(me,[me.name,me.greeting]);	// Hello John, 你好！
    sayHi.call(me,me.name,me.greeting);	// Hello John, 你好！
    //bind()会返回函数实例，后面加()直接执行
    sayHi.bind()();	// Hello Terry, how are you?
    sayHi.bind(me)();	// Hello John, 你好！


例子三：

    function Thing() {
    }
    Thing.prototype.foo = "bar";
    Thing.prototype.logFoo = function () { 
        function doIt() {
            console.log(this.foo);
        }
        doIt.apply(this);
    }
    function doItIndirectly(method) {
        method();
    }
    var thing = new Thing();
    doItIndirectly(thing.logFoo.bind(thing)); //logs bar


注：javascript对象所有属性都是公开的（public）， 没有私有（private）。 

##callee & caller

###callee
返回正被执行的 Function 对象，也就是所指定的 Function 对象的正文。

callee 属性是 arguments 对象的一个成员，它表示对函数对象本身的引用，这有利于匿 函数的递归或者保证函数的封装性

常见用法：

arguments.length 是实参长度,	arguments.callee.length 是形参长度

###caller
返回一个对函数的引用，即调用了当前函数的函数体。

对于函数来说，caller 属性只有在函数执行时才有定义。 

如果函数是由 javascript 程序的**顶层**调用的，那么 caller 包含的就是 null 。

如果在**字符串上下文**中使用 caller 属性，那么结果和 functionName.toString 一样，也就是说，显示的是函数的反编译文本。

    function CallLevel(){
    	if( CallLevel.caller == null ){
    		console.log("CallLevel was called from the top level.")
    	}else{
    		console.log("CallLevel was called by another function:\n" + CallLevel.caller);
    	}
    }
    function funCaller(){
    	CallLevel();
    }
    CallLevel();	// null
    funCaller();	// funCaller()


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

###Scoping & Hoisting

    var scope = "hello";
    function scopeTest() {
        console.log(scope);	// undefined
        var scope = "no";
        console.log(scope);	// no
    }

声明提前、全局变量优先级低于局部变量

参考：[Scoping & Hoisting](http://segmentfault.com/a/1190000000348228)

##闭包
由于变量的作用域，通常我们无法从函数外部获取到它的局部变量，方法就是在函数内部再定义一个函数。

###闭包的概念
闭包就是函数的局部变量集合，只是这些局部变量在函数返回后会继续存在。

闭包就是就是函数的“堆栈”在函数返回后并不释放，我们也可以理解为这些函数堆栈并不在栈上分配而是在堆上分配

当在一个函数内定义另外一个函数就会产生闭包

一般来讲，当函数执行完毕后，局部活动对象就会被销毁，内存中仅保存全局作用域。但是，闭包的情况又有所不同。在另一个函数内部定义的函数会将**包含函数的活动对象**添加到他的作用域链中。

函数执行完毕后，其活动对象也不会被销毁，因为匿名函数的作用域链仍然在引用这个活动对象。 该函数的执行环境的作用域链会被销毁，但他的活动对象仍然会留在内存中；直到匿名函数被销毁后，函数的活动对象才会被销毁。

例子：

    function Thing(){}
    Thing.prototype.foo = "bar";
    Thing.prototype.logFoo = function(){
    	function doIt(){
    		console.log(this.foo);
    	}
    }
    var thing = new Thing();
    thing.logFoo();	//undefined

**内层函数通过闭包获取外层函数里定义的变量值，不是直接继承自 this;**


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


###闭包的用途
1、可以读取函数内部私有变量

2、让这些变量始终保持在内存中

3、避免全局变量的污染

###闭包的精髓
闭包中局部变量是**引用**而非拷贝；

多个函数绑定同一个闭包，因为他们定义在同一个函数内

当一个循环中赋值函数时，这些函数将绑定同样的闭包

外部函数所有局部变量都在闭包内，即使这个变量声明在内部函数定义之后

每次函数调用的事后创建一个新的闭包

###闭包的应用

    var singleton = (function(){
    	var privateVariable;
    	function privateFunction(x){
    		...privateVariable...
    	}
    	return {
    		firstMethod: function(a, b){
    			...privateVariable...		
    		},
    		secondMethod: function(c){
    			...privateFunction()...
    		}
    	};	
    })();


###闭包中的this
在闭包中函数作为某个对象的方法调用时，要特别注意，该方法内部匿名函数的this指向的是**全局变量**。

    var scope = "golobal";
    var object = {
        scope:"local",
        getScope:function(){
            return function(){
                return this.scope;
            }
        }
    }
    console.log(object.getScope()());	// golobal



解决方法一：赋值 _this

    var scope = "golobal";
    var object = {
        scope:"local",
        getScope:function(){
            var _this = this;
            return function(){
                return _this.scope;
            }
        }
    }
    var val = object.getScope();
    console.log(val());	// local


解决方法二：使用 apply || call

    var name = "golobal";
    function Thing(){}
    Thing.prototype.name = "local";
    Thing.prototype.logName = function(){
    	function doIt(){
    		console.log(this.name);
    	}
    	doIt();	// golobal
    	doIt.apply(this);	// local
    }
    var thing = new Thing();
    thing.logName();


总结：

1、在函数闭包中，不使用this对变量进行访问时，函数会通过文法环境中的外部引用（指针），一级级地往上找（单向链表），直到找到（或者最终找不到）对应的变量。这个结构是在函数定义的时候就决定了的。

2、在函数闭包中，使用this对变量进行访问时，和绝大多数语言不同，JavaScript的this保存的是调用环境的上下文，也就是说this中的内容是在调用的时候决定的，所以访问到的是当前环境下的对应变量，并不会像前一种情况一样进行逐级查找。

###注意点
1、由于闭包会使得变量都被保存在内存中，内存消耗很大，所以不能滥用闭包，否则会造成网页的性能问题，在IE中可能导致内存泄露。

解决方法是：在函数退出之前，将不是哦你给的局部变量全部删除。

    function bindEvent(){
        var target = document.getElementById("elem");
        var name = target.name;
        // 通过创建target.name副本减少对外部变量的循环引用以及手动重置对象
        target.onclick = function(){
            console.log(name);
        }
        // 赋值 NULL 回收
        target = null;
     }


2、闭包会在父函数外部，改变父函数内部变量的值。所以，如果你把父函数当做对象使用，把闭包当做它的公用方法，把内部变量当做它的私有属性，者一定要小心，不要随便改变父函数内部变量的值。

注：**this的指向是由它所在函数调用的上下文决定的，而不是由它所在函数定义的上下文决定的。**

###参考文章：

[详解js闭包](http://segmentfault.com/a/1190000000652891)--例子非常好

[理解javascript闭包](http://segmentfault.com/a/1190000002463063)

[阮一峰-学习Javascript闭包](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)

##模拟块级作用域
使用闭包来定义公共函数，可以访问私有函数和变量

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




##类和继承
注意： `new`会调用一个构造函数， `Object.create`没有调用构造函数

###prototype 和 __proto__ 的区别
父原型可以通过 __proto__ 进行访问

    function Person(){}
    Person.prototype.name = 'nick';
    var p1 = new Person();
    var p2 = new Person();
    p1.name === p2.name;	// true
    p1.__proto__ === Person.prototype;	// true
    Person.prototype.__proto__ === Object.prototype

相关链接：

[你应该知道的javascript](http://www.cnblogs.com/jianjialin/articles/1712988.html)

[了解javascript中的prototype和__proto__](http://www.qkeye.com/blog-42-457818.html)


###类
构造函数模式用于定义实例属性，原型模式用于定义方法和共享属性。

    function Thing(){
    	//在这里面定义属性每个实例互不影响
    	this.things = [];
    }


###继承


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
    Sub.prototype.constructor = Sub;
    Sub.prototype.sayAge = function(){
    	alert(this.age);
    };


##事件

###onclick 和 ontachmove的区别——延迟不一样
touchstart -> touchmove -> touchend -> click

所以：click在移动手持设备上带来的延迟

###阻止事件冒泡的方式

1、return false;		// 阻止冒泡和默认事件本身

2、event.stopPropagation();		// 非IE，只会阻止冒泡，但不会阻止默认事件

3、window.event.cancelBubble = true;		// IE 方式

###组织浏览器默认行为

1、e.preventDefault();		// 非IE，阻止默认事件，但不会阻止事件冒泡

2、window.event.returnValue = false;		// IE

###自定义事件
**简单的自定义事件**：既是定义一个对象以及其方法，可以在其中包含 click等其他事件

**高级自定义事件**：进行封装函数、参数

[JavaScript自定义事件](http://www.cnblogs.com/dolphinX/p/3254017.html)

[鑫空间](http://www.zhangxinxu.com/wordpress/2012/04/js-dom%E8%87%AA%E5%AE%9A%E4%B9%89%E4%BA%8B%E4%BB%B6/)

###事件类型
`event.type: `

###跨浏览器事件对象
在事件处理程序内部， `this` 始终等于 `currentTarget` 的值， `target` 只包含事件的实际目标。

1、如果事件处理程序指定给了**目标元素**,那么 `this == currentTarget == target`

2、如果**事件处理程序存在于父节点中**，那么 

    document.body.onclick = function(event){
    	console.log(currentTarget == document.body);	// true 
    	console.log(this == document.body);	// true
    	console.log(event.target == document.getElementById("myBtn"));	// true
    }



    var EventUtil = {
    	getEvent: function(event){
    		return event ? event : window.event;
    	},
    	getTarget: function(event){
    		return event.target || event.srcElement;
    	},
    	// 阻止默认行为
    	preventDefault: function(event){
    		if( event.preventDefault ){
    			event.preventDefault();
    		}else{	// IE
    			event.returnValue = false;
    		}
    	},
    	// 阻止事件冒泡
    	stopPropagation: function(event){
    		if( event.stopPropagation ){	//
    			event.stopPropagation();
    		}else{	// IE
    			event.cancelBubble = true;
    		}
    	}
    }


###跨浏览器事件处理

    var EventUtil = {
    	addHandler: function(element, type, handler){
    		if( element.addEventListener){	// 非IE DOM 2级
    			element.addEventLister(type, handler, false);
    		}else if(element.attachEvent){	// IE DOM 2级
    			element.attachEvent("on" + type, handler);
    		}else{	// DOM 0级
    			element["on" + type] = handler; // btn.onclick, 属性通过数组来访问！
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


###事件委托（事件代理）delegate	
在javascript中，添加到页面上的函数都是对象，都会占用内存，内存中的对象越多，性能就越差。

对事件处理程序过多问题的解决方案就是**事件委托**。事件委托利用率事件冒泡，只指定一个事件处理程序，就可以管理某一类型的所有事件。

利用事件委托，只需在DOM书中尽量最高的层次上添加一个事件处理程序。

    var list = document.getElementById("myLinks");
    EventUtil.addHandler(list, "click", function(event){
    	event = EventUtil.getEvent(event);	// event || window.event
    	var target = EventUtil.getTarget(event);	// event.target || event.srcElement
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


这段代码消耗很低，因为只取得一个DOM元素，只添加一个事件处理程序，占用内存更至少。

##Ajax

###Ajax的原理
Ajax是Asynchronous JavaScript and XML的缩写。

在非Ajax请求中，用户触发一个HTTP请求到服务器,服务器对其进行处理后再返回一个新的HTML页到客户端, 每当服务器处理客户端提交的请求时,客户都只能**空闲等待**,并且哪怕只是一次很小的交互、只需从服务器端得到很简单的一个数据,都要返回一个完整的HTML页,而用户每次都要浪费时间和带宽去重新读取整个页面.

Ajax,通过XmlHttpRequest对象来向服务器发异步请求,从服务器获得数据，然后用javascript来操作DOM而更新页面。

XMLHttpRequest的5种状态

0：XMLHttpRequest 对象还没有完成初始化

1：XMLHttpRequest 对象开始发送请求

2：XMLHttpRequest 对象的请求发送完成

3：XMLHttpRequest 对象开始读取服务器响应

4：XMLHttpRequest 对象读取服务器响应结束


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


script.a.com上的b.html

    document.domain = 'a.com';

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

401   Unauthorized（未授权）

403   Forbidden（禁止）

404   (请求失败)，请求所希望得到的资源未被在服务器上发现。

408   Request Timeout（请求超时）

**5xx（服务器错误）**

500   Internal Server Error

503   (服务不可用)

##CSS

###垂直居中
1、方法一——使用table-ceil

#wrapper {display:table;}
#cell {display:table-cell; vertical-align:middle;}

**优点：**
content 可以动态改变高度(不需在 CSS 中定义)。当 wrapper 里没有足够空间时， content 不会被截断

**缺点：**
Internet Explorer(甚至 IE8 beta)中无效，许多嵌套标签(其实没那么糟糕，另一个专题)

2、方法二——绝对定位

    #content {
    	position:absolute;
    	top:50%;
    	height:240px;
    	margin-top:-120px; /* negative half of the height */
    }

**优点：**
适用于所有浏览器

不需要嵌套标签

**缺点：**
没有足够空间时，content 会消失(类似div 在 body 内，当用户缩小浏览器窗口，滚动条不出现的情况)

3、方法三——绝对定位，可不确定高度

    .wrapper{
    	border:1px solid #f80;
    	position: relative;
    	height: 100px;
    }
    .content{
    	border:1px solid #000;
    	height: 80px;
    	width: 80%;
    	position: absolute;
    	top: 0;
    	bottom: 0;
    	left: 0;
    	right: 0;
    	margin: auto;
    }

**优点：**
简单

**缺点：**
IE(IE8 beta)中无效

无足够空间时，content 被截断，但是不会有滚动条出现

4、方法四——单行文本置中

    #content {height:100px; line-height:100px;}



###position
**static**:元素框正常生成

**fixed**:生成绝对定位的元素，相对于浏览器窗口进行定位。

**relative**:可以通过设置垂直或水平位置，让这个元素“相对于”它的起点进行移动。

**absulote**：绝对定位的元素的位置相对于最近的已定位祖先元素(relative / absulote)，如果元素没有已定位的祖先元素，那么它的位置相对于最初的包含块(body)。

###超越行内 style 样式

    //具有最高优先级
    box{color:red !important;} 	// ie 7/8/FF


###字体大小
**px**:是基于像素的单位.在浏览网页过程中，屏幕上的文字、图片等会随屏幕的分辨率变化而变化

**em**:浏览器默认的字体大小为 1em = 16px;(body中设置font-size决定)。“em”是相对于其父元素来设置字体大小的，这样就会存在一个问题，进行任何元素设置，都有可能需要知道他父元素的大小，在我们多次使用时，就会带来无法预知的错误风险。

**pt**:是point(磅)缩写，是一种固定长度的度量单位,大小为1/72英寸。如果在web上使用pt做单位的文字，字体的大小在不同屏幕（同样分辨率）下一样，这样可能会对排版有影响，但在Word中使用pt相当方便。因为使用Word主要目的都不是为了屏幕浏览，而是输出打印。当打印到实体时，pt作为一个自然长度单位就方便实用了：比如Word中普通的文档都用“宋体 9pt”，标题用“黑体 16pt”等等，无论电脑怎么设置，打印出来永远就是这么大。

**rem(css3)**:是相对于根元素 <html>，这样就意味着，我们只需要在根元素确定一个参考值

###为何img、input等内联元素可以设置宽、高

img、input 是替换元素；替换元素一般有内在尺寸，所以具有width和height，可以设定。例如你不指定img的width和height时，就按其内在尺寸显示，也就是图片被保存的时候的宽度和高度。
对于表单元素，浏览器也有默认的样式，包括宽度和高度。

##jQuery的选择器与getElementById的区别
`getElementById`返回的是当前HTML元素，`$("#id")`返回的是一个对象数组

实际上 `$("#id")[0] == getElementById`

##网页重构
1、使用马克曼量取精准尺寸（标注：尺寸、颜色）；

2、使用photoshop自动切图脚本，自动切图；

3、使用腾讯 alloydesign 像素级对比设计稿（超高还原度）

##判断一个对象是不是数组

1、对function , String, Number , Undefined 等几种类型的对象来说，完全可以胜任，对于Array
`typeof(arr) == object`

2、javascript中 instancedof 会返回一个boolean 值。
`arr instanceof Array == true`

3、

    function isArray(obj){
    	return Object.prototype.toString.call(obj) === '[object Array]';
    }


##写一个input实时请求加载

    function showResult(str) {
      if (str.length==0) { 
        document.getElementById("livesearch").innerHTML="";
        return;
      }
      if (window.XMLHttpRequest) {
        // code for IE7+, Firefox, Chrome, Opera, Safari
        xmlhttp=new XMLHttpRequest();
      } else {  // code for IE6, IE5
        xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
      }
      xmlhttp.onreadystatechange=function() {
        if (xmlhttp.readyState==4 && xmlhttp.status==200) {
          document.getElementById("livesearch").innerHTML=xmlhttp.responseText;
        }
      }
      xmlhttp.open("GET","livesearch.php?q="+str,true);
      xmlhttp.send();
    }

    <form>
    <input type="text" size="30" onkeyup="showResult(this.value)">
    <div id="livesearch"></div>
    </form>

##参考文章

[HTML head头标签](<meta http-equiv="X-UA-Compatible" content="IE=Edge,chrome=1">)