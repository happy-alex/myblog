---
title: hooks背后的设计思想
date: 2018-12-13
categories: js
tags: react
---

此前在React Conf 2018 上 react公布了自己最新的实验性API: hooks

之前我们提到react组件中复用代码主要有两个方式：高阶组件以及render props。
hooks给函数式组件引入了state的概念，可以更方便的抽离业务逻辑代码，实现业务逻辑复用和组件复用

hooks的基本介绍有很多文章，这里就不再赘述。下面我们主要来聊聊hooks背后的设计思想

### 允许多次调用useState

Q: 为什么hooks支持useState多次调用， 而不是一次useState维护所有状态？

```javascript
function Card(){
    // 示例1 多次调用useState
    const [name, setName] = useState('bobo');
    const [age, setAge] = useState(25);

    // 示例2 只调用一次useState, 这种方式hooks也支持，但并不推荐
    const [person, setPerson] = useState({
        name: 'bobo',
        age: 25
    });
}
```

单个useState维护一个复杂状态（参考上述示例2），hooks也支持。但hooks更建议将这种复杂状态拆分成多次调用useState(), 保证每一个useState尽可能简单（示例1）。
多个useState调用的好处在于，方便随时提取state和effect到自定义的hook中，以复用逻辑
另一个很重要的原因是，如果hooks只允许调用一次useSatate, 那么外部代码逻辑就无法引入到函数组件内执行，自定义hook复用也就无从谈起

### 为什么要依赖调用顺序 

hooks的一大特性就是只能用在函数顶层作用域内，不能用在if/else等条件判断，循环语句内，原因就是react内部并不真正保存state，它通过调用顺序来追踪state。

**多个useState的调用是借由一个数组来维护的，数组内的索引来实现对多个state的“识别”。**

如果用在if/else等语句内，当函数组件重新执行时可能会改变调用顺序，导致错误。

那么为什么要这么设计？

useState()可以接收一个参数， 能否通过这个参数来来作为react追踪的依据？

由于函数组件内useState会被多次调用，传入useState()的参数如果一样，就会出现名字冲突，react追踪时就会混乱。
虽然开发者可以通过代码检查避免这种错误写法，但显然这并不是一个好的API设计

既然简单的命名区分有冲突问题，那能否使用Symbol特性来保证传入useState的值唯一不重复？
这看似能解决上边的问题，但其实并没有解决，假设我们从外部引入了一个公共函数, 这个函数内部封装的hook使用Symbol作参数，在函数组件内多次调用这个公共函数时，还是会造成react追踪错误
```javascript
// useFormInput.js
const valueKey = Symbol();
function useFormInput() {
  const [value, setValue] = useState(valueKey);
  return {
    value,
    onChange(e) {
      setValue(e.target.value);
    },
  };
}

// function component
function Person() {
  const name = useFormInput();
  const age = useFormInput();
  return (
    <div>
      <input {...name} />
      <input {...age} />
    </div>    
  );
}
```
上面的写法等同于下面这样，并没有解决问题
```javascript
  const [name, setName] = useState(valueKey);
  const [age, setAge] = useState(valueKey);
```

### 更深层次的考虑
上面我们尝试过一些让react保持对函数组件内state追踪的方式，都存在一些问题。事实上即便不使用调用顺序，也有许多其他方式来实现hook效果，像是闭包，参数加密等等。

比如上面提到过使用简单的Symbol作追踪，依然有冲突的问题，但通过闭包隔离的方式就能解决名字冲突的问题。实现如下：
```javascript
function createUseFormInput() {
  // 这样能保证每次调用，实例都是唯一的
  const valueKey = Symbol();  

  return function useFormInput() {
    const [value, setValue] = useState(valueKey);
    return {
      value,
      onChange(e) {
        setValue(e.target.value);
      },
    };
  }
}
```
问题是这些方式不见得比现有的维护调用顺序的hook规则更简单。
比如上面的方案，你就需要额外注意createUseFormInput工厂函数和useFormInput hook实例的区别

[关于hooks更多的考虑可以看这里](https://overreacted.io/why-do-hooks-rely-on-call-order/)

尽管hooks的实现机制还有很多争议，但就像Dan在文章中指出的，目前依赖调用顺序的hooks也有很多优点：

>由于没有命名空间的概念，开发人员复制粘贴组件代码，抽离code，提升成为一个自定义Hook的过程将更加简单，安全。
>hook规则足够简单，一定程度上减少了bundle size
>数组的方式是一个相对高效的实现，避免了Map查找的过程


### 后记

hooks并不完美，但目前它是解决现有问题的一个权衡。
因为hooks是实验性质的API, 所以上面我们的讨论在未来某个时候可能就是过时的，甚至是错误的，你需要注意到这一点