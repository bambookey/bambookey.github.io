---
layout: post
title: linux性能常用命令
category: tech
---

今晚67服务器上的日志收集启动不起来了，查看启动日志也没有异常，查看启动GC日志有问题，原来是内存不足了。

这时候先是top了一下，但是这个命令除了能大概看一下大盘以外，并不能精确的解决问题。

其实这时候应当使用的命令是ps,当然，不是平时使用的ps -elf

这个命令可以查看当前系统进程性能状态

常见查看CPU，内存性能可以通过以下组合命令

linux下获取占用内存资源最多的进程：

ps aux|head -1
ps aux|grep -v PID|sort -rn -k +3|head

其中第一句主要是为了获取标题（USER PID %CPU %MEM VSZ RSS TTY STAT START TIME COMMAND）。
接下来的grep -v PID是将ps aux命令得到的标题去掉，即grep不包含PID这三个字母组合的行，再将其中结果使用sort排序。
sort -rn -k +3该命令中的-rn的r表示是结果倒序排列，n为以数值大小排序，而-k +3则是针对第3列的内容进行排序，再使用head命令获取默认前10行数据。(其中的|表示管道操作)

简化的方法就是

top + M -> mem排序

top + P -> cpu排序

最后是对top命令表头的一些解释

PID：进程的ID
USER：进程所有者
PR：进程的优先级别，越小越优先被执行
NInice：值
VIRT：进程占用的虚拟内存
RES：进程占用的物理内存
SHR：进程使用的共享内存
S：进程的状态。S表示休眠，R表示正在运行，Z表示僵死状态，N表示该进程优先值为负数
%CPU：进程占用CPU的使用率
%MEM：进程使用的物理内存和总内存的百分比
TIME+：该进程启动后占用的总的CPU时间，即占用CPU使用时间的累加值。
COMMAND：进程启动命令名称
