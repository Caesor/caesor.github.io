---
layout: blog
title: 使用Javascript读取本地图片并显示
categories: Font-end
tags : javascript
---
想要在页面显示本地图片，以前我们通常的做法是将选择的图片文件上传至服务器后端，后端对其进行存储，再将图片的URL返回到前端，前端通过这个URL来显示图片。而HTML5的**FileReader**接口支持本地预览，**FileReader**接口主要是将文件读入内存，并提供相应的方法，来读取文件中的数据，当然就能显示本地图片而不用上传了。

**点击进入查看[demo]({{site.demourl}}/file_reader.html)**

目前比较高端的浏览器都实现了FileReader接口。直接代码示例：

**HTML**

	<input tyle = "file" id="input_file">
	<section id="result"></section>

**Javascript**

	var result = document.getElementById('result');
	var input = document.getElementById('input_file');
	if(typeof FileReader === 'undefined'){
		result.innerHTML = "Sorry, your browser can't support RileReader!";
		input.setAttribute('disabled','disabled');
	}else{
		input.addEventListener('change', readFile, false);
	}
	function readFile(){
		var file = this.files[0];
		if(!/image\/w+/.test(file.type)){
			alert("The file must be image!");
			return false;
		}
		var reader = new FileReader();
		reader.readAsDataURL(file);
		reader.onload = function(){
			result.innerHTML = '<img src="' + this.result + '" alt=""/>'
		}
	}

**另附：FileReader的方法和事件**

![picture1]({{site.blogimgurl}}/2014-11-23-01.png "filereader")


