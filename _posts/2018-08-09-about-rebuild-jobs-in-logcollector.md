---
layout: post
title: 关于日志收集系统最后一次登录相关任务的重构
category: tech
---

日志收集系统有一个异步计算用户最近登录记录的功能

实现原理大致是：将当前活动状态下的域名丢入线程池，线程池处理每个域名，标记每个域名的最后检测时间，
获取检测时间之前的所有登录记录，去重，取每个账号最后一次登录记录。

自助查询就可以不经计算，直接从最后一次登录数据库中获取记录了。

但随着时间推移，慢慢来了新需求。

1. 推送最后一次登录记录

2. 推送可能是异常IP的登录记录

这两个推送依赖最后一次登录的数据，表面看起来推送任务可以做成独立的，但从本质上来讲，推送任务
依赖于最后一次登录的计算任务，即如果没有了计算任务，推送可以进行，但将会变得无意义。之前的配置文件类似下面

```
<lastLoginLogCalculator>
    <calConf1></calConf1>
    <calConf2></calConf2>
    <pushLastLogin></pushLastLogin>
</lastLoginLogCalculator>
<abnormalLoginLogPusher>
</abnormalLoginLogPusher>
```

这样的抽象明显是不合理的。
我目前的经验来看，首先两个推送行为应该是并列关系，其次两个推送行为对最后一次登录行为存在依赖关系，
而且两个推送任务应当互不影响且具有独立的开关功能

最终抽象出来的结果是

```
<lastLoginLogCalculator>
    <calConf1></calConf1>
    <calConf2></calConf2>
    <push-switch/>
    <pushLastLogin>
        <switch/>
        <url/>
    </pushLastLogin>
    <pushAbnormalLoginLog>
        <switch/>
        <url/>
    </pushAbnormalLoginLog>
</lastLoginLogCalculator>
```

这仅仅是一个表层的抽象，实际上既然有一个总体依赖关系，二者的初始化保持依赖关系即可。感觉这类问题的解决
使用消息队列是很好的，获取每个域下面的最近登录列表，广播到两个exchange中去，分别对两种情况进行匀速消费即判别和推送，
不过因为种种原因，现在还是采用线程池的方式，通过future控制线程池的任务数，并对每个域名进行标记，防止同一时刻的重复计算，
但这个代码不符合分布式的思想，所以感觉除了做到解耦合、控制计算速率、监控刷新效率外，也不能做到什么了，不过写起来很难受，记录一下。
