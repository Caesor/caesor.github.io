---
layout: blog
title: Cookie、Session 和 LocalStorage
categories: font-end
tags: optimization
---
##说在前面
数据既可以在浏览器本地存储，也可以在服务器端存储。

浏览器端可以保存一些数据，需要的时候直接从本地获取，sessionStorage、localStorage和cookie都由浏览器存储在本地的数据。

服务器端也可以保存所有用户的所有数据，但需要的时候浏览器要向服务器请求数据。

1、服务器端可以保存用户的持久数据，如数据库和云存储将用户的大量数据保存在服务器端。

2、服务器端也可以保存用户的临时会话数据。服务器端的session机制，如jsp的 session 对象，数据保存在服务器上。实现上，服务器和浏览器之间仅需传递session id即可，服务器根据session id找到对应用户的session对象。会话数据仅在一段时间内有效，这个时间就是server端设置的session有效期。

服务器端保存所有的用户的数据，所以服务器端的开销较大，而浏览器端保存则把不同用户需要的数据分布保存在用户各自的浏览器中。

浏览器端一般只用来存储小数据，而服务器可以存储大数据或小数据。

服务器存储数据安全一些，浏览器只适合存储一般数据。

##概念

###Cookie
cookie，容量4kb，默认各种浏览器都支持，缺陷就是**每次请求，浏览器都会把本机存的cookie发送到服务器**，无形中浪费带宽。

###Web Storage
Web Storage实际上由两部分组成：sessionStorage与localStorage。

sessionStorage用于本地存储一个会话（session）中的数据，这些数据只有在同一个会话页面才能访问并且当会话结束后数据也随之销毁。因此sessionStorage**不是**一种持久化的本地存储。

localStorage用于持久化本地存储，除非主动删除数据，否则数据是永远不会过期的。

##区别

###共同点
都是保存在浏览器端，且同源的。

###区别
cookie数据始终在同源的HTTP请求中携带，即cookie在浏览器和服务器间来回传递。而sessionStorage和localStorage不会自动把数据发给服务器，仅在本地保存。cookie数据还有路径（path）的概念，可以限制cookie只属于某个路径下。

存储大小限制不同。cookie数据不能超过**4k**，同时因为每次http请求都会携带cookie，所以cookie只适合保存很小的数据，如会话标识。sessionStorage和localStorage 虽然也有存储大小的限制，但比cookie大得多，可以达到5M或更大。

数据有效期不同。sessionStorage：始终有效，窗口或浏览器关闭也一直保存，因此用作持久数据；cookie只在设置的cookie过期时间之前一直有效，既是窗口或浏览器关闭。

作用域不同，sessionStorage不在不同的浏览器窗口中共享，即使是同一个页面；localStorage在所有同源窗口中都是共享的；cookie也是在所有同源窗口中都是共享的。

Web Storage支持事件通知机制，可以将数据更新的通知发送给监听者。Web Storage的 API 接口使用更方便。

###sessionStorage与页面的JS数据对象的区别
页面中的JS对象或数据的生存期是仅当当前页面有效，因此刷新页面或转到另一个页面这样的重新加载页面的情况，数据就不存在了。而sessionStorage只要是同源的的同窗口（或tab）中，刷新页面或进入同源的不同页面，数据始终存在。

也就是说只要这个浏览器窗口没有关闭，加载新页面或重新加载，数据仍然存在。

##Web Storage与Cookie相比优势
1、存储空间更大

2、存储内容不会发送到服务器

3、更多丰富的API

4、独立的存储空间：每个域（包括子域）有独立的存储空间，各个存储空间是完全独立的，因此不会造成数据混乱

##Web Storage带来的好处
1、减少网络流量：一旦数据保存在本地，就可以避免在向服务器请求数据。因此减少不必要的数据请求，减少数据在浏览器和服务器件不必要的来回传递。

2、快速显示数据：性能好，从本地读取数据比通过网络从服务器获得数据快得多，本地数据可以即时获得。在家上网也本身也可以有缓存，因此整个页面和数据都在本的的话，可以立即显示。

3、临时存储：很多时候只需要在用户浏览伊组页面期间使用，关闭窗口后数据就可以丢弃了，这种情况使用sessionStorage非常方便。

##参考文章
[浏览器本地数据（sessionStorage、localStorage、cookie）与server端数据](http://han.guokai.blog.163.com/blog/static/13671827120112694851799/)