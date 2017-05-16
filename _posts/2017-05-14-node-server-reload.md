---
layout: blog
title: Node Server零基础——开发环境文件自动重载
categories: back-end
tags: node koa
---

很多Node服务端开发新同学所面临的第一问题就是，使用Express或者Koa开启一个服务器，修改逻辑文件之后，如何不手动重启服务器，让逻辑文件自动重载生效？

## 前言
在web前端开发中，我们会借助Grunt、Gulp和Webpack等工具的Watch模块去监听文件变化，那服务端应该怎么做？其实文件变化的监听依然可以借助构建工具，但我们还需要自动重启服务或者热重载。本文将介绍三种常见的方法。
## 方案一：fs.watch
使用node原生的fs.watch方法监听文件改动，所谓的“热重载”也不过是及时清除内存中的文件缓存。示例如下：
```
const fs = require('fs'),
    path = require('path'),
    projectRootPath = path.resolve(__dirname, './src');

const watch = project => {
    require('./src'); // 启动APP，自动检索到 src/index.js
    try { // 监听文件夹
        fs.watch(project, { recursive: true }, cacheClean)
    } catch(e) {
        console.error('watch file error');
    }
}
// 清除缓存
const cacheClean = () => {
    Object.keys(require.cache).forEach(function (id) {
        if (/[\/\\](src)[\/\\]/.test(id)) {
            delete require.cache[id]
        }
    })
}
// 启动开发模式
watch(projectRootPath);
```
**注意：**在服务器入口文件`src/index.js`中引用中间件时需要套一层函数，并使用require的方式引入模块才能清除缓存。比如：
```
// 引入中间件或控制器
app.use(async (ctx, next) => {
    await require('./controllers/main.js')(ctx);
});
// 或引入路由
app.use(async (ctx, next) => {
    await require('./router').routes()(ctx, next)
})
```
## 方案二：应用进程管理器
此处以PM2为例，**supervisor、forever**等类似的进程管理工具异曲同工，这里不再赘述。

PM2是一款带有负载均衡功能的Node应用进程管理器，具有 --watch 配置项，用来监听应用目录的变化，一旦发生变化，立即重启。详见：[Auto restart apps on file change](http://pm2.keymetrics.io/docs/usage/watch-and-restart/)。他是真正意义上的重启，不是热替换。
**缺点**：PM2并不提供优雅的方式告知用户何时重启或者杀掉进程。
以下是一个简单的 PM2 配置 （开发环境） start.js，启动进程 `node start.js`。
```
const pm2 = require('pm2');

pm2.connect(function(err) {
    if (err) {
        console.error(err);
        process.exit(2);
    }

    pm2.start({
        "watch": ["./app"], // 开启watch模式，并监听app文件夹下的改动
        "ignore_watch": ["node_modules", "assets"], // 忽略监听的文件
        "watch_options": {
            "followSymlinks": false // 不允许符号链接
        },
        name: 'httpServer',
        script: './server/index.js', // APP入口
        exec_mode: 'fockMode', // 开发模式下建议使用fockModel
        instances: 1, // 仅启用1个CPU
        max_memory_restart: '100M' // 当占用100M内存时重启APP
    }, function(err, apps) {
        pm2.disconnect(); // Disconnects from PM2
        if (err) throw err
    });
});
```
每次修改文件之后保存（Ctrl+S），会有个黑框闪一下，说明应用已经成功重启了。

## 方案三：chokidar + babel 
**chokidar**是对 fs.watch / fs.watchFile / fsevents的一层封装。它的优势包括解决（出自chokidar文档）：
1、在OS X下不能获取文件名；
2、在OS X下Sublime修改文件后不能获取到修改事件；
3、修改文件会触发两次事件；
4、不提供文件递归监听；
5、高CPU使用率；
6、...
这里使用**babel**的原因是想要支持最新的js语法，包括ES2017、Stage-x，以及 import / export default 等模块语法。
下面提供一个完整的监听重载配置文件，并通过注释说明功能和意义。
```
const projectRootPath = path.resolve(__dirname, '..'),
    srcPath = path.join(projectRootPath, 'src'),    // 源文件
    appPath = path.join(projectRootPath, 'app'),    // 编译后输出文件夹
    devDebug = debug('dev'),
    watcher = chokidar.watch(path.join(__dirname, '../src'))

// 启动chokidar监听文件改动
watcher.on('ready', function () {
    // babel编译文件夹目录
    babelCliDir({
        outDir: 'app/',
        retainLines: true,
        sourceMaps: true
    }, [ 'src/' ]) // compile all when start

    require('../app') // 启动APP（编译后的文件）
    
    // 添加监听方法
    watcher
        // 文件新增
        .on('add', function (absPath) {
            compileFile('src/', 'app/', path.relative(srcPath, absPath), cacheClean)
        })
        // 文件修改
        .on('change', function (absPath) {
            compileFile('src/', 'app/', path.relative(srcPath, absPath), cacheClean)
        })
        // 文件删除
        .on('unlink', function (absPath) {
            var rmfileRelative = path.relative(srcPath, absPath)
            var rmfile = path.join(appPath, rmfileRelative)
            try {
                fs.unlinkSync(rmfile)
                fs.unlinkSync(rmfile + '.map')
            } catch (e) {
                devDebug('fail to unlink', rmfile)
                return
            }
            console.log('Deleted', rmfileRelative)
            cacheClean()； //清除缓存
        })
})

// 动态编译文件
function compileFile (srcDir, outDir, filename, cb) {
    const outFile = path.join(outDir, filename),
        srcFile = path.join(srcDir, filename);
        
    try {
        babelCliFile({
            outFile: outFile,
            retainLines: true,
            highlightCode: true,
            comments: true,
            babelrc: true,
            sourceMaps: true
        }, [ srcFile ], {
            highlightCode: true,
            comments: true,
            babelrc: true,
            ignore: [],
            sourceMaps: true
        })
    } catch (e) {
        console.error('Error while compiling file %s', filename, e)
        return
    }
    console.log(srcFile + ' -> ' + outFile)
    cb && cb() // 通常为清除缓存
}
// 清除缓存
function cacheClean () {
    Object.keys(require.cache).forEach(function (id) {
        if (/[\/\\](app)[\/\\]/.test(id)) {
            delete require.cache[id]
        }
    })
    console.log('App Cache Cleaned...'）
}
// 监听程序退出
process.on('exit', function (e) {
    console.log('App Quit')
})
```

**注意：**为了能让缓存失效，我们同样需要在use里包裹一层函数，并以 require 的方式引入路由
```
app.use(async (ctx, next) => {
    await require('./router').routes()(ctx, next)
})
```
## 方案四：开发插件
nodemon和node-dev都是可用于node.js开发版插件，提供简单易用的开发环境。以nodemon为例，全局安装或本地安装都可
`npm install nodemon -g`
然后通过 `nodemon ./server.js localhost 8080`启动开发进程。独立、简单，好用！

详见：[remy/nodemon](https://github.com/remy/nodemon)

## 综上
| 方案     | 优势     | 劣势 |
| :------------- | :------------- | :------------- | 
| fs.watch         | 无依赖，简单      | 监听兼容较差，不能监听app.use层面的改动 |
| PM2         | 配置简单，真正的重启 | 不支持ES6，无通知机制  | 
| chokidar + babel | 兼容性强，支持最新语法  | 需引入配置文件 |
| nodemon | 兼容性强，简单易用  | 需引入插件 |

每个方法都有不同的适用场景。如果想要尝试最新语法，推荐试用方案三；如果追求简单快捷，方案二是不错的选择。

##这就结束了吗？
如果我既想用最新的语法特性，又需要像PM2那样简单，怎么办？babel构建工具（如webpack）对于每个前端开发并不陌生，再加一款PM2足以解决所有问题。