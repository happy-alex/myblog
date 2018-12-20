---
title: this绑定
date: 2017-06-01
categories: js
tags: js
---

this的指向问题，就记住4句话：默认绑定，显式绑定，隐式绑定，new绑定

## 默认绑定
默认指向全局对象上，例如window；如果是严格模式，则指向undefined

## 显式绑定
使用apply,call,bind强制绑定this

## 隐式绑定
使用.运算符，调用对象上的函数，自动绑定为该调用对象，但**可能会丢失绑定关系**

```javascript
var obj = {
    name: 'bobo',
    say: function(){
        console.log(this.name);     // this指向obj
    }
}
```

下面介绍两种丢失this绑定关系的场景
### 给函数起别名，修改引用关系
```javascript
var obj = {
    name: 'bobo',
    say: function(){
        console.log(this.name);
    }
}
var test = obj.say;     // 重新赋值后，绑定关系丢失，本质上是修改了一次地址引用
test();                 // undefiend
```

### 函数传参
```javascript
var obj = {
    name: 'bobo',
    say: function(){
        console.log(this.name);
    }
}
var test = obj.say;
setTimeout(test, 100);      // undefined
```

### 总结
>将函数的引用重新赋给其他变量，本质上是修改函数的上下文环境，将函数上下文暴露到了全局作用域下（参见下面的例子）
>回调函数作为参数传入时，是一次隐式的赋值操作，同样修改了函数的上下文
>上述两种操作都会导致原函数体内this绑定失效

```javascript
var demo = {
    name: 'demo',
    print: function(){
        console.log('test1', this.name);
        var obj = {
            name: 'bobo',
            say: function(){
                console.log('test2', this.name);
            }
        }
        var test = obj.say;
        test();    
    }
}
demo.print();
// 输出
// test1 demo
// test2 ''
```

## new绑定
将this绑定到新创建的对象实例上，这也是new关键字的作用
```javascript
function Person(name){
    this.name = name;
}
var me = new Person('bobo');
```

## 优先级
new绑定 > 显式绑定 > 隐式绑定

## 后记
读完上面，请解释React中为什么存在this绑定的问题？