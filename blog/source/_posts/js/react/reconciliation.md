---
title: reconciliation
date: 2017-04-19
categories: js
tags: react
---
以下翻译自官网文档，有删减

## 动机

React在下一次render时会返回一棵新的React元素树，React需要弄清楚如何有效地更新UI以匹配最新的树。

这个算法问题有一些通用的解决方案，即生成将一棵树转换成另一棵树的最小操作次数。然而现有技术的算法具有O（n3）的复杂度，其中n是树中元素的数量。

如果我们在React中使用它，显示1000个元素将需要大约10亿次比较。这太贵了。相反，React基于两个假设实现了一个启发式O（n）算法：

>不同类型的两个元素将产生不同的树。
>开发人员可以通过key属性来暗示哪些子元素可以在不同渲染中保持稳定。

实际上，这些假设对几乎所有实际用例都有效。

## diff算法

在diff两棵树时，React首先比较两个根元素。 根据根元素的类型，行为会有所不同。

### 不同类型的元素
如果两个根元素类型不同，直接卸载旧树，创建新树。

### 相同类型的DOM元素
当比较两个相同类型的React DOM元素时，React查看两者的属性，保持相同的底层DOM节点，并且仅仅更新变化的属性
在处理DOM节点之后，React然后对子节点进行递归。

### 相同类型的组件
当组件更新时，实例保持不变，以便在渲染之间保持状态。 React更新底层组件实例的props以匹配新元素，并在底层实例上调用componentWillReceiveProps（）和componentWillUpdate（）。

接下来，调用render（）方法，diff算法对前一个结果和新结果进行递归。

### 递归DOM节点
默认情况下，当对DOM节点的子节点进行递归时，React会同时迭代两个子节点列表，并在出现差异时生成突变。

例如直接在子节点末尾添加元素时，下面的例子可以很简单实现
```javascript
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>

```
React可以匹配first和second, 然后插入third

如果只是这样简单的实现它，在子节点列表开头插入一个元素可能会有更糟糕的性能。 例如，在这两棵树之间进行转换效果不佳：
```javascript
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

```
React将突变每个子节点，而不是意识到它可以保持Duke和Villanova子树不变。这种低效率可能是一个问题。


### keys
为了解决这个问题，React支持一个key属性。 当子节点有key时，React使用key属性将原始树中的子节点和后续树中的子节点相匹配。 例如，在上面的低效demo中添加一个key可以使更新更有效率：
```javascript
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```
当用索引作key时，重新排序容易导致组件状态出现问题。 组件实例根据key进行更新和重用。如果key是索引，则移动子节点顺序会更改它的值。 因此，不受控组件比如input，状态可能会出现[意想不到的问题](https://codepen.io/pen?&editable=true&editors=0010)。

## 后记
Q: 为什么只有渲染数组时才需要声明key，其他声明式JSX元素不需要？
A: 声明式JSX元素在React.createElement()中的参数位置始终是固定的，这个位置就是天然的key，而动态变化的数组其内部索引经常变化

Q: index作key有什么危险？数组如果不存在动态变化(增加，删除，重新排序)是否可以用index作key？
A: 参见上述“意想不到的问题”链接。如果数组内容不存在动态变化，用index作key也是可以的。