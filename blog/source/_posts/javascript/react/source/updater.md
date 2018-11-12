---
title: updater
date: 2018-09-27
categories: js
tags: react
---

以下翻译自：http://makersden.io/blog/look-inside-fiber

## updater
每个React 组件实例都有一个updater，它在组件reconcilier时被注入，负责组件和react核心间的通信。

updater主要有4种作用:
1. 在树中查找组件对应的fiber实例
```javascript
get: function(key) {
    return key._reactInternalInstance;
}
```
2. 向调度器查询该fiber的优先级

3. 将更新变化push进fiber的更新队列
这个逻辑与react首次渲染时的逻辑很像，唯一区别是setState不是一个优先级最高的更新，新的更新将根据优先级添加到fiber的更新队列中。

4. 使用确定的优先级调度更新工作
Fiber会从正在调度更新的节点处开始遍历返回节点，直到它到达根节点。(其FiberNode.return为null)。遍历过程中，如果每个节点的pendingWorkPriority低于当前跟定的级别，就会把pendingWorkPriority提升，这种情况的pendingWorkPriority为LowPriority低优先级。遍历到达根节点后，Fiber开始更新工作，低优先级的任务使用requestIdleCallback进行调度，走到了setState流程的末尾。

## render/reconciliation

这个阶段会克隆构建一个fiber tree的副本--workInProgress。构建过程中如果有优先级较高的任务（SynchronousPriority）出现，构建工作会停下来。
完成调度的函数时performWork。其目的是捕获和处理在reconciliation过程和构建tree时期内发生的错误。它通过执行一个while循环来实现，在这个循环中，workLoop函数一直被调用，直到出现错误是退出循环。

workLoop找到fiber节点，继续工作。包括为找到最高优先级的根fiber而查找scheduledRoot list。

然后，它检查是否存在截止时间以及当前工作优先级是否低于TaskPriority，这意味着该工作的执行必须遵守当前帧的时间限制。如果没有deadline或优先级足够高，那么它会运行一个正常的while循环，直到它找不到更多的工作要做。如果存在deadline（或优先级较低），则它会运行一个while循环，直到有工作要完成且deadline尚未到期。 只要条件成立，就fiber而言，我们处于延迟批次中。 （同样，任何铃声响起？batchedUpdates在我描述渲染时的方式回来？）这里的逻辑决定了是继续处理fiber task还是将task推迟到下一个渲染帧执行，以避免在UI中引起抖动。。

执行当前工作单元的过程包括两个阶段：beginWork和completeWork。

未完待续...