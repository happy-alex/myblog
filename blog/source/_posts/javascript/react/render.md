---
title: React渲染DOM的奥秘
date: 2018-04-13
categories: js
tags: [react, jsx]
---

> 总结： React Component Render -> JSX -> React.createElement -> Virtual DOM -> DOM

## JSX -> React Element

一个基本的JSX语法：
```javascript
render() {
    return (
        <div className="page" ref="refPage">
            <Header title="去哪儿" />
            <p>this is a div</p>
            this is a text
        </div>
    );
}
```

上述代码通过babel（JSX -> React.createElement()）的帮助，会被编译为
```javascript
return React.createElement(
    'div',
    {
        className: 'page',
        ref: 'refPage'
    },

    // 自定义组件，当发现类型为class或者function时，递归完成构建
    React.createElement(
        Header, 
        {
            title: '去哪儿'
        },
        null
    ),

    // 原生DOM节点
    React.createElement(
        'p',
        null,
        'this is a div'
    ),

    // 普通文本
    'this is a text'
);
```

通过调用React.createElement(type, config, ...children) 方法生成对dom的抽象描述--element;
最终得到结构形如：
```javascript
{
    type: 'div',
    ref: 'refPage',
    key: null
    props: {
        className: 'page',
        children: [
            {
                type: Header,
                ref: null,
                key: null,
                props: {
                    title: '去哪儿',
                    children: []
                }
            },
            {
                type: 'p',
                ref: null,
                key: null,
                props: {
                    chidren: 'this is a p'
                }
            },
            'this is a text'
        ]
    },
}
```
上面的例子列举了children中的3个类型：自定义组件，原生DOM节点，String；此外还有 null, undefined, number, boolean, array等。


## Reace Element -> Dom

调用ReactDOM.render() 将element渲染到真实DOM.

## 深入

React在生成最终描述DOM的轻量对象--Element的过程，其实就是React reconciliation(调和)过程。一般这个过程在首次渲染和setState之后触发。
