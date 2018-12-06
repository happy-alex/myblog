---
title: class component 中的 super(props)
date: 2018-04-14
categories: js
tags: react
---

react中class component常见写法
```javascript
class Card extends React.Component {
    constructor(props) {

        // 此处this还不存在

        super(props);
        this.state = {
            name: ''
        }
    }
    render() {

        // 我们并没有显示定义this.props，这里的值怎么来的
        console.log(this.props)
        return <p>demo</p>
    }
}

```

### this.props是怎么来的？

React.Component中会帮你自动绑定好this.props
```javascript
// Component的构造器中自动赋值，super(props)就是用来执行这里的
class Component {
    constructor(props) {
        this.props = props;
    }
}

// 在调用开发者自定义的构造器后，二次赋值
const instance = new Card(props);
instance.props = props;
```

### 为什么子类构造器内要调用super(props)?

上面可以看到React会有一个this.props二次赋值的情况，即使super()不传props，react也能正常工作，那为什么还需要把props传给super？

考虑这种情况：
```javascript
class Card extends React.Component {
    constructor(props) {
        super();                    // 😬 We forgot to pass props
        console.log(props);         // ✅ {}
        console.log(this.props);    // 😬 undefined 
    }
}
```
因此react始终建议调用 super(props)

### context

react中context会当做第二个参数传给类的constructor，那为什么我们没有调用super(props, context)?

本来这样做是对的，但实际上context并不经常使用，尤其在constructor中，所以就没那么强调了

### 银弹

如果有一天提案通过，类的实例属性可以不再写在构造器中，就不需要关心constructor里的this.props问题了
```javascript

// 彻底抛弃构造器写法
class Card extends React.Component {
    state = {}
    render() {
        //...
    }
}
```


[以上翻译自Dan的博客](https://overreacted.io/why-do-we-write-super-props/)