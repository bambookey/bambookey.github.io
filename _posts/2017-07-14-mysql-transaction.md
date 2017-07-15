---
layout: post
title:  MySQL事务操作
category: tech
---

本周在使用MySQL过程中遇到了两个问题，解决方式都是之前没有见过的，最近刚好在看《高性能MySQL》，这些知识真的需要补充一下了。

**表被锁死**
问题描述：由于要对表增加一个字段，索引做一定的调整，操作是直接在NaviCat上进行的，执行过后GUI卡死，之后对该数据库的任何操作均超时，没有任何反应。
解决方法：问题原因在于在修改时刚好有人使用了该测试数据库，使用命令

```
show processlist;
```
找到其中阻塞的查询，kill掉即可。

**事务提交导致的阻塞**
问题描述：插入会导致UNIQ索引冲突的数据且两次事务开启后均未提交，导致错误 **ERROR 1205 (HY000): Lock wait timeout exceeded; try restarting transaction**
解决方法：

1. 查看数据库事务隔离级别
select @@tx_isolation;

2. 查看数据库当前线程情况
show processlist;

3. 查看INNODB的事务表INNODB_TRX，找到其中锁定的事务线程
SELECT * FROM information_schema.INNODB_TRX\G;
找到字段trx_mysql_thread_id
kill掉，解决
