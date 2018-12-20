---
title: generator动手实现
date: 2018-12-10
categories: js
tags: [es6, js]
---

```javascript
function *test() {
    console.log('generator demo');
    yield 'aa';
    yield 'bb';
}
var gen = test();
gen.next();
```

手动实现
```javascript
function wrap(cb) {
    // 维护现在应该跳到哪一个switch分支，以便顺序执行
    var obj = {
        next: 0
    }
    return function() {
        return {
            next: function(){
                var v = cb(obj);

                // 如果没有值，说明generator已经执行完了
                if (!v) {
                    return {value: undefined, done: true};
                }
                return {value: v, done: false};
            },
            throw: function(err){
                throw Error(err);
            }
        }
    };
}

// 上面test函数的转换实现
var test2 = wrap(function(ctx){

    // while是用来处理源代码中在循环语句里使用了yield
    while(1) {
        switch(ctx.next) {
            case 0:
                console.log('generator demo')
                ctx.next = 1;
                return 'aa';
            case 1:
                ctx.next = 2;
                return 'bb';
            case 2:
            default:
                return;
        }
    }
});
```