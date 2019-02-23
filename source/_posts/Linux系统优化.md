---
title: Linux系统优化
date: 2019-02-23 13:30:51
tags: [Linux]
categories: 白科技
---
本文是关于Ubuntu或CentOS桌面系统的基本配置
<!--more-->
# 修改默认文件夹名为英文
```
export LANG=en_US
xdg-user-dirs-gtk-update
export LANG=zh_CN
```

# 修改主题图标字体
```
mv Sierra-light /usr/share/themes/

mv macOS11 /usr/share/icons/`

mv *.ttf /usr/share/fonts/chinese/
chmod 755 *.ttf
mkfontscale
mkfontdir
fc-cache
```