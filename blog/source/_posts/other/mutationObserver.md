---
title: MutationObserver
date: 2017-05-19
categories: other
tags:
---

### MutationOberver 
可以检测 DOM 的变化。

```javascript

// dom批量变化之后才会触发回调
var observer = new MutationObserver(mutations => {
    mutations.forEach(item => {

        // 每一条变化
        console.log(item);
    });
});

// 开始监听
observer.observe(document.body, {

    // 指定要监听的属性变化
    attributes: true,
    characterData: true,
    childList: true,
    subtree: true,
    attributeOldValue: true,
    characterDataOldValue: true
});

// 停止监听
observer.disconnect();

// 在回调执行前返回最新的DOM变化
observer.takeRecords();
```


### 替代方案
1. Mutation events （每次dom变化都会触发，性能太差，已被废弃）
2. 轮询/脏检查
3. 动画 （dom变化可能会触发动画执行，监听动画事件从而捕捉到dom变化的时间点）