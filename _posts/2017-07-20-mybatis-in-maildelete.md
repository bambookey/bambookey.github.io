---
layout: post
title:  Mybatis in Maildelete
category: tech
---

项目表和查询语句提交DBA，第一次操作还是出现了一些问题，记录一下，积累经验。
犯了一个问题就是动态SQL过多，其中内嵌有各种条件语句，比如一条动态SQL能够应付几乎所有情况下的查询。这也导致DBA看不清我的意图，在最初设计时，应当将SQL与指定的业务相绑定，也有利于维护，其实动态SQL不应当被滥用，而应当被用在恰当的场景。比如查询过程中需要根据domainId(=)或domainIds(WHERE IN)进行查询，此时使用如下情景就比较合适，SQL也比较直观。

```xml
<where>
    <choose>
        <when test="domainId != null">
            domain_id = #{domainId}
        </when>
        <otherwise>
            domain_id IN
            <foreach collection="domainIds" item="domainId" open="(" separator="," close=")">
                #{domainId}
            </foreach>
        </otherwise>
    </choose>
    AND tid = #{tid}
    AND mid = #{mid}
</where>
```
