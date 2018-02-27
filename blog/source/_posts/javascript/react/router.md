---
title: 前端路由设计杂谈
date: 2018-02-13
categories: js
tags: [react, router]
---

## 前言
用户通过访问不同的url，获得不同的页面展示。很长一段时间这部分工作都是后端MVC来做的。后端通过配置不同的controller从而决定不同url应该展示不同的view。

无论是传统的MVC模式还是方兴未艾的前端SPA模式，路由设计的目的只有一个，即**保证url和视图的同步**。

网上有很多文章讲了spa，以及相关路由的问题。这些基础科学我就不写了，主要谈下我对前端路由设计的认识。

前端路由是一个状态机，监听location的变化，经过控制器(比如react-redux中的Router组件)的处理，渲染出不同的UI。


前端路由的匹配，基本上是根据下面3个条件来完成的匹配。
- 路径
- 嵌套关系
- 优先级
<br/>

## SPA路由的实现
spa路由需要完成两件事：
- 更新url页面不刷新
- 监听url的变化，渲染不同的UI.

### 实现更新url页面不刷新？
1.hash方式
url有很多组成部分，其中哈希片段部分（即#部分）更新导致url变化，但页面是不会刷新的。

2.h5的historyAPI
History API主要有两个方法：history.pushState()，history.replaceState()。这两个方法可以对浏览器的历史栈进行操作，将传入的url和相关数据压栈，并将浏览器地址栏的url替换成传入的url，且不刷新页面。

### 监听url变化
1.hash方式
通过window.onhashchange来监听
2.h5的historyAPI
window.onpopstate可以监听history的变化。

这两种方式都存在一定的兼容性问题，推荐使用history库，react-router核心也是用的这个库。


<br/>
## History
history提供3种历史管理模式：browserHistory，hashHistory，memoryHistory.

1.BrowserHistory 
需要服务端路由支持，假设当前路由为qunar.com/index,那么刷新浏览器后，将会请求服务器上的/index，如果服务器不支持对/index的访问，页面将404，因此实现上需要服务器拦截这种前端发出的特定路由请求并返回一个html文件

2.HashHistory 
完全由前端控制实现的路由，不依赖服务端任何配置。 qunar.com/#/index 刷新后页面由前端控制渲染。
使用这种路由的url，其末尾经常会有类似#?_k=koomro这样的字符。那么_k是什么？
每一个location允许有自己的state，**state存储在sessionStorage当中**。
当一个history在路由间跳转时，我们直接通过window.location.hash = newhash完成，且不用传输state。**_k就是state在sessionStorage中存储的key**。 
当浏览器发生前进和后退时，可以根据这些_k取出state。
url中hash部分浏览器是不会发送给服务端的(可以通过代理工具抓包验证下)。因此如果后端需要依赖参数处理请求，请不要把参数放在hash部分。这一点尤其应该在hashHistory的应用中注意下。

3.MemoryHistory 我还没找到这种模式的应用场景，待后续补充...

4.线上生产环境推荐使用browserHistory。why？
- 上面讲hashHistory的时候说过，hash部分是不会被发送到服务端的，服务端只能知道页面级别.htm。而hash部分的参数服务端处理不了
- 理论上服务端应该知道一次请求的所有细节，而hashHistory明显跳出了服务端可控范围，作为线上生产环境这可能会带来风险。
- history这个库在实现browserHistory是使用的浏览器自带的h5提供的API，但hash由于没有替换历史记录的功能，hashHistory使用了polyfill实现功能，因此在处理一些边边角角浏览器时可能解决的并不好，存在隐患。


<br/>
## react-router
react-router是官方推荐的react应用的路由库。最新版本是v4。之前我用的都是v2版本，最近在项目中切到了v4.

### router v2
```javascript
// 方法一：传入一个独立的routes配置文件。然后通过this.props.children渲染子路由
<Router history={history} routes={routes} />

// 方法二：
<Router history={browserHistory}>
    <Route path='/' component={App}>
        <IndexRoute component={Home} />
        <Route path='about' component={About} />
        <Route path='features' component={Features} />
    </Route>
</Router>
```

后来官方觉得v2组件化的不够彻底，尤其routes文件这种方式不伦不类，所以在v4的时候本着**一切皆组件**的理念彻底来了一次彻底重构。所有匹配的路由都应当显式声明在jsx文件中。


### router v4
react-router            // 核心库，提供router，route等概念
react-router-dom        // 适配web端，提供browserHistory,hashHistory等组件，在web应用中应当直接从这个库里引用组件
react-router-native     // 适配诸如react-native等
react-router-redux      // 将redux和history结合起来

v2为了同步路由状态到redux中，经常还要搭配react-router-redux来使用。v4中可以不再使用react-router-redux这个库了。

Router作为根组件，没有做什么特别的东西，只是把当前应用的路由状态设为整颗组件树的context。context.router = {history, route}，使其子组件route可以拿到当前路由相关数据。
Route组件从context.router中拿到当前的路由进行匹配从而渲染。

### Q&A
1.url发生改变，Route组件如何重新匹配？
> Router组件内部维护了一个state={match: {}}。这个match的值是一个对象。通过history.listen方法监听url，只要url发生了变化，就会重新setState，把match设置为一个新的对象，由于match总是指向一个全新的对象引用，所以state总被认为发生了改变，从而导致Router组件进行更新。

> Router作为路由管理中的顶层父组件，添加了一个名字为router的context。被Router包裹的所有子组件，都拥有这个router，当Router组件更新的时候，会重新设置context.router，子组件Route知道context.router发生了改变，进行重新匹配进而渲染不同组件。

2.被react-reudx的@connect包裹的组件接收不到this.context.router上的变化。所以如果connect包裹了Route组件，会导致UI不能不正常渲染。
```javascript
// 不知道有没有人写过这样的代码，如果有你就知道我在说什么了。
class App extends React.Component {
    render () {
        return (
            <div>
                <div>
                    <Link to="/index" />
                    <Link to="about" />
                </div>
                <div>
                    <Route path="/index" component={Index}></Route>
                    <Route path="/about" component={About}></Route>
                </div>
            </div>
        );
    }
}
const myApp = connect()(App);
<Provider store={store}>
    <HashRouter>
        <App />
    </HashRouter>
</Provider>
```
现象：如果使用上面这样的代码，你会发现点击切换Link，浏览器地址栏的url一直在变，但是对应的视图却不更新。why?

connect作为高阶组件，它在wrappedComponent之上进行了一些处理。**connect重写了shouldComponentUpdate方法，只会对比显式传给他的props**，从而决定是否要更新组件。而context.router的变化并不会导致其props的改变，所以connect高阶组件的shouldComponentUpdate方法总是返回false，其被包裹组件不更新，所以Route组件也就不再响应路由了。

结论：**Route组件不能被connect包裹**，否则路由管理将会失效，而正确的打开方式是connect作为Route的子组件。

<br/>
## vue-router
TODO...