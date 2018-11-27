---
title: passive
date: 2018-11-01
categories: fe
tags: js
---

## 背景
浏览器多进程结构

Browser Process 浏览器底层操作，文件请求，网络请求

Renderer Process 渲染进程，负责一个tab内的所有工作

Plugin Process 管理插件，如flash

GPU Process


Render Process 多线程：

1. 主线程/内核线程 Main thread   js执行，样式计算，布局，绘制

2. 工作线程 Worker thread

3. 合成器线程 Compositor thread 合成图层

4. 光栅线程 Raster thread

浏览器工作流 ： 解析文档 -> 构建dom -> 样式计算 -> 布局 -> 绘制 -> 合成 -> 渲染


## passive

部分用户手势输入事件（触摸和滚动，如touch相关事件)可以不经内核线程, 由合成器线程直接处理。比如一个有滚动条的页面，即使有死循环，页面仍然可以快速滑动。
但手势输入事件有可能需要内核线程参与，比如如果事件监听器内部定义了preventDefault, 浏览器就不应该产生手势输入事件。
而浏览器只有等到内核线程执行了js代码，才知道是否阻止了默认行为。
由于要等到内核线程执行代码才能判断，这就导致用户的手势事件无法快速产生，无法快速滑动，导致页面卡顿。
通过指定passive: true, 浏览器就知道监听器内没有调用preventDefault函数来阻止掉默认行为，就可以快速响应手势输入事件，从而使页面“变快”

总结：passive用来决定监听器内执行e.preventDefault()是否有效


## 参考
https://mp.weixin.qq.com/s/cb8VJOmAB1Yrv-ct4jJ3JQ