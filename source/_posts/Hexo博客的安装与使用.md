---
title: Hexo博客的安装与使用
date: 2017-09-27 15:55:22
tags:
- HEXO
categories:
- 关于HEXO的二三事
---
HEXO的安装和配置指南，包括从零搭建、多终端编辑博客等
<!--more-->
# Chapter01 配置
## 教程
- [NexT主题配置](http://theme-next.iissnan.com/getting-started.html#avatar-setting)
- [HEXO官方文档](https://hexo.io/docs/index.html)
- 国内原因，用`npm install -g cnpm --registry=https://registry.npm.taobao.org`,将`npm`改用淘宝源的`cnpm`（cmd中执行即可）
- 接下来需要执行`$ cnpm install -g hexo-cli`来安装hexo

## 配置标签页等
我使用的主题是NexT的主题，其中主题的配置文件需要修改，才能得到对应的界面和效果。

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
# Chapter02 使用
## 修改主题命令流程

```bash
$ cnpm install hexo-deployer-git --save
$ hexo clean
$ hexo g
$ hexo d
```
# Chapter03 多台终端
由于不可能只在一个终端写博客，大部分情况都是属于工作电脑和私人电脑的两台电脑的情况，所以本小节针对多台终端如何轻松还原博客编写环境
## 基本思路是通过管理两个项目
- `git@github.com:you_name/you_name.github.io.git` 用户管理hexo生成的博客代码
- `git@github.com:you_name/hexo.git` 用于管理博客源代码

## 搭建流程
1. 创建项目*you_name.github.io*和*hexo*；
2. D/E/F盘Git Bash中执行`git clone git@github.com:you_name/hexo.git`；
3. 进入hexo文件夹，进入Git Bash按顺序执行

```bash
$ cnpm install hexo
$ hexo init
$ cnpm install
$ cnpm install hexo-deployer-git
```
4. 配置好_config.yml中的信息
5. 利用TortoiseGit把hexo提交到远程仓库
6. 以后每一次修改文件，都需要push到hexo仓库的master分支。

## 本地资料丢失
1. D/E/F盘Git Bash中执行`git clone git@github.com:you_name/hexo.git`；
2. 进入hexo文件夹，进入Git Bash按顺序执行

```bash
$ cnpm install hexo
$ cnpm install
$ cnpm install hexo-deployer-git
```
# Chapter04 升级

进入hexo文件夹，进入Git Bash按顺序执行

1. `npm-check` 检查更新

```bash
$ npm install -g npm-check
$ npm-check
```

2. `npm-upgrade` 更新

```bash
$ npm install -g npm-upgrade
$ npm-upgrade
```

3. 更新全局包

```bash
$ npm update <name> -g
或者
$ npm update -g
```

4. 更新生产环境依赖包

```bash
$ npm update <name> -g
或者
$ npm update -g
```

5. 安装 swig

```bash
$ npm i hexo-renderer-swig
```