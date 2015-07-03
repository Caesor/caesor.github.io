---
layout: blog
title: Sass、Compass学习笔记
categories: font-end
tags: sass compass
---

##使用SASS消除样式表冗余
```sass
// 变量
// 通过变量来复用属性值、集中管理、便于皮肤改版
$font-family-global:Microsoft Yahei,"\5FAE\8F6F\96C5\9ED1",arial;
$space:12px;
$yellow:#FFA100;
// 嵌套
// 通过嵌套来避免在选择器中重复写相同的元素
.new{
	@include border-radius(20px);// “compass/css3” 模块
	color: $text-color-normal;
	.new-title{
		font-size: $font-size;
		// 使用 @at-root 将该类 提前到嵌套之外
		@at-root .new-title-second{
			font-size:  $font-size * 0.5;
		}
	}
}
// 混合器（有冗余）
// 复用一大段规则，但是输出的 css 在每一个包含进来的敌方，都会复制一段样式
@mixin horizontal-list {
	li {
		float: left;
	}
}
#header ul.nav{
	@include horizontal-list;
	float: right;
}
// 带参数、默认参数的混合器
@mixin table-padding($spacing: 10px){
	tr,th{
		padding: $spacing;
	}
}
.content-table{
	@include table-padding;
}
// 继承
// 可以通过选择器继承减少冗余
// 解决了使用混合器是每次都复制样式的问题
.btn{ // 输出的css中有 .btn 样式
	border:none;
	padding: 10px 20px;
}
.btn-default{
	@extend .btn;
	background-color: #fff;
	color: red;
}
.btn-pirmary{
	@extend .btn;
	background-color: #000;
	color: white;
}
// 使用占位符避免输出 “父类”
%button-reset { // 输出的css中没有 .button-reset 样式
	margin: 0;
	padding: .5em 1.2em;
	text-decoration: none;
	cursor: pointer;
}
.save {
	@extend %button-reset; 
	color:white;
	background-color: $body-bg;
}
// 使用插值
@mixin thing($class, $prop){
	.thing.#{$class} {
		prop-#{$prop}:val;
	}
}
@include thing(foo, bar);
```

##注意事项

###命名
Sass 变量名、混合器名、函数名中用中划线生命的变量可以使用下划线的方式引用，反之亦然：$link-color 和 $link_color 指向同一个变量

###父类选择器表示父 &
```sass
article a{
	color: blue;
	&:visited{color:#555}
	&:focus{color:#f00;}
	&:hover{color:red}
	&:active{color:#f00}
}
```

###导入sass文件
Sass 的 @import 规则在省城CSS文件是就把相关文件导入进来。且使用Sass的@import规则并不需要指明被导入文件的全名。
`// 导入sass部分文件 themes/_color.scss
// 部分文件不会在编译时单独编译这个文件生成css文件
@import "themes/color"`

###变量默认值
`$fancybox-width:400px !default;`

###嵌套导入
`.blue-theme{ @import "blue-theme"}`

###使用继承的最佳实践
继承只会在生成CSS是肤质选择器，而不会复制大段的css属性。

不要再CSS规范中使用后代选择器去继承CSS规则；


##Compass
是一个Sass框架，他有一套实用的工具，并在长期的实战中总结出最佳实践。

###几个模块
```
// CSS 重置
@import "compass/reset"
// html5重置
@include reset-html5
// 表格辅助器
@import "compass/utilities/tables"
// CSS3 模块
@import "compass/css3"
// 使用 CSS3 圆角
@include border-radius(5px)
// 用省略号代表阶段内容
@include ellipsis
```

###用Compass制作精灵图
```sass
@import "compass/utilities/sprites"
@import "icons/*.png";
// 为每个精灵撰写必要的CSS(background-image, background-position)
@include all-icons-sprites
// 单数精灵输出css
.add-button{
	@include icons-sprite(box-add);
}
// 配置 compass 精灵
// 间距
$icons-spacing: 4px;
// 设置精灵地图布局
$icons-layout: smart;
// 改变默认基础类(.icons-sprites)
$icons-sprite-base-class: ".spritey-mcspriterson";
// 使用精灵辅助器（最佳实践）
$icons: sprite-map("icons/*.png", $layout: smart);
// 撰写精灵的CSS
.next{
	background: sprite($icons, arrow) no-repeat; 
}
```

###URL 配置(config.rb文件中)
```
// 使用相对位置
relative_assets = true
// 修改根路径(如果使用，应关闭相对路径)
http_path = "/super-market"
// 默认图片文件夹
images_dir = 'images';
// 被复制到一个相对与站点根目录的文件夹
http_images_dir = "imgs"
#bottom{background-image: image-url("head.png");}
```

###输出样式
**（开发推荐）**:expanded // 正常

:nested // 层次嵌套式

:compact // 每个选择器压缩至单行显示

**（部署推荐）**:compressed // 压缩

##高性能样式表
1、会比使用带有服务器端@Import 的 HTTP请求

2、使用压缩减少传输时间——gzip压缩

3、图片压缩

4、使用资源托管提高页面加载速度

5、个别特殊内容可以使用内联 dataURL

6、降低选择器的复杂性，避免过分嵌套

##其他

###添加版权提示
```
$copyright-year: unquote("2015");
$company-name: unquote("Github, Inc.");
/*!
  copyright © #{$copyright-year}, #{$company-name}. All Rights Reserved.
*/
```

