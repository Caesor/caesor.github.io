---
layout : blog
title : 玩转swift字符串——Advanced
tags : swift ios
categories : ios-development
---
最近在尝试学一些iOSswift开发，要做一个“四则运算计算器”，其中涉及到很多字符串运算，终于在不断查阅文档和stackoverflow的情况下完成。感慨道，swift的开发手册太过于简单呐，完全不够用，很多东西还得自己摸索，但其中唯一不变的真理就是：“请在 **playground** 里面好好玩耍吧！！！”

在这里，我把我搜集到的知识与大家分享一下

##String 和 Character
**字符串声明**
`var str = "Hello, playground"
let str2 = String()`

**判断是否为空**
`str.isEmpty`

**字符串长度**
`countElements(str)
//如果是NSString，那么长度就是
var str3:NSString = str
str3.length`

**字符串的遍历**
`for eachChar in str{
	println eachChar
}
//这时有人会问，那么我想从后往前遍历呢？！
for eachChar in reverse(str){
	println eachChar
}`

**字符串拼接**
`//拼接字符串型
str += " Hello"
//拼接非字符串型
var ch:Character = "!"
str.append(ch)
//或者
str += "\(ch)"`

**字符串的插值**
`//String Interpolation
let imInt:Int = 2
let imDouble:Double = 3.14
let imBool:Bool = true
let imString:String = "hello"
let imTuple = (2,4)
let imOptionnal:Int? = nil
let imCharacter:Character = "!"
println("\(imInt)\n\(imDouble)\n\(imBool)\n\(imString)\n\(imTuple)\n\(imOptionnal)\n\(imCharacter)")`

##String基础操作
**字符串比较**
`//String Comparison
let str_a = "abc"
let str_b = "abc"
str_a == str_b	//true
let str_c = "abd"
str_a < str_c	//因为 d 要比 c 靠后，所以 true
let str_d = "abcd"
str_c > str_d	//虽然 str_c 有三个，str_d 有四个，但是 d 比 c 要靠后， 所以 true
str_b < str_d	//前三位相同，但是str_d 有第四位，所以 true`

**前缀／后缀 我在这里就不说了，每个文档都会讲**

##导入Foundation使用更多字符串功能
**使用 Foundation**, 其实每个".swift"文件都已经默认导入 Foundation 了，所以不需要我们再在头部导入
`//default import Foundation
str.capitalizedString 	//大写首字母
str.uppercaseString 	//全部转大写
str.lowercaseString 	//全部转小写`

**字符串修剪**
`var str5 = "     !hi!!!    "
//清除空格
str5.stringByTrimmingCharactersInSet(NSCharacterSet.whitespaceCharacterSet())
//清除特定字符 如“!”
str5.stringByTrimmingCharactersInSet(NSCharacterSet(charactersInString:" !"))`

**字符串分割**
`var str6 = "welcome to play swift"
//以空格作为分割依据
str6.componentsSeparatedByString(" ")
var str7 = "welcome to play swift!Step-by-step learn from now"
//以特定字符作为分割依据 如:" ","!","-"三个
str7.componentsSeparatedByCharactersInSet(NSCharacterSet(charactersInString: " !-"))`

**字符串链接**
`var str8 = "-"
str8.join(["2","3","4","5"])	//输出：2-3-4-5`

**String、Int、Float、Double 互相转换**
`//String to Int
var str = "4"
str.toInt()
//String to Float
var str1 = "3.14"
//下面的方法 return 3.14000010490417 ，个人建议最好还是使用下面的方法转换成double比较好
NSString(string: str1).floatValue
//String to Double
var str2 = "1.3333"
NSString(string: str2).doubleValue	//return 1.3333
//Int to String
var num = 234
String(num)
//float to String
var num1 = 3.14
String(format:"%f", num1)   //return "3.140000"
String(format:"%.2f", num1) //return "3.14"`

**Something else**

如果你对 3.140000 这种形式的数字感到恶心的话，那么你就对了因为我也是这样，试想：
`var result = 3.125 + 4.875 	//return 8.0 , It's good!
//但是你如果使用String(format:)进行转换
String(format: "%f", result) 	//return 8.00000  It's bad!
//所以你不如直接使用“字符串插值”的方法
“\(result)”`

##String.Index 和 Range
在swift语言中，你会发现很多函数参数中要求 String.Index 类型，那下面我们就用实例来说明什么是 String.Index 类型
**范围**
`str7 = "Welcome to play swift!step by step learn from now"
str7.rangeOfString("step")	//return 22..<26
//从后往前搜索，需要第二个参数
str7.rangeOfString("step", options:NSStringCompareOptions.BackwardsSearch)	//return 30..<34
//我们同样可以设置第二个参数，表示匹配串忽略大小写限制“原字符串中是 Welcome，我们搜索 welcome”
str7.rangeOfString("welcome", options: NSStringCompareOptions.CaseInsensitiveSearch)	//return 0..<7`
**String.Index 是什么？**
`str7 = "Welcome to play swift!step by step learn from now"
str7.startIndex 	//0
str7.endIndex 		//49
//range between 0..<49
let strRange = Range<String.Index>(start:str7.startIndex, end:str7.endIndex)
//注意这里，startIndex 和 endIndex 是 String.Index 类型的，不是 Int 类型！！
let startIndex:String.Index = str7.startIndex 				//0
let endIndex:String.Index = advance(str7.startIndex, 10) 	//10
//这里定义了一个 Range 类型，由 start 和 end 两个参数构成
let searchRange = Range<String.Index>(start:startIndex, end:endIndex)
//在定义的范围“searchRange”内进行搜索操作
str7.rangeOfString("step", options: NSStringCompareOptions.CaseInsensitiveSearch, range: searchRange)`

**截取子串**
`var toIndex = advance(str7.startIndex, 4)
//从字符串开头开始到即第4个字符
str7.substringToIndex(toIndex) // Welc
var fromIndex = advance(str7.startIndex, 14)
//从字符串某个位置开始向后 14 位
str7.substringFromIndex(fromIndex) // y swift!step by step le...
//在字符串 4 位置开始，到 14 位置结束
str7.substringWithRange(Range<String.Index>(start:toIndex, end:fromIndex)) //ome to pla`

**获取字符串最后一个字符**
`//方法一
str7.substringFromIndex(str7.endIndex.predecessor())
//方法二－较繁琐
var lastChar:String.Index = advance(str7.startIndex, countElements(str7) - 1)
var lastChar = str7.substringFromIndex(lastChar)`

**字符串插入**
`var insertIndex = advance(str7.startIndex, 22)
//在第 22 字符的后面插入
str7.insert("!", atIndex: insertIndex)`

**字符串删除**
`str7.removeAtIndex(insertIndex)
str7.removeRange(Range<String.Index>(start:str7.startIndex, end:insertIndex))`

**字符串替换**
`var replaceEndIndex = advance(str7.startIndex, 13)
str7.stringByReplacingCharactersInRange(Range<String.Index>(start:str7.startIndex,end:replaceEndIndex), withString:"Step-by-step")`