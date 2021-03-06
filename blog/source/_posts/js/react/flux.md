---
title: flux/redux/mobx
date: 2017-05-11
categories: js
tags: [react, redux, mobx]
---

### flux vs redux
```bash
reducer(state, action) -> newState
```
1.flux是一种架构设计思想，用来在大型项目中实现数据的单向流动，而redux是flux这种设计思想下的一种具体实现。（也有人不同意这种意见）
2.flux里的概念：store，action，dispatcher；redux的概念：store，action，reducer。redux中不额外提供专门的派发器dispatcher，而是通过store.dispatch实现
3.flux里store有多个；redux全局只维护一个store，其包含整个应用的state。
4.redux里的store只有一个，包含了整个应用的state。
  action用来描述改变的行为。action是一个简单对象，包含需要执行的操作类型（type指定）和一些其他负载（payload）。
  reducer(state，action)=> newState。改变由reducers 来执行，reducers 是没有副作用的纯函数，将先前的state和一个action作为参数。它们会返回由应用action产生的新的state。
  reducer有多个，每个reducer维护state树的一支，redux用combineReducers()方法将多个reducer合并起来。

补充：
在hybrid架构中，每次打开一个页面，都是新建一个webview。因此spa + redux模式在这种场景下并不适用，因为redux只有一个全局store，维护了多个子页面的store, 而在一个webview中，非本页面的store都没被利用，无疑是一种浪费

### mobx vs redux
核心理念：数据驱动
mobx看起来更像是把vue的数据绑定机制单拎出来，再做增强，同时使用es7的decorator语法让API更简单

mobx的工作方式：
```bash
        change          Derivations
action --------> state -------------> computed 
                      |    
                       -------------> reaction
```
1.redux中的action要求是一个简单对象，用以描述发生了什么改变；而mobx的action是一个函数，用来修改状态
2.redux里reducer(state, action) -> newState； mobx里彻底抛弃了reducer，原本reducer要做的工作全在action里完成
3.mobx能精确知道数据的变化，所以可以减少diff的成本，一定程度上性能会比redux更快
4.redux要求state应该是一个纯对象，且全局单一；而mobx里的state更自由，不要求state的数据结构和类型


### mobx vs vuex

vuex的典型工作流
```bash
        commit             mutate          render
action --------> mutation --------> state --------> view
```
vuex从设计上区分了异步任务和同步任务，分别放在对应的action和mutation里实现；而mobx并不区分这个，同步和异步都在action里完成
vuex里的getters就是mobx里的computed