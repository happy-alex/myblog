---
title: 清理磁盘
date: 2018-10-07
categories: linux
tags:
---

最近有次应用发布，部署失败了，查原因是beta机的磁盘满了，这里mark下如何找到过大size的文件

1.查看当前文件系统磁盘占用情况 
```bash
df -h
```

2.查看当前目录下所有文件夹的size 
```bash
du -sh *
```

