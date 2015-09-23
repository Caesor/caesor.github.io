---
layout: blog
title: AngularJS学记笔记
categories: font-end
tag: angular
---
##一些非常好的资源
[AngularJS Book](https://github.com/peiransun/angularjs-cn)

[AngularJS编码风格](https://github.com/johnpapa/angular-styleguide)

[AngularJS资源汇总](https://github.com/jmcunningham/AngularJS-Learning/blob/master/ZH-CN.md)

##最佳实践

###单一职责

###使用javascript闭包

    (function() {
        'use strict';

        angular
            .module('app')
            .factory('logger', logger);

        function logger() { }
    })();

###Module
1、避免命名冲突

2、避免使用变量来命名 module

3、Only set once and get for all other instances.

    - Use `angular.module('app', []);` to set a module.
    - Use `angular.module('app');` to get a module.

4、使用有名字的函数，避免使用匿名函数

###Controller
1、避免使用函数表达式

##分页——Paging
