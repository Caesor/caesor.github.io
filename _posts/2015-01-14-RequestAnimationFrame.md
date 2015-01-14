---
layout: blog
title: 使用 requestAnimationFrame 更好的实现你的js动画
categories: font-end
tags: javascript requestAnimationFrame
---

一直以来，JavaScript的动画都是通过定时器和间隔来实现的。虽然使用CSS transitions 和 animations 使Web开发实现动画更加方便，但多年来以 JavaScript 为基础来实现动画却很少有所改变。直到 Firefox 4的发布，才带来了第一种对 JavaScript 动画的改善的方法。但要充分认识改善，这有利于帮助我们了解web动画是如何演变改进的。

##定时器 Timer

用于创建动画的第一个模式是使用链式setTimeout()调用。在Netscape 3′s hayday 的很长一段时期，开发者都记得一种在网络上随处可见的固定式最新行情状态栏，通常它类似于这样：

	(function(){
	  var msg = "新的广告",
	  len = 25,
	  pos = 0,
	  padding = msg.replace(/./g, " ").substr(0,len),
	  finalMsg = padding + msg;
	  function updateText(){
	    var curMsg = finalMsg.substr(pos++, len);
	    window.status = curMsg;
	    if (pos == finalMsg.length){ pos = 0; }
	    setTimeout(updateText, 100);
	  }
	  setTimeout(updateText, 100);
	})();

如果你想在浏览器中测试这段代码，你可以新建一个 pre 标签用来模拟window.status

这种让人烦恼的web模式，后来遭到对 window.status 禁用的抵抗，但随着 Explorer 4和Netscape 4 的发布，浏览器第一次给开发者更多对页面元素的控制权限，这种技术再次出现。这样就出现了使用javascript动态改变元素大小、位置、颜色等的一种全新动画模式。例如，下面就是一个将div宽度变化成100%的动画（类似于进度条）：

	(function(){
	  function updateProgress(){
	    var div = document.getElementById("status");
	    div.style.width = (parseInt(div.style.width, 10) + 5) + "%";
	    if (div.style.width != "100%"){ setTimeout(updateProgress, 100); }
	  }
	  setTimeout(updateProgress, 100);
	})();

尽管动画在页面上的地方不同，但基本原理却是一样的：做出改变，用setTimeout()间隔使页面更新，然后setTimeout又执行下一次变化，这个过程反复执行，直到动画完成（见进度条动画），早期的状态栏动画是相同的技术，只是动画不一样而已。

##间隔动画 interval

随着成功将动画引入web，新的探索开始了。一个动画已经无法满足了，现在需要多个动画。首次尝试为每个动画创建多个动画循环，在早期的浏览器中使用 setTimeout()来创建多个动画是有点复杂的，所以开发商开始使用 setInterval() 一创建单一的动画循环，来管理页面上所有的动画，一个使用 setInterval() 的基本动画像这样：

	(function(){
	  function updateAnimations(){
	    updateText(); 
	    updateProgress();
	  }
	  setInterval(updateAnimations, 100);
	})();

创建一个小动画库，updateAnimations（）方法将每一个动画(同时看到一个新闻股票和进度条在一起运行)循环执行并进行适当的改变。如果没有动画需要更新，该方法可以退出而不做任何事情，甚至停止动画循环，直到有更多的动画更新做好准备。

动画问题比较棘手的问题是延迟应该为多少。间隔一方面必须足够短，从而使不同的动画都能流畅的进行，别一方面还要足够长，使得浏览器可以完成渲染。大多数浏览器的刷新频率为60HZ，即每秒60次刷新，大多数浏览器的刷新频率都不会比这个更频繁，因为他们知道，最终用户是得不到更好的体验的。

鉴于此，为流畅动画的最佳时间间隔为1000毫秒/ 60，约17ms。在这个频率你会看到流畅的动画，那是因为你最大的接近了浏览器能达到的频率。跟以前的动画相比，你会发现17ms间隔的动画更加平滑，也更快（因为动画更新更频繁，没有做其他任何修改的情况下），多个动画可能需要节流，以免17ms的动画完成得太快。

##问题

即使使用setInterval()为基础的动画循环比多套使用setTimeout()的动画循环高效，这里还是存在问题。无论是setInterval()还是setTimeout()都无法达到精确，这个延迟即你指定的第二个参数仅仅表示何时代码会添加到浏览器的可能被执行的UI线程队列中。如果队列中有其他工作在此之前，那代码将会等到他完成才会执行。简而言之，毫秒级的延迟不是表示何时代码会执行，而是表示何时代码会添加进队列。如果UI线程处于繁忙状态或在处理用户动作，那么代码将不会被马上执行。

平滑动画的关键是理解下一帧何时被执行，直到现在都没有一个方法来保证下一帧将会在浏览器中被绘制。随着 Canvas 的日益流行和新的基于浏览器的游戏的出现，开发商对setInterval()和setTimeout()的不精准越来越感到失望。

浏览器的计时器分辨率加剧了这个问题，计时器对毫秒不精准，这里有一些常见的计时器分辨率：

Internet Explorer 8 and earlier 15.625ms

Internet Explorer 9 and later 4ms.

Firefox and Safari ~10ms.

Chrome has a timer 4ms.

IE在版本9之前的的分辨率为15.625，所以0~15之间的任意值可能是0或15，但没有分别。IE9的计时器分辨率改进为4ms，但涉及到动画时也是不具体的，chrome的计时器分辨率为4ms，firefox 和 safari的为10ms。因此即使你把间隔设定为最佳的显示效果，你也仅仅是得到这个近似值。

##mozRequestAnimationFrame

Mozilla 的 Robert O’Callahan 在思考这个问题，并想出了一个独特的方案。他指出CSS transitions 和 animations的优势在于浏览器知道哪些动画将会发生，所以得到正确的间隔来刷新UI。而javascript动画，浏览器不知道动画正在发生。他的解决方案是创建一个mozRequestAnimationFrame()方法来告诉浏览器哪些javascript代码正在执行，这使得浏览在执行一些代码后得到优化。

mozRequestAnimationFrame()方法接受一个参数，是一个屏幕重绘前被调用的函数。这个函数用来对生成下合适的dom样式的改变，这些改变用在下一次重绘中。你可以像调用setTimeout()一样的方式链式调用mozRequestAnimationFrame()，例如：

	function updateProgress(){
	  var div = document.getElementById("status");
	  div.style.width = (parseInt(div.style.width, 10) + 5) + "%";
	  if (div.style.left != "100%"){
	    mozRequestAnimationFrame(updateProgress);
	  }
	}
	mozRequestAnimationFrame(updateProgress);

##浏览器兼容性写法

在很多人热忠于chrome时，随即创建了webkitRequestAnimationFrame()方法。这个版本与firefox的版本在两方面有着细微的差别。一方面，它不通过回调函数传递时间代码，你将无法知道下次重绘何时发生，另一方面，它添加了第二个可选参数来确定哪一个DOM元素发生改变。因此，如果你知道重绘发生在页面哪个部分的元素内，你可以限制重绘发生的区域。

应该不会感到惊讶，有没有相应的mozAnimationStartTime，因为如果没有下一个重绘的时间信息不是很有益。有，只是webkitCancelAnimationFrame()取消了之前计划的重绘。

如果你不需要精确的时间差异，你可以用下面的方式来创建一个用于Firefox4和chrome10+的动画：

	//标准语句，可以作为统一的 API 准备使用
	window.requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame;
	window.cancelAnimationFrame = window.cancelAnimationFrame || window.webkitCancelAnimationFrame || window.mozCancelAnimationFrame || window.msCancelAnimationFrame;

	var start = null;
	var d = document.getElementById('SomeElementYouWantToAnimate');
	function step(timestamp) { 
	  if (start === null) start = timestamp;
	  var progress = timestamp - start;
	  d.style.left = Math.min(progress/10, 200) + "px";
	  if (progress < 2000) {
	    requestAnimationFrame(step);
	  }
	}
	requestAnimationFrame(step);

这种模式使用可用的方法来创建以花费多少时间为理念的循环动画。Firefox使用时间代码信息是有用的，而Chrome默认为欠精准的时间对象。当用这种模式的时候，时间的差异给你一种多少时间过去了的想法，但不会告诉你Chrome的下一次重绘出现在何时。不过这比只有多少时间过去了的模糊概念要好些。

##总结

mozRequestAnimationFrame()方法的介绍为推动Javascript 动画及web的历史发展有着非常重要的作用。如前所述，JavaScript动画的态几乎和JavaScript的初期一样。随着浏览器逐渐推出CSS transitions 和 animations，很高兴看到基于JavaScript的动画的关注，因为这些在基于 Canvas 的游戏领域将变得更重要和更与CUP联系紧密。知道Javascript何时尝试动画，允许浏览器做更多的优化处理，包括在tab处于后台或移动设备电量过低时停止进程。

该requestAnimationFrame（）API现在正由W3C起草一个新议案，并正由Mozilla和Google努力使之成为Web大舞台的一部分。很高兴能看到这两大集团这么迅速的兼容（可能不完全）实现。


[原帖地址](http://kimhou.com/?reqp=1&reqr=nzcdYaOuo3yvqTLhpTW6)

[参考文献](https://developer.mozilla.org/zh-CN/docs/Web/API/window.requestAnimationFrame)
