---
layout: blog
title: Angular 采坑小计
categories: font-end
tags: angular
---
1、在低版本angular中（<1.3,运营平台当前使用版本v1.2.15）使用`ng-repeat`时会对即将循环输出的数据做重复检测，如果发现元素有重复，就会报错 **Duplicates in a repeater are not allowed.**。
比如：

    $scope.todo = ["eat","eat","eat","eat","eat","eat","sleep","sleep","sleep","sleep"];
    <div ng-repeat="elem in todo">

解决办法：使用`ng-repeat`时添加`track by`语句，比如：

    <div ng-repeat="elem in todo track by $index">

2、在低版本angular中（<1.3）中，在`form`中使用`ng-repeat`输出双向绑定的`input`时，为了避免`name`属性重复而使用`{{index}}`,比如：

    <form name="todoform">
        <div ng-repeat="elem in todo track by $index">
            <input type="text" name="rank{{$index}}" ng-model="elem" required/>
            <span ng-show="todoform.rank{{$index}}.$error.required">none</span>
        </div>
    </form>

这样表单验证并不会正常触发！
解决方法：使用`ng-form`

    <form name="todoform">
        <div ng-repeat="elem in todo track by $index">
            <ng-form name="subform">
                <input type="text" name="rank" ng-model="elem" required/>
                <span ng-show="subform.rank.$error.required">none</span>
            </ng-form>
        </div>
    </form>

**注意**：在1.3及以上版本中使用 `name="rank{{$index}}"`可正常验证，但是千万不要使用`name="rank[{{$index}}]"`!

3、在书写`factory`时请按照angular标准依赖规范来，不然压缩之后会无法加载依赖。
错误的书写方式：

    app.directive('sensitiveWords', function($http){
        return {
            ...
        }
    });

正确的书写方式：

    app.directive('sensitiveWords', ['$http', function($http){
        return {
            ...
        }
    }]);


4、做一个多tab选项卡激活效果不要太复杂

    <ul>
        <li ng-class="btnActive('default')" ng-click="changeRankStyle(0)">系统默认</li>
        <li ng-class="btnActive('school')" ng-click="changeRankStyle(1)">校园风格</li>
        <li ng-class="btnActive('kunfu')" ng-click="changeRankStyle(2)">武侠风格</li>
    </ul>


    $scope.changeRankStyle = function(n){
        switch(n) {
            case 0:
                $scope.selectedStyleName = 'default';
                break;
            case 1:
                $scope.selectedStyleName = 'school';
                break;
            case 2:
                $scope.selectedStyleName = 'kunfu';
                break;
        }
    };
    $scope.btnActive = function(category){
        return category == $scope.selectedStyleName ? 'rank-title-active':'';
    }


5、在书写表单验证时，请使用`directive`增加复用性和维护性，并减少`scope`下的变量！
如：

     <input type="text" class="ranklist-item-tab"
        name="rank"
        ng-model="selectedRankStyle[separate($index,2)]"
        length-limit="rank"
        special-symbol="rank"
        sensitive-words="rank"
        required/>

6、`ng-class`花式用
方案A：多选

    <div ng-class="{error: isError, warning: isWarning，success: isSuccess}"></div>

方案B：二选一

    <div ng-class="{true: 'alert-success', false: 'alert-danger'}[isActive]"></div>

方案C：条件选择

    <div ng-class="warningLevel()"></div>

7、`scope`下变量的直接赋值是“传引用”，如果想要“传值”，请使用`angular.copy(object)`
8、如果想使用`filter`
这样是可以的：

    <span class="rank-logo">LV{{ $index | separate:2}}</span>

这样也是ok的：

    <span class="logo-color-{{$index | separate:2}}"></span>

但这样是不行的：

    <input name="rank[{{$index | separate:2}}]" />
