---
layout: post
title: 在双硬盘上安装Win10出现的问题
categories: Softwares
description: 在双硬盘上安装Win10出现的问题
keywords: Windows, Windows10, Install, Problems
---



> 原创
>
> 垃圾Thinkpad E480

垃圾Thinkpad E480又“罢工”（这次是蓝屏，上次是直接烧主板）。

吐槽完毕。开始重装系统。从学校网上下载win10，制作系统盘。

然后从U盘启动安装，可是，可是。。。居然具体安装时候出现以下问题：

1.“选定要安装的磁盘上的分区不在推荐次序中”

解决方法：将要安装的磁盘前面的内容删除。使得系统安装去为磁盘部分1。

```
|---------|-------------------------------------------------|------------------|
  未分区                   系统安装区（磁盘部分1）                   未分区
```

2.“Windows 无法对计算机进行启动到下一个安装阶段的准备”

本笔记本有两个磁盘，将不需要安装系统的磁盘扣下来，再次安装系统。系统安装完毕后，再安装上去。