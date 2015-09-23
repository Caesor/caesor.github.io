---
layout: blog
title: Angular-Working with Promises(翻译)
categories: font-end
tags: javascript ES6
---
##Angular中的Promise
Promise 是一种对未来将要发生的所关注事件的**“承诺”**，比如：对一个服务器的Ajax请求的响应。Promise 并不是AngularJS独有的，可以在很多库中找到它的身影，包括 jQuery，但不同库的开发者所表达的设计思想和个人偏好还是有所不同的。

一个 Promise 需要两个对象：一个用来接收通知的 promise 对象，还有一个用来发送通知的 deferred 对象。对于大多数需求来说，最简单理解 promise 对象的方式就是把它当成一个特殊的事件；而 deferred 对象就用来触发未来的“一些任务或活动”。

我并没有在说到“一些任务或活动”时有所含糊，因为 promise 能够用来表示未来发生的任何事。我们接下来将用一个例子而不是 Ajax 请求来展现这种方式的灵活性。

##理解为什么Promise并不是传统的事件
在这个点上，我们也许会疑惑为什么我们仅仅为了达到和传统的javascript事件处理器一样的效果，而做了这么多麻烦的事来创建 deferred 对象和 promise 对象？

promise 确实表现出了与事件相同的基本功能：它们允许一个组件去指明未来发生某些特定的事情之后会触发的响应函数，比如一个按钮点击或者是Ajax请求。Promise和传统事件两者都提供了在未来事件发生时注入一个事先定义好的响应函数。

而且只有在当你开始详细深究promises和事件的区别时，才会发现它们在AngularJS应用中的差别是多么的明显。

###一旦状态改变，就不会再变
Promises 表示一个活动的单例，一旦它 resoleved 或者 rejected，promise 就不能再用了。

这一点很重要，因为传递给观察者的信号“只有第一次”的选择有效。如果我们使用的传统的javascript点击事件，那么每次点击都将会触发绑定的相应事件，无论是第一次点击还是第十次点击，每次点击结果都是就用户的决定而改变。

这是一个巨大的不同点，这让 promise 适合于针对触发特殊事件而发出响应，而事件响应则会被重复触发且可能结果完全不同。或者从另一方面来说， promise 因为只针对用户的一次决定或者特别的Ajax响应而发出“一次响应”而显得尤为珍贵。

###对输出结果的触发响应
事件允许当触发时发出一个响应回调。promise 同样能够做到，但是当事件不能正常触发或者输出失败时我们同样可以定义响应信号，因为事件失败可以通过定义 deferred 对象的 reject 方法来实现。当事件失败时会触发一个在promise中定义好的 error 回调函数。

这个机制能确保活动不能正常发生或者输出失败时，有一个明确的结果来表明状态。这对比如Ajax请求来说非常重要，因为它可以用来提示用户这里出现了问题。

##Promise in ES6

###概述
Promise 对象用来进行延迟（deferred）和异步（asynchronous）计算。一个Promise出于以下四种状态之一：

+ pending:初始状态，非 fulfilled 或 rejected
+ fulfiled:成功的操作。
+ rejected:失败的操作
+ settled:Promis已被fulfilled或rejected，且不是pending

###语法
    /*
     * @param executor: 函数对象，带有两个实参 resolve 和 rejected
     * 第一个参数用来完成 fulfill 当前 promise
     * 第二个参数用来拒绝 reject
     * 一旦操作完成即可调佣这些函数
     */
    new Promise(executor);
    new Promise(function(resoleve,reject){...})

###方法
`Promise.length` 构造器参数的数目

`Promise.all` 返回一个Promise，当iterable参数里所有的promise都被解决后，该promise也会被解决
**如果任一传入的promise被拒绝了， All Promise立刻带着该promise的拒绝原因进入拒绝（rejected）状态，不会再理其他传入的promise是否被解决。**

`Promise.race(iterable)` 返回一个Promise，当iterable参数里的任意一个子Promise被接受或拒绝后，该promise**马上**也会用子Promise的成功值或失败原因被接受或拒绝。

`Promise.reject(reason)` 用失败原因reason拒绝一个Promise对象

`Promise.resolve(value)` 用成功值value解决一个Promise对象昂。如果该value为可继续的（thenable, 既带then方法），返回的Promise对象会“跟随”这个value，采用这个value的最终状态；否则的话返回值会用这个value满足（fullfil）返回的Promise对象。

`Promise.prototype.catch(onRejected)` 添加一个否定（rejection）回调到当前promise，返回一个新的promise。如果这个回调被调用，新promise将以他的返回值来resolve，否则如果当前promise进入fulfilled状态，则以当前promise的肯定结果作为新的promise的肯定结果。

`Promise.prototype.then(onFulfilled, onRejected)` 添加肯定和否定回调到当前promise，返回一个新的promise，将以回调的返回值来resolve

##参考文章
[Pro AngularJS chapter 20]()

[MDN官方文档-Promise](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)

[阮一峰-ECMAScript 6入门](http://es6.ruanyifeng.com/#docs/promise)

