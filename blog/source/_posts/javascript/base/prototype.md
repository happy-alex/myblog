---
title: 解析js中的原型
date: 2018-07-31
tags: js
---

一张图来说明：

![原型链说明图](https://camo.githubusercontent.com/71cab2efcf6fb8401a2f0ef49443dd94bffc1373/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f332f31332f313632316538613962636230383732643f773d34383826683d35393026663d706e6726733d313531373232)

总结：
1. 只有函数才拥有prototype属性，构造函数的.prototype为一个对象
2. 每个对象都有一个隐式的原型属性[[prototype]], 但由于[[prototype]]是内部属性，不能访问，用 __proto__ 代替， 浏览器实现存在兼容性
3. 每个对象的 __proto__ 指向了其构造函数的原型对象，形成原型链，因此对象可以访问到不属于其定义的属性

4. Object.prototype 和 Function.prototype是由js引擎创建的, 引擎将Function.prototype.__proto__指向Object.prototype，Object.prototype.__proto__指向null

5. Object 是所有对象的爸爸，所有对象都可以通过 __proto__ 找到它
6. Function 是所有函数的爸爸，所有函数都可以通过 __proto__ 找到它

7. 对象的 __proto__ 指向原型， __proto__ 将对象和原型连接起来组成了原型链
8. 所有的函数都可以认为是通过new Function出来的，所以所有函数都能通过原型链 __proto__ 找到Function.prototype, 即Function.__proto__ === Function.prototype。
9. Object和Function都是构造函数，所以Object也是Function的实例对象。Object.__proto__ === Function.__proto__

[深度解析原型中的各个难点](https://github.com/KieSun/Blog/issues/2)