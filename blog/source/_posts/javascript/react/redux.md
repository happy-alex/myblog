---
title: react-redux
date: 2017-05-28
categories: js
tags: [react, redux]
---

[redux源码分析](https://github.com/kenberkeley/redux-simple-tutorial/blob/master/redux-advanced-tutorial.md)
注意区分redux和react-redux
redux是一个状态管理库，理论上可以和任何库合作。react-redux是针对react做的适配。

### react-redux
只有两个东西Provider 和 connect
```javascript
<Provider store={store} />  //这个地方把store作为一个普通props传入，Provider内部把传入的store放在组件树的全局上下文context上，这样理论上其组件树上的每一个子组件都能通过context.store拿到全局state。
```
connect本质上一个高阶组件，它返回一个包装过的UI组件。比如允许UI组件直接通过props获得actionCreators。
react中如果子组件想拿到全局上下文的数据，即this.context有值，必须在组件外额外声明。connect帮助开发人员处理了这部分。

redux
三驾马车 store，action，reducer

利用redux的createStore生成一个store，这个store具有全局唯一性。
createStore(),接受3个参数，initialState，reducer，中间件。
```javascript
const reducers = {
    component1: function(state, action) {
        return newState;
    },
    component2: function(){},
    ...
};
const store = createStore(
     initialState, // 初始state，主要用于服务端渲染
     combineReducers(reducers),
     middleware
);
```

initialState 设置初始状态，主要用于服务端渲染时同步状态。
combineReducers()返回的还是一个reducer函数,通过他把多个reducer组织起来，形成最终的reducer
middleware提供了一个出口，允许在dispatch前后统一做一些操作。
一般来讲dispatch(action)是同步的，这个action只能是一个普通对象，如果想异步dispatch，则需要借助redux-thunk提供的thunkMiddleware中间件。他的工作是允许dispatch一个函数。

得到的store对象结构如下：
```javascript
store = {
     dispatch: function(action){},
     getState: function(){},
     replaceReducer: function(){},
     subscribe:function(){}
}
```

store.getState()得到整棵树的顶层state。
为了模块化的管理state，有多个reducer，因此有多个state，如何保证state不重名？通过combineReducers({key: value})中的key来隔离
多个reducer中的各自分散state被汇总到全局store中，初始化的时候redux会首先dispath一个type为@@redux/INIT的action，通过触发这个action，流通到所有的reducer中，从而得到所有的state。


### 实际工作中使用action
```javascript
class A extends React.Component {
    loadData() {
        this.props.fetchData().catch(err => {});
    }
}
```
标准写法是store.dispatch(action)，为什么实际业务开发中能够通过this.props.fetchData()这种方式来dispatch？

答案就是bindActionCreators。this.props.fetchData是通过@connect被bindActionCreators包装过的。逻辑上把bindActionCreators(action, dispatch)处理成{action: dispatch(action)}，并作为props传入UI组件，这样UI组件就可以直接this.props.action()来触发dispatch了