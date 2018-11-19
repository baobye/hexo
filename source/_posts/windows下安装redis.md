---
titile: windows下安装redis
---
## windows下安装redis

github下载地址：https://github.com/MSOpenTech/redis/tags

<!--more-->

## 下载

* 这里下载的是Redis-x64-3.2.100版本，我的电脑是win7 64位，所以下载64位版本的，在运行中输入cmd，然后把目录指向解压的Redis目录。
* 启动命令
``` bash
redis-server redis.windows.conf
```

## 出现的问题
[6644] 02 Apr 23:11:58.976 # Creating Server TCP listening socket *:6379: bind: No such file or directory

``` bash
redis-cli.exe
shutdown
exit
redis-server.exe redis.windows.conf
```