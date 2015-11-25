---
layout: blog
title: 谈谈 AngularJS 中的 ng-class
categories: font-end
tags: angular
---
初学 AngularJS 总是对其中的一些细枝末节不适应，可能是因为太适应 jquery了吧，现在这里将 `ng-class` 的使用总结一下

##方案一

    <div ng-controller="HeaderController">
        <div ng-class="{error: isError, warning: isWarning}">{{messageText}}</div>
        <button ng-click="showError()">Simulate Error</button>
        <button ng-click="showWarning()">Simulate Warning</button>
    </div>

    function HeaderController($scope){
        $scope.isError = false;
        $scope.isWarning = false;
        $scope.showError = function(){
            $scope.messageText = 'This is an error';
            $scope.isError = true;
            $scope.isWarning = false;
        };
        $scope.showWarning = function(){
            $scope.messageText = 'Just a warning. Please carry on';
            $scope.isWarning = true;
            $scope.isError = false;
        };
    }

这种方法适用于**添加多个class**，但是对于我们上面的需求，每次都得给 isWarning 和 isError 两个值交替赋值，你是不是觉得很low 很麻烦啊！那就对了，请继续往下看。

##方案二

    <div ng-class="{true: 'alert-success', false: 'alert-danger'}[isActive]">
    </div>

    function Ctr($scope) { 
        $scope.isActive = true;
    }

这种方案适用于**两个class二选一**

##方案三
通过返回一个 className 到 ng-class 实现添加 class

    <span ng-class="warningLevel()"></span>

    $scope.warningLevel = function(){
        return $scope.incompleteCount() < 3 ? "label-success" : "label-warning";
    }