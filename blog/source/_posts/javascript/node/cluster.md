---
title: cluster
date: 2017-12-10
categories: node
tags: node
---

cluster模块 = net + child_process 组合应用

## 进程管理
最初的node多进程模型/抢占式：

**发送fd给所有worker: master通过执行cluster.fork()，fork出多个worker进程，worker执行listen()。进程间通过传递文件描述符fd完成通信，由于多个worker间存在竞争问题导致惊群现象。**

改进/Round-Robin：

**不直接将fd分发给所有worker：master创建socket，绑定端口，执行listen()，通过round-robin等算法挑选一个worker，将拿到的链接客户端句柄fd直接交给该子进程。从而避免惊群，实现负载均衡。**


进程管理要考虑的几个关键点：

负载均衡    -->     如何利用多核cpu: 抢占式，Round-Robin算法
进程守护    -->     reload，子进程exit或disconnect时，父进程重新fork一个子进程
优雅退出    -->     捕获到uncaughtException错误，通知master进行fork，断开已有链接且不接受新链接，处理完已有请求，断开数据库等连接，一段时间后主动kill自己


## 进程通信
```bash
实现进程通信    -->   IPC通道
                    |
                    |--> 发送fd, eg.master分发客户端链接给worker
                    |
                    |--> 不发送fd，eg.简单字符串传递，双向通信
```
具体实现：fork()时，父进程先构造一个IPC通道并监听，父进程创建子进程，并告诉子进程通道的fd，子进程启动时根据fd去连接这个已存在的通道，此后就可以双向通信了


## 扩展阅读

[cluster源码实现](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)
[当我们谈论 cluster 时我们在谈论什么](http://taobaofed.org/blog/2015/11/10/nodejs-cluster-2/)
