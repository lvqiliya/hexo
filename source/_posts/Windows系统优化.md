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

# U盘无法识别解决方法
```
在打开的cmd运行窗口里输入
diskpart
回车，然后输入
list disk
回车，查看目前电脑上所有的磁盘，如果U盘为2，那么输入以下命令将焦点指定到该磁盘
select disk 1
回车，然后再输入
clean
最后启动设备管理器进行格式化
```
