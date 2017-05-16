---
layout: blog
title: 多场景下的图片优化策略
categories: optimization
tags: image
---

## 一、常规策略
0、能不用图片就不用
透明、边框、圆角、阴影、渐变都可以用CSS实现，SVG、canvas或iconfont等在主流浏览器中都已经普遍支持，可代替图片，且体积更小更具优势。SVG可无失真无限放大缩小；Canvas可实现2D、3D动态效果等。

1、压缩
+ 简单压缩。使用在线图片压缩网站如：[https://tinypng.com/](https://tinypng.com/)，[智图](https://zhitu.isux.us/)
+ 使用构建工具，grunt(grunt-imagemin)、gulp(gulp-book)、webpack(url-loader)

2、CDN部署 & 合理缓存设置 & 多尺寸支持
CDN部署不用多说，几乎是任何图片服务的标配，要求支持 gzip 压缩。
针对不同的图片类型设置不同的浏览器缓存时间（max-age），包括：用户头像、Logo、icon、文案配图等；
针对不同的设备尺寸，采用不同尺寸的图片大小（PC、Mobile、Pad）；
针对不同的设计稿图片尺寸需求，拉取不同的图片尺寸（用户头像采用100像素，新闻类列表页可采用400像素，详情页拉取1000像素），可节省流量、缩短请求传输时间

3、使用Data-url/base64
在http1.x时代，为了减少小图片请求带来的损耗（主要是协议头）而将其转化为Base64编码内嵌在CSS或者HTML中。存在的问题是转换为Base64编码的体积要比原图本身大。通常我们只将小于8kb的图片转换为Base64

4、合并精灵图
使用构建工具将多个小图合并在一张大图上，特点是减少HTTP请求数。缺点是：“一荣俱荣，一损俱损”。一旦加载失败，所有图片都会展示失败。

5、接入webP、sharpP等优秀图片格式，减少带宽流量，加快图片加载速度
sharpP是腾讯自研的图片压缩方案，压缩效果要比webP更加优秀，具体介绍请前往：[sharpP——引领新一代图片压缩技术](http://km.oa.com/group/1719/articles/show/237185?kmref=search&from_page=1&no=1)。

在实际项目中的策略优先级是：SharpP > WebP > 普通格式（jpg、png）
判断当前环境是否支持，可参考下列代码：
```
// webP
window.isSupportWebP = !![].map && document.createElement('canvas')
.toDataURL('image/webp').indexOf('data:image/webp') === 0;

// sharpP
var img = new Image();
img.src = 'https://gpic.qpic.cn/gbar_pic/S1enqicZz6UKoibvAiby8vpNOBuYLDHic8tSsVyVHF2XQmtsG7z7zo2MYA/?tp=sharp'; // 一像素的sharpP图片
img.onload = e => window.isSupportSharpP = 1;
img.onerror = e => window.isSupportSharpP = 0;
```
SharpP和WebP接入详情可参考：
[基于CDN的sharpP图片无痛接入方案(优化60%+)](http://km.oa.com/group/20251/articles/show/274758?kmref=search&from_page=1&no=3)
[WebP无痛接入方案](http://km.oa.com/group/20251/articles/show/241615?kmref=search&from_page=1&no=1)

6、渐进增强
图片加载有**逐行扫描**和**交错扫描**，在弱网络环境下，使用交错扫描加载方式的大图会先让用户看到图片的大致轮廓，对用户更加友好。具体可在Photoshop图片导出中选择**交错**体验尝试。
![alt](http://7tszky.com1.z0.glb.clouddn.com/Fo8q3huYFyQma_rsSvo28dUyd7mN)

7、渐进增强 2
知名博客发布网站[Medium](https://medium.com/)中的图片采用了 **纯色 -> 低质量拉伸图片+高斯模糊 -> 正常标准图片**的加载过程。
![](http://km.oa.com/files/photos/pictures/201702/1487232930_1_w639_h178.png)
可以通过分析其代码发现，其原理是这样的：
1）渲染一个div，通过对容器设置宽高使其比例和大小与最终图片的比例和大小相同，避免图片加载出来的时候导致的页面的重排；
2）使用 img 标签来加载一张原图质量的 10% ~ 20%的小图。由于体积小，所以可以很快加载并显示；
3）小图加载完成后使用 canvas 绘制高斯模糊透明效果覆盖在小图之上。同时请求最终要加载的大图；
4）最终大图加载完成后隐藏canvas。

**缺点：**费流量，适合WiFi和有线网络。实现稍微复杂

8、按需加载
懒加载：需要的时候才去加载，不需要不加载。
预加载：对于即将看到的图片预拉取，比如图片查看器中，预先拉取当前图片中后2~3张。推荐使用[AlloyViewer— H5图片查看器](https://github.com/AlloyTeam/AlloyViewer)
常见的方式如：
```
// 长列表
if(document.documentElement.clientHeight > targetElement.getBoundingClientReact().top){
    // load image
}

// 图片查看器
const PRELOADNUM = 3
if(index <= current + PRELOADNUM && index >= current - PRELOADNUM){
    // load image
}
```
9、请求失败占位图 & 重试机制
图片加载失败后需要有占位图（纯色/图片），并且有稍后自动重试的功能。
```
state={
    src: http://image.qq.com/12345.png,
    retry: 0
}
render() {
    return <img onError={this.onError.bind(this)} src={src} />
}
onError(e, src){
    if(this.state.retry > 0) { // 仅重试一次，否则加载默认图
        this.setState({ src: http://image.qq.com/defaultImg.png })
    }else {
        let { src. retry } = this.state,
            src += '&time=' + ~~(Math.random()*1E6);
        this.setState({ src, retry： ++retry });
    }
}
```

10、媒体查询 & `picture` 标签
实验性功能详见[HTML picture元素](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/picture)

## 二、活动页
活动H5页面的特点是具有高并发访问量。尤其是在节假日访问量飙涨的状况下，对访问量的预估对于图片优化具有非常重大的意义。
比如图片CDN服务器的并发访问临界值为200W每秒，活动峰值访问为300W每秒。每一个访问页面的用户都会拉取评论列表中的用户头像，这时我们应该怎样优化？
1、活动页面从设计上就应该避免大量的图片展示，尽可能减少图片（去除用户头像、评论列表头像）；
2、使用一张背景大图，避免多个图片使用绝对定位层叠
3、如果有可能，最好使用CSS、SVG、canvas或iconfont代替图片；
4、减少图片请求数，小图片采用base64内嵌在CSS中；
5、采用离线包、离线包预先Server Push；

**极限条件下策略**
1、CDN图片请求失败展示默认头像
2、可打包一定数量默认头像至APP离线包，请求失败时分配随机头像；
3、活动页**首屏**评论区采用假数据，预先将前十位用户头像打入离线包

**注意：**避免使用精灵图，一旦精灵图请求失败，一组图片都会挂掉

## 三、手Q内H5应用
1、UI图片资源离线化
离线包是利器、离线包是利器、离线包是利器。重要的事说三遍
2、完全支持接入WebP、SharpP
3、其他遵循常规策略

## 四、动画和游戏
>常见的动画实现方式有 gif、Video、SVG、Canvas。动画和游戏越复杂，需要的图片就越多，便会导致加载耗时增加。

1、单图动画 & 骨骼动画
使用精灵图合并资源。
缺点：大图有很多空白区域；大图真的很“大”，每一个样式里面都要书写绝对定位（可通过自动化工具解决）

2、帧动画
通常的帧动画素材都是一组连贯的图片，横向或者竖向。单帧动画的特点是：每一帧图片的相似度较高。因此可利用图片diff算法删减并记录相同的部分，利用二进制文件保存图片序列。可大幅压缩素材的传输体积。

3、Blob合并文件加载（推荐使用）
在node.js中可通过 `new Blob([data], {"type":"text/html"}）`创建一个Blob对象，其中可保存图片二进制数据。提供`Blob.close()`方法关闭Blob对象释放底层资源，提供了`Blob.slice([start[, end[, contentType]]])`方法切割数据。
实验表明：使用Blob时合并文件分片传输相比单纯的传图具有更小的传输体积，更快的加载速度。

更多详情可参考：
[利用 html5 Blob对象优化Web页面](http://km.oa.com/group/15849/articles/show/282933?kmref=search&from_page=1&no=1)
[H5动画-Web前端性能优化](http://km.oa.com/group/2068/articles/show/281538)

## 五、迎接HTTP2.0时代
1、多路复用、头部压缩让图片并行加载更快；
2、由于每个域名都需要创建一个连接，因此图片资源需要**域名收归**。创建的连接数减少，意味着对延迟和TCP拥塞控制的敏感度降低，可大幅降低连接延迟；
3、精灵图、Base64转化都不再有必要，拆分开来可以更好的利用浏览器缓存；
4、目前主流浏览器(PC端Chrome、Firefox等)都支持DNS预解析(link dns-prefetch)、TCP预连接(link preconnect)、资源预加载(link prefetch、preload）等新特性，可用于图片加载速度提升。