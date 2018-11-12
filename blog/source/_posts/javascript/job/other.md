---
title: 工作中奇奇怪怪
date: 2017-05-21
categories: work
tags: 
---

1. 关于checkbox元素，不同浏览器下表现不一致

设置checked=“false”和checked=“0“,checked=“null"只要设置了checked(无论其值是什么)的input都会默认选中状态。
vue1下，设置checked为0或"0"，chrome下表现为未选中；safari表现为选中。如果设为null或者false时，两者均表现未选中

chrome/safari测试通过js修改checked属性
设置checked为false,0,null,NaN可以使其未选中
设置为”false”,”0”,undefined会选中

一个元素如果是readonly，jQuery下其val和prop仍然可以进行编辑，而attr不行

2. react的受控组件(setState)在ios11中文输入法下表现异常，比如想输入中国共产党，输入拼音zongguo，再输入g时可能会得到zongguogzongguo.改由非受控组件可以解决这个问题