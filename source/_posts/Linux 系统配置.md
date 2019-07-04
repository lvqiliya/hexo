---
title: Linux 系统配置
date: 2019-02-23 13:30:51
tags: [Linux]
categories: 白科技
---
本文是关于Ubuntu或CentOS桌面系统的基本配置，以及开发环境的配置。
<!--more-->

## Ubuntu 修改镜像源

```shell
# 备份文件
sudo cp sources.list sources.list.bak

# 修改文件
sudo vi sources.list

# 添加以下内容到文件最前面
deb http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-security main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-updates main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-proposed main restricted universe multiverse
deb-src http://mirrors.aliyun.com/ubuntu/ bionic-backports main restricted universe multiverse

# 更新源
sudo apt-get update

# 更新已安装包
sudo apt-get upgrade
```

## 杂项

- 修改默认文件夹名为英文

```shell
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN
```

- 修改主题图标字体

```shell
mv Sierra-light /usr/share/themes/
mv macOS11 /usr/share/icons/`
mv *.ttf /usr/share/fonts/chinese/
chmod 755 *.ttf
mkfontscale
mkfontdir
fc-cache
```

## Docker

本配置仅适用于 Cent OS ，更多具体操作可详见 [Docker中文文档](https://docs.docker-cn.com/)

- 卸载旧版本

```shell
$ sudo yum remove docker \
                  docker-common \
                  docker-selinux \
                  docker-engine
```

- 设置镜像仓库

```shell
# 安装所需的软件包。
$sudo yum install -y yum-utils device-mapper-persistent-data lvm2

# 使用下列命令设置 stable 镜像仓库。
$ sudo yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo
```

- 安装步骤

```shell
# 更新 `yum` 软件包索引
sudo yum makecache fast

# 安装最新版本的 Docker CE
sudo yum install docker-ce

# 启动 Docker
sudo systemctl start docker

# 验证 Docker 是否安装成功
sudo docker run hello-world

# 适用于 Linux 的安装后步骤
# 以非 root 用户身份管理 Docker，完成后注销并重新登录
sudo groupadd docker
sudo usermod -aG docker qly

# 将 Docker 配置为在启动时启动
sudo systemctl enable docker
```

## Zsh

安装 zsh 并使用 oh-my-zsh 配置

```shell
# 下载 zsh
sudo apt-get install -y zsh

# 下载 oh-my-zsh，下列任一方式即可
# via curl
sh -c "$(curl -fsSL https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
# via wget
sh -c "$(wget https://raw.github.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"

# 为 root 用户修改默认 shell 为 zsh
chsh -s /bin/zsh root
# 为当前用户修改默认 shell 为 zsh
chsh -s /bin/zsh
# or
chsh -s `which zsh`
```

wsl-terminal 的使用 zsh，需要修改 `etc/wsl-terminal.conf`，使 shell=/bin/zsh 即可。
