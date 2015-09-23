---
layout: blog
title: 让 Source Map 助你一臂之力
categories: font-end
tags: css less sass compass
---
##从源码转换讲起
通常，我们需要编译 LESS 文件来得到对应 CSS 文件在 HTML 文件中引入。同样，我们需要压缩、合并javascript、CSS文件来投入生产环境。

与此同时，这使得实际运行的代码不同与开发代码，让我们调试和阅读困难重重。

通常，浏览器调试工具会提示哪一行出现了错误，或者哪一行CSS代码需要调整。但是这对于转换后的代码毫无用处。因为所有字符都被压缩到了一行中。

##什么是 Source Map
source Map 是一个信息文件，里面存储这位置信息。也就是说，转化后代码的每一个位置，所对应转换前的位置。

有了它，出错的时候，可以再调试窗口只看到原始代码，这无疑给开发者带来了很大方便。

##Grunt中启用 SourceMap

今天我就在这里介绍如何简单的配置 Grunt 获得 sourceMap 文件，让我们可以再浏览器调试窗口看到 压缩或编译之前的文件样式。

废话不多说，直接上代码

    module.exports = function(grunt){
      ...
      grunt.initConfig({
        ...
        less: {
          development: {
            options: {
              path: ["app/less"],
              sourceMap: true,	//是否生成 sourceMap 文件
              outputSourceFiles: true,	//将 LESS 文件 放入 map
              //将此sourceMapURL 添加进对应的less 文件的最后，以此寻找对应的 Map 文件
              sourceMapURL: 'main.css.map',
              // 定义Map文件的输出路径及文件名
              sourceMapFilename: '.tmp/css/main.css.map'
            },
            files: {
              '.tmp/css/main.css' : 'app/less/*.less'
            }
          }
        }
        ...
      })
    }

运行 grunt Task 之后你会在main.css 文件的最后一行看到

    /*# sourceMappingURL=main.css.map */

同时你会发现在统计目录下看到main.css.map 文件，这样当你打开Chrome 调试器时就看在 style 选项卡中看到对应的样式已经对应到了原来的 LESS 文件上。

![picture1]({{site.blogimgurl}}/2015-06-06-01.png "source map")

大功告成！

其他 uglify、cssmin 等一些常用 Task 配置雷同。

