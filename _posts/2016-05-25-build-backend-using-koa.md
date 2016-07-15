---
layout: blog
title: 使用 Koa + ES6 快速打造后端服务小记
categories: back-end
tags: node koa
---
最近为了尝试使用服务端直出基于react-router的单页应用，了解了一下使用` Koa`搭建web服务器，在此总结分享给大家。

#项目启动
1、文件夹结构
合理的文件组织方式是一个好的项目开端，更是开发一个高可维护性web App的基本准则。以下是我的目录结构：
![](http://km.oa.com/files/photos/pictures/201605/1464101155_80_w406_h379.png)

2、服务器跑起来
Koa 的helloworld程序这里不再复制，可从[koa官网](http://koajs.com/)直接看到 demo，

3、使用 ES6 编写 Webpack 配置
要编写 React 应用，Webpack 神器少不了！大家都知道在控制台运行 `webpack` 命令会自动寻找并加载 `webpack.config.js` 文件。但是，直接使用 ES6 编写 `webpack.config.js` 会在运行 `webpack` 时报错，此时只需将文件名 `webpack.config.js`改为 `webpack.config.babel.js` 即可完美支持ES6语法。

4、ES6 全栈
- 安装 `babel-cli` 使用 `babel-node` 命令替代 `node` 开启服务。
- 安装 `babel-preset-es2015`，`babel-plugin-transform-runtime`，`babel-preset-react`
- 根目录下创建 `.babelrc` 文件
```
{
    "presets": ["es2015", "react"],
    "plugins": ["transform-runtime"]
}
```

#项目进行时
##一、Koa 中间件
所谓“我们不创造代码，我们只是代码的搬用工”，站在巨人的肩膀上可以让我们快速搭建一个系统。在[这里](https://github.com/koajs/koa/wiki#middleware)可以找到所有官方推荐的中间件

###我的推荐
日志工具： koa-logger
Session： koa-session
请求数据解析： koa-body
路由： koa-router
静态文件服务： koa-static
favicon： koa-favicon

##二、配置文件
项目基于模块和中间件的，无论是别人的中间件，还是自己编写的中间件，为了提高模块的可复用性，总是会为中间件提供一系列配置参数。在项目中为了方便统一管理中间件配置，我建议在configs文件夹中创建 `main.js` 存放所有的配置信息，单个复杂配置可单独保存文件在configs文件夹中，例如：
```
\\ main.js

import path from 'path'

const config = {
    name: "Nemo's blog",
    keys: ['nemo'],
    port: process.env.PORT || 8008,
    bodyparser: { strict: false },
    markdown: {
        root: path.resolve(__dirname, '../docs'),
        baseUrl: '/docs',
        layout: path.resolve(__dirname, '../public/blog.html'),
        cache: false
    },
    static: path.resolve(__dirname, '../public'),
    favicon: path.resolve(__dirname, '../public/favicon.ico')
}

export default config;
```

##三、数据库管理
1、mongoose 半壁江山
如果使用 mongoDB 作为数据库，那么我强烈建议使用 `mongoose` 开发，为什么呢？
- 可以为文档创建一个模式结构
- 可以对模型中的对象/文档进行验证
- 应用程序数据可以通过类型强制转换转换成对象模型
- 可以使用中间件来应用业务逻辑挂钩
- 比 mongoDB 使用更容易

2、sequelize 一统江湖
如果使用mySQL 作为数据库，那么我强烈“安利” [sequelize](http://sequelize.readthedocs.io/en/latest/)，sequelize 提供类似于 mongoose 的方式管理数据库，但是更强大百倍，其提供的 BelongsTo(1:1)、HasOne(1:1)、HasMany(1:m)、BelongsToMany(n:m)非常适合关系型数据库的映射。让你在进行数据库操作时编写的代码更加优雅方便，容错性更强。

##四、请求数据校验
其实在第三步中如果使用 `sequelize` 就自带有数据校验的功能，如果您刚好又不想使用它，那我想“总有一款适合你！”这里推荐 `koa-validate` 中间件，具有较强的功能和优雅的写法。

##五、路由
好的路由可以让使用者不读说明wiki就可以明白要干什么，并且怎么干。REST（表述性状态传递）的API正式我们所建议的。需要注意的是REST是**设计风格**而不是**标准**，其定义了一组体系架构原则，我们可以根据这些原则设计以系统资源为中心的Web服务。

###REST架构风格最重要的架构约束有以下6个：
- 客户-服务器：通信只能有客户端单方面发起，表现为请求-响应形式
- 无状态：通信的会话状态应该全部由客户端负责维护
- 缓存：相应内容可以在通信链的某处被缓存，以改善网络效率
- 统一接口：通信链的组件之间通过统一的接口相互通信，以提高交互的可见性
- 分层系统：通过限制组建的行为（即每个组件只能“看到”预期交互的紧邻层），蒋家沟分解为若干等级的层
- 按需代码：支持通过下载并执行一些代码（如：js）对客户端的功能进行拓展

这里也不敢托大，使用 `koa-router` 编写的路由规则，大胆使用 `patch`、`delete`等请求方法。
栗子：
```
import router from 'koa-router'
import * as project from '../controllers/project.controller'

const koarouter = router()
koarouter
    .prefix('/cgi-bin/')
    .get('/project', project.view)
    .post('/project'
        , superAdmin
        , project.add)
    .patch('/project/:pid'
        , authenticate
        , project.update)
    .delete('/project/:pid'
        , superAdmin
        , project.remove)
    
export default koarouter
```

##六、Koa错误处理
在 Koa 中，推荐统一使用 try / catch / throw 的方式来进行错误的触发和捕获。例如可以这样使用：
```
try {
    yield getDataAsync();
} catch(e) {
    // error handle
}
```
在[这里](http://taobaofed.org/blog/2016/03/18/error-handling-in-koa/)可以找到比较详细的介绍和原因。

#项目部署
##一、压缩
推荐使用中间件 `koa-compress`

##二、缓存
对静态文件服务中间件进行配置 `maxage` 等参数，如 `koa-static`中间件：
```
static: {
    path: path.resolve(__dirname, '../public/dist'),
    option: { 
        maxage: 30 * 24 * 60 * 60 * 1000, 
        gzip: true 
    }
},
```
##三、安全
1、为防止 CSRF 攻击，我们可以安装中间件 `koa-csrf`，通过它来为每一个表单创建一个 CSRF token 进行安全校验
2、为防止 XSS 攻击，我们可以在页面响应头部添加 Content-Security-Policy 来过滤不符合配置策略的内容
3、由于一般情况下我们会将静态资源（js / css）部署到 CDN 上，因而 CDN 和 html 页面的主域名一般情况下是不同的。因此添加`Access-Control-Allow-Origin` 头部是非常有必要的，其中的好处之一就是避免跨域脚本报错仅能捕捉到 `script error`。这里我推荐使用`koa-cors`中间件配置头部。

更多安全策略可以参见[这篇文章](https://blog.risingstack.com/node-js-security-checklist/)。

到此为止所用到的中间件都已介绍完毕，这里给出我的 `app.js` 入口文件（有不同意见请大胆提出，轻拍即可）
![](http://km.oa.com/files/photos/pictures/201605/1464101232_57_w721_h829.png)

##四、稳定
1、使用 `babel` 全站转码
尽管可以可以将项目直接部署到线上，但是为了让 PM2 更好的支持，我建议还是使用 `babel` 整站编译到 `dist` 文件夹下，直接部署 `dist` 文件夹到服务器。

2、PM2部署
[PM2](http://pm2.keymetrics.io/docs/usage/quick-start/) 是一个带有负载均衡功能的Node应用的进程管理器。它可以让你的web服务利用全部的服务器上的所有CPU，并保证进程永远都活着，0秒的重载。
使用方式一：
安装 PM2 到全局：使用 PM2 启动应用，比如：`pm2 start app.js -i 0`

使用方式二：
安装到局部（推荐），使用配置文件启动：
`sudo npm install pm2 --save`
```
//pm2-start.js

var pm2 = require('pm2');

var instances = process.env.WEB_CONCURRENCY || -1; // Set by Heroku or -1 to scale to max cpu core -1
var maxMemory = process.env.WEB_MEMORY || 512;    // " " "

pm2.connect(function() {
    pm2.start({
        script    : './dist/app.js',
        name      : 'your-web-app',
        exec_mode : 'cluster',
        instances : instances,
        max_memory_restart : maxMemory + 'M',   // Auto restart if process taking more than XXmo
        log_file : './log/combined.outerr.log',
        error_file: './log/err.log',
        out_file: './log/out.log',
        merge_logs: true,
        env: {
            "NODE_ENV": "development",
            "AWESOME_SERVICE_API_TOKEN": "xxx"
        },
        env_testing: {
            "NODE_ENV": "testing",
        },
        env_production: {
            "NODE_ENV": "production",
        }
    }, function(err) {
        if (err) 
            return console.error('Error while launching applications', err.stack || err);
        
        console.log('PM2 and application has been succesfully started');

        // Display logs in standard output 
        pm2.launchBus(function(err, bus) {
            console.log('[PM2] Log streaming started');
            bus.on('log:out', function(packet) {
                console.log('[App:%s] %s', packet.process.name, packet.data);
            });
            bus.on('log:err', function(packet) {
                console.error('[App:%s][Err] %s', packet.process.name, packet.data);
            });
        });
    });
});
```
控制台运行： `node pm2-start.js` 启动pm2

3、运行日志
你可能已经注意到在上方的 `pm2` 配置文件中已经有这样的配置：
```
log_file : './log/combined.outerr.log',
error_file: './log/err.log',
out_file: './log/out.log',
merge_logs: true
```
它表示在根目录下 log 文件夹生成一个**合并日志**、一个**请求流水日志**以及一个**错误日志**

##五、Nginx 部署
Nginx 想必大家并不陌生，这里也正是看中了 Nginx 出色的 HTTP反向代理能力，才把它放置在 Node.js 前端，用来处理我们的各种需求。尤其是当项目后期访问量加大是，一个进程、一台服务器已经不能满足我们的需求，这是 Nginx 就可以发挥自己反向代理的能力——在Nginx 后端添加多个服务器或启动多个进程来分担访问压力。
当然，还有一些不得不说的优点
1、静态文件性能
Node.js 静态文件受制于他的单线程异步I/O，使用  Nginx 处理静态文件的性能基本上是纯 Node.js的2倍以上。
2、反向代理规则
有事我们希望根据 session、IP等一些特殊规则和需求，使用Nginx 配置文件就可以简单实现。
3、扩展性
最典型的就是加入 Lua 语言的拓展。胶水语言Lua赋予了Nginx复杂逻辑判断能力，并保持一贯的高效性。
4、稳定性和转发性能
Nginx 在同样的负载下，相比 Node.js 占用的CPU和内存资源更少。
5、安全性
Nginx 经过一些配置可以有效地抵挡类似 slowloris等的 Dos 攻击。而 Node 的这方面还不够。
6、运维管理
Nginx 的反向代理配置非常方便，轻松的修改一些配置就可以在一台服务器上让多个站点同时占用 80 端口。






