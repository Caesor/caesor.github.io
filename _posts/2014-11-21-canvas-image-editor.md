---
layout: blog
title : Canvas图片被跨域数据污染的问题——Canvas tainted by cross-origin data
categories : Font-end
tags : canvas javascript
---
最近在写一个有关canvas图片处理的demo，总是遇到
__Uncaught SecurityError: Failed to execute 'getImageData' on 'CanvasRenderingContext2D': The canvas has been tainted by cross-origin data.__
的报错，让我很苦恼！
于是在**stackoverflow**上提问，终于明白了一些，在这里分享给大家。

##先说说canvas图片处理

Canvas API 用于在网页实现生成图像，并且可以操作图像内容，基本上踏实一个可以用Javascript操作的位图（bitmap）。Canvas允许将图像文件插入画布，做法是读取图片后，使用drawImage方法在画布内进行**重绘**

**getImageData** 方法可以用来读取Canvas的内容，返回一个对象，包含了每个像素的信息。
`var imageData = context.getImageData(0, 0, canvas.width, canvas.height);`
imageData对象有一个data属性，他的只是一个一维数组。该书组的纸，依次是每个像素的红、绿、蓝、alpha通道值，因此该 **数组的长度 = 图像的像素宽度 *  图像的像素高度 * 4** ，每个值得范围是 0－255。这个数组不仅可读，而且可写，因此通过操作这个数组的值，就可以达到操作图像的目的。修改这个数组以后，使用 **putImageData** 方法将数据内容重新绘制在Canvas上。
`context.putImageData(imageData, 0, 0);`

对图像数据作出修改以后，可以使用 **toDataURL** 方法，将Canvas数据重新转化为一般的图像文件形式

    function convertCanvasToImage(canvas){
    	var image = new Image();
    	image.src = canvas.toDataURL("image/png");
    	return image;
    }

**save** 方法用于保存上下文环境，**restore** 方法用于恢复到上一次保存的上下文环境。

##什么是“被污染”的Canvas

当你在本地的 .html 中将本地图片插入Canvas之后，对Canvas进行 getImageData 和 putImageData 操作时浏览器会有报错，这是因为：尽管你可以在你的Canvas中没有CORS协议使用外部图片，但是这样做会污染 Canvas。一旦Canvas被污染了，你就不能把Canvas中的图像倒出来。比如你就不能使用 canvas.toBlob(), toDataURL(), 或者 getImageData() 方法，一旦调用这些方法，浏览器就会抛出一个安全错误提示。

**解决方法**

得需要个服务器才行。。。。当 .html 文件和图片文件在同一个URL下之后 Canvas 就不会出现跨域污染的问题。

在Ubuntu和Mac OS下，我是使用git和github进行代码管理的，于是安装了 **jekyll** 进行本地调试，只要在该文件目录下运行Jekyll就OK了。
`$ jekyll server`
就可以解决该问题啦～

---------------------------------------------------------------
**参考文章：[CORS enabled image](https://developer.mozilla.org/en-US/docs/Web/HTML/CORS_enabled_image)**


