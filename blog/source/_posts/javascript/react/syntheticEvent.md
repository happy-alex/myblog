---
title: React 合成事件
date: 2018-04-02
categories: js
tags: react
---

1. react 的合成事件和 dom 原生事件是两个东西，都有各自的“捕获冒泡”机制，防止混用。
2. react中绑定的事件最终都绑定在document上，类似于事件委托的性质，dom原生事件通过冒泡机制到doc上，然后依次触发doc上绑定的react回调函数。
3. react绑定的事件如果想区分是在冒泡还是捕获阶段执行，通过事件函数后缀Capture区别。比如onClick在冒泡阶段执行， onClickCapture发生在捕获阶段。
4. 事实上合成事件都是注册在真实dom事件的冒泡阶段，并不存在所谓的“捕获”，onClickCapture所谓的捕获阶段执行只是react把doc上事件绑定的监听序列从后往前依次触发执行。
5. 合成事件总是晚于dom原生事件的捕获冒泡监听函数，可以简单理解为先执行一遍dom原生的事件捕获和冒泡阶段，再走一遍react自定义事件的“捕获和冒泡”。


备注
https://juejin.im/post/59db6e7af265da431f4a02ef