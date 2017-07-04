---
layout: post
title:  RPC调用和DB操作混合下的事务操作
category: tech
---

在邮件删除业务中遇到这么一种情形：对批量邮件进行删除，删除操作分为两部分，一部分是调用邮件系统第三方接口对邮件进行删除；另一部分是在数据库中记录删除的记录。这两个操作没有直接的先后顺序，但在真正业务发生过程中需要对其进行考虑。

由于涉及到对来自两个不同源的方法调用，则需要保证该操作的原子性。

**1 全部完成还是部分完成**

这个取决于具体业务场景，内因在于，子任务之间是否具有关联性。在删除邮件场景下，子任务之间均独立，则应当允许部分完成以最大满足用户需求，遇到失败时。若不同任务之间存在依赖关系，则不允许部分完成。

**2 如何保证对一条记录操作的原子性**

RPC调用受被调用方和网络环境的影响较大，相比于DB操作更容易出问题，应当作为着重考虑的地方。经过考虑，总结出以下方法：

- RPC-DB-RPC

- 1)RPC调用：失败则结束后续动作
- 2)DB操作：失败则执行步骤3)
- 3)RPC反向操作回滚：

由于这种混合操作的瓶颈是在RPC调用上，这种方法将希望寄托于在DB操作出现异常时RPC调用正常，不是一种很好的方法，但这是我们最初能想到的。

- DB-RPC-COMMIT

- 1)DB操作
- 2)RPC调用
- 3)DB提交

这是种经典的方法，思路来自数据库的事务操作，将RPC调用放在数据库的事务操作中，确保RPC成功后再提交事务，而在提交之前，已经确保了数据库的正常连接和DB操作。

- 在Spring-Mybatis下的操作

- 1)建立@Transactional Service方法
- 2)DB操作
- 3)RPC调用

这是在本项目中对方法2)的具体实践，由于在sping-mybatis中更推荐注解式的事务操作，因此采用此法。
贴一段代码记录一下：

```java
@Override
public int removeBatchByMids(List<MailDelRecord> records, String createId) throws ServiceException {
    Date nowDate = new Date();
    int sucCnt = 0;
    for (MailDelRecord record: records) {
        record.setCreateTime(nowDate);
        record.setUpdateTime(nowDate);
        try {
            removeSingle(record, createId);
            sucCnt++;
        } catch (ApiException e) {
            LOG.error("hmail delete failed one record. record:{}", JSON.toJSONString(record), e);
            continue;
        }
    }

    if (sucCnt != records.size()) {
        LOG.warn("mail delete partly. sucCnt:{}, recordSize:{}", sucCnt, records.size());
        throw new ServiceException("MAIL.DELETE.PARTLY");
    }
    return sucCnt;
}

@Override
@Transactional
public void removeSingle(MailDelRecord record, String createId) throws ApiException, DaoException {
    record.setStatus(RecordStatus.DELETE.getCode());
    record.setUpdateId(createId);
    record.setCreateId(createId);
    mailDelDao.insert(record);
    mailDeleteApi.remove(record.getMailTo(), record.getMid());
}
```
