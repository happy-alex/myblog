---
title: node模块机制
date: 2017-07-10 19:01
categories: node
tags: node
---

node采用commonJS实现模块管理

## module.exports

1. node同一进程内只有一个上下文，不同文件内部只是node对js文件进行的封装，源代码被包裹在一个自执行函数中，文件内部的 require 和 module 等由node注入自执行函数中。
2. 标准的导出命令只有 module.exports, exports 其实是个语法糖，本质上 exports = module.exports，所以才能通过exports导出内容

```javascript
// a.js
function log() {
    console.log('this is a log');
}

function test(){}

// 写法一
exports.log = log;

// 写法二
module.exports = {
    log
};

// 错误写法, 无法导出函数test，这种写法等同于为exports重新赋值，仅仅修改了exports的指向，但module.exports并没有被修改
exports = { test };

// b.js
const logHelper = require('./a.js');   // 引入一个模块
logHelper.log();
```

## require
require命令第一次加载脚本文件时，就自动执行整个脚本，然后在内存中生成一个对象,形如：
```javascript
{
    // 模块名
    id: 'a',

    // 模块输出的接口
    exports: {
        log: function(){},
        ...
    },

    // 脚本是否执行完
    loaded: true,
    ...
}
```
这个对象会被缓存起来。
- 如果需要用到这个模块，则到exports上取值使用
- 如果重复require同一个模块，则node从系统缓存中取值，返回第一次脚本执行的结果，除非手动清除系统缓存。


## commonJS vs ESModule

> commonJS 输出的是一个值的拷贝，一旦输出，模块内部变化不会影响这个值； ESModule输出的是值的引用，随时反应模块内部值的变化

- ESModule export 输出的是接口， 和接口对应的值是动态绑定关系; commonJS module.exports输出的是值的缓存，不能动态更新

> commonJS 在代码运行时才会加载； ESModule只是一个静态定义，在代码静态分析时就会生成。

- import 具有最高优先级，会被js引擎做静态分析优化，在模块中最先执行，必须置于模块顶层，因此无法做条件判断(有个提案叫import()可完成异步动态加载)
- require 运行时加载， 同步

> commonJS中顶层this指向当前模块；ESModule顶层this指向undefined

```javascript

var i = 0;

// 错误写法，export不能输出一个值
export i;

// 正确，default可视作一个变量名
export default i;
```

## ESModule 加载 commonJS
```javascript
// a.js
// 等同于export default function log(){}
module.exports = function log(){}

// b.js
import log from './a';
log();
```
tips: commonJS的缓存机制依然有效


## commonJS 加载 ESModule
1. import()提案
2. babel-register转码
```javascript
// react ssr
// 引入babel-register来转码react组件
require('babel-register')({

    // 忽略哪些文件
    ignore(filename) {},
    presets: [ 'env', 'react', 'stage-0' ],
    plugins: [
        'transform-decorators-legacy',
        [
            'module-alias',
            [
                // alias别名配置
                { src: './src/xx', expose: '$XX' },
            ]
        ],
        [
            'babel-plugin-transform-require-ignore',
            {
                extensions: [ '.scss' ]
            }
        ],
        [
            'transform-runtime',
            {
                helpers: false,
                polyfill: false,
                regenerator: true,
                moduleName: 'babel-runtime'
            }
        ]
    ]
});

// react本身有适配
const React = require('react');

// 业务组件ESModule
const Card = require('./src/card').default;
```