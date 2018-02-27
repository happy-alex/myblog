---
title: setState
date: 2017-07-19
categories: js
tags: react
---


在更新react组件时，我们常常使用setState方法传入一个对象来更新组价的状态。但其实除了状态对象，该方法还允许传入一个状态计算函数。

setState()的参数支持对象和函数两种。
1. 对象式setState ---> this.setState({ showModal: true});
2. 函数式setState
```javascript
this.setState((state, props) => {
    // 当前组件的state和props值作为参数传入，
    // 通过返回一个新的state对象来进行state的更改。
    return newState;
});
```


为什么setState要允许传入函数？
我们知道state更新是异步的，多个setState连续调用会被合并成一个批处理更新操作。
```javascript
this.state = {count: 0};
function add() {
    this.setState({count: this.state.count+1});
    this.setState({count: this.state.count+1});
    this.setState({count: this.state.count+1});
}
console.log(this.state.count); //?
```
上面的代码，处于性能考虑，React不会真的分3次去setState，而是把传入的这3个对象合并成一个，即只会得到一个对象{count: this.state.count+1},对这个对象执行setState操作，导致最终只setState了一次，我们的得到结果是1，而不是3。


函数式setState就是为了解决这个问题滴：
当 React 碰到多次 setState(func)调用时，它会按照调用的顺序把这些函数func放进一个队列。
接下来，React 会依次调用**队列**中的方法，把上一个func中更改的状态传递给当前的方法，从而不断更新state。
我们改写上面的例子,改用函数式setState将会得到3。
```javascript
function add() {
    this.setState((state) => {count: state.count+1});
    this.setState((state) => {count: state.count+1});
    this.setState((state) => {count: state.count+1});
}
console.log(this.state.count);  //3
```

###搞事情
如果混用两种方式，会产生意想不到的效果。举个例子（伪代码）。
```javascript
this.state = {
    count: 0;
}

function add(state, props) {
    return {
        count: state.count + 1;
    };
}

function handle() {
    this.setState(add);
    this.setState(add);
    this.setState({ count: this.state.count + 1});
    this.setState(add);
}

handle();

console.log(this.state.count); // 你猜是啥
```
你可能期望输出的count是4，但实际输出的是2。
原因是React会依次合并所有setState产生的效果。虽然第2个调用时产生的效果count值为2，但由于第三次调用setState时，采用了对象式setState调用，一下子强行把积攒的state更新效果清空，this.state.count又开始从0算起。因此对象式setState调用会影响到当前队列下此前函数式setState调用积攒的效果，切忌两种方式混用。



一般来讲，按照React组件的生命周期，state的更新会导致新旧虚拟DOM的diff过程，从而导致最终的组件render。我们知道，shouldComponnetUpdate方法返回false时，会强制组件没有render。那这个时候组件的state的值到底改变没有呢，答案是没有。我们可以认为，组件的state只有在render函数重新执行时才会被改变。也就是说setState更新组件state并不是一定的，还需要得到render和shouldComponentUpdate的“承认”。



###补充：

React推荐的构造组件的写法是 class Card extends React.Component{};
这条原型链是 Card -> React.Component -> React内部自定义对象（定义了setState，forceUpdate等方法） -> Object


###疑惑
1.“多次”调用setState会合并state更新操作，那这个“多次”React究竟是怎么算的？



