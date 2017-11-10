---
title: 关于HEXO的配置与使用
date: 2017-09-27 15:55:22
tags:
- HEXO
categories:
- 关于HEXO的二三事
---

# 配置
## 教程
- [NexT主题配置](http://theme-next.iissnan.com/getting-started.html#avatar-setting)
- [HEXO官方文档](https://hexo.io/docs/index.html)

## 配置标签页等
我使用的主题是NexT的主题，其中主题的配置文件需要修改，才能得到对应的界面和效果。<!--more-->

打开`themes\next\_config.yml`，找到`menu`之后，建议复制需要的紧接着下面然后注释掉原本的，然后不需要修改其他的。

标签页、分类页、关于页需要执行以下命令
```bash
hexo new page "tags"
hexo new page "categories"
hexo new page "about"
```
以标签页为例，`index.md`文件需要修改一下
```
---
title: tags
date: 2017-11-11 11:11:11
//新增以下type
type: "tags"
---
```
同理其他页面也新增即可。


# 使用

## 修改主题命令流程

```bash
$ cnpm install hexo-deployer-git --save
$ hexo clean
$ hexo g
$ hexo d
```
