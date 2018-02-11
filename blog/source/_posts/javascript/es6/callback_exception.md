---
title: 异步回调的异常处理
date: 2017-07-20 21:40:50
tags: es6
---

上一篇文章提到了js事件循环机制中的两种队列类型：micro-task，macro-task。
这里简单mark一下工作中遇到的相关异常处理问题。

### Promise

1.Promise只能捕获来自上一级的异常, then()和catch()返回的仍然是一个Promise。
2.在Promise的回调中，throw和reject都可以抛出错误。但是throw和reject有本质的不同:
>throw是条同步语句的，reject是异步的。
如果在一个**异步上下文**中throw出错误，在当前上下文中是没有办法捕获到这个错误的。(throw唯一的用途可能就是在debug时及时暴露出各种错误，不至于被Promise吞掉。)
在**异步回调函数**中，除我们自己throw语句外，任何其他原因造成的错误都会导致我们无法捕捉的异常。（原因见下面扩展部分）
因此异步回调中，要避免意料之外的错误抛出，明确reject出任何可能的异常。【见下面示例1】

结论就是:除非明确知道使用throw在干什么，否则都请使用reject回调来抛出错误异常。

>值得注意的是，和传统的try/catch代码块不同，如果没有使用catch方法来指定错误处理，Promise对象抛出的错误不会被传递到外层代码。正常情况下，不会有任何反应，但浏览器中可能会打印出错误，不过这并不会终止脚本的执行。



### 扩展：回调中的异常处理
> 实质：在异步回调中，由于回调函数的执行栈脱离了当前运行的上下文环境，导致throw同步抛出的异常无法被当前作用域捕获。
```javascript
示例1：
function fetch(callback) {
    setTimeout(() => {
        throw Error('请求失败') //异步回调中同步抛出的错误不会被外层代码所捕获
    })
    callback();
}
try {
    fetch(() => {
        console.log('请求处理') // 永远不会执行
    })
} catch (error) {
    console.log('捕获异常', error) // 错误永远捕获不到
}
```

macro-task脱离了上下文环境，异常无法被当前作用域捕获，因此Promise的处理应该在macro级别的回调中使用reject的方式，而非throw Error，避免抓不到异常。
micro-task没有离开当前作用域，因此异常可以被捕获。

### 参考
[详见promise、generator、async/awiat的异常处理](http://mp.weixin.qq.com/s/UYT42aiZ4oVbVmcoR2LrhQ)