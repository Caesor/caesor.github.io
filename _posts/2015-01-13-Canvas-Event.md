---
layout: blog
title: Canvas 事件处理
categories: font-end
tags: canvas javascript
---
最近在学习一些有关Canvas的API，现在将Canvas事件处理的一些函数分享给大家。

很多人在学习canvas时会想，平时在javascript中处理一些事件响应的函数和操作在Canvas中同样有效吗？答案是肯定的。今天我在这里汇总分享一些Canvas 事件处理的方法。

##DOM0 级事件处理程序

通过Javascript制定事件处理程序的传统方式，就是将一个函数赋值给一个事件处理程序属性。这种为事件处理程序赋值的方法是在第四代 Web 浏览器中出现的，而且至今仍然为所有现代浏览器所支持。原因一是简单，二是跨浏览器的优势。

每个元素都有字的事件处理程序属性，这些属性通常全部是小写，例如：onclick。

在 Canvas 中，包括：

鼠标点击事件 onclick

鼠标按下事件 onmousedown

鼠标移动事件 onmousemove

鼠标按键释放事件 onmouseup

事件处理程序、以及标准的鼠标坐标获取示例：

	canvas.onclick = function(e){
		alert("Clicked")
		var e = window.event || e;
		var rect = this.getBoundingClientRect();
		// mouseX、mouseY既为鼠标在canvas画布中对应的位置
		var mouseX = e.clientX - rect.left;
		var mouseY = e.clientY - rect.top;
	}

以这种方式添加的事件处理程序会在事件流的冒泡阶段被处理。

也可以删除通过 DOM0 级方法制定的事件处理程序，只要将下面这样将事件处理程序属性的值设置为 null 即可

`canvas.onlick = null`

将事件处理程序设置为 null 之后，再点击 canvas将不会有任何动作发生

##DOM2 级事件处理程序

DOM2 级事件定义了两个方法，用于处理制定和删除事件处理程序的操作： addEventListener() 和 removeEventListener()。 他们都接受 3 个参数：要处理的事件名、作为事件处理程序的函数和一个布尔值。最后这布尔值参数如果是 true，表示在捕获阶段调用事件处理程序；如果是 false ，表示在冒泡阶段调用事件处理程序。

在 Canvas 中，包括：

鼠标点击事件 click

鼠标按下事件 mousedown

鼠标移动事件 mousemove

鼠标按键释放事件 mouseup

事件处理程序示例：

	canvas.addEventListener("click",function(){
		alert(this.id);
	}, false);

**使用DOM2 级方法添加事件处理程序的主要好处是可以添加多个事件处理程序**，例子：

	canvas.addEventListener("click",function(){
		alert(this.id);
	},false);
	canvas.addEventListener("click",function(){
		alert("Hello World");
	},false);

通过 addEventListener() 添加的事件处理程序只能使用  removeEventListener() 来移除；移除时传入的参数与添加处理程序是使用的参数相同。这也意味着**通过 addEventListener 添加的匿名函数将无法移除**

	canvas.removeEventListener("click",function(){ //没用用
		alert(this.id);
	},false);

在这个例子中，我们使用的 `addEventListener()` 添加来一个事件处理程序。虽然使用的 `removeEventListener()` 使用了相同的参数，但是实际上第二个参数与传入的 那一个是完全不同的函数。

	var handler = function(){
		alert(this.id);
	}
	canvas.addEventListener("click", handler, false);

	canvas.removeEventListener("click", handler, false);//有效！！

##IE事件处理

由于今天阐述的内容主要是针对 canvas 的，在IE浏览器中，对 canvas的支持很差，我们在这里先不讨论针对 IE浏览器的事件处理

**但是有一点非常重要，也是很多公司面试题中经常出现的：**

在使用 attachEvent() 绑定多个事件时，这些事件的处理蜀轩不是以添加他们的顺序执行，而是以相反的顺序被触发

##手机触控板触发事件

在 Canvas 作为 HTML5 中最为重要的成员之一，在移动端的地位不能小觑。而在各类手机触屏中的事件与PC端桌面又是有所不同的。

在移动终端如 iphone、ipod  Touch、ipad上的web页面触屏时会产生ontouchstart、ontouchmove、ontouchend、ontouchcancel 事件，分别对应了触屏开始、拖拽及完成触屏事件和取消。

当按下手指时，触发ontouchstart；

当移动手指时，触发ontouchmove；

当移走手指时，触发ontouchend。

当一些更高级别的事件发生的时候（如电话接入或者弹出信息）会取消当前的touch操作，即触发ontouchcancel。一般会在ontouchcancel时暂停游戏、存档等操作。

##跨平台事件处理程序

为你的Canvas添加一组跨平台事件监听让它可以根据你当前的设备自适应，这是一件多酷的事情啊！

不多说，直接上代码！！！

	addHandlerToBoard: function(){
		//charge the device
		var hastouch = "ontouchstart" in window ? true : false;
		var tapstart = hastouch ? "touchstart" : "mousedown";
		var tapmove = hastouch ? "touchmove" : "mousemove";
		var tapend = hastouch ? "touchend" : "mouseup";

		// mousedown
		this.canvas.addEventListener(tapstart, function(e){
			//important!!! for Smooth as silk
			e.preventDefault();

			if(hastouch){
				var mouseX = e.targetTouches[0].pageX;
				var mouseY = e.targetTouches[0].pageY;
			}else{
				var e = window.event || e;
				var rect = this.getBoundingClientRect();
				var mouseX = e.clientX - rect.left;
				var mouseY = e.clientY - rect.top;
			}
			
		}),
		//mousemove
		this.canvas.addEventListener(tapmove, function(e){

		}),
		//mouseup
		this.canvas.addEventListener(tapend, function(e){

		})
	}