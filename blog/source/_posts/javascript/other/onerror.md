---
title: 关于window.onerror的一二事
date: 2018-03-02
categories: other
tags: js
---

为了保障线上服务的可靠运行，我们经常需要在前端侧做一些监控，当出现问题的时候可以及时发现，尽早修复，避免造成更大的损失。从监控目的来讲，我认为可以分为两块，一块是业务监控(监控业务指标，比如用户访问量，点击率等)，一块是异常监控(如脚本执行异常等等)。

今天只谈下脚本异常监控。既然代码是人写的，就难免会出现bug。问题是如何减少甚至避免这种情况，当出现问题时能够及时发现，降低损失。前端一直没有一个统一的完善的被各方广泛接受的异常监控处理方案。各家公司基本上都有自己的处理方案，落实到前端来，异常监控基本上分为两个方法。
1.try/catch
2.window.onerror

### try/catch
try/catch基本上是最简单最容易操作的一个捕获异常的方式，但他的缺点很明显一是只能捕获其代码块里的异常，不适合全局引用。二是不能捕获回调中的异常。

### window.onerror
window.onerror可以拿到详细的错误信息，包括错误堆栈，行列号等等。但使用上，不能和一般业务代码放在同一个代码块里，否则将可能拿不到脚本中的异常信息。**window.onerror必须置于所有的script标签之前**。

为什么window.onerror需要提前声明？假设出现了语法错误，语法都不过，同一个代码块中的onerror当然没有办法执行。

### crossorigin
script标签本身具有跨域特性，因此可以在a.com下引用b.com的脚本并执行。
当script引用了其他域名下的脚本时，脚本执行报错，onerror捕获不到详细异常信息的。
这个时候就需要用到crossorigin属性。

crossorigin用来声明当前脚本中发生的异常可以被脚本所在当前域名捕获。
```javascript
// a.com/index.htm
<script>
    window.onerror = function(e) {
        console.log(e);
    }
</script>
<script src="b.com/index.js" crossorigin></script>
```
使用了crossorigin的脚本，其服务端响应必须设置响应头Access-Control-Allow-Origin: true，如果没有这个设置，会引发页面跨域错误。

### 备注
如果脚本文件404这种错误，window.onerror是无法捕获的，只能在标签元素上监听onerror方法，如下：
```javascript
<script>
    function onerror() {}
</script>
<script src="" onerror="onerror()"></script>
```