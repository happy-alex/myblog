---
title: new 关键字
date: 2017-05-19
categories: other
tags: 
---

```javascript
function Dog() {
    this.name = 'dog';
}
var obj = new Dog();
```

## new 到底做了什么

可以理解为
```javascript
var obj = {}; 
obj.__proto__ = Dog.prototype;
Dog.call(obj);
```

1. 初始化一个空对象obj
2. 将obj的__proto__指向Dog.prototype;
3. 将构造函数的this绑定到对象obj上
4. 返回这个对象