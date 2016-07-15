---
layout: blog
title: 从 Array 理解函数式编程
categories: font-end
tags: function array
---
>写 js 很久了，有没有发现用到数组的地方特别多？ 无论是 JSON 返回字段，还是页面的列表，都跟数组操作紧密相关。为何我们如此偏爱数组？

让我们来看一个例子：

```
function process(number){
    console.log('[number] %s', number);
}
const zero = [];
zero.forEach(process);

const one = [42];
one.forEach(process);

const multiple = [1,2,3,4];
multiple.forEach(process);
```

可以大开脑洞**举一反三：**无论操作的对象是**一个还是多个**，无论是**数字、字符串、复杂对象还是文件**，我们都可以通过数组将他们组织起来，通过**统一的方法依次作用于每个元素**。以此来提升编程效率和内容组织。

与此同时，js 为数组提供的相应 API 亦是非常强大，在此并不一一列举。但是，您是否真的完全掌握**这些方法**？使得自己的代码因此而简短、易读、耐用？

“函数式编程”的思想为开发者处理数据集时提供了抽象的工具方法，使得这些方法都高度可重用，可组合。当你总结这些方法的时候会惊奇的发现其实都可以归结为以下5种方法：

>1、map
2、filter
3、concatAll
4、reduce
5、zip

接下来就让我们看看是不是这么回事。

###例一：数组遍历
```
//机智的你肯定立刻想到了这么办！
const names = ["Ben", "Jafar", "Matt","Priya","Brian"];
for(let i = 0; i < names.length; i++){
    console.log(names[i]);
}
```
搞定了！可是你有没有想过这个问题：我们真的需要为打印的 names 指定顺序吗？如果不这样做，怎么做？
```
//或许你想到了使用 forEach 方法，good！
names.forEach((name)=>{
    console.log(name);
});
```
我们会惊喜的发现 forEach 函数指明了我们需要对每一个数组元素的操作，与此同时**隐藏了数组遍历的细节**。

###例二：字段投影
投影出数据结构中的 ｛id, title｝
```
const newReleases = [
    {
        "id": 70111470,
        "title": "Die Hard",
        "boxart": "http://cdn-0.nflximg.com/images/2891/DieHard.jpg",
        "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": [4.0],
        "bookmark": []
    },{
        "id": 654356453,
        "title": "Bad Boys",
        "boxart": "http://cdn-0.nflximg.com/images/2891/BadBoys.jpg",
        "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": [5.0],
        "bookmark": [{ id:432534, time:65876586 }]
    },{
        "id": 65432445,
        "title": "The Chamber",
        "boxart": "http://cdn-0.nflximg.com/images/2891/TheChamber.jpg",
        "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": [4.0],
        "bookmark": []
    },{
        "id": 675465,
        "title": "Fracture",
        "boxart": "http://cdn-0.nflximg.com/images/2891/Fracture.jpg",
        "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": [5.0],
        "bookmark": [{ id:432534, time:65876586 }]
    }
];
var videoAndTitlePairs = [];

// 为了不被你喷，这次毫不犹豫的使用 forEach
newReleases.forEach( video => {
    videoAndTitlePairs.push({id: video.id, title:video.title});
});
```
总结一下：所有的数组对象都进行了两种操作：
1、遍历原数组
2、把每个元素的所需键值对插入新数组
**为何我们不把它抽象出一个新方法？避免中间变量，形式更优雅？**
```
// Arrary 现有！ great！
newReleases.map( video => {id: video.id, title:video.title} );
```
通过使用 map 方法，我们把想要的映射效果应用到了数组上，与此同时**隐藏了操作细节**。

###例三：数据过滤
想象我们需要找到 newRelease 数组中所有 Rating = 5.0 的元素，我们应该怎么办？
```
// 似乎没什么概念，万能的 for 循环搞起！不， forEach 优雅一点
var videos = [];
newRelease.forEach( video => {
    if(video.rating === 5.0){
        videos.push(video);
    }
});
```
总结一下：所有的数组对象都进行了两种操作
1、遍历原数组
2、把通过条件判断的元素插入新数组
**为何我们不把它抽象成一个新方法？**
```
// 这个方法早就有了！
newRelease.filter( video => video.rating === 5.0 );
```
就是这么简单！

###例四：数据合并
有些时候，我们仅仅是需要数组层级中的某一个层，比如：
```
var movieLists = [{
    name: "New Releases",
    videos: [{
        "id": 70111470,
        "title": "Die Hard"
    },{
        "id": 654356453,
        "title": "Bad Boys"
    }]
},{
    name: "Dramas",
    videos: [{
        "id": 65432445,
        "title": "The Chamber"
    },{
        "id": 675465,
        "title": "Fracture"
    }]
}];
```
抽出其中的 videos id 组成一个数组。
虽然 Array 有 concat 方法合并数组，但是**为了方便串式调用，我们来自己实现一个 concatAll 方法**
```
Array.prototype.concatAll = () => {
    let result = [];
    this.forEach(sub => {
        result.push.apply(result, sub);
    });
    return result;
}
```
```
movielists
    .map( movielist => movielist.videos.map(video => video.id))
    .concatAll();     
//得到的结果  [70111470, 654356453, 65432445, 675465]
```
###例五：综合用例
```
// 数据源
const movieLists = [{
    name: "Instant Queue",
    videos : [{
        "id": 70111470,
        "title": "Die Hard",
        "boxarts": [
            { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard150.jpg" },
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard200.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 4.0,
        "bookmark": []
    },{
        "id": 654356453,
        "title": "Bad Boys",
        "boxarts": [
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg" },
            { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys150.jpg" }

        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 5.0,
        "bookmark": [{ id:432534, time:65876586 }]
    }]
},{
    name: "New Releases",
    videos: [{
        "id": 65432445,
        "title": "The Chamber",
        "boxarts": [
            { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber150.jpg" },
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber200.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 4.0,
        "bookmark": []
    },{
        "id": 675465,
        "title": "Fracture",
        "boxarts": [
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
            { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture150.jpg" },
            { width: 300, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 5.0,
        "bookmark": [{ id:432534, time:65876586 }]
    }]
}];

```
期望结果：选出 boxart.width === 150 的数据，且映射出 id, title, boxart。输出结果如下格式
```
[{
    "id": 675465,
    "title": "Fracture",
    "boxart":"http://cdn-0.nflximg.com/images/2891/Fracture150.jpg" 
},{
    "id": 65432445,
    "title": "The Chamber",
    "boxart":"http://cdn-0.nflximg.com/images/2891/TheChamber150.jpg"
},{
    "id": 654356453,
    "title": "Bad Boys",
    "boxart":"http://cdn-0.nflximg.com/images/2891/BadBoys150.jpg"
},{
    "id": 70111470,
    "title": "Die Hard",
    "boxart":"http://cdn-0.nflximg.com/images/2891/DieHard150.jpg"
}];
```
```
movieLists
    .map( movieList => {
        return movieList.videos
            .map( video => {
                return video.boxarts
                    .filter( boxart => boxart.width === 150 )
                    .map( boxart => { id: video.id, title: video.title, boxart: boxart.url };
            })
            .concatAll();
    })
    .concatAll();
```
###例六：数组元素比较
设想我们要找到如下数据中面积最大的
```
const boxarts = [
    { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
    { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture150.jpg" },
    { width: 300, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" },
    { width: 425, height:150, url:"http://cdn-0.nflximg.com/images/2891/Fracture425.jpg" }
]
```
听起来好简单的样子！
```
// 万能的遍历啊！难道你告诉我这次不用遍历？！
var maxSize = -1,
    largestBoxart;
    
boxarts.forEach( boxart => {
    let currentSize = boxart.width * boxart.height;
    if(currentSize > maxSize){
        largestBoxart = boxart;
        maxSize = currentSize;
    }
});
return largestBoxart;
```
总结上面的过程，我们总是使用上一次的计算结果来计算当前的值，这个过程称为 `Reduction`。在上面我们通过遍历的方式实现了这个过程，那么我们是否能抽象出这样一种方法对数组进行 Reduce 操作呢？
```
// Array 原生自带，直接拿来用!
boxarts.reduce( (pre, cur) => pre.width * pre.height > cur.width * cur.height ? pre : cur; );
```
简短的一行，就是这么**任性**！

> 注： reduce 回调完成的参数列表是 `function(previousValue, currentValue, currentIndex, array)`

###例七：数据压缩
有时候我们需要将两个数组中的数据合成一个键值对存入一个新的数组。让我们来实现这样一种方法。
```
Array.zip = (first, second, combineFunc) => {
    let results = [];
    for( let i = 0; i < Math.min(first.length, second.length); i++ ){
        resuls.push(combineFuc(first[i], second[i]));
    }
    return results;
}
```
这里有一组数据源
```
const videos = [{
    "id": 70111470,
    "title": "Die Hard",
    "boxart": "http://cdn-0.nflximg.com/images/2891/DieHard.jpg",
    "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
    "rating": 4.0,
},{
    "id": 654356453,
    "title": "Bad Boys",
    "boxart": "http://cdn-0.nflximg.com/images/2891/BadBoys.jpg",
    "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
    "rating": 5.0,
},{
    "id": 65432445,
    "title": "The Chamber",
    "boxart": "http://cdn-0.nflximg.com/images/2891/TheChamber.jpg",
    "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
    "rating": 4.0,
},{
    "id": 675465,
    "title": "Fracture",
    "boxart": "http://cdn-0.nflximg.com/images/2891/Fracture.jpg",
    "uri": "http://api.netflix.com/catalog/titles/movies/70111470",
    "rating": 5.0,
}],
bookmarks = [
    {id: 470, time: 23432},
    {id: 453, time: 234324},
    {id: 445, time: 987834}
];
```
```
Array.zip( videos, bookmarks, (video, bookmark) => { videoId: video.id, bookmarkId: bookmark.id });
//    [{
//        bookmarkId: 470, videoId: 70111470
//    },{
//        bookmarkId: 453, videoId: 654356453
//    },{
//        bookmarkId: 445, videoId: 65432445
//    }]
```
###例八：综合用例
```
var movieLists = [{
    name: "New Releases",
    videos: [{
        "id": 70111470,
        "title": "Die Hard",
        "boxarts": [
            { width: 150, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard150.jpg" },
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/DieHard200.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 4.0,
        "interestingMoments": [
            { type: "End", time:213432 },
            { type: "Start", time: 64534 },
            { type: "Middle", time: 323133}
        ]
    },{
        "id": 654356453,
        "title": "Bad Boys",
        "boxarts": [
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg" },
            { width: 140, height:200, url:"http://cdn-0.nflximg.com/images/2891/BadBoys140.jpg" }

        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 5.0,
        "interestingMoments": [
            { type: "End", time:54654754 },
            { type: "Start", time: 43524243 },
            { type: "Middle", time: 6575665}
        ]
    }]
},{
    name: "Instant Queue",
    videos: [{
        "id": 65432445,
        "title": "The Chamber",
        "boxarts": [
            { width: 130, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber130.jpg" },
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/TheChamber200.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 4.0,
        "interestingMoments": [
            { type: "End", time:132423 },
            { type: "Start", time: 54637425 },
            { type: "Middle", time: 3452343}
        ]
    },{
        "id": 675465,
        "title": "Fracture",
        "boxarts": [
            { width: 200, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture200.jpg" },
            { width: 120, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture120.jpg" },
            { width: 300, height:200, url:"http://cdn-0.nflximg.com/images/2891/Fracture300.jpg" }
        ],
        "url": "http://api.netflix.com/catalog/titles/movies/70111470",
        "rating": 5.0,
        "interestingMoments": [
            { type: "End", time:45632456 },
            { type: "Start", time: 234534 },
            { type: "Middle", time: 3453434}
        ]
    }]
}];
```
取出每一个 video 的 id,title, type为 middle 的 interesting moment,以及最小的 boxars的url
```
// 方法一：
movieLists
    .map( movielist => movielist.videos.map( video => {
        id: video.id,
        title: video.title,
        time: video.interestingMoments
            .filter(intere => i.type === 'Middle')[0].time,
        url: video.boxarts
            .reduce((pre, cur) => pre.width * pre.height > cur.width * cur.height ? pre.url : cur.url;
        })
    })
    .concatAll();
    
// 方法二：
movieLists
    .map( movielist => { movielist.video
        .map( video => {
            return Array.zip (
                video.boxarts.reduce((pre, cur) => pre.width * pre.height > cur.width * cur.height ? pre.url : cur.url),
                video.interestingMoments.filter(intere => i.type === 'Middle'),
                (boxarts, interestingMoments) => {
                    id: video.id, 
                    title: video.title, 
                    time: interestingMoments[0].time, 
                    url: boxarts.url
                }
            );
        })
        .concatAll();
    })
    .concatAll();

// 方法三：既然复杂的集合既要 map，又要 concatAll。索性我们就聚合一个方法 concatMap()
Array.prototype.concatMap = (projectionFunctionThatReturnArray) => {
    this.map( item => projectionFunctionThatReturnArray(item) )
        .concatAll();
}
    
/***************************************************************
[{
    id: 70111470,
    time: 323133,
    title: "Die Hard",
    url: "http://cdn-0.nflximg.com/images/2891/DieHard200.jpg"
},{
    id: 654356453,
    time: 6575665,
    title: "Bad Boys",
    url: "http://cdn-0.nflximg.com/images/2891/BadBoys200.jpg"
},{
    id: 65432445,
    time: 3452343,
    title: "The Chamber",
    url: "http://cdn-0.nflximg.com/images/2891/TheChamber200.jpg"
},{
    id: 675465,
    time: 3453434,
    title: "Fracture",
    url: "http://cdn-0.nflximg.com/images/2891/Fracture300.jpg"
}];
****************************************************************/
```

纵观以上五种方法及各用例，无一例外都包含以下几点特性：
>1、程序 = Function( input )，函数求值过程无副作用；
2、避免了中间状态和变量；
3、没有 if/while/switch/for 等控制流；
4、函数作为返回值，可串式调用

这正是**函数式编程**的精髓！让你的代码会更简短、更有描述性、更耐用（可维护、热升级）。掌握它们，也很容易明白异步编程的关键。

一句话，这五个方法将是你最强大、最灵活、最有用的函数。



####参考文章：[Functional Programming in Javascript](https://github.com/ReactiveX/learnrx)