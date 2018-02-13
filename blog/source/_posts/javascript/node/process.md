---
title: 守护进程
date: 2017-08-17 21:25
categories: node
tags: js node
---

区分3个概念：前台进程，后台进程，守护进程

前台进程: node index.js
后台进程: node index.js &
守护进程: nohup node index.js & 

### 前台进程和后台进程
二者的本质不同在于**后台进程没有继承标准输入**，因此当开启一个后台进程后，用户还可以在控制终端上输入其他命令。
后台进程继承了当前session的标准输出和标准错误，所以后台进程的所有输出仍然可以在当前session中显示

### 守护进程不等于后台进程。
node index.js & 产生了一个后台进程。
但这个后台进程并不是一个守护进程，因为它没有脱离终端，如果关闭终端，这个进程还是会被杀死。


### 守护进程
守护进程是在后台运行不受终端控制（输入，输出）的进程。一般的网络服务都是以守护进程的方式运行。
node项目常常需要用守护进程的方式启动服务，常见3种方式：nohup, forever, pm2。
```javascript
nohup node index.js &    // nohup不能自动把前台进程变为后台进程，必须加&
forever start index.js
pm2 start index.js
```
其中forever和pm2是node应用专有的启动工具，两者都能保证当进程意外退出时立即重启，但pm2提供比forever更强大的功能，日志收集和监控。

sudo service xx start/stop 启动/停止执行脚本

### 参考
[http://www.ruanyifeng.com/blog/2016/02/linux-daemon.html]