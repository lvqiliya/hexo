---
title: Linux 开发环境配置
date: 2019-02-28 16:23:21
tags: [Linux, Docker]
categories: 白科技
---
本文是关于 Linux 系统开发环境的基本配置
<!--more-->
# Docker
本配置仅适用于 Cent OS ，更多具体操作可详见 [Docker中文文档](https://docs.docker-cn.com/)
## 安装前置
### 卸载旧版本
```shell
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```
## 安装 Docker CE
### 设置镜像仓库
1. 安装所需的软件包。
```shell
$sudo yum install -y yum-utils device-mapper-persistent-data lvm2
```
2. 使用下列命令设置 stable 镜像仓库。
```shell
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```
### 安装步骤
1. 更新 `yum` 软件包索引。
```shell
$ sudo yum makecache fast
```
2. 安装最新版本的 Docker CE。
```shell
$ sudo yum install docker-ce
```
3. 启动 Docker。
```shell
$ sudo systemctl start docker
```
4. 验证 Docker 是否安装成功。
```shell
$ sudo docker run hello-world
```
### 适用于 Linux 的安装后步骤
1. 以非 root 用户身份管理 Docker，完成后注销并重新登录。
```shell
$ sudo groupadd docker
$ sudo usermod -aG docker qly
```
2. 将 Docker 配置为在启动时启动。
```shell
$ sudo systemctl enable docker
```
