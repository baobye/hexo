---
title: linux下webpack不自动重启
date: 2018-11-22 09:45:29
tags:
---

# linux下webpack-dev-server修改文件后不自动重启

## 1、Debian,RedHat 系统

``` bash
echo fs.inotify.max_user_watches=524288 | sudo tee -a /etc/sysctl.conf && sudo sysctl -p

```

## 2、ArchLinux 系统

``` bash
echo fs.inotify.max_user_watches=524288 | sudo tee /etc/sysctl.d/40-max-user-watches.conf && sudo sysctl --system

```

参考网站：https://blog.csdn.net/xs20691718/article/details/81146985
https://github.com/guard/listen/wiki/Increasing-the-amount-of-inotify-watchers