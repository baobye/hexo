---
title: linux服务器修改ssh默认22端口方法
date: 2018-07-24 21:33:24
tags:
---


* 登录服务器，打开sshd_config文件

```bash
vim /etc/ssh/sshd_config

```
* 找到#Port 22，默认是注释掉的，先把前面的#号去掉，再插入一行设置成你想要的端口号，注意不要跟现有端口号重复

```bash
Port 22
Port 26580

```

* 保存后退出，执行重启命令
```bash
service sshd restart
```