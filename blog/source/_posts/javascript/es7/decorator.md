---
title: decorator-es7
date: 2017-04-12 20:57:42
tags: es7 decorators
---

前段时间在工作中，团队因为react的组件里的this的问题就事件写法产生了分歧，
```javascript
class App extends Component {
    // 第一种写法
    onTapCard = () => {
    }
    render () {
        return <Card onTap={this.onTapCard} />;
    }

    // 第二种
    onTapCard() {
    }
    render () {
        return <Card onTap={() => {this.onTapCard()} />;
    }
}
```

后来有人介绍了@autobind这个装饰器，自动绑定好this，于是问题解决。下面说一下decorator是个神马东西。

ES7从python借来了decorator概念。这个语法糖通过ES5的Object.defineProperty实现。
首先回顾下Object.defineProperty的用法：描述对象的属性，比如是否可枚举，可写可读等，语法Object.defineProperty(obj, prop, descriptor)。

那ES7的decorator具体做了什么呢？
```javascript
// 这里定义了装饰器的执行内容（@autobind就相当于这个）
function readonly(target, key, descriptor) {
  descriptor.writable = false;
  return descriptor;
}

class Dog {
    @readonly
    say() {
        return 'bobo';
    }
}
```
上面这个class在转换为es5代码后，等价于
```javascript
// 第一步
function Dog() {}

// 第二步,如果配置了@readonly，js引擎就会执行下面这段代码，跳过第三步
var descriptor = {
  value: function () { return 'bobo' },
  enumerable: false,
  configurable: true,
  writable: true
}

/* decorator 接收的参数与 Object.defineProperty 一致 */
/* 这里的readonly就是上面提前写好的一个函数 */
descriptor = readonly(Dog.prototype, 'say', descriptor) || descriptor;

/* 相当于下面的第三步 */
Object.defineProperty(Dog.prototype, 'say', descriptor);

// 第三步
Object.defineProperty(Dog.prototype, 'say', {
    value: function() { return 'bobo'; },
    enumerable: false,
    configurable: true,
    writable: true
});
```

### 作用在类和方法上的decorator
decorator可以作用在方法上，也可以作用在类上，不同点在于
* 作用在 **方法** 上的 decorator 接受的第一个参数（target）是 **类的 prototype** ：decorator(Dog.prototype, prop, descriptor)
* 如果在 **类** 上的 decorator 接受的第一个参数 （target） 是 **类本身** ：decorator(Dog, prop, descriptor)

我们可以自定义一个decorator
```javascript
function enumerable (isEnumerable) {
  return function(target, key, descriptor) {
    descriptor.enumerable = isEnumerable
  }
}

class Dog {
  @enumerable(false)
  eat () { }
}
```

总而言之，**ES7的decorator作用就是返回一个新的descriptor,并将这个属性描述符应用在目标方法上**
> decorator 让我们有机会在代码的执行期间改变其行为

备注：
1. core-decorators提供了很多实用的装饰器，文章开头的@autobind就是有这个库提供的
2. decorator虽然目前还是个提议，但是可以通过babel转译来使用

相关链接
1. https://zhuanlan.zhihu.com/p/20139834?columnSlug=FrontendMagazine
