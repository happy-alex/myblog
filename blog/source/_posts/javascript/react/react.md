---
title: react
date: 2017-04-13 21:28:07
tags:
---
TODO...
想写一写react系列文章,计划从以下几个步骤来

1.react小书 
2.fiber架构（diff算法）
3.如何实现一个迷你的react框架

小tip：
1. 组件的props.children其实本质是一个数组，**JSX会把插入表达式数组中的JSX一个个罗列显示**(map渲染列表的原理)。
2.this和bind的关系 a是怎么由card传到父组件的
    function handle(a,b) {this.props.print(a)} <card onClick={this.handle.bind(this)} />
推荐阅读：
1. [react核心概念，入门教程](http://huziketang.com/books/react/lesson1)
2. [如何实现一个virtual-dom算法](https://github.com/livoras/blog/issues/13)
3. [redux源码分析](https://github.com/kenberkeley/redux-simple-tutorial/blob/master/redux-advanced-tutorial.md)
3. [redux中文官网](http://www.redux.org.cn/)