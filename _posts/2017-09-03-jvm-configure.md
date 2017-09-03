---
layout: post
title:  JVM遇到的问题
category: tech
---

周五定时任务上线了，主要是负责三个方面的业务：

* 邮件群发
* 账号批量导入
* 反垃圾通知信定时提醒

周五晚上一切正常，周六有用户使用账号批量导入功能，发现系统没有反馈。没有反馈的实际原因是因为调度线程此时已经OOM了。而且是PermGen的OOM，后来通过运维平台查了一下，PermGen的默认最大值为82M。

项目采用springboot打包再解压的方式运行，直接运行boot中的JarLauncher文件，这样使得配置文件可以在线上进行修改，今天将MaxPermGen调大，并且设置了堆内存，再观察一次看看情况如何。

```
nohup java -Xmx$MAX_HEAPSIZE -XX:PermSize=$PERMSIZE -XX:MaxPermSize=$MAX_PERMSIZE org.springframework.boot.loader.JarLauncher &
```
