---
title: 联合主键下的mapper文件对数据库的批量更新
date: 2020-05-11 23:02:42
categories: 数据库
tags: [工作中的问题,数据库]
---

​	工作中使用到了联合主键，业务需求要在一系列的操作以后进行更新的操作，传统形式下的批量更新就有点不好用了。

工作中的业务当然不能拿出来说明了，随便建立一张表，说明一下问题。

表结构如下：

![](1.PNG)

内部数据如下：

![](2.PNG)

主要任务是要吧表中的 **AMOUNT** 字段修改批量修改，那一套的请求、分层、数据库、和mapper生成就省略了。随处都可以找到的。

先上mapper批量更新的语句：

``` xml
<update id="updateByBatch" parameterType="java.util.List">
    update dcwt_test
    set AMOUNT =
    CASE
    <foreach collection="list" item="item" index="index"> // 此处看一下应该能理解的
      when PARTID=#{item.partid} AND BRESQ=#{item.bresq}
      then #{item.amount}
    </foreach>
    END
    where
    <foreach collection="list" item="item" index="index"> // 主要就是字符串拼接 OR
      (PARTID=#{item.partid} AND BRESQ=#{item.bresq}) 
      <if test="index!=list.size-1"> // 还真没想到这玩意还能这么写的
        OR
      </if>
    </foreach>
</update>
```

更新结果：

![](3.PNG)

