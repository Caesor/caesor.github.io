---
layout: blog
title: Javascript 中的绝句
categories: font-end
tags: javascript
---
##特殊字符的魅力

###说在前面—鸭子类型
鸭子类型是动态类型的一种风格，在这种风格中，一个对象有效的语义，不是由继承自特定的类或者实现特定的接口，而是由当前方法和属性的集合决定。

“当看到一只鸟走起来像鸭子、游泳起来像鸭子、叫起来也像鸭子，那么这只鸟就可以被称为鸭子。”

在鸭子类型中，关注的不是对象的类型本身，而是它是如何使用的。例如，在不使用鸭子类型的语言中，我们可以编写一个函数它接受一个任意类型的对象，并调用它的**走**和**叫**方法。如果这些需要被调用的方法不存在，那么将引发一个运行时错误。任何拥有这样的正确的**走**和**叫**教方法的对象都可被函数接受的这种行为引出了以上表述。

###取两次反
	// 一个感叹号可以将其转换成Boolean类型的值，双感叹号将其转换成**对应**的Boolean值
	!![] //true; ![] //false
	!!{} //true; !{} //false
	!!false //false; !false //true
	!!true //true; !true //false
	!!undefined //false; !undefined //true
	!!null //false; !null //true
	!!"" //false; !"" //true

###取整同时转成数值型
	/* JS里面所有的数值类型都是双精度浮点数，进行位运算是，会首先将这些数字转换为整数
	 * 由于鸭子类型的存在，JS将尝试位运算的所有对象尝试转化为整数(包括：NaN, undefined)
	 * x | 0 == x
	 * x ^ 0 == x
	 * ~~x == x
	 */
	'10.567890'|0	// 结果: 10

	'10.567890'^0	// 结果: 10

	-2.23456789|0 	// 结果: -2

	~~-2.23456789 	// 结果: -2

	'10.56fdbas'|0	// 结果: 0
	'10.56fdbas'^0	// 结果: 0
	~~'10.56fdbas'	// 结果: 0

###日期转数值
	var d = +new Date(); //1295698416792 时间戳

###类数组对象转数组：
	var arr = [].slice.call(arguments)

###漂亮的随机码：
	Math.random().toString(16).substring(2);	//14位

###合并数组：
	var a = [1,2,3];
	var b = [4,5,6];
	Array.prototype.push.apply(a, b);
	console.log(a); 	// [1,2,3,4,5,6]

###交换值：
	a= [b, b=a][0];

###将一个数组插入另一个数组的指定位置：
	var a = [1,2,3,7,8,9];
	var b = [4,5,6];
	var insertIndex = 3;
	a.splice.apply(a, Array.concat(insertIndex, 0, b));
	// a: 1,2,3,4,5,6,7,8,9

###删除数组元素：
	var a = [1,2,3,4,5];
	a.splice(3,1);

###快速取数组最大和最小值
	Math.max.apply(Math, [1,2,3]) //3
	Math.min.apply(Math, [1,2,3]) //1
	(出自http://ejohn.org/blog/fast-javascript-maxmin/)

###条件判断：
	var a = b && 1; 
	// 相当于
	if (b) {
	  a = 1
	}

	var a = b || 1; 
	// 相当于
	if (b) {
	  a = b;
	} else {
	  a = 1;
	}

###判断IE:
	var ie = /*@cc_on !@*/false;

##参考文章
[javascript绝句欣赏](http://site.douban.com/106371/widget/notes/22456/note/142716442/)