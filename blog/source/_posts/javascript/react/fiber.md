---
title: fiber
date: 2018-09-26
categories: js
tags: react
---

## 背景React15
### react核心思想：
内存中维护一颗虚拟DOM树，数据变化时（setState），自动更新虚拟DOM，得到一颗新树，然后diff新老虚拟DOM树，找到有变化的部分，得到一个change(patch)，将这个patch加入队列，最终批量更新这些path到DOM中。

### react 执行render()和setState()进行渲染时主要有两个阶段：
调度阶段 （Reconciler）: 用新数据生成一颗新树，遍历虚拟dom，diff新老virtual dom树，搜集具体的UI差异，找到需要更新的元素，放到更新队列中。
渲染阶段（Renderer）: 遍历更新队列，通过调用宿主环境的API，实际更新渲染对应元素。宿主环境，比如dom,native等。

### 3种实例
1.DOM，  对应真实的DOM节点
2.Element   描述UI，通过React.createElement()得到
3.Instance      React维护的虚拟DOM, 根据Element创建，对组件和DOM节点的抽象表示，维护组件内部状态和与DOM树的关系

### 优化实践
react本身为了提高页面渲染性能，推出了一些最佳实践
1.vdom 减少对dom的直接操作
2.无状态组件   减少组件内部状态和复杂度
3.shouldComponentUpdate  减少diff的次数
4.immutable   减少diff的成本
但以上都是针对js执行而提出的方法，具体到浏览器渲染，如何避免长时间的线程占用并没有给出好的建议。

## why Fiber的来源
Fiber之前的Reconciler阶段采用的是Stack Reconciler， 其自顶向下遍历vdom tree, 递归组件执行任务，过程无法中断。
假设有一个层级很复杂的组件，在顶层组件内执行setState, 那么调用栈可能会很长。由于调用栈过长，中间可能还有一些复杂操作，这些任务无法中断，就导致主线程被长时间阻塞。由于浏览器里渲染和js执行共一个主线程，在对响应要求高的场景，比如手势，动画等，就容易造成卡顿，延迟等现象，从而影响用户体验。

Fiber的出现就是为了解决这个问题。
Fiber 的解决思路：把渲染更新过程拆分成多个子任务，每次只做一小部分，做完看是否还有剩余时间，如果有继续下一个任务；如果没有，挂起当前任务，将时间控制权交给主线程，等主线程不忙的时候在继续执行。


## what Fiber是什么

计算机通常追踪程序执行的方式是使用调用堆栈。执行函数时，不断的把堆栈帧加入到堆栈中，一个堆栈帧就表示一个要执行的工作。
在处理UI时，如果执行太多的工作，就可能导致动画丢帧。

新版本的浏览器实现了有助于解决这个问题的API：requestIdleCallback 和 requestAnimationFrame.
requestIdleCallback 调度在空闲期间调用的低优先级函数
requestAnimationFrame调度在下一个动画帧上调用的高优先级函数。

问题是，为了使用这些API，就需要一种方法将渲染工作分解为增量单元。如果仅依赖于调用堆栈，它将继续工作直到堆栈为空，无法中断。
为了实现增量渲染的调度，就必须重新实现这个堆栈帧，以便可以将堆栈帧保留在内存中，然后按照自己的调度算法执行他们。同时由于这些堆栈栈是我们手动处理的，我们还可以加入并发或者错误边界等功能。

因此Fiber就是重新实现的堆栈帧，本质上Fiber也可以理解为是一个虚拟的堆栈帧，将可中断的任务拆分成多个子任务，通过按照优先级来自由调度子任务，分段更新，从而将之前的同步渲染改为异步渲染。
react内部有自己的优先级判断逻辑，比如动画，用户交互等任务优先级就明显要高。


### Fiber的目标
- 将耗时长可中断的任务拆分成多个子任务
- 对正在做的工作调整优先级，可以重做，复用上次的结果

### Fiber特性
- 增量渲染，把一个渲染任务拆分成多个子任务，平均到多个渲染帧中执行，每次只做一小段，做完后就把时间控制权上交给主线程
- 在渲染更新时，能够暂停，复用任务
- 不同类型的任务具有不同的优先级
- 并发方面的其他能力
- 错误边界


## how Fiber实现

简单来说就是，**时间分片 + 链表结构** 。
fiber就是维护每一个分片的数据结构。

### fiber && fiber tree
react中没有明确的Virtual Dom， 可以把fiber理解为我们习惯上的虚拟Dom概念。
react在render中第一次渲染时，利用React.createElement会创建一课Element树，同时会创建一颗结构一模一样的fiber tree，不同的Element类型对应不同类型的Fiber Node。
一个Fiber Node可以认为是一个对象，它表示组件需要做的工作。一个Element对应一个或多个Fiber Node。

上面提到Fiber要做增量更新，所以就要额外维护一些上下文信息，所以react 扩展出了 fiber tree 的概念，更新过程就是根据输入的数据以及当前fiber tree，构造出新的fiber tree(workInProgress tree)。上面我们提到了Instance, Fiber基于此进行了扩展，添加了一些其他概念： 
```javascript
{
    // 搜集diff结果，每个workInProgress tree节点都有一个effect list
    // 当前节点更新完毕，会向上merge effect list
    effect

    // reconcile过程中的快照，用于断电恢复
    workInProgress

    // fiber tree 与vdom tree类似，描述增量更新需要的上下文信息
    fiber
}
```


```javascript
// <Card />

// FiberNode 结构如下：
{
    // fiber类型
    type: Card,

    // type对应标记
    tag: 2,

    // tag类型，二进制，不同tag代表不同类型
    effectTag: 1,
    firstEffect: null,
    lastEffect: null,

    // 单链表结构，方便遍历fiber树上游副作用的节点
    nextEffect: FiberNode|null,

    // 标记子树上待更新任务的优先级
    pendingWorkPriority: number，

    // 第一个子fiber
    child: FiberNode|null,

    // 指向父fiber，表示当前节点处理完毕后，应该向谁提交自己的结果effect list
    return: FiberNode|null

    // 兄弟fiber
    slibing: FiberNode|null,

    // 当前父fiber中的位置
    index: 0,

    // fiber实例对象，指向当前组件实例
    stateNode: Card,

    // setState待更新状态
    updateQueue: null,

    // component中的相关props、state
    memoizedProps: {},
    memoizedState: {}
    pendingProps: {},

    // 和组件中的key,ref保持一致
    key: null,
    ref: null,

    // fiber更新时基于当前fiber克隆出的镜像，更新时记录两个fiber diff的变化；更新结束后alternate替换之前的fiber成为新的fiber节点
    alternate: {}
}
```
从上述fiber数据结构，可以看出fiber tree 是一个单链表结构，通过child, slibing, return完成结构关联。

### Fiber Reconciler
reconciler过程分为两个阶段:
#### Reconciliation
过程可中断。
将每个fiber节点作为最小工作单位，通过自顶向下逐个遍历fiber node，构造workInProgress tree(一颗新的fiber tree), 得到patch结果
具体过程如下：
```bash
1. 如果当前节点不需要更新，直接clone 子节点， 跳到步骤5；如果需要更新，则修改tag，记录更新类型
2. 更新当前节点状态，context，props，state等
3. 调用shouldComponentUpdate, 如果返回值为false，则跳转步骤5
4. 调用组件实例的render方法，得到一个新的子节点，同时为子节点创建Fiber Node（会尽量复用现有fiber，子节点的增删也发生在这里）
5. 如果没有产生child fiber, 该工作单元结束，把effect list归并到return上，并把当前FiberNode的sibling作为下一个工作单元。如果有child fiber，将child指向作为下一个工作单元。
6. 检查有没有剩余时间，如果有继续执行下一个工作单元；如果没有，等到下一次主线程空闲时再开始执行下一个工作单元
7. 如果没有下一个工作单元，回到workInProgress tree 根节点，reconciliation节点结束，进入pendingCommit状态。
```
从上述过程可以看出，1-6的实际执行逻辑其实是一个work loop，每执行完一次loop，都要检查有没有剩余时间，进行控制权的交换。
由于每做完一个loop，都要把effect list向上归并到return，因此等到loop结束时，workInProgress根节点上的effect list就是收集到的所有effect。

**构建workInProgress tree的过程就是diff的过程**。
通过requestIdleCallback来调度执行一个任务，每完成一个任务，都回来检查下有没有优先级更高的任务。每完成一个任务，都要把时间控制权交换给主线程，知道下一个requestIdleCallback回调再继续构建workInProgress tree.

PS.Fiber之前的Reconciler被叫做Stack Reconciler，就是因为这些调度上下文信息是由系统堆栈来保存的，以便和Fiber Reconciler区分开。

componentWillMount
componentWillReceiveProps
shouldComponentUpdate
componentWillUpdate

由于Reconciliation阶段是可中断的，因此处在这一阶段的生命周期钩子函数可能被多次调用，存在副作用，从而引起bug。所以版本16之后会逐渐废除掉这些API（不包括scu函数）

但 componentWillReceiveProps 和 componentWillUpdate 在实际业务场景中比较有用，所以16新增了两个API static getDerivedStateFromProps 和 getSnapshotBeforeUpdate 用以解决相同场景下的业务问题。

#### Commit
过程不可中断，一直执行直到更新完成。对dom应用上一个过程得到的patch
```bash
1.处理effect list, 更新dom树，调用组件生命周期函数，更新ref等内部状态
2.Commit结束时，把所有更新commit到DOM树上
```
componentDidMount
componentDidUpdate
componentWillUnmount

### fiber tree && workInProgress tree


## v15 vs v16
源码结构的简单变化：
name | v15 | v16 | 备注
-----|----|------|----- 
tree | vdom tree | fiber tree | vdom是个简单树结构，fiber tree是个单链表
内部方法 | mountComponent/updateComponent | beginWork/completeWork/commitWork
内部组件 | ReactDomComponent/ReactCompositeComponent | ReactDomFiberComponent/ReactFiberClassComponent
核心机制 | Stack Reconciler | Fiber Scheduler


## 参考文档
(https://zhuanlan.zhihu.com/p/37095662）
1.[完全理解Fiber](http://www.ayqy.net/blog/dive-into-react-fiber/)
2.[fiber](http://makersden.io/blog/look-inside-fiber) 如果404，请先访问http://makersden.io
3.[https://github.com/acdlite/react-fiber-architecture](https://github.com/acdlite/react-fiber-architecture)
4.(http://zxc0328.github.io/2017/09/28/react-16-source/)