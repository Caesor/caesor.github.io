---
layout: blog
title: PC版部落项目React实践系列（五）——Webpack进阶
categories: font-end
tags: react webpack
---
##说在前面
PC兴趣部落目前由于使用了React直出，页面各项性能指标使人悦目。本篇将深入探讨目前PC部落所采用webpack打包优化策略，以及探讨PC部落并未使用的 webpack Code Splitting 代码分包、异步模块加载特性。看看它们又是如何对PC部落的性能起到进一步的催化作用。

##为什么要使用webpack
如果你曾经使用过 Broserify, RequireJS 或类似的打包工具，并注重：代码分包、异步加载、静态资源打包（图片/CSS）。那么 webpack 就是帮你构建项目的利器！简单一句话：在webpack中，所有资源都被当作是模块，js可以引用 css , css 中可以嵌入图片 dataUrl。

###webpack特性
对应不同文件类型的资源，webpack有对应的模块 loader ，比如对于 less， 使用的是 `less-loader`，你可以在这里找到 [所有loader](http://webpack.github.io/docs/list-of-loaders.html).
webpack 具有requireJS 和 browserify 的功能，但仍有自己的新特性：
1、对 CommonJS、AMD、ES6的语法做了兼容；
2、对js、css、图片等资源文件都支持打包；
3、串联式模块加载器以及插件机制让其具有更好的灵活性和拓展性，例如对 coffeeScript、ES6的支持；
4、有独立的配置文件 webpack.config.js;
5、可以将代码切割成不同 chunk，实现按需加载，降低了初始化时间；
6、支持 SourceUrls 和 SourceMaps，易于调试；
7、具有强大的 Plugin 接口，大多是内部插件，使用起来比较灵活；
8、webpack 使用异步 IO 并具有多级缓存，使得 webpack 在增量编译上更快！

###PC部落为什么混用Grunt和webpack
自React诞生以来，耳熟能详的是 React+webpack 开发大法，而且在大多数 React 网络教程中也很少提及同时采用了 Grunt 联合构建项目。

Grunt 可以对整个项目文件做复制、删除、合并、压缩等等。而Webpack 的优势在于对静态文件（js/jsx/coffeeScript/css/less/sass/iamges）按不同模块加载（包括按需加载）——这正是我们对webpack感兴趣的地方，各个模块组建化（可以将一个组建的图片、样式、脚本、页面放在同一个文件夹中）。所以，在项目中二者分工不同，各司其职。

**注：**使用 gulp 替换 grunt 当然也是没有问题。

#webpack配置
webpack 有多种配置方式，由于PC部落中静态资源文件较多，使用配置文件进行打包会方便很多。

通常情况下，如果我们只使用 webpack 构建项目，那么配置 webpack.config.js 即可。由于在PC部落中使用了 grunt，并在 grunt 组合任务中调用 webpack 任务，因此需要在 grunt 的任务配置中添加 webpack.js（使用了`load-grunt-config`插件） 进行配置。

#####配置总览

    var taskConfig = {
        dev: {
            entry: {    // 入口文件，考虑到多页面资源缓存，我们打成多个包
                "index": path.resolve(config.srcPath, "pages/index/index.jsx"),
                "detail": path.resolve(config.srcPath, "pages/detail/detail.jsx"),
                ...
            },
            resolve: {  // 请求重定向，显示指出依赖查找路径
                alias: {
                    img: path.resolve(config.srcPath + 'img'),
                    comps: path.resolve(config.srcPath + 'pages/components')
                    ...
                }
            },
            output: {   // 输出文件
                path: config.devPath + '/js',                               // 文件绝对路径
                filename: "[name].min.js",                                  // 输出文件名
                publicPath: "http://s.url.cn/qqun/xiaoqu/buluo/p/js/",      // 公共访问路径，替换CDN
                chunkFilename: "[name].chunk.min.js"                        // 异步加载时需要被打包的文件名
            },
            module: {   // 各类文件 loader
                noParse: [],            // 忽略解析的文件
                preLoaders: [{          // 预加载的模块
                    test: /\.jsx$/,
                    exclude: /node_modules/,
                    loader: 'jsxhint-loader'
                }],
                loaders: [{             // 各式加载器
                    test: /\.jsx$/,
                    loader: 'jsx-loader',
                    include: path.resolve(config.srcPath)
                }, {
                    test: /\.less$/,
                    // 使用“！”链式loader，从右向左依次执行
                    loader: ExtractTextPlugin.extract("style-loader", "css-loader!less-loader"),
                    include: path.resolve(config.srcPath)
                }, {
                    test: /\.(jpe?g|png|gif|svg)$/i,
                    // inline base64url for <=1500 images
                    loader: 'url-loader?limit=1500&name=images/[name].[hash].[ext]',    
                    include: path.resolve(config.srcPath)
                }]
            },
            externals: {    // 指定采用外部 CDN 依赖的资源，不被webpack打包
                "react": "React",
                "react-dom": "ReactDOM"
            },
            plugins: [
                ...
                // 公共模块独立打包配置
                new CommonsChunkPlugin("common", "common.min.js", ["index", "detail", "barindex", "search"]),
                // 独立打包css文件以外链形式加载
                new ExtractTextPlugin("../css/[name].min.css")
            ],
            watch: true,
            keepalive: true,
            lessLoader: {
                lessPlugins: [
                    new LessPluginAutoPrefix()
                ]
            }
        },


##webpack打包优化

###1、请求重定向
`resolve.alias` 是webpack 的一个配置项，它的作用是把用户的一个请求重定向到另一个路径。
比如：

    resolve: {  // 显示指出依赖查找路径
        alias: {
            comps: 'src/pages/components'
        }
    }

这样我们在要打包的脚本中的使用 `require('comps/Loading.jsx');` 其实就等价于 `require('src/pages/components/Loading.jsx')`。这犹如《高性能javascript》中给查询压力较大的对象给了一个别名，通过使用别名可以将本例减少几乎一半的时间。

###2、忽略对已知文件的解析
`module.noParse`，如果你**确定一个模块中没有其它新的依赖**，就可以配置这项，webpack 将不再扫描这个文件中的依赖。
比如我们在入口文件 `entry.js` 中检测到对资源 `src/pages/components/ueditor.min.js`资源的请求，如果我们配置：

    module: {
        noParse: [/ueditor/]
    }

`noParse`规则中的`/ueditor/`一条生效，所以 webpack 直接把依赖打包进了 `entry.js`。增加这样的配置会让 webpack 编译时间更短。

###3、使用公用CDN
考虑到web上有很多的公用 CDN 服务，那么我们可以将 `react` 从 bundle 中分离出来，进而不会被 webpack 打包， 作为外部依赖引用 CDN 。 方法是使用 `externals` 声明一个外部依赖。
如：

    module:{
        externals: {
            // 方式一：申明为外部依赖并指定别名
            "react": "React",
            "react-dom": "ReactDOM"
            // 方式二：true 为外部依赖，false 则不是
            a: false,   // a is not external
            b: true     // b is external
        },
    }

并在 HTML 代码中加上一行

    <script src="//cdn.bootcss.com/react/0.14.2/react.js">
    <script src="//cdn.bootcss.com/react/0.14.2/react-dom.js">

这样我们在js中引入`React = require('react')` ， webpack 就不会把 react 打包进来而直接引用CDN，这样做可以让 webpack 编译时间缩减一大半！


##系列插件

###CommonsChunkPlugin 
开发中需要将**多个页面的公用模块独立打包**，从而可以利用浏览器缓存机制来提高页面加载效率，减少页面初次加载时间，只有当某功能被用到时才去动态加载。这就要使用到 webpack 中的 CommonsChunkPlugin 插件。

#####使用：

    var CommonsChunkPlugin = require("webpack/lib/optimize/CommonsChunkPlugin");
    module.exports = {
      ...
      /*
       * @param 1  将公共模块提取，生成名为 common 的chunk
       * @param 2  最终生成的公共模块的 js 文件名
       * @param 3  公共模块提取的资源列表
       */
      new CommonsChunkPlugin("common", "common.min.js", ["index", "detail", "barindex", "search"])
    }

###ExtractTextPlugin
webpack 中编写js文件时，可以通过 `require` 的方式引入其他静态资源，可通过loader对文件自动解析并打包文件。
通常我们会将 js 文件打包合并，css 文件在页面header中嵌入 style 的方式载入页面。但在**开发**过程中我们并不想将样式打包在脚本中（最好可以独立生成css文件，以外链形式加载）。
ExtractTextPlugin 插件可以帮我们达到这样的效果。

#####安装：
`npm install extract-text-webpack-plugin –save-dev`

#####使用：

    var ExtractTextPlugin = require("extract-text-webpack-plugin");
    module.exports = {
      ...
      plugins: [ new ExtractTextPlugin("../css/[name].min.css") ]
    }

这样配置就可以将 js 中的 css 文件提取，并以指定的文件名来进行加载。

###LessPluginAutoPrefix
顾名思义，就是autoPrefix插件，用来补全CSS的厂商前缀（-webkit-, -moz-, -o-）;

#####使用：

    var LessPluginAutoPrefix = require('less-plugin-autoprefix');
    var taskConfig = {
        dev: {
            ...
            lessLoader: {
                lessPlugins: [
                    new LessPluginAutoPrefix()
                ]
            }
        }


##Code Splitting
对于一个大型的web app，我们把所有的 js 文件合成一个显然是非常低效的，因为有些 js 模块并不是我们当前页面所需要的（这会大大增加页面首屏渲染时间）。Webpack 就是这样一种神器，为您提供优质的代码分包服务，从此“妈妈再也不用担心页面按需加载的问题了”！

###方式一：require
`require(dependencies, callback)` 遵从 AMD 规范定义的**异步**方法。使用该方法时，所有的依赖被**异步加载并从左至右立即执行**，依赖都被执行后，执行`callback`。

###方式二：require.ensure
`require.ensure(dependencies, callback)` 遵从 CommonJS 规范，在需要的时候才下载依赖的模块。当所有的依赖都被加载完毕，便执行 `callback`（注：require作为callback的参数）。
细心的同学可能还记得 `output` 配置中有

    output: {
        ...
        chunkFilename: "[name].chunk.min.js"
    }

`chunk` 到底是什么？ `chunk` 又是怎么生成的呢？
为了实现部分资源的异步加载，有些资源是不打包到入口文件里面的。于是我们使用 `require.ensure` 作为代码分割的标识。`require.ensure` 会创建一个 chunk ，且可以指定该 chunk 的名称（注：如果这个chunk已经存在了，则将本次依赖的模块合并到已经存在的chunk中），最后这个 chunk 在 webpack 构建时会单独生成一个文件。
比如我们要根据当前运行平台，加载两个不同的UI组建，那么：

    var platform = Util.getPlatform();
    if( platform === "ios"){
        require.ensure(['./components/dialog'], function(require){
            ...
        }, 'popup'); // 最后一个参数是 chunk 名
    }
    if( platform === "android"){
        require.ensure(["./components/toast"], function(require){
            ...
        }, 'popup');
    }

通过webpack打包之后，会生成一个 `popup.chunk.min.js` 文件。在不同的运行平台上，我们会发现 popup.chuck.min.js 文件的内容是相同的（因为我们配置的 chunk 名都是 popup）。
如果我们想让按需加载的模块再次拆分成 dialog 和 toast，两个文件，仅仅需要将 `require.ensure` 中配置的chunk 名改不同，即可在代码被执行时加载单一文件。

###注意点：
1、`require` ：加载模块，并立即执行；
2、`require.ensure`：仅仅加载模块，但不会执行；
3、不用在 html 中显示调用生成的 chunk 文件，按需加载时会自动调用；
4、不用担心第三方库被反复打包的问题，因为我们已经使用 `CommonsChunkPlugin` 对公共部分进行提取。