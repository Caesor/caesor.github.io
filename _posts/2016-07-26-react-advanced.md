---
layout: blog
title: React进阶——高复用性组件设计
categories: font-end
tags: react
---
对于`react`开发，我相信几乎所有前端开发者都能跟你讲出一套一套的什么组件化啊、生命周期啊、虚拟DOM啊等等，更有很多`react`狂热粉和深度用户已经开始实践`react`服务端直出、`react-native`等技术。

那么本文谈什么呢？关于react的文章还能讲出什么新的花样呢？

今天我们来谈谈最基础的`react`组件设计。还没等我张口，或许有些人就会发问：“组件设计还用你来讲吗？不就是拆分模块，提取公共模块吗”？没错，确实是这么回事，但是其中也大有文章！

>本文的内容主要来自于FackBook ShihChi Huang的演讲“React tips while building large scale application”

##Container Component
###What
如下图所示，Container Component就是包裹在普通组件外层的“包裹组件”，主要用来处理数据、loading/error占位图等。而包裹在其中的普通组件只单一负责`render`。
![](http://km.oa.com/files/photos/pictures/201607/1469516257_38_w328_h287.png)
###How
```
class FooContainer extends Component {
    constructor(props) {
        super(props);
        this.state = { data: [] };
    }
    render() {
        return (
            <Foo {...this.state} />
        );
    }
}
```
###Why
通常，我们把包裹组件和普通组件区分为：Container Component和UI Component。
**Container Component**专注于：
1、管理data 和 state；
2、用于逻辑测试。

**UI Component**专注于：
1、UI展示；
2、可复用的模块。

###Example
通常我们可能会这样设计一个ToDoAPP
```
class TodoApp extends Component {
    constructor(props) {
        super(props);
        this.state = { todos: [] };
    }
    componentDidMount() {
        fetch('/todos.json').then(
            todos => {
                this.setState({
                    error: null,
                    isLoaded: true,
                    todos
                });
            },
            error => {
                this.setState({ error });
            }
        );
    }
    render() {
        const todos = this.state.todos
            .map(todo => <Todo {...todo} />);
        return <ul>{todos}</ul>;
    }
}
```
那如果还要加入一些 timeout逻辑、失败后重试逻辑、数据分组处理、数据过滤等等时，componentDidMount 中的逻辑会变得臃肿复杂，阅读困难。
让我们再来使用Container Component的思维重构一次：
```
//数据处理、异常处理、占位图等
class TodoContainer extends Component {
    componentDidMount() {
        this.setState({ isLoading: true });
        fetch('/todos.json').then(
            todos => {
                this.setState({
                    error: null,
                    todos,
                    isLoading: false
                });
            },
            error => this.setState({ error })
        );
    }
    render() {
        return <Todo {...this.state} />;
    }
}
//只关注UI展示逻辑
class Todo extends Component {
    render() {
        const { error, isLoading, todos } = this.props;
        if (error) { return <ErrorPage />; }
        if (isLoading) { return <Spinner />; }
        if (!todos.length) {
            return <EmptyResult />;
        }
        const lists = todos.map(todo => {
            return <TodoItem {...todo} />;
        });
        return <ul>{lists}</ul>
    }
}
```
有木有清晰很多？

##无状态组件（Stateless Component）
或许你曾今这样写过一个模块：
```
class Todo extends Component {
    _renderItem(todo) {
        return <li>{todo.title}</li>
    }
    render() {
        const items = this.props.todos
            .map(this._renderItem);
        return <ul>{items}</ul>;
    }
}
```
把_renderXXX 作为一个子组件其实是一种不好的设计方式，也就是官方称为的“反模式”。这种设计风格会导致状态被保存在 _renderItem 中，每次调用`render`会创建一个新的 _renderItem 实例。
这里建议的设计模式如下所示：
```
function TodoItem(props) {
    return <li>{props.title}</li>
}
class Todo extends Component {
    render() {
        return <ul>{this.props.todos.map(TodoItem)}</ul>;
    }
}
```
即“无状态组件”，这种组件没有状态，没有生命周期，只是简单地接受props渲染成DOM结构。由于React在渲染时省掉了“组件类”实例化的过程，因此**开销非常低**。

##装饰者模式（Decorator）
在长列表中，如果要给每一个项绑定左滑删除的按钮，我们应该怎样设计呢？考虑到“删除组件”的可复用性，我们可以尝试使用“装饰者”。
![](http://km.oa.com/files/photos/pictures/201607/1469516412_40_w551_h259.png)
```
function MakeDeletable(Child) {
    class Deletable extends Component {
        render() {
            const {onDelete, ...otherProps} = this.props;
            return (
                <div>
                    <Child {...otherProps} />
                    <button className="delete-todo" onClick={onDelete} />
                </div>
            );
        }
    }
    return Deletable;
}

// ES7语法中还可以这样使用
@MakeDeletable
function BaseTodoItem(props) {
    return <label>{props.content}</label>;
};
// 或常规方式
const DeletableItem = MakeDeletable(BaseTodoItem);

class TodoItem extends Component {
    render() {
        const { isCompleted, content } = this.props;
        const className = classNames({
            'is-completed': isCompleted
        });
        return (
            <li className={className}>
                <DeletableItem
                    checked={isCompleted}
                    content={content}
                    onChange={this._onChange}
                    onDelete={this._onDelete}
                />
            </li>
        );
    }
}
```
##装饰者—之高阶函数实战
在兴趣部落独立APP私密部落升级公开部落的信息表单中，需要在多个页面复用“上传图片”的功能组件，但由于采用了ES6的写法，无法使用mixin，让人苦恼不已。使用高阶函数，可以帮助我们解决这个问题。
![](http://km.oa.com/files/photos/pictures/201607/1469516493_11_w433_h312.png)
```
// 高阶函数的定义
export default let ImageUpload = ComposedComponent => class extends Component {
    render(){
        let { content, loading } = this.state;
        return (
            <div className="imageUpload">
                <Loading show={loading}/>
                <ComposedComponent 
                    {...this.props} 
                    {...this.state} 
                    onChangeImg={ e => this.onChangeImg(e) } 
                />
            </div>
        );
    }

    componentDidMount() {
        //右上角“完成”按钮绑定事件
        mqq.ui.setTitleButtons({
            right: {
                title: '完成',
                callback: function(){
                    self.exchangeUrl(); // 点击完成时再换取URL
                }
            }
        })
    }
    // 更换按钮
    onChangeImg() {
        mqq.media.getPicture(self.props.opt, function(ret, images) {
            // 先展示出 base64
            self.setState({
                content: images[0].data
            });
        });
    }
    // 图片base64 换 URL
    exchangeUrl() {
        upload(self.state.content, {
            ...
            self.props.reStore(url);
        });
    }
}

// 高阶函数的使用
class Avatar extends Component {
    render(){
        let { content } = this.props,
            divStyle = {
                backgroundImage: 'url("' + content + '")'
            };
        return (
            <div className="avatar-container">
                <div className="avatar" style={divStyle}></div>
                <i onTouchTap={this.props.onChangeImg.bind(this)}>更换头像</i>
                <p>添加一张有代表性的图片作为部落头像</p>
            </div>
        );
    }
}
// ES6方式导出或使用ES7@装饰者
export default ImageUpload(Avatar);
```
高阶函数加无状态组件，大大增强了整个代码的可测试性和可维护性。我们也因此可写出组合性更好的代码。
更多细节，可拓展阅读：[React and ES6 - Part 4, React Mixins when using ES6 and React](http://egorsmirnov.me/2015/09/30/react-and-es6-part4.html)
##map / filter / reduce
这一块内容其实在我的另一篇文章中已经有所阐述：[从 Array 理解函数式编程](http://km.oa.com/group/19674/articles/show/259044)。如果你能够在数据处理时尝试考虑使用这些函数会让你的逻辑更加清晰。这里不做赘述，一个例子即可帮你get到point。
```
var todos = [];
return this.props.todos
    .filter(todo => !todo.isCompleted)
    .filter(todo => todo.title.endsWith(‘today’))
    .map(todo => {
        const {id, title, isCompleted} = todo;
        return {id, title, isCompleted};
    })
    .reduce((todos, todo) => {
        const {id, title, isCompleted} = todo;
        todos[id] = {title, isCompleted};
        return todos;
    });
```

##结语
读到这里，或许你会回想到：哦！当初我的组件原来可以这么设计！或许也只是我个人yy。所谓“我们不创造代码，我们只是代码的搬用工”。真心希望我的总结能帮助到大家。