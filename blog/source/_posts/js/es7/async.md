---
title: async/await动手实现
date: 2018-12-10
categories: js
tags: [es7, js]
---

**async/await本质上是一个generator/yield的语法糖，通过内置一个执行器，来实现自动执行。**

generator/yield的实现可参考上一篇 [generator动手实现](/js/es6/generator/)

## 自动执行器

```javascript
// 一个简单的自动执行器
var autoGen = function(fn) {
    var gen = fn.apply(this, arguments);

    // 返回Promise,是因为async要求总是返回一个Promise
    return new Promise((resolve, reject) => {
        function step(key, arg){
            try {
                var info = gen[key](arg);
            } catch(err) {

                // 捕获异常，以便抛出
                return reject(err);
            }

            if (info.done) {
                return resolve(info.value);
            }

            // info.value可能是一个promise, 所以应当在then之后再恢复执行
            Promise.resolve(info.value).then(_ => {
                step('next', info.value);
            }, err => {
                step('throw', err);
            });
            
        }
        return step('next');
    });
}
```

下面验证下我们刚才写的执行器是否能正常工作
```javascript
function *test() {
    var a = yield 1;
    console.log(a);
    var b = yield 2;
    console.log(b)
}
autoGen(test);
console.log('after gen');

// 输出结果
// after gen
// 1
// 2
// Promise {<resolved>: undefined}
```

## async/await

async/await的实现可以看做是先把async/await转换为generator/yield语法，再通过上面的自动执行器包裹执行

## forEach
```javascript
async function fn(v) { return Promise.resolve(v); }
var arr = [1, 2, 3];
arr.forEach(async item => {
    await fn(item)
});
```
这种写法实际上并不是顺序执行的, 它等同于
```javascript
var temp = async function(item) {
    await fn(item)
}
for (var i = 0, len = arr.length; i < len; i++) {
    temp(arr[i]);
}
```
every/some等函数都存在这个问题，如果希望顺序执行可直接使用for循环

## 补充：一个细微差别
```javascript
const p = Promise.resolve();
(async () => {
  await p; console.log('after:await');
})();
p.then(() => console.log('tick:a'))
 .then(() => console.log('tick:b'));
```

以上在node8下运行结果：
```bash
after:await 
tick:a 
tick:b
```

node10下运行结果
```bash 
tick:a 
tick:b
after:await
```
