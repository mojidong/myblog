---
title: "mybatis批处理(batch)"
date: 2019-03-24T06:54:17+08:00
lastmod: 2019-03-24T06:54:17+08:00
description: "mybatis批处理(batch)"
categories: ["java", "mybatis", "ibatis"]
tags: ["java", "mybatis", "ibatis"]
---

最近一个老的项目从`ibatis`升级到了`mybatis`，之前的批处理方法需要在`mybatis`中重新实现。
实际实现过程中发现并没有如`ibatis`类似的`api`需要一些特殊处理。下面来讲讲`mybatis`中实现批处理的方式。

## sql处理方式
sql的处理方式批较简单，直接将对应的sql变成批量sql，如下：

**批量插入**

```xml
<insert id="batchInsert">
insert into 
    user(id, gmt_create, gmt_modified, username, age, sex, phone)
    values
    <foreatch item="user" collection="userList" separator=",">
    (#{user.id}, now(), now(), #{user.username}, #{user.age}, #{user.sex}, #{user.phone})
    </foreatch>
</insert>
```

**批量更新**

- 单行sql

```xml
<update id="batchUpdate">
update user set 
    age = #{age}
where id in 
    <foreatch item=id collection="idList" open="(" separator="," close=")">
    #{id}
    </foreatch>
</update>
```

- 多行sql

```java
<update id="batchUpdate">
    <foreatch item=user collection="userList" separator=";">
    update user set 
        age = #{user.age},
        phone = #{user.phone}
    where 
        id = #{user.id}
    </foreatch>
</update>
```


**批量删除**

- 单行sql

```xml
<delete id="batchDelete">
delete from user 
where id in 
    <foreatch item=id collection="idList" open="(" separator="," close=")">
    #{id}
    </foreatch>
</update>
```

- 多行sql

```java
<update id="batchUpdate">
    <foreatch item=id collection="idList" separator=";">
    delete from user where id = #{id}
    </foreatch>
</update>
```

> 多行sql需在mysql链接参数上增加`allowMultiQueries=true`

用sql批量处理方式存在局限性，最明显的是长度问题，另外就是`in`条件存在性能问题。

## jdbc batch方式

`ibatis`中`execute`的回调`api`可以直接实现，底层使用的是`jdbc`的`batch`能力，代码如下：

```java
public int batchUpdate(String statementId, List<Map<String, Object>> params) {
        
    return getSqlMapClientTemplate().execute(executor -> {

        int count = 0;
        int batchSize = 0;
        executor.startBatch();
        for (Map<String, Object> param : params) {
            executor.update(statementId, param);
            batchSize++;
            if (batchSize == 50) {
                count += executor.executeBatch();
                executor.startBatch();
                batchSize = 0;
            }
        }

        if (batchSize > 0) {
            count += executor.executeBatch();
        }

        return count;
    });
}
```

`mybatis`中不在提供`execute`这种`api`了，实际还是支持的但预要另外特殊处理下，代码如下：

```java
@Autowired
private SqlSessionFactory sqlSessionFactory;

private SqlSession batchSqlSession;

@PostConstruct
public void init() {
    // 初始化批量处理的sql session
    batchSqlSession = new SqlSessionTemplate(sqlSessionFactory, ExecutorType.BATCH);
}

public int batchUpdate(String statementId, List<Map<String, Object>> params) {

    List<BatchResult> batchResults = new ArrayList<>();

    int batchSize = 0;
    for (Map<String, Object> param : params) {
        batchSqlSession.update(statementId, param);
        batchSize++;
        if (batchSize == 50) {
            batchResults.addAll(batchSqlSession.flushStatements());
            batchSize = 0;
        }
    }

    if (batchSize > 0) {
        batchResults.addAll(batchSqlSession.flushStatements());
    }

    return batchResults.stream().map(BatchResult::getUpdateCounts).flatMapToInt(Arrays::stream).sum();
}
```

> 这里的核心是初化一个`BATCH`的`SqlSession`，上面代码设置的`BatcSize`为50，你可以根据自已的情况进行调整，但这个值不建议过大，会导致游标过大的错误。

## 总结
1. 单行`sql`这种方式比较简单直接针对`sql`进行处理即可，无需特殊编码，但是存在`sql`过长和性能问题。
2. `jdbc batch`这种方式需要额外的编码，但不存在`sql`过长和性能问题。

这里推荐`jdbc
batch`方式，没有副作用，不存任何隐患。

> 笔者这里生产环境中3w数据的更新只需5s，未使用`BATCH`之前在15-20s左右。
