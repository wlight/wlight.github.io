# Mybatis大量数据的插入或更新操作方案思考


<!--more-->

[Mybatis大量数据的插入或更新操作方案思考(使用ON DUPLICATE KEY UPDATE)](https://www.cnblogs.com/sueyyyy/p/13034526.html)

## 背景

　　最近需要向数据库中插入5w+数据，但是在插入的过程中需要先根据某个字段进行判断，如果数据库中已经存在记录则进行更新，否则插入。通常这种情况下，我们会在代码中编写一条查询语句，查询数据库中是否存在相关记录。然后通过if条件判断是进行更新还是进行插入。这种情况对于数据量不多的时候是感觉不到很大的差异的。但是在数据量特别大的情况下，如果操作每一条数据之前都要先查询数据库再判断是更新还是插入，就需要对数据库进行多次的连接和断开操作，极大的降低了系统性能同时增加了服务器的压力。

## 解决方案

　　我们可以将判断逻辑交给数据库去完成，这样原先两次的连接的操作只需要一次就可以实现。经过查阅发现Mysql中的 *ON DUPLICATE KEY UPDATE* 语句可以实现该需求。

## **索引**

在使用该语法之前，我们先回忆一下mysql中主键索引，唯一索引和普通索引的关系。

### **主键索引**

主键索引是唯一索引的特殊类型。 
数据库表通常有一列或列组合，其值用来唯一标识表中的每一行。该列称为表的主键。 
在数据库关系图中为表定义一个主键将自动创建主键索引，主键索引是唯一索引的特殊类型。主键索引要求主键中的每个值是唯一的。当在查询中使用主键索引时，它还允许快速访问数据。主键索引不能为空。每个表只能有一个主键

### **唯一索引**

不允许两行具有相同的索引值。但可以都为NULL，笔者亲试。 
如果现有数据中存在重复的键值，则数据库不允许将新创建的唯一索引与表一起保存。当新数据将使表中的键值重复时，数据库也拒绝接受此数据。每个表可以有多个唯一索引.

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200602231127178-357619770.png)

### **普通索引**

一般的索引结构，可以在条件删选时加快查询效率，索引字段的值可以重复，可以为空值

## ON DUPLICATE KEY UPDATE 的使用

我们先看一下测试表的结构 主键为id

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200602233304417-2029473985.png)

 表中现有测试数据

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603230014293-706553605.png)

**case 1:**含有ON DUPLICATE KEY UPDATE的INSERT语句中包含主键，且主键在表中已存在，执行更新操作

 如果update语句后的主键不是insert语句后的主键，且表中已存在，那么更新或者插入操作都不会成功执行，会抛出主键冲突异常。

```sql
INSERT INTO applyinfo ( id, username, phone, position, company )
VALUES
	( '1', 'name03', 'phone03', 'position03', 'company03' ) 
ON DUPLICATEKEY UPDATE 
	id = '2',
	position = 'position02',
	username = 'name03',
	phone = 'phone03',
	company = 'company03';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603230608489-780604513.png)

 如果update语句后的主键是就是insert语句后的主键或者没有填写主键，那么会更新当前记录。

```sql
INSERT INTO applyinfo ( id, username, phone, position, company )
VALUES
	( '1', 'name03', 'phone03', 'position03', 'company03' ) 
ON DUPLICATEKEY UPDATE 
	id = '1',
	position = 'position03',
	username = 'name03',
	phone = 'phone03',
	company = 'company03';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603230418000-373564700.png)

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603230444326-1133671963.png)

**case 2:**含有ON DUPLICATE KEY UPDATE的INSERT语句中包含主键，且主键在表中不存在，那么会直接执行插入操作，不执行update之后的语句。

```sql
INSERT INTO applyinfo ( id, username, phone, position, company )
VALUES
	( '4', 'name04', 'phone04', 'position04', 'company04' ) 
ON DUPLICATEKEY UPDATE 
	id = '4',
	position = 'position044',
	username = 'name044',
	phone = 'phone044',
	company = 'company044';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603231413112-1106658812.png)

 ![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603231445248-1030845605.png)

 **case 3:**含有ON DUPLICATE KEY UPDATE的INSERT语句中不包含主键

```sql
INSERT INTO applyinfo ( username, phone, position, company )
VALUES
	( 'name05', 'phone05', 'position05', 'company05' ) 
ON DUPLICATEKEY UPDATE 
	id = '4',
	position = 'position044',
	username = 'name044',
	phone = 'phone044',
	company = 'company044';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603231635459-936500742.png)

 ![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603231702063-72556442.png)

## **补充**

**ON DUPLICATE KEY UPDATE的INSERT语句的规则除了适用于主键之外，还适用于唯一索引字段**

例如，我们在这里给applyInfo表的phone字段添加一个唯一索引

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603231919996-1008280377.png)

 **case 1:**这里我们可以看到insert语句后面的phone字段为“phone05”，已经在表中存在，那么则会执行update方法。

```mysql
INSERT INTO applyinfo ( username, phone, position, company )
VALUES
	( 'name05', 'phone05', 'position05', 'company05' )
ON DUPLICATEKEY UPDATE 
	position = 'position044',
	username = 'name044',
	phone = 'phone044',
	company = 'company044';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603232224470-1090950247.png)

 ![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603232235439-1312466321.png)

 **case 2:**同样，不存在则会执行插入操作

```sql
INSERT INTO applyinfo ( username, phone, position, company )
VALUES
	( 'name05', 'phone06', 'position05', 'company05' )
ON DUPLICATEKEY UPDATE 
	position = 'position066',
	username = 'name066',
	phone = 'phone066',
	company = 'company066';
```

![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603232355607-437856883.png)

 ![img](https://cdn.jsdelivr.net/gh/wlight/cdn-images/blog-images/1320942-20200603232414702-1804121725.png)

## MyBatis语句示例

```java
int saveOrUpdateTrans(List<BisOwnRisk> bisOwnRiskList);
```

 在mybatis中进行批量增加或修改的sql为：

```sql
INSERT INTO bis_own_risk_trans (
    username,
    idNo,
    phone,
    deviceId,
    ip,
    bankCard
    )
    VALUES
    <foreach collection="list" item="item" index="index" separator=",">
      (
      #{item.username}, #{item.idNo}, #{item.phone},
      #{item.deviceId},  #{item.ip}, #{item.bankCard}
      )
    </foreach>
    ON DUPLICATEKEY UPDATE
    username =VALUES(username),
    idNo =VALUES(idNo),
    phone =VALUES(phone),
    deviceId =VALUES(deviceId),
    ip =VALUES(ip),
    bankCard =VALUES(bankCard)
```

## 总结

- ON DUPLICATE KEY UPDATE需要有在INSERT语句中有存在主键或者唯一索引的列，并且对应的数据已经在表中才会执行更新操作。而且如果要更新的字段是主键或者唯一索引，不能和表中已有的数据重复，否则插入更新都失败。
- 不管是更新还是增加语句都不允许将主键或者唯一索引的对应字段的数据变成表中已经存在的数据。

## 注意

　　**在并行且开启事务的时候使用INSERT ON DUPLICATE KEY UPDATE语句时，会出现死锁。**关于使用 ON DUPLICATE KEY UPDATE语句造成死锁现象的分析可以参考如下链接。

　　*[Mysql死锁排查：insert on duplicate死锁一次排查分析过程](https://www.cnblogs.com/sueyyyy/p/13047035.html)*


