## 渲染奥秘


## JSX生成element

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

上述代码会被编译为
```javascript
return React.createElement(
    'div',
    {
        className: 'page',
        ref: 'refPage'
    },

    // 自定义组件
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
通过 React.createElement(type, config, ...children) 方法生成element;
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


## element 生成 vNode
jsx -> element -> vnode  -> dom