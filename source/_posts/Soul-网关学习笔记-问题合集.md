---
title: Soul 网关学习笔记-问题合集
abbrlink: d0eef086
date: 2021-01-17 11:51:00
tags: soul
categories: 源码学习
---
记录问题
<!--more-->
## install 报错

笔者下载了 `Maven Helper` 插件，自定义命令跳过测试即可。

```mvn
clean install -DskipTests
```

## soul-examples 模块下子模块启动报错

一般来说针对 soul-examples 模块执行 install 命令后即可成功启动。
