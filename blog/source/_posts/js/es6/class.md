---
title: class/extends实现
date: 2018-08-03 15:44
tags: [js, es6, babel]
---

分析下es6中class/extends的实现

```javascript
import React, { Component } from 'react';
class List extends Component {
    constructor() {

        // 调用父类构造函数
        super();
        this.state = {};
    }

    render() {
        return <p className="desc">这是一个react组件</p>
    }
}
```

babel转码es5后
```javascript
Object.defineProperty(exports, "__esModule", {
    value: true
});

var _react = require('react');

var _react2 = _interopRequireDefault(_react);

// 判断模块导出的方法是export or export default
function _interopRequireDefault(obj) { 
    return obj && obj.__esModule ? obj : { default: obj }; 
}

function _classCallCheck(instance, Constructor) { 
    if (!(instance instanceof Constructor)) { 
        throw new TypeError("Cannot call a class as a function"); 
    } 
}

/**
 * 如果父类构造函数存在，返回父类构造函数
 * self 子类this
 * call 父类构造函数
 */
function _possibleConstructorReturn(self, call) { 
    if (!self) { 
        throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); 
    } 
    return call && (typeof call === "object" || typeof call === "function") ? call : self; 
}

// 实现两个构造函数的继承
function _inherits(subClass, superClass) { 
    if (typeof superClass !== "function" && superClass !== null) { 
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass); 
    } 

    // 实例属性继承-->将subClass.prototype指向父类原型superClass.prototype
    subClass.prototype = Object.create(superClass && superClass.prototype, { 
        constructor: { 
            value: subClass, 
            enumerable: false, 
            writable: true, 
            configurable: true 
        } 
    }); 

    // 静态属性继承-->子类构造函数的__proto__指向父类构造函数，以便子类能够继承父类的静态属性和方法
    if (superClass) 
        Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; 
}

// 给指定对象添加方法
var _createClass = function () { 
    function defineProperties(target, props) { 
        for (var i = 0; i < props.length; i++) { 
            var descriptor = props[i]; 
            descriptor.enumerable = descriptor.enumerable || false; 
            descriptor.configurable = true; 
            if ("value" in descriptor) 
                descriptor.writable = true; 
            Object.defineProperty(target, descriptor.key, descriptor); 
        } 
    } 

    // 构造函数，实例属性，静态属性
    return function (Constructor, protoProps, staticProps) { 
        if (protoProps) defineProperties(Constructor.prototype, protoProps); 
        if (staticProps) defineProperties(Constructor, staticProps); 
        return Constructor; 
    }; 
}();

var List = function(_Component) {

    // 实现extends关键字
    _inherits(List, _Component);

    // 实现constructor()方法
    function List(props) {

        // 检查this指向是否正确，只能通过new List的方式调用，防止通过List()调用
        _classCallCheck(this, List);

        // 实现super关键字, 和es5的差别就是这里将父类this返回了，子类去修饰父类this
        // 在子类的实例对象上，调用了父类的构造函数方法，以便将父类的属性拷贝到子类上
        var _this = _possibleConstructorReturn(this, 

            // 这里拿到的是父类的this，List.__proto__在_inherits函数中已经指向了父类构造函数
            ( List.__proto__ || Object.getPrototypeOf(List) ).call(this, props)
        );

        // es5继承：没有返回父类，只是单纯的借用父类构造函数，子类无法获取父类this，导致无法继承原生JS对象
        // ( List.__proto__ || Object.getPrototypeOf(List) ).call(this, props)

        // 对父类/子类this进行增强
        _this.state = {};
        return _this;
    }

    // 绑定react的多个生命周期钩子
    _createClass(List, [{
        key: 'render',
        value: function render() {
            return _react2.default.createElement(
                'p',
                {className: 'desc'},
                '2222'
            );
        }
    }]);
    return List;
}(_react.Component);
```

总结：

> es6继承: 先通过super关键字将父类实例的属性和方法添加到this上，再调用子类的构造函数完成对this的修饰。
具体实现上相比es5使用的构造函数复制，es6使用的是将子类构造函数的原型链指向父类的构造函数。