---
title: Windows系统优化
date: 2019-02-23 13:56:39
tags: [Windows]
categories: 白科技
---
本文是关于Windows系统的基本配置
<!--more-->
# 新建照片查看器
1. 进入注册表
```
计算机\HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Photo Viewer\Capabilities\FileAssociations
```
2. 新建字符串
```
.jpg PhotoViewer.FileAssoc.Tiff
.png PhotoViewer.FileAssoc.Tiff
```