---
layout: blog
title: 为什么使用Webpack
categories: font-end
tags: webpack javascript
---
##Webpack
如果你懂 Broserify, RequireJS 或者类似的打包工具，并且注重这些东西:
+ 代码分包
+ 异步加载
+ 静态资源(图片, CSS)的打包

那么，Webpack 就非常合适！

###介绍
WebPack 是一个模块打包工具，你可以使用WebPack管理你的模块依赖，并编绎输出模块们所需的静态文件。它能够很好地管理、打包Web开发中所用到的HTML、Javascript、CSS以及各种静态文件（图片、字体等），让开发过程更加高效。

为了将模块化技术用于浏览器，人们造出了一大堆工具比如：RequireJS、Browserify、LABjs、Sea.js、Duo等。同时，由于Javascript的标准没有对模块的规范进行定义，所以伟大的程序员们又定义了一系列不同的模块定义，比如：CommonJS、AMD、CMD、UMD等。

Webpack同时支持同步模式(CommonJS)和异步模式(AMD形式)的模块

webpack的优势：
+ require.js的所有功能它都有
+ 编绎过程更快，因为require.js会去处理不需要的文件

###为什么使用 Webpack？
+ 她像 Browserify, 但是将你的应用打包为多个文件. 如果你的单页面应用有多个页面, 那么用户只从下载对应页面的代码. 当他么访问到另一个页面, 他们不需要重新下载通用的代码。
+ 他在很多地方能替代 Grunt 跟 Gulp，因为他能够编译打包 CSS, 做 CSS 预处理, 编译 JS, 打包图片, 还有其他一些.

它支持 AMD 跟 CommonJS, 以及其他一些模块系统, (Angular, ES6). 如果你不知道用什么, 就用 CommonJS

###样式表和图片
首先更新你的代码用 require() 加载静态资源

    require('./bootstrap.css');
    require('./myapp.less');

    var img = document.createElement('img');
    img.src = require('./glyph.png');

当你引用 CSS(或者 LESS 吧), Webpack 会将 CSS 内联到 JavaScript 包当中, require() 会在页面当中插入一个`<style>`标签.

当你引入图片，Webpack 在包当中插入对应图片的 URL，这个 URL 是由 require() 返回的。

你需要配置 Webpack（添加loader）

    modele.exports = {
        entry: { // 多个进入点
            './main.js',
            './feed.js'
        },
        output: {
            path: './build', // 图片和 JS 会到这里来
            filename: '[name].min.js' // 模板基于上边 entry 的 key
        },
        module: {
            loaders: [
                { test: /\.less$/, loader: 'style-loader!css-loader!less-loader'},
                { test: /\.css$/, loader: 'style-loader!css-loader!css-loader'},
                { text: /\.js$/, loader: 'uglify-loader'},
                { test: /\.(png|jpg)$/, loader: 'url-loader?limit=8192'} // <= 8k的图片使用内联base64 URLS
            ]
    }

针对 profile, 在页面当中插入`<script src="build/Profile.min.js"></script>`. feed 页面也是一样。

###优化公用代码
feed 页面跟 profile 页面共用必要代码(比如 React 还有通用的样式和 component). Webpack 可以分析出来他们有多少共用模块, 然后生成一个共享的包用于代码的缓存。

    // webpack.config.js
    var webpack = require('webpack');
    var commonsPlugin =
      new webpack.optimize.CommonsChunkPlugin('common.js');

    module.exports = {
      entry: {
        Profile: './profile.js',
        Feed: './feed.js'
      },
      output: {
        path: 'build',          // 打包文件存放的绝对路径
        publicPath: '/bulid/',  // 网站运行时的访问路径
        filename: '[name].js'   // 打包后的文件名
      },
      plugins: [commonsPlugin]
    };

在上一个步骤的 script 标签前面加上 `<script src="build/common.js"></script>` 你就能得到廉价的缓存了.

###异步加载
CommonJS 是同步的, 但是 Webpack 提供了异步指定依赖的方案. 这对于客户端的路由很有用, 你想要在每个页面都有路由, 但你又不像在真的用到功能之前就下载某个功能的代码.

声明你想要异步加载的那个"分界点". 比如:

    if (window.location.pathname === '/feed') {
      showLoadingState();
      require.ensure([], function() { // 语法奇葩, 但是有用
        hideLoadingState();
        require('./feed').show(); // 函数调用后, 模块保证在同步请求下可用
      });
    } else if (window.location.pathname === '/profile') {
      showLoadingState();
      require.ensure([], function() {
        hideLoadingState();
        require('./profile').show();
      });
    }

Webpack 会完成其余的工作, 生成额外的 chunk 文件帮你加载好.

Webpack 在 HTML script 标签中加载他们时会假设这些文件是怎你的根路径下. 你可以用 output.publicPath 来配置.

    // webpack.config.js
    output: {
        path: "/home/proj/public/assets", // path 指向 Webpack 编译能的资源位置
        publicPath: "/assets/" // 引用你的文件时考虑使用的地址
    }

##browserify
Browserify 可以让你使用类似于 node 的 require() 的方式来组织浏览器端的 Javascript 代码，通过预编译让前端 Javascript 可以直接使用 Node NPM 安装的一些库。在浏览器中，调用browserify编译后的代码，同样写在`<script>`标签中。

用 Browserify 的操作，分为3个步骤。

1. 写`node`程序或者模块
2. 用`Browserify` 预编译成 `bundle.js`
3. 在`HTML`页面中加载`bundle.js`