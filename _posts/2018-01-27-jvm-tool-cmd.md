---
layout: post
title: JVM常用命令
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


一般来讲，排查问题从三个角度出发：CPU，内存（物理内存，虚拟内存），IO
先不考虑java实例的情况，从Linux系统层级出发，常用的命令就包括：
top：查看全局情况
vmstat：查看pageIn/Out，Swap情况，IO，cpu都可以查看，这个命令很好用，不仅仅是用top
free：可以查看内存使用情况，不过感觉如果使用了vmstat就没有什么使用的必要了

这里记录几个概念：
负载：分配到CPU的进程数量，在vmstat中的r可以看到，如果数量高于CPU的核心数量就比较危险了

从系统层面定位完这些问题后，如果是java应用服务器，就可以重点使用java命令对其进行排查了：
Mem:jmap, pmap
CPU:jstack
jvmstat:jstat
