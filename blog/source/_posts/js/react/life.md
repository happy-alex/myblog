---
title: react的生命周期
date: 2017-04-11 20:26:58
categories: js
tags: react
---

## react v15
mark一下v15 react的几个生命周期函数：
1.componentWillMount
2.componentDidMount
3.componentWillReceiveProps
4.shouldComponentUpdate
5.componentWillUpdate
6.render
7.componentDidUpdate
8.componentWillUnmount


### componentWillMount

有人会在这个方法里调用ajax来请求后端数据。这种做法是不妥的，原因如下：
1.会阻碍组件的实例化，影响组件渲染。渲染应该是纯粹的，不该触发I/O操作。
2.如果将ajax放在willMount中，可能会导致在一个尚未挂载的组件上进行setState，这可能会导致某些不确定隐患。在componentDidMount中执行ajax，将是十分安全的。
可以执行setState,但不建议这样做, 如果想在render之前指定state，应该使用default state



### render
将jsx渲染到实际dom中


### componentDidMount
在这个方法里，组件已经成功挂载，可以处理许多跟dom有关的操作，可以处理需要组件未挂载之前不能做的事情，比如事件监听。
可以调用setState，这里setState调用的效果会在下一次render进行。不理解？就是说如果这里进行了setState操作，会导致render执行两次。


### componentWillReceiveProps
**首次渲染的时候，不会调用这个方法的，只有当后续props改变时才会执行。
一个典型使用场景是，通过对传入的props做处理，得到组件的state。比如一个tab选项卡，当前tab项应该是组件内部状态，但有时候业务要求可以任意指定当前选中tab，就需要根据传入的props做处理。
有些人会在这里diff新老props的不同，从而做些其他操作，比如动画。
严格说来这些都是副作用，应该尽力避免。这些副作用容易破坏state的，引入其他影响，可能导致组件状态变得不可预测。
可以setState


### shouldComponentUpdate
禁止setState
很多人喜欢在这个地方来控制组件是否进行更新，来优化性能。
[这里有另一种不同意见](http://www.infoq.com/cn/news/2016/07/react-shouldComponentUpdate)，过度使用这个特性可能会导致的问题。主要是scu会拖慢组件的更新速度，如果要比较的对象很大，比如层层嵌套的大家伙，很明显将会很耗时,假设比较花费了1ms，而直接渲染花费时间还不到1ms，那么个时候这种比较反而是累赘的，有副作用。请谨慎使用。
在比较两个层级比较复杂的对象时，这种比较就很难进行，因此引入了不可变数据--immutable。

### componentWillUpdate
在组件更新之前执行，跟willReceiveProps的功能部分重复，除了这里不允许调用setState。willReceiveProps接受到了新的props，但组件并不一定会更新（scu）,但willUpdate一定发生在render之前。因此如果组件里面存在scu方法，同时还想组件在更新之前做点什么的话，代码应该放在这里执行。


### componentDidUpdate
可以调用setState，但要避免死循环。
这里可以进行一些dom相关操作，比如获取更新后的dom宽高。


### componentWillUnmount
禁止setState
清理组件的残留，比如abort请求，解绑事件等


### 各生命周期是否可以调用setState方法
生命周期 | 是否允许调用setState | 备注
----|------|----- 
WillMount | yes | 不建议调用
Didmount | yes
WillReceiveProps | yes | 不建议调用
scu | no
WillUpdate | no
DidUpdate | yes
WillUnmount | no


## react v16
react 16采用fiber架构实现了异步渲染，对原来的声明周期做了改变
逐步废弃了下面3个生命周期函数
componentWillMount
componentWillReceiveProps
componentWillUpdate

这3个函数都容易引发副作用，在15版本可能中这些副作用只是会影响性能；但在16的异步渲染模式中可能直接引起bug。

新增两个生命周期函数，以替代上面废弃的函数
static getDerivedStateFromProps
getSnapShortBeforeUpdate

渲染错误相关处理函数
static getDerivedStateFromError 专做UI降级处理
componentDidCatch 专做错误上报