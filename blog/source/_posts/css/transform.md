---
title: transform对普通元素的副作用
date: 2017-06-06
categories: css
tags: css
---

> 父元素如果有transform属性，会导致子元素的fixed失效（相当于absolute定位）。暂时没有好的解决办法。

移动端如果用到了fixed定位，建议使用position:absolute来实现类似效果。

最近在做一个移动端页面开发的时候遇到了一个fixed的问题。

使用场景：
页面中有一个筛选栏绝对定位，当页面滚动过一定距离，筛选栏固定定位在页面头部。
但是我在开发过程中，发现修改筛选栏的样式，改绝对定位为固定定位，不起作用。
这让我很疑惑。。。

后来发现，这个页面的滚动效果是通过transform实现的。我想到可能跟这个有关系，于是我开始查资料。

>  父元素如果有transform属性，会导致子元素的fixed失效（将fixed定位处理为absolute定位）。

开头提到的问题就是有这个引起的。


最后的解决方案：
页面初始化时，仍然可以将筛选栏放在transform容器中；但是当需要将筛选栏固定在页面头部时，不仅要修改样式，同时还要将dom节点放在body根下，脱离开transform容器。
如果有更好的解决办法，联系我~~thx

延伸
在网上查找相关资料时，发现transform还有许多自己以前不知道的副作用，
http://www.zhangxinxu.com/wordpress/2015/05/css3-transform-affect/

### 总结几点：
1. 父元素如果有transform，会导致子元素的fixed定位失效。
2. transform会提升元素的垂直地位，可以理解为transform将元素的z-index设置变大。
3. 存在关系树 a -> b -> ... -> f -> g,如果g是absolute定位，且a的overflow属性不为visible（设置overflow:hidden;），那么g不会受到overflow属性的影响，溢出部分不会隐藏。
   如果b到f之间的元素使用了transform属性，则元素g受overflow属性影响，溢出部分会隐藏。
   设置a采用transform，g的表现因浏览器而异。
4. 元素采用absolute定位，设置其宽度为100%，则宽度相对于元素最近已定位祖先元素宽度计算。但是如果元素的祖先链上存在设置了transform的元素，那么元素的宽度会相对于transform元素计算。