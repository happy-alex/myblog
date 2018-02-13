---
title: shadowDom
date: 2017-05-19
categories: html
tags:
---

html5中有一个新标签：
```javascript
    <template>123</template>
```
它的特点包括：
1.会隐藏内部节点，比如<img/>不会显示，也不会请求图片资源。
2.要想获取template内部的文档片段documentfragment，，需要使用template.content属性（childNodes为[]），后续可以通过document.importNode(node,deep)方法克隆节点。
3.其默认display值为none，所以会隐藏显示内部元素。修改template的display属性为block，设置样式，比如宽高，背景样式均可以生效；
但其内部节点的样式也不会生效。比如文字123不会显示。


### 参考资料 
[ShadowDom](https://aotu.io/notes/2016/06/24/Shadow-DOM/)


TODO...
[webcomponents]