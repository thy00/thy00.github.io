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

### 解决办法

观察代码发现，事务本身不会出现死锁的情况，因为是删除操作不存在需要数据一致的情况只需要查询存在然后删除就可以了，存在表锁的表已经加了悲观锁，然后没有表锁的语句的查询语句放置在事务之外，所以不会是进入死锁，查看打印的语句也没有发现死锁。

最后回到超长事务这里，发现在事务里面我有做一个从oss删除的操作。。。问题就在这里吧，删除操作需要数秒，从而导致在并发情况下事务一直不能提交从而会有各种timeout。问题解决

## Druid监控

除了使用mysql自己的查询方法，也可以在服务端监控sql语句的执行情况

在测试的时候有一个现象，在执行第二次循环的时候，后台的错误日志打印出很多的事务等待超时的记录

> Caused by: com.mysql.jdbc.exceptions.jdbc4.MySQLTransactionRollbackException: Lock wait timeout exceeded; try restarting transaction

第一个怀疑连接池有事务没有主动提交，查看监控

![image-20200108122223715](../images/2019-12-25-transaction/image-20200108122223715.png)

打开和关闭的次数一致，则证明连接没有泄露或者未提交

然后查看连接持有时间

![image-20200108122425550](../images/2019-12-25-transaction/image-20200108122425550.png)

发现连接持有时间在1s以上的占比很大，因为连接池的可用连接固定，如果连接时间过长就会导致，其他正常的连接无法拿到连接，可以认为这些持有连接时间在1s以上的堵塞了连接池的正常工作。

再查看事务时间分布

![image-20200108122807043](../images/2019-12-25-transaction/image-20200108122807043.png)

明显有些事务不合理

![image-20200108123009278](../images/2019-12-25-transaction/image-20200108123009278.png)

可以看见红色的最慢执行时间占比较高，在测试刚开始的时候都是正常的，但是运行一段时间之后，执行时间边长，可以预期的是执行遇到了堵塞，找到执行分布时间较长的对比，可以推理出执行时间应该是被那种经常性的执行较长时间语句堵塞，并且并发较高错误数较多，找到了需要优化的语句。

针对特定的语句需要优化

1. 优化表结构
2. 将表锁优化为行锁
3. 将悲观锁优化为乐观锁？

