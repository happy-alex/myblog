---
title: tree-shaking
date: 2018-08-16
categories: builder
tags: webpack
---

最早由rollup提出，webpack2+ 引入支持 tree-shaking功能，在最终打包产物中去除无效或没有使用的代码

### tree-shaking过程分为两步：
1. webpack 对 export, import导出导入内容进行标记
    将所有的 import 标记为 harmony import
    将用到的 export 标记为 /* harmony export <typeName> */typeName为webpack内部的一个值，比如是immutable
    将未使用的 export 标记未 /* unuse harmony export <functionName> */ functioName为未使用但export出的函数名称
2. uglifyJS 等工具将未使用的代码清除


### tips:
1. 工作原理基于es6的静态分析，因此只能使用esmodule的项目中

2. tree-shaking 只针对于模块的顶层变量，如果是函数或类内部存在冗余代码，无能为力。因为 webpack 不知道函数或类的内部代码究竟做了什么，直接清除可能会导致某些副作用

3. 某些情况tree-shaking可能不会像你预期的那样工作，考虑下面这个case:
```javascript
// a.js
export default function log {
}

// tree-shaking不知道这两行代码具体是做啥的，引入这个模块存在副作用，即便log方法在b.js中未使用，这里还是会保留这两行代码
Array.prototype.say = function(){}
console.log('this is a file');

// b.js
// log方法未使用，但打包产物中并不会直接去掉a.js的所有代码
import log from './a';
console.log('this is b file');
```
4. webpack4对bundle做了一个优化，默认所有代码都处于一个闭包当中（更早版本每个模块文件都是一个闭包，然后互相调用），编译器可以直接识别代码是否无效，加以删除，因此可以解决上述提到的副作用


<!-- from: https://mp.weixin.qq.com/s/Ue0kNOMQS7mH-2-9BhYk8Q -->
