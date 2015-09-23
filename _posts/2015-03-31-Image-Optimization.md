---
layout: blog
title: web优化之图片加载
categories: font-end
tags: optimization
---
##前言
图片资源是Web项目的中很重要的组成部分，也是交互设计中的一个很重要的体现，往往一张图能胜过千言万语，所谓“一图胜百言”就是这个道理。大量的图片也会给服务器带来很大的压力，据统计：图片内容已经占到了互联网内容容量的62%，也就是说超过一半的流量和时间都用来下载图片。

如何最大优化图片资源？

图片优化的手段有哪些？

##即时加载机制
即时加载是最常见的加载方式，一般都是通过给 `img` 标签制定具体的src 值，那么等页面渲染的时候，遇到该标签的话，就会下载指定URL图片，并渲染出来。

在现代浏览器运行上面的代码你就会发现以下几点：

1、大部分浏览器都会**并发**下载图片，图片的下载**没有阻塞**；

2、`src` 的值如果相同，也指挥下载**一次**，也就是说相同的图片，不会下载多次；

3、图片的下载和渲染会阻塞整个页面的渲染

4、图片越大，下载的时间就越长

###即时加载的运用场景是
图片量少，页面少于3屏

##预先加载机制
图加载图片是提高用户体验的一个很好方法。图片预先加载到浏览器中，访问者可以享受到几块的加载速度。这对图片画廊以及占据很大比例网站来说非常有利，它保证了图片快速、无缝的发布，也可以帮助用户在浏览你网站内容时获得更好的用户体验。

###预先加载的原理
**预知用户将发生行为，提前加载后用户所需的图片**。

比如：漫画有好几屏，等页面把漫画加载好了，我们就慢慢往下拉滚动条，看完了当前页，我们按下连接跳到下一页面。当我们欣赏漫画的时间，用来加载下一页或者两页。这样当我们看完当前页，进入下一页的时候，由于前面已经加载完毕，页面瞬间加载完毕。

###方法一：使用CSS和Javascript实现预加载
单纯使用CSS，可容易、高效的预加载图片

    #preload-01 { background: url(http://domain.tld/image-01.png) no-repeat -9999px -9999px; } 

使用该方法加载图片会同页面的其他内容一起加载，增加了页面的整体加载时间。为了解决这个为题，我们增加一些javascript来推迟预加载时间，直到页面加载完毕。

    function preloader() {  
        if (document.getElementById) {  
            document.getElementById("preload-01").style.background = "url(http://domain.tld/image-01.png) no-repeat -9999px -9999px";  
            document.getElementById("preload-02").style.background = "url(http://domain.tld/image-02.png) no-repeat -9999px -9999px";  
            document.getElementById("preload-03").style.background = "url(http://domain.tld/image-03.png) no-repeat -9999px -9999px";  
        }  
    }  
    function addLoadEvent(func) {  
        var oldonload = window.onload;  
        if (typeof window.onload != 'function') {  
            window.onload = func;  
        } else {  
            window.onload = function() {  
                if (oldonload) {  
                    oldonload();  
                }  
                func();  
            }  
        }  
    }  
    addLoadEvent(preloader);  


###方法二：仅使用Javascript实现预加载
上述方法在实际实现过程中会耗费太多时间。

    function preloader() {  
        if (document.images) {  
            var img1 = new Image();  
            var img2 = new Image();  
            var img3 = new Image();  
            img1.src = "http://domain.tld/path/to/image-001.gif";  
            img2.src = "http://domain.tld/path/to/image-002.gif";  
            img3.src = "http://domain.tld/path/to/image-003.gif";  
        }  
    }  
    function addLoadEvent(func) {  
        var oldonload = window.onload;  
        if (typeof window.onload != 'function') {  
            window.onload = func;  
        } else {  
            window.onload = function() {  
                if (oldonload) {  
                    oldonload();  
                }  
                func();  
            }  
        }  
    }  
    addLoadEvent(preloader);    


###方法三：使用Ajax实现预加载
使用Ajax实现图片预加载，不仅仅预加载图片，还会预加载CSS、javascript等相关的东西。

**使用Ajax比直接使用javascript的优越之处在于javascript和CSS的加载不会影响到当前页面。**

    window.onload = function() {  
        setTimeout(function() {  
            // XHR to request a JS and a CSS  
            var xhr = new XMLHttpRequest();  
            xhr.open('GET', 'http://domain.tld/preload.js');  
            xhr.send('');  
            xhr = new XMLHttpRequest();  
            xhr.open('GET', 'http://domain.tld/preload.css');  
            xhr.send('');  
            // preload image  
            new Image().src = "http://domain.tld/preload.png";  
        }, 1000);  
    };  

##延迟加载机制
图片延迟加载——懒加载，通常用于图片比较多的网页。

如果一个页面图片比较多，而且页面高度或宽度有好几屏，页面初次加载时，只显示可视区域的图片，当页面滚动的时候，图片进入可视区域进行加载，这样可以显著提高页面的加载速度，更少的图片并发请求也可以减轻服务器的压力。如果用户仅仅在首屏停留，还可以节省流量。

###延迟加载的原理
页面初次加载时获取图片在页面中的位置并**缓存**（每次取offset的值会引发页面的重排-reflow），计算出可视区域，当图片的位置出现在可视区域中，将src的值替换成真实的地址，此时图片就开始加载了。

    <img src="images/placeholder.png" alt="" real-src="images/realimg.jpg">

当页面滚动的时候，在判断图片已经缓存的位置值是否出现在可视区域内，进行替换src加载。**当所有的图片都加载完之后，相应的出发时间卸载，避免重复操作引起的内存泄露。**

###非常好的延迟插件

[jquery_layload](https://github.com/tuupola/jquery_lazyload/)


##相关文章
[3-ways-preload-images](http://perishablepress.com/3-ways-preload-images-css-javascript-ajax/)


