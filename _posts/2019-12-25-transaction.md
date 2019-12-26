---
title:  超长事务
categories: 
 - 事务
tags: 日常

---



java.sql.SQLTransientConnectionException: HikariPool-1 - Connection is not available, request timed out after 30000ms



数据库连接超时，需要查看哪个连接部分事务没有停止

Lock wait timeout exceeded; try restarting transaction

## mysql慢日志查询

1. 慢日志开启

```mysql
#查看是否开启
show variables like '%slow_query_log%';

#开启
set global slow_query_log = 1;
```

开启之后查看获得慢日志文件路径

![image-20191225160802348](/images/2019-12-25/image-20191225160802348.png)

2. 慢日志过期时间修改

```sql
# 查看过期时间
show variables like 'long_query_time';
```

因为是在测试环境，所以直接修改配置文件内容

```ini
slow_query_log_file = /slow.log
long_query_time = 1
```

修改慢日志的路径，并且配置需要记录的过期时间

3. 查看打印的慢日志

打印发现没有查询很慢的语句，于是需要排查没有结束的事务

```mysql
select * from sys.processlist
```

![image-20191226161950639](/images/2019-12-25/image-20191226161950639.png)

里面有几个比较重要的字段
里面有几个比较重要的字段

- `db`  线程的默认数据库
- `command`  对于前台线程，线程正在代表 client 执行的命令类型，如果 session 是 idle 则为`Sleep`

- `last_statement`  如果当前没有执行语句或等待，则由线程执行的最后一个语句

- `last_statement_latency`  How long the last statement executed

可以看出确实有部分语句事务执行时间过长

select taskdetail0_.id as id1_ ... uid=157734172287109 for update

select taskdetail0_.id as id1_ ... uid=157734176692108 for update

```mysql
select * from performance_schema.events_statements_current
```

查到最后执行的查询语句

由查到的查询语句找到了超长事务的语句，我这里结果就不展示了。

观察代码发现，事务本身不会出现死锁的情况，因为是删除操作不存在需要数据一致的情况只需要查询存在然后删除就可以了，存在表锁的表已经加了悲观锁，然后没有表锁的语句的查询语句放置在事务之外，所以不会是进入死锁，查看打印的语句也没有发现死锁。

最后回到超长事务这里，发现在事务里面我有做一个从oss删除的操作。。。问题就在这里吧，删除操作需要数秒，从而导致在并发情况下事务一直不能提交从而会有各种timeout。问题解决