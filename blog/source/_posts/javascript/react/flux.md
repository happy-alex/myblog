---
title: flux vs redux
date: 2017-05-11
categories: js
tags: react
---

### 简单对比下flux和redux
1.flux是一种架构设计思想，用来在大型项目中实现数据的单向流动，而redux是flux这种设计思想下的一种具体实现。（也有人不同意这种意见）
2.flux里的概念：store，action，dispatcher；redux的概念：store，action，reducer。redux中不额外提供专门的派发器dispatcher，而是通过store.dispatch实现
3.flux里store有多个；redux全局只维护一个store，其包含整个应用的state。
4.redux里的store只有一个，包含了整个应用的state。
  action用来描述改变的行为。action是一个简单对象，包含需要执行的操作类型（type指定）和一些其他负载（payload）。
  reducer(state，action)=> newState。改变由reducers 来执行，reducers 是没有副作用的纯函数，将先前的state和一个action作为参数。它们会返回由应用action产生的新的state。
  reducer有多个，每个reducer维护state树的一支，redux用combineReducers()方法将多个reducer合并起来。
   
