---
title: 事件循环机制
date: 2017-05-20 21:40:50
categories: js
tags: es6
---
网上讲事件循环的文章很多，这里mark下自己的认识。

> 每一个线程都有自己的事件循环(event loop)机制，事件循环按照事件队列里的任务(task)“依次”执行。
> 浏览器中的工作线程也有自己的event loop，浏览器确保多个task可以按照一定顺序执行，并在多个task之间可能进行UI渲染。

### 概念
Tasks: 任务队列
Microtasks: 微任务队列
Stack: js执行堆栈

js有两种事件，一种是task，一种是microtask
浏览器会执行Tasks中的每个task，在多个task间择机进行rendering。
microTask在每一个task执行后紧接着执行，前提是**当前执行栈stack是空闲的**，没有其他的js任务在执行。

如何理解当前栈是空闲的？
当没有js在执行的时候，就认为其是空闲的。比如当前正在执行一个函数，如果这个函数还没有返回，就认为当前栈不是空闲的。

### 一个有意思的case
```javascript
<div id="outer">
    <div id="inner"></div>
</div>

var inner = document.getElementById('inner');
var outer = document.getElementById('outer);

function onClick() {
    console.log('click');

    setTimeout(function() {
        console.log('timeout');
    }, 0);

    Promise.resolve().then(function() {
        console.log('promise');
    });
}
inner.addEventListener('click', onClick);
outer.addEventListener('click', onClick);

// 1. inner.click()
// 2. 手动点击inner元素
```
1.inner.click()执行完当前js后，事件向父元素冒泡，我们认为其冒泡过程是作为click()执行的，整个过程中执行栈均没有空闲，无法执行microtask。
2.手动点击inner，触发了inner的监听事件，执行完inner的监听事件后，冒泡到outer，执行outer的监听事件。由于在冒泡到outer前，inner的Listener已经执行完成并且返回了，所以虽然此轮任务尚未结束，但此时执行栈是空闲的，可以执行microtask。


### 备注
有些文章把上面的task也称之为macrotask，以便和microtask区分。
> macro-task包括：script(整体代码,主进程), setTimeout, setInterval, setImmediate, I/O, UI rendering。
> micro-task包括：process.nextTick, Promise回调, Object.observe(已废弃), MutationObserver(DOM变化监听器)


```javascript
setTimeout(function(){ console.log(0); }, 0);
new Promise(function(resolve, reject) {
    resolve(1);         // micro-task
    console.log(2);     // 一旦新建，立即执行, macro-task
}).then((res) => {
    console.log(res);   // micro-task
}, () => {});
console.log(3);         // 输出结果是2, 3, 1, 0
```
Promise新建属于当前stack， 创建后会立即执行，首先输出2。Promise的回调属于micro-task任务，在本轮事件循环的末尾执行，输出1。setTimeout回调的执行则发生在下一轮event中，最后输出0。

> 在 JavaScript 函数中，只有 return / yield / throw 会中断函数的执行，其他的都无法阻止函数运行到结束的
> 不同的浏览器实现存在差异，有些浏览器下setTimeout的回调可能会先于Promise回调执行


### 参考：
https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/

