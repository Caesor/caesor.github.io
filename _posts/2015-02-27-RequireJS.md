---
layout: blog
title: require.js 学习笔记
categories: font-end
tag: requireJS
---

##AMD——异步模块定义简介

###传统javascript的问题
让我们来看看一般情况下JavaScript代码是如何开发的：通过`<script>`标签来载入JavaScript文件，用全局变量来区分不同的功能代码，全局变量之间的依赖关系需要显式的通过指定其加载顺序来解决，发布应用时要通过工具来压缩所有的JavaScript代码到一个文件。当Web项目变得非常庞大，前端模块非常多的时候，手动管理这些全局变量间的依赖关系就变得很困难，这种做法显得非常的低效。

###AMD的引入
AMD（Asynchronous Module Definition）提出了一种基于模块的异步加载JavaScript代码的机制，它推荐开发人员将JavaScript代码封装进一个个模块，对全局对象的依赖变成了对其他模块的依赖，无须再声明一大堆的全局变量。通过延迟和按需加载来解决各个模块的依赖关系。模块化的JavaScript代码好处很明显，各个功能组件的松耦合性可以极大的提升代码的复用性、可维护性。这种非阻塞式的并发式快速加载JavaScript代码，使Web页面上其他不依赖JavaScript代码的UI元素，如图片、CSS以及其他DOM节点得以先加载完毕，Web页面加载速度更快，用户也得到更好的体验。

CommonJS的AMD规范中只定义了一个全局的方法：

    define(id?, dependencies?, factory); 

**id**表示该模块的标识，为可选参数。

**dependencies**是一个字符串Array，表示该模块依赖的其他所有模块标识，模块依赖必须在真正执行具体的factory方法前解决，这些依赖对象加载执行以后的返回值，可以以默认的顺序作为factory方法的参数。dependencies也是可选参数，当用户不提供该参数时，实现AMD的框架应提供默认值为[“require”，”exports”，“module”]。

**factory**是一个用于执行该模块的方法，它可以使用前面dependencies里声明的其他依赖模块的返回值作为参数。若该方法有返回值，当该模块被其他模块依赖时，返回值就是该模块的输出。

###RequireJS简介
RequireJS是一个非常小巧的JavaScript模块载入框架，是AMD规范最好的实现者之一。最新版本的RequireJS压缩后只有14K，堪称非常轻量。它还同时可以和其他的框架协同工作，使用RequireJS必将使您的前端代码质量得以提升。

目前最新版本的RequireJS 1.0.8在IE 6+、Firefox 2+、Safari 3.2+、Chrome 3+、Opera 10+上都工作的很好。

###RequireJS的机制

RequireJS使用head.appendChild()将每一个依赖加载为一个script标签。

RequireJS等待所有的依赖加载完毕，计算出模块定义函数正确调用顺序，然后依次调用它们。

在同步加载的服务端JavaScript环境中，可简单地重定义require.load()来使用RequireJS。build系统就是这么做的。该环境中的require.load实现可在build/jslib/requirePatch.js中找到。

未来可能将该部分代码置入require/目录下作为一个可选模块，这样你可以在你的宿主环境中使用它来获得正确的加载顺序。

##RequireJS实战

###data-main属性
当你下载RequireJS之后，你要做的第一件事情就是理解RequireJS是怎么开始工作的。当RequireJS被加载的时候，它会使用data-main属性去搜寻一个脚本文件（它应该是与使用src加载RequireJS是相同的脚本）。data-main需要给所有的脚本文件设置一个根路径。根据这个根路径，RequireJS将会去加载所有相关的模块。下面的脚本是一个使用data-main例子：

    <script src="scripts/require.js" data-main="scripts/app.js"></script>

另外一种方式定义根路径时使用的配置函数，后面我们将会看到。requireJs假设所有的依赖都是脚本，那么当你声明一个脚本依赖的时候你不需要使用.js后缀。

###配置 RequireJS
如果你想改变RequireJS的默认配置来使用自己的配置，你可以使用require.configh函数。config函数需要传入一个可选参数对象，这个可选参数对象包括了许多的配置参数选项。下面是一些你可以使用的配置：

baseUrl——用于加载模块的根路径。

paths——用于映射不存在根路径下面的模块路径。

shims——配置在脚本/模块外面并没有使用RequireJS的函数依赖并且初始化函数。假设underscore并没有使用RequireJS定义，但是你还是想通过RequireJS来使用它，那么你就需要在配置中把它定义为一个shim。

waitSeconds——指定最多花多长等待时间来加载一个JavaScript文件，用户不指定的情况下默认为7秒。

deps——加载依赖关系数组
下面是使用配置的一个例子：

    require.config({
        //By default load any module IDs from scripts/app
        baseUrl: 'scripts/app',
        //except, if the module ID starts with "lib"
         paths: {
            lib: '../lib'
        }, 
        // load backbone as a shim
        shim: {
            'backbone': {
                //The underscore script dependency should be loaded before loading backbone.js
                deps: ['underscore'],
                // use the global 'Backbone' as the module name.
                exports: 'Backbone'
            }
        },
        waitSeconds: 10
    });

在这个例子中把根路径设置为了scripts/app，由lib开始的每个模块都被配置在scripts/lib文件夹下面，backbone 加载的是一个shim依赖。

###使用RequireJS定义一个javascript模块
模块是进行了内部实现封装、暴露接口和合理限制范围的对象。ReuqireJS提供了define函数用于定义模块。按章惯例每个Javascript文件只应该定义一个模块。define函数接受一个依赖数组和一个包含模块定义的函数。**通常模块定义函数会把前面的数组中的依赖模块按顺序做为参数接收。**

    define(["./cart", "./inventory"], function(cart, inventory) {
            //return an object to define the "my/shirt" module.
            return {
                color: "blue",
                size: "large",
                addToCart: function() {
                    inventory.decrement(this);
                    cart.add(this);
                }
            }
        }
    );

注：尽量不要提供模块的ID，如AMD规范所述，这个ID是可选项，如果提供了，在RequireJS的实现中会影响模块的可迁移性，文件位置变化会导致需要手动修改该ID。


###使用RequireJS加载一个javascript文件

    require(['jquery'], function ($) {
        //jQuery was loaded and can be used now
    });

require方法里的这个字符串数组参数可以允许不同的值，当字符串是以”.js”结尾，或者以”/”开头，或者就是一个URL时，RequireJS会认为用户是在直接加载一个JavaScript文件，否则，当字符串是类似”my/module”的时候，它会认为这是一个模块，并且会以用户配置的baseUrl和paths来加载相应的模块所在的JavaScript文件。

这里要指出的是，RequireJS默认情况下并没有保证myFunctionA和myFunctionB一定是在页面加载完成以后执行的，在有需要保证页面加载以后执行脚本时，RequireJS提供了一个独立的domReady模块，需要去RequireJS官方网站下载这个模块，它并没有包含在RequireJS中。

    require(["domReady!", "./js/a.js", "./js/b.js"], function() {
    	myFunctionA();
    	myFunctionB();
    });   

##综合运用RequireJS
当RequireJS与其他框架一起工作的时候，显然它是可以作为统一的加载器来加载其他框架。鉴于jQuery、Dojo等已经支持了AMD，那么就有可能在定义自己的模块的时候，通过适当配置，直接把其他框架的模块作为依赖对象。

    <script>
    	require = {
    		packages: [
    			{
    				name : "dojo",
    				location : "dojo",
    				main : "lib/main-brower",
    				lib : "."
    			},{
    				name : "dijit",
    				loction : "dijit",
    				main : "lib/main",
    				lib : "."
    			}
    		],
    		paths : {
    			require : "./js" 
    		},
    		ready : function(){
    			require(["my/module"], function(module){});
    		}
    	}
    </script>

在定义module这个模块时就可以直接将Dojo和Dijit里的模块作为依赖对象了。

把jQuery作为一个依赖模块来使用也非常简单，只需要在RequireJS里的config里做相应的配置就可以了。

    <script>
    require.config({
    	paths : {
    		"jquery" : "./js/jquery-1.7"
    	}
    });
    require(["jquery"], function($){});
    </script>

**参考文章：**

[http://requirejs.org/docs/api.html](http://requirejs.org/docs/api.html)

[http://www.csdn.net/article/2012-09-27/2810404](http://www.csdn.net/article/2012-09-27/2810404)

[http://www.oschina.net/translate/getting-started-with-the-requirejs-library?cmp](http://www.oschina.net/translate/getting-started-with-the-requirejs-library?cmp)



