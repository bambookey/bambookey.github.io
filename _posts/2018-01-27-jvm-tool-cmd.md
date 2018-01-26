---
layout: post
title: JVMc常用命令
category: tech
---

jps
-l 输出主类全名
-m main参数
-v vm参数

pmap -进程号
第一列，内存块起始地址
第二列，占用内存大小
第三列，内存权限
第四列，内存名称，anon表示动态分配的内存，stack表示栈内存


jmap -histo 进程号
定位内存问题，查看有多少实例
B  byte
C  char
D  double
F  float
I  int
J  long
Z  boolean

dump : 生成堆转储快照
finalizerinfo : 显示在F-Queue队列等待Finalizer线程执行finalizer方法的对象
heap : 显示Java堆详细信息
histo : 显示堆中对象的统计信息
permstat : to print permanent generation statistics
F : 当-dump没有响应时，强制生成dump快照
加:live 显示活着的对象



jstack
定位CPU问题
deadlock,wait,loop

jstat
虚拟机状态，类装载、内存、垃圾收集、JIT编译等问题

类装载
jstat -class 11589
Loaded  Bytes  Unloaded  Bytes     Time
  9544 18286.7        0     0.0      11.56

jstat -gc 12538 5000
