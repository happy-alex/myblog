---
title: react-redux
date: 2017-05-28
categories: js
tags: [react, redux]
---

## 背景
本质上，react只是一个dom的抽象层，允许你通过jsx的语法，组件的思想构建虚拟dom。
react提供的组件通信方式只有一个方式---父子组件传参，当应用复杂起来，简单的传参已经不满足需要。这个时候引入了redux。


## redux
[redux源码分析](https://github.com/kenberkeley/redux-simple-tutorial/blob/master/redux-advanced-tutorial.md)

tip: 注意区分redux和react-redux
redux是一个状态管理库，理论上可以和任何库合作。
react-redux是针对react做的适配, 下文的redux指代react-redux，不再具体区分。

redux其实只有两个东西：Provider 和 connect
```javascript
// 这个地方把store作为一个普通props传入，Provider内部把传入的store放在组件树的全局上下文context上，这样理论上其组件树上的每一个子组件都能通过context.store拿到全局state。
<Provider store={store} />  
```
connect本质上是一个高阶组件，它返回一个包装/增强过的UI组件。
react中如果子组件想拿到全局上下文的数据，即this.context有值，必须在组件内额外声明。connect帮助开发人员处理了这部分。
connect接收上下文context上的store，把它转换成UI组件的props传入，从而使得UI组件内可以直接通过this.props获取全局store，增强了原始UI组件。

redux
三驾马车 store，action，reducer

### store
利用redux的createStore生成一个store，这个store具有全局唯一性。
createStore()接受3个参数，initialState，reducer，中间件。
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
middleware提供了一个出口，暴露给开发者，允许在 dispatch(action) 前后统一做一些操作。

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

为了模块化的管理state，有多个reducer，因此有多个state，如何保证state不重名？
通过combineReducers({key: value})中的key来隔离
多个reducer中的各自分散state被汇总到全局store中，初始化的时候redux会首先dispath一个type为@@redux/INIT的action，通过触发这个action，流通到所有的reducer中，从而得到所有的state。


### reducer
reducer就是一个纯函数，接收一个action，返回新state。reducer的工作可表示为 reducer(state, action) -> newState

###  action
一般来讲dispatch(action)是同步的，这个action只能是一个普通对象，如果想异步dispatch，则需要借助redux中间件来完成。社区著名的有redux-thunk，redux-saga等。

```javascript
class A extends React.Component {
    loadData() {
        this.props.fetchData().catch(err => {});
    }
}
```
标准写法是store.dispatch(action)，为什么实际业务开发中能够通过this.props.fetchData()这种方式来dispatch？

答案就是bindActionCreators。this.props.fetchData是通过@connect被bindActionCreators包装过的。逻辑上把bindActionCreators(action, dispatch)处理成{action: dispatch(action)}，并作为props传入UI组件，这样UI组件就可以直接this.props.action()来触发dispatch了


### 中间件
view -> state 中使用 dispatch(action) 给reducer，这个过程可以被拦截。因此react-redux中允许你定制自己的中间件来完成这个拦截处理
react-redux通过 applyMiddleware方法来串联多个中间件
```javascript
export default function applyMiddleware(...middlewares) {
  return (createStore) => (reducer, preloadedState, enhancer) => {
    const store = createStore(reducer, preloadedState, enhancer)
    let dispatch = store.dispatch
    let chain = []

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
 
### redux-thunk
核心思想：扩展action的类型，使其支持function，从而实现异步action
他的工作就是允许redux能够dispatch一个函数。
上面提到action可以被拦截，这里的redux-thunk和下边的redux-saga都是著名的中间件，他们的目的都是为了使redux支持异步action
```javascript
// redux-thunk源码实现
function createThunkMiddleware(extraArgument) {

    // {dispatch, getState}和next均由redux在执行applyMiddlaware方法时注入，next实质上仍然指向store.dispatch
    return ({ dispatch, getState }) => next => action => {
        if (typeof action === 'function') {
            return action(dispatch, getState, extraArgument);
        }
        return next(action);
    };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
redux-thunk的缺点很明显，就是他的业务逻辑过于散乱，这些副作用可以放在UI组件，也可以放在thunk内部。

### redux-sage
实现更精细的异步action管理
核心思想：利用generator特性来yield 各种副作用，用同步的方式描述异步任务。
redux-saga中，UI组件不再执行副作用或其他任务，UI组件总是dispatch一个action来说明哪里发生了改变，通知saga来执行这些任务。
上面提到thunk管理副作用时比较混乱，redux-saga就很好的解决了这个问题，saga将所有的副作用，异步任务都集中在一起管理。
saga标榜的另一个优点就是测试方便，因为所有的副作用都被集中到了一起管理，且使用yield执行。因此可以遍历generator函数，不断的执行next()得到结果，测试这个结果

这里需要提一下的是，yield 后面的表达式并不意味着saga立即执行异步调用，而是返回一个简单的action文本描述信息，然后saga middleware负责解释执行具体异步操作，换句话说，**yield的仅仅是一个描述函数调用信息的简单对象**。
```javascript
put({type: 'INCREMENT'})    // => { PUT: {type: 'INCREMENT'} }
call(delay, 1000)           // => { CALL: {fn: delay, args: [1000]}}
```

### dva
核心思想：约定大于配置
淘宝的一个脚手架，直接集成了react + react-router + react-redux + redux-saga
通过将redux中的store，reducer和saga写在一个model文件中, 极大提升了开发体验
```javascript
// 一个model的简单配置
app.model({
    namespace: 'count',
    state: {
        id: 0
    },
    reducers: {
        update(state) {
            return {
               id: state+1
            };
        }
    },
    effects: {
        *add(action, { call, put }) {

            // dosomething... yield异步操作

            // 执行完成后再dispatch一个action
            yield put({ type: 'update' });
        }
    },
    subscriptions: {}
});

```

### 后记
react-redux是一个单向数据流方案，但社区还有一些其他知名的数据流方案
函数响应式数据流方案：mobx，rxjs等，这以后我们再讨论
