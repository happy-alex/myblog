---
title: 浅谈js中的继承
date: 2018-08-01 15:44
categories: js
tags: [js, babel]
---

## es5中的继承
es5中常见继承写法，(命名取自《js高级程序设计》)

### 原型链继承
```javascript
    function SupClass() {
        this.name = 'sup';
        this.hobby = ['sing', 'draw'];
    }
    SupClass.prototype.say = function() {};
    function SubClass() {
        this.name = 'sub';
    }
    SubClass.prototype = new SupClass();
    var obj = new SubClass();
```

缺点： 
1. 如果SupClass存在引用类型的值，则在SubClass的所有实例上都共享了这个值
2. 不能向SupClass的构造函数中传递参数


### 借用构造函数继承
```javascript
    function SupClass() {
        this.name = 'sup';
        this.hobby = ['sing', 'draw'];
    }
    SupClass.prototype.say = function() {};
    function SubClass(...props) {
        SupClass.call(this, ...props);
        this.name = 'sub';
    }
    var obj = new SubClass();
```

缺点：
1. 子类无法继承SupClass.prototype定义的属性

### 组合继承 
```javascript
    function SupClass() {
        this.name = 'sup';
        this.hobby = ['sing', 'draw'];
    }
    SupClass.prototype.say = function() {};
    function SubClass(...props) {
        SupClass.call(this, ...props);
        this.name = 'sub';
    }
    SubClass.prototype = new SupClass();
    var obj = new SubClass();
```
结合原型链和构造函数，借用原型链实现对原型属性和方法的继承，借用构造函数实现对实例属性的继承，常见
缺点：
1. 调用了两次父类构造函数
2. 子类原型上存在冗余的父类的实例属性

### 寄生组合继承
对组合继承的优化，推荐方案。

```javascript
    // 一般的组合继承存在调用两次父类构造函数的问题
    function SupClass() {
        this.name = 'sup';
        this.hobby = ['sing', 'draw'];
    }
    SupClass.prototype.say = function() {};
    function SubClass(...props) {

        // 第一次调用父类构造函数
        SupClass.call(this, ...props);
        this.name = 'sub';
    }

    // 优化前
    // 第二次调用父类构造函数，这次可避免，关键点是这里如何得到一个父类原型的副本, 其实可以直接clone一个父类原型
    SubClass.prototype = new SupClass();

    // 优化后
    // 效果等同于inherit(SubClass, SupClass)
    var prototype = SupClass.prototype;
    prototype.constructor = SubClass;
    SubClass.prototype = prototype;


    var obj = new SubClass();
```
基于这种继承方法，可以抽象出一个工厂函数, 将子类原型指向父类原型
```javascript

// 这里的inherit可以有很多实现方式，比如
//SubClass.prototype = Object.create(SupClass.prototype, {constructor: {value: SubClass}});
function inherit(SubClass, SupClass) {

    // 克隆一个父类原型的副本, 这里有很多写法，造成inherit的写法不同
    function Bridge() {}
    Bridge.prototype = SupClass.prototype;
    var prototype = new Bridge();

    // 将子类构造函数的原型指向父类原型副本，并重写指向后子类原型的constructor
    prototype.constructor = SubClass;
    SubClass.prototype = prototype;

    return SubClass.prototype;
}


function SubClass() {}
function SupClass() {}
inherit(SubClass, SupClass);
var obj = new SubClass();
```


补充： 另外两种不常见的继承
### 原型继承
适用于只希望基于简单对象进行扩展，返回一个原型为指定参数的对象，类似Object.create();
```javascript
    function create(obj) {
        function Bridge(){}
        Bridge.prototype = obj;
        return new Bridge();
    }
    var obj = create({});
```

### 寄生继承
实现一个工厂函数，对传入的对象，进行增强
```javascript
    function create(obj) {
        function Bridge(){}
        Bridge.prototype = obj;
        return new Bridge();
    }
    function createInstance(obj) {
        var instance = create(obj);
        instane.say = function(){}
        return instance;
    }
    var obj = create({});
```

## es6的继承
es6的继承和上面提到的寄生组合继承类似，本质上，es6的继承是es5继承的一个语法糖。
es6通过关键字extends来实现继承，可以实现继承一个普通类或者原生JS对象。
```javascript
class List extends Component {
    constructor() {
        super();
        this.name = 'demo';
    }
}
```

babel转码后
```javascript
function _inherits(subClass, superClass) { 
    if (typeof superClass !== "function" && superClass !== null) { 
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass); 
    } 

    // 将subClass.prototype指向父类原型superClass.prototype
    subClass.prototype = Object.create(superClass && superClass.prototype, { 
        constructor: { 
            value: subClass, 
            enumerable: false, 
            writable: true, 
            configurable: true 
        } 
    }); 

    // 子类构造函数的__proto__指向父类构造函数，以便子类能够继承父类的静态属性和方法
    if (superClass) 
        Object.setPrototypeOf ? Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass; 
}

// 检查构造器函数是否是通过new调用
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

var List = function(Component) {
    // 实现extends关键字
    _inherits(List, Component);

    function List(props) {
        // 检查this指向是否正确，保证该函数只能通过new List的方式调用，防止直接调用List()
        _classCallCheck(this, List);

        // 实现super关键字
        // 在子类的实例对象上，调用了父类的构造函数方法，以便将父类的属性拷贝到子类上
        var _this = _possibleConstructorReturn(this, 
            ( List.__proto__ || Object.getPrototypeOf(List) ).call(this, props)
        );

        _this.name = 'demo';
        return _this;
    }
    return List;
}(Component);
```
tips: 在class Obj extends Object{}时，无法通过super来向Object构造函数传值, es6中Object()会忽略通过非new方式传入的参数


## 继承原生JS对象
原生JS对象：Array, Date, Math, String, Boolean, Number, RegExp, Function, Object.

如何实现一个MyArray对象，继承原始的Array?
```javascript
function MyArray() {
    Array.apply(this, arguments);

    // 这种方式就可以实现对Array的继承，why?
    // return Array.apply(this, arguments);
}
inherit(MyArray, Array);
var arr = new MyArray();
arr[0] = 1;
console.log(arr, arr.length);   // MyArray {0: 1}, 0
```
通过上面的例子可知， 无法用es5的办法实现对原生JS对象的继承，why?
因为我们定义的子类MyArray无法获得原生构造函数的内部属性。原生构造函数会忽略我们通过call或者apply绑定的this。
**es5的继承总是先新建子类对象的this，再父类对象的属性添加到子类上**，由于我们无法获取原生构造函数的内部属性，就无法正常调用原生构造函数完成继承。

```javascript
class MyArray extends Array {}
var arr = new MyArray();
arr[0] = 1;
console.log(arr, arr.length);   // [1], 1
```
利用extends关键字可以实现对原生JS对象的继承，why?
**es6的extends继承总是先新建父类对象的this，再调用子类构造函数完成初始化**，不存在上面提到的无法获取内部属性的问题，故可以正常完成继承。

## es5 vs es6
> es5继承: 先新建子类对象的this，再将父类对象的属性添加到子类上。
> es6继承: 先通过super关键字将父类实例的属性和方法添加到this上，再调用子类的构造函数完成对this的修饰。
> es6继承: 子类构造函数的原型链指向父类的构造函数；ES5中使用的是构造函数复制，没有原型链指向

## 参考
1. [setPrototypeOf](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/setPrototypeOf)