---
title: 记录一些 react的相关tips
date: 2017-04-13 21:28:07
categories: js
tags: react
---

1. 组件的props.children其实本质是一个数组，**JSX会把插入表达式数组中的JSX一个个罗列显示**(map函数渲染列表的核心原理)。

2. 推荐在有复杂状态的组件上使用key属性，确保根据key在合适的时机销毁和重建组件，提供性能（http://www.tuicool.com/articles/UVvaMz）。

3. 更新style时不要直接去修改原始style对象，这会引起react的警告，更好的办法是使用一个新的style对象。
   (http://stackoverflow.com/questions/33295615/why-was-mutating-style-deprecated)
   ![这里写图片描述](http://img.blog.csdn.net/20180213144507338?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvdGh6eDI2NWJvYm8=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

4. ref属性有两种形式，一种设置字符串，一种设置回调函数。字符串形式的ref，由于是字符串引用，所以无法知道组件何时被卸载，易造成内存泄漏。回调函数形式的ref，当组件挂载/卸载/渲染时，均会触发函数的执行。当组件卸载时，把ref指向null，从而可以防止内存泄漏

5. setState({}, cb); react会在setState完成后触发回调函数cb，从而可以拿掉更新后的state。