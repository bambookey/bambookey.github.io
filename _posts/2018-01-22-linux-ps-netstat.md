---
layout: post
title: Linux 下关于进程和端口的一些命令
category: tech
---

经常遇到一些需要查看进程和端口使用情况的命令，多数是需要监控当前服务的运行情况。

查看指定进程，最常用的就是：
```
ps -ef | grep tomcat
```

最近在使用SpringBoot中需要按端口获取进程，常用的有：
```
netstat -nltp | grep 19090
netstat -anp | grep 19090
```
和
```
lsof -i:19090
```
