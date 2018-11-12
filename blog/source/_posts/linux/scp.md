---
title: 从远程服务器上下载日志文件
date: 2018-10-07
categories: linux
tags:
---

下载ip为10.86.11.12服务器上目录为 /home/q/www/bnb/logs/monitor.log 文件到本地/Users/workspace目录下

### sz命令
windows比较适合，对mac不友好

### scp命令
```bash
scp <remote_username>@<remote_ip>:<remote_path> <local_path>

eg. scp pengbo.guo@10.86.11.12:/home/q/www/bnb/logs/monitor.log /Users/workspace
```
注意：如果线上机器是本地经跳板机登录的，则需要先将文件下载到跳板机，再从跳板机下载到本地。

### 远程搭建一个简单http服务（推荐）
在服务器上要下载文件的同目录下，启动一个简单的http服务器。
本地浏览器直接访问服务器ip加端口号，即可看到服务器上当前目录的所有文件，然后点击下载

```bash
启动http服务 python -m SimpleHTTPServer 8090
访问：10.86.11.12:8090
```