---
title: Git和TortoiseGit的配置
date: 2017-11-20 22:36:48
tags: [Git, tortoisegit]
categories: 白科技
---
重装了n次系统，每次都要安装git和tortoisegit两个东西，故写点配置总结。
<!--more-->
# Git配置
> $ git config --global user.name "lvqiliya"

> $ git config --global user.email lvqiliya@163.com

# ssh配置
1. 生成ssh
> $ ssh-keygen -t rsa -b 4096 -C "lvqiliya@163.com"

2. 验证ssh是否生效
> $ ssh -T git@github.com

# TortoiseGit配置
1. 打开PuTTYgen
Firstly, loading an existing private kiy file,then save private key to .ssh
![PuTTYgen设置](/images/git config/PuTTYgen.png)

2. 打开Pageant
Add Key the private key
![PuTTYgen设置](/images/git config/Pageant.png)

搞定
