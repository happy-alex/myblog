---
title: babel
date: 2017-08-14 17:36:42
categories: js
tags: babel
---

在web工程里经常会引用一大堆babel包，它们主要用来是干什么的
babel支持命令行操作
引入babel-cli包，内置有两个命令babel, babel-node
babel index.js -o b.js  // 把index.js用babel转码后输出到b.js
babel-node index.js     // 可以直接运行脚本，不需要额外处理

babel-register
每次require文件时的一个hook钩子。
每当使用require加载.js，.es等文件时，会先用babel进行转码（实时转码，只适合开发环境），不需要手动转码。
只针对require文件，不会对当前文件转码。

babel-core
代码里手动调用babel的api进行转码。


babel-polyfill
babel默认只转换新的句法，不转换api，比如Generator，Set， proxy等。如果想要使用这些api，就需要引入babel-polyfill。
