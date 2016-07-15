---
layout: blog
title: React 开发技巧和最佳实践（翻译）
categories: font-end
tags: react
---
**原文**：[REACT TIPS AND BEST PRACTICES](http://aeflash.com/2015-02/react-tips-and-best-practices.html)

去年我花了大量的时间研究 React。那段时间我写了很多组件，然后重构、再重写，发掘了很多最佳实践和反模式。

我并不打算在此介绍什么是 React 以及你应该如何使用它——网络上已经有大量文章介绍 React。此文面向的读者是对 React 有了解并且已经自己尝试写过一两个组件的人。

##一、使用PureRenderMixin
`PureRenderMixin` 是一个重写了`shouldComponentUpdate`方法而且只 re-render `props` 和 `state` 真实改变 的 mixin。它对 React 的顶层高性能所作出巨大的优化。这意味着你可以任性的使用 `setState` 方式而不用担心“虚假无效 re-render”的出现。你也完全没有必要去手动为你的代码添加如下部分：
```
if (this.state.someVal !== computedVal) {
    this.setState({someVal: computedVal});
}
```
但是，`PureRenderMixin` 要求你的 `render` 方法必须是纯净的（组件输出的 `props` 和 `state` 必须是一致的）。这意味着你不能够在 `render` 方法中直接使用 class 属性，或者任何会变化的属性！
```
render: function () {
    if (this._previousFoo !== this.props.foo) { // <-- IMPURE
        return renderSomethingDifferent();
    }
}
```
`PureRenderMixin`本身非常容易使用，我已经非常顺利的将它加到我的几个组件之中并且工作正常。如果`PureRenderMixin`的引入导致组件奔溃，那么你需要重构你的组件了——通常是因为你写了一些奇奇怪怪的东东。

在使用 `PureRenderMixin` 时一个非常重要的细节就是，它仅仅是简单比较了 `nextProps` 和 `nextState` 两个属性。如果你的字段是一个 Object 或者 Array，那么修改或添加某一个属性并不会触发组件的更新！因为对象的引用源（original reference）并没有改变。破解之法就是修改你的 `props` 属性时返回一个对象新的引用，以此来改变对象。

除此之外还有很多解决这个问题的策略。可以考虑使用**不可变的数据结构**，如：[Immutable.js](https://github.com/facebook/immutable-js）)和不可变性辅助插件 `React.addons.update` 来为你减压，有兴趣的童鞋自行查阅相关文档。

这些库虽然对 `props` 中的对象做了额外的操作，但是为了能够有效的检测出无用的 re-render 还是非常值得的。请铭记——越少的 `props` 意味着越少导致组件 re-render 的因子。

如果你不使用 `PureRenderMixin`，你将错过 React 的最佳特性之一。这些简单的代码能让你的程序性能有很大的提高！

##二、使用 PropTypes
随着项目APP的成长，越来越多的组件累积成一个庞大复杂的层级结构，追查丢失或错误的 `props` 将会是我们心中隐隐的痛。

幸运的是， React 提供了 `propTypes` 方法去验证 `props`。每一个 React 类都可以定义一个 `propTypes` 映射关系来指定一个验证性函数对每一个 `props` 进行验证。如果验证函数返回失败，则会在 console 中弹出一条 warning 信息。如果你丢失一个 isRequired 的 prop 或者期望是 object 时传了一个 string ，console 中也会打印出非常有利的信息。

1、`PropTypes`是一个非常方便的接口文档，来描述你的组件所期望的 `props` 以及组件的功能；
2、为只要你设定好 `NODE_ENV="production"`，`propTypes` 在生产环境中并不会触发类型检查以至于拖慢你的APP。因为它只在开发模式下进行类型检查。通过执行 `uglify --compress` 亦可手动执行；
3、不是 isRequired 的 propType 都有该在 `getDefaultProps()`中声明对应的字段。（但这种方法并不总是可行的，因为 `getDefaultProps` 只在创建新的class时被调用一次，而不是在实例创建时被调用。因此所有实例都共享一套 `props`）。

##三、避免State
“避免状态”是所有编程都应该避免的，React 组件也不例外。即使 React 提供了 `setState` 来方便的改变状态，我们也应该避免去使用它。

即便我们使用了 `PureRenderMixin`，State 也会导致错综复杂的组件和虚假无用的 re-renders。而且最糟糕的做法就是在 `componentDidMount` 或 `componentDidUpdate` 之后调用 `setState` 方法，它会导致 `render` 方法被调用两次。如果它的一个子组件中又调用了 `setState` ，那么 `render` 方法就有可能在子组件中被调用四次！因此随着你的组件嵌套层级的加深，组件元素的进一步组合就会导致属性和结构的颠簸以及 `render` 次数的指数增长。

坦白来说，我们仍然会需要用到 `state`，但在使用的时候需要将它完美的封装在组建内部。如果一个父组件需要了解有关子组件太多的状态信息，那么你的组件结构将很不稳定——是你需要重构的时候了！

##四、集中管理 state
当我们确实需要 state 并且组件之间有需要状态的通信时，那该怎么办？在这种情况下，我们应该将所有 state 完全剥离层级并集中管理。于是乎，**Flux**、**reflux** 等跨组件通信工具闪亮登场！在**Stores** 中保存 APP 的一系列 state，**Actions** 由事件驱动并修改 **Stores**——那么你的整个APP state 保存在单一的原子中。单项数据流就这么形成了。

在实践中，组件主要还是依赖于 `props`，而不是调用 `setState`，他们通过中央数据存储来通信，也通过 事件、流、通道、回调、函数调用这些直接的方式来通信。

集中化状态管理和单项数据流的优势在于让你的 APP 更能让别人理解。每一块数据都是来自于单一源，这使得你的 APP 简化了。

如下图举例说明：
![图一](http://aeflash.com/imgs/data_flow1.svg)

这可以被看做是一个简化的 Flux 数据模型。你仅仅需要知道每当中央数据存储被更新时，就会在根组件上用新的属性调用 `render()` 函数，然后更新就通过组件层级开始层层传播。

就算你不使用 Flux，你也会自己实现一套类似的“发布订阅者模式”来便于组间通信。最佳实践总是引导你朝向看似相同的结果。

当我们调用 `setState()` 
或者有一个组件之间的通信，这个图看起来就没那么好看了：

![图二](http://aeflash.com/imgs/data_flow2.svg)

数据流发生了自循环，每一个 prop 不在有单一的源。App 从根本上变得复杂了——混乱的螺旋而不是简单地循环。组件开始无用的 re-render。

你也许会由单向数据流猜测到父组件不得不去管理它所需要的 `props` 已经和它的子组件耦合了。那我说这种“单向耦合”是可以接受的。如下代码所示：
```
render(){
    return (
        <div ref="container" className={this.props.foo}>
        {
            this.props.childProps.map(function(props){
                return (
                /* 子组件仅仅接收与它相关的属性，并不关心他自己真的需要什么 */
                    <Child {...props} />
                )
            });
        }
        </div>
    )
}
```

##五、在Render()中多做文章
当你发现你将太多的逻辑写在了 `componentWillReceiveProps` 或 `componentWillMount`中时，你可以试着将这些逻辑移至 `render()` 中。对 `props` 的预处理完全可以放在 `render` 函数中，不要担心在一些情况下会发生重复计算。 `PureRenderMixin` 已经最小化了`render` 的函数调用。如果你想对 `render` 函数瘦身，这是典型的过早优化，这确实会导致减少bug的产生，但是同样也会增加你在 `componentWillMount` 和 `componentWillReceiveProps` 中的重复代码！
```
//bad
componentWillMount(){
    this.setState({
        computedFoo: compute(this.props.foo)
    });
}
componetWillReceiveProps(){
    this.setState({
        computedFoo: compute(nextProps.foo);
    });
}
render(){
    return (
        <div className={this.state.computedFoo}></div>
    ）
｝

//better
render(){
    var computedFoo = compute(this.props.foo);
    return (
        <div className={this.state.computedFoo}></div>
    )
}
```
这个策略同时也帮你减少了 `props` 和 `state` 字段的数目，甚至有可能帮你完全避免 `state`。越少的 `state` 让组件越简单，会让 `PureRenderMixin` 的工作更加容易。

不要害怕在 `render()` 中调用返回其他组件的方法！但是要避免将所有事情都委托给 `render()` 方法。

##六、Mixins，赞！
Mixin 是创造功能性可复用代码块最便捷的方式，可被多个组件使用。之前我们已经介绍过 `PureRenderMixinx` 已经覆盖了 `shoudComponentUpdate()` 方法。

mixin 一个非常酷的特性是并不会覆盖组件生命周期中的方法，而是把mixin 中的方法加进去。这意味着 mixin 可以在 `componentWillMount()` 时做一些安装配置，然后在 `componentWillUnmount()` 时将这些配置卸载。与此同时并不会影响到原有寄主组件的 `componentWillMount` 方法。

这样，我们就可以尝试将所有 `state` 加入到 mixin 中，而让所有底层组件无状态。当我们有需要使用包含状态的组件时加上带有 `state` 的 mixin 包装。

##七、使用实例属性
这里的实例属性指的是例如： `this.foo` 而不是 `this.props.foo`。请注意，由于 `PureRenderMixin` 的严格性，我们不能在 `render()` 方法中使用实例属性。当我们组件中的状态并不会影响 `render` 函数时使用实例属性将会非常方便；但是当 `state` 会导致 re-render 时，实例属性不合适了。例如：
```
//这里我们并不希望时间状态的改变导致 re-render
componentWillReceiveProps(){
    this._timer = Data.now();
}
onLoadHandler(){
    this.trigger('load:time', Date.now - this._timer);
}
render(){
    return (
        <img src={this.props.src} onLoad={this.onLoadHandler} />
    )
}
```



