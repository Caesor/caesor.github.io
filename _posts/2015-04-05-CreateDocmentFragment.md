---
layout: blog
title: createDocumentFragment
categories: font-end
tags: javascript
---
[翻译原文](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/createDocumentFragment)

##语法
```
//docFragment 是一个空的DocmentFragment 对象的引用
var docFragment = document.createDocumentFragment();
```
DocmentFragments 是一个DOM节点，但从来都**不属于**主DOM节点树的一部分，通常用来创建一个文档片段，将元素插入文档片段，然后将该片段插入DOM节点树。在DOM节点树中，文档片段被他的所有子节点所替换。

由于 document fragment 在**内存**中，并不是DOM节点树的一部分，所以**将子节点append到其中时并不会引起页面重排**。因此使用 document fragments 浏览器渲染会有**更好的性能**。

所有浏览器均支持documentFragment，包括 IE6。

##例子

    var ul = document.getElementsByTagName("ul")[0]; // assuming it exists
    var docfrag = document.createDocumentFragment();
    var browserList = ["Internet Explorer", "Mozilla Firefox", "Safari", "Chrome", "Opera"];
    browserList.forEach(function(e) {
      var li = document.createElement("li");
      li.textContent = e;
      docfrag.appendChild(li);
    });
    ul.appendChild(docfrag);
    // a list with well-known web browsers


##createElement vs createDocmentFragment

1、createElement创建的元素可以使用innerHTML，createDocumentFragment创建的元素使用innerHTML并不能达到预期的修改文档内容的效果，只是作为一个属性而已。

    var fragment_1 = document.createDocumentFragment();
    fragment_1.innerHTML = '<p>通过createDocumentFragment创建</p>';
    document.body.appendChild(fragment_1);
    // 无法生效，因为无法通过 innerHTML 插入内容
    var fragment_2 = document.createElement('p');
    fragment_2.innerHTML = '<p>通过createElement创建</p>';
    document.body.appendChild(fragment_2);
    // 但可以查询 innerHTML 属性
    alert(fragment_2.innerHTML)

2、createElement 创建的元素可以重复操作，但 createDocumentFragment创建的元素是一次性的，添加之后就不能操作了。

    function $(id){
        return document.getElementById(id);
    }
    var outer = $('outer');
    var inner = $('inner'); 
    $('btn_1').onclick = function(){
        var div = document.createElement('div');
        div.innerHTML = '<p>测试createElement</p>';
        document.body.appendChild(div);
        setTimeout(function(){
        	//此次append时，原来body 中的div自动清除，因为这是一个 “浅拷贝”
            outer.appendChild(div);
            setTimeout(function(){
                outer.removeChild(div);
            },1000)
        },1000)
    }
    $('btn_2').onclick = function(){
        var p = document.createElement('p');
            p.innerHTML = '测试DocumentFragment';
        var fragment = document.createDocumentFragment();
        	//只能通过这种方式赋值，但值并没有赋值到 innerHTML 上
            fragment.appendChild(p);
            //innerHTML 只是一个属性，不能修改 content
            fragment.innerHTML = '<p>测试DocumentFragment</p>';
            fragment.innerHTML = '<span>测试DocumentFragment</span>';
        document.body.appendChild(fragment);
        setTimeout(function(){
            outer.appendChild(fragment); //报错，因为此时文档内部已经能够不存在fragment了
            setTimeout(function(){
                outer.removeChild(fragment);
            },1000)
        },1000)
    }
