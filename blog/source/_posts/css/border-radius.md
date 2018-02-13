---
title: border-radius的实现原理
date: 2017-04-12 19:35:31
categories: css
tags: css
---

**border-radius的实现原理就是用一个椭圆/圆去切割元素的每个角**

border-radius的原始写法是border-radius: 25px/50px;
这代表这个元素的左上，右上，右下，坐下4个角将依次被一个水平半径为25px，垂直半径为50px的椭圆进行切割，从而得到4个角


举个栗子
![](http://img.blog.csdn.net/20161207005704619?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveGlhb2VybWluZ24=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)

上面这幅图表示：用一个水平宽为200px，垂直高为100px的椭圆贴着左上角，进行裁剪，左上角多出来（蓝色线区域）的地方就是元素被切去的区域，剩下的就是最终元素表现出来的形状

*说到了这里，发现了一个问题。当设置border-radius为100%时，也能够将一个正方形切割为圆形，这是为什么？*

W3C 对于border-radius的重合曲线有这样的规范：
> **如果两个相邻的角的半径和超过了对应的盒子的边的长度，那么浏览器要重新计算保证它们不会重合。**

假设一个正方形，如果正方形左上角圆角半径被设置成了100%，那么圆角就会从这个正方形左下角跨到右上角，相当于把圆角半径设置成方形的大小。如果同时把右上角的圆角半径也设置成为100%，则两个相邻圆角合起来就有200%。这种情况自然是不允许出现的，所以浏览器就会重新就算，匀出空间给右边的圆角，同时缩放两个圆角的半径直到它们可以刚好符合这个方形，所以半径就变成了50%。可以粗暴的理解为：超过50%后的效果基本都是一样，都会被重新计算为50%。




常见写法
``` bash
    // 等同于 border-radius: 50px/50px
    border-radius: 50px;

    // 切割圆弧半径设置为元素宽高的一半，百分比是相对于元素的宽高而言
    border-radius: 50%/50%;

    // 注意，用这种方式分开设置角时要先设置所有角的水平半径，再设置所有角的垂直半径 
    border-radius: 25% 0 0 0/50% 0 0 0;
```


相关链接
1. [http://blog.csdn.net/xiaoermingn/article/details/53497607](http://blog.csdn.net/xiaoermingn/article/details/53497607) 讲的很好，赞
2. [https://zhuanlan.zhihu.com/p/20128284](https://zhuanlan.zhihu.com/p/20128284)
