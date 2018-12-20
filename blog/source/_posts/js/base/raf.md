---
title: requestAnimationFrame/requestIdleCallback
date: 2017-08-01 15:44
categories: js
tags: [browser]
---

## 帧

我们看到的网页，是由浏览器一帧一帧绘制出来的，一般浏览器的刷新频率1分钟可达60帧，即我们常说的60fps。如果一个渲染帧的执行时间超过了16.67ms（1000/60），那么人眼就会感觉到卡顿，因此我们常说为了保持页面流畅度，要尽可能使浏览器在16.67ms内完成渲染。

### 浏览器在一帧内做了哪些工作

从网上找了一张图：

![帧]](https://image-static.segmentfault.com/821/964/821964204-5ad6fd8d6ac94_articlex)

一个渲染帧中先后包含 与用户的交互，js执行，调用raf函数，样式计算，布局计算（计算样式，执行渲染），页面绘制，合并图层 等多个时序。


### js耗时造成丢帧
从上边的流程图来看，如果某个时刻有大量的js执行，导致执行js脚本耗时过长，将会丢掉该帧的绘制阶段，最终在下一个空闲帧内一次性全部绘制出来。这就是我们常说的“丢帧”。

### 优化手段
1. 减少js耗时

上面提到执行耗时超过16.67ms的脚本可能会导致丢帧，从而使页面变卡。因此首先想到的手段就是，如果有耗时长的任务，那么将一个大任务拆分成多个子任务，分散到不同的帧中去执行。

2. 样式的读写分离
如果对一个dom的样式进行频繁读写，为了得到一个正确的css值，那么浏览器可能会多次触发layout阶段，造成阻塞。


## requestAnimationFrame
在下一帧执行raf指定的回调。可以将一个耗时过长的js任务作为其回调传入，以便其在下一帧执行。


## requestIdleCallback
requestIdleCallback在一个帧的空闲阶段执行。如果页面绘制在16.67ms内完成，那么剩余的空闲时间就是执行 requestIdleCallback 的时机。
既然ric在帧的空闲阶段执行，就意味着ric有可能得不到执行。毕竟如果浏览器一直处于繁忙阶段，那么将没有空闲时间供ric执行回调，为了解决这个问题，引入了超时机制。
使用如下：
```javascript
function work(deadline) {
    // 如果有空闲时间或达到超时阈值，执行回调
    if ((deadline.timeRemaining() > 0 || deadline.didTimeout) && hasMoreWork) {
        dowork();
    }

    if (hasMoreWork) {
        requestIdleCallback(work, {timeout: 1000});
    }
}
requestIdleCallback(work, {timeout: 1000});
```
由于ric的执行时间是在渲染帧的空闲时间，因此requestIdleCallback并不适合执行那些容易导致整个帧耗时过长的任务，避免增加当前帧的耗时。比如：dom修改，promise回调等等。

## fiber
react16之后引入了fiber的概念，fiber可以根据优先级自行调度任务，就是基于requestIdleCallback实现的。但ric存在很大兼容性问题，react还因此自行实现了ric的功能。

## 参考

https://segmentfault.com/a/1190000014457824
