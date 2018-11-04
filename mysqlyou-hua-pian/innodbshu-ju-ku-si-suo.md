# InnoDB数据库死锁

## 场景描述

在update表的时候出现DeadlockLoserDataAccessException异常 \(Deadlock found when trying to get lock; try restarting transaction...\)。

## 问题分析

这个异常并不会影响用户使用，因为数据库遇到死锁会自动回滚并重试。用户的感觉就是操作稍有卡顿。但是监控老是报异常，所以需要解决一下。

## 解决方法

在应用程序中update的地方使用try-catch。

我自己封装了一个函数，如下。

```
/**
     * 2016-03-15
     * linxuan
     * handle deadlock while update table
     */
    private void updateWithDeadLock(TestMapper mapper, Test record) throws InterruptedException {
        boolean oops;
        int retries = 5;
        do{
            oops = false;
            try{
                mapper.updateByPrimaryKeySelective(record);
            }
            catch (DeadlockLoserDataAccessException dlEx){
                oops = true;
                Thread.sleep((long) (Math.random() * 500));
            }
            finally {
            }
        } while(oops == true && retries-- >0);
    }
```

我用的是mybatis，所以只需将mapper传进函数，如果不用mybatis，需要自己创建并关闭数据库连接。

## 数据库死锁

数据库死锁是事务性数据库 \(如SQL Server, MySql等\)经常遇到的问题。除非数据库死锁问题频繁出现导致用户无法操作，一般情况下数据库死锁问题不严重。在应用程序中进行try-catch就可以。那么数据死锁是如何产生的呢？

InnoDB实现的是行锁 \(row level lock\)，分为共享锁 \(S\) 和 互斥锁 \(X\)。

* 共享锁用于事务read一行。
* 互斥锁用于事务update或delete一行。

当客户A持有共享锁S，并请求互斥锁X；同时客户B持有互斥锁X，并请求共享锁S。以上情况，会发生数据库死锁。如果还不够清楚，请看下面的例子。

首先，客户A创建一个表T，并向T中插入一条数据，客户A开始一个select事务，所以拿着共享锁S。

```
mysql> CREATE TABLE t (i INT) ENGINE = InnoDB;
Query OK, 0 rows affected (1.07 sec)

mysql> INSERT INTO t (i) VALUES(1);
Query OK, 1 row affected (0.09 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM t WHERE i = 1 LOCK IN SHARE MODE;
+------+
| i    |
+------+
|    1 |
+------+
```

然后，客户B开始一个新事务，新事务是delete表T中的唯一一条数据。

```
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
```

删除操作需要互斥锁 \(X\)，但是互斥锁X和共享锁S是不能相容的。所以删除事务被放到锁请求队列中，客户B阻塞。

最后，客户A也想删除表T中的那条数据：

```
mysql> DELETE FROM t WHERE i = 1;
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
```

[lima](https://www.cnblogs.com/lin-xuan/)

keep coding, don't settle

## [InnoDB数据库死锁](https://www.cnblogs.com/lin-xuan/p/5280614.html)

**目录**

* [场景描述](http://www.cnblogs.com/lin-xuan/p/5280614.html#_label0)
* [问题分析](http://www.cnblogs.com/lin-xuan/p/5280614.html#_label1)
* [解决方法](http://www.cnblogs.com/lin-xuan/p/5280614.html#_label2)
* [延伸：数据库死锁](http://www.cnblogs.com/lin-xuan/p/5280614.html#_label3)
* [数据库死锁例子](http://www.cnblogs.com/lin-xuan/p/5280614.html#_label4)

**正文**

[回到顶部](http://www.cnblogs.com/lin-xuan/p/5280614.html#_labelTop)

## 场景描述

在update表的时候出现DeadlockLoserDataAccessException异常 \(Deadlock found when trying to get lock; try restarting transaction...\)。

[回到顶部](http://www.cnblogs.com/lin-xuan/p/5280614.html#_labelTop)

## 问题分析

这个异常并不会影响用户使用，因为数据库遇到死锁会自动回滚并重试。用户的感觉就是操作稍有卡顿。但是监控老是报异常，所以需要解决一下。

[回到顶部](http://www.cnblogs.com/lin-xuan/p/5280614.html#_labelTop)

## 解决方法

在应用程序中update的地方使用try-catch。

我自己封装了一个函数，如下。

按 Ctrl+C 复制代码

按 Ctrl+C 复制代码

我用的是mybatis，所以只需将mapper传进函数，如果不用mybatis，需要自己创建并关闭数据库连接。

[回到顶部](http://www.cnblogs.com/lin-xuan/p/5280614.html#_labelTop)

## 延伸：数据库死锁

数据库死锁是事务性数据库 \(如SQL Server, MySql等\)经常遇到的问题。除非数据库死锁问题频繁出现导致用户无法操作，一般情况下数据库死锁问题不严重。在应用程序中进行try-catch就可以。那么数据死锁是如何产生的呢？

InnoDB实现的是行锁 \(row level lock\)，分为共享锁 \(S\) 和 互斥锁 \(X\)。

* 共享锁用于事务read一行。
* 互斥锁用于事务update或delete一行。

当客户A持有共享锁S，并请求互斥锁X；同时客户B持有互斥锁X，并请求共享锁S。以上情况，会发生数据库死锁。如果还不够清楚，请看下面的例子。

[回到顶部](http://www.cnblogs.com/lin-xuan/p/5280614.html#_labelTop)

## 数据库死锁例子

首先，客户A创建一个表T，并向T中插入一条数据，客户A开始一个select事务，所以拿着共享锁S。

```
mysql> CREATE TABLE t (i INT) ENGINE = InnoDB;
Query OK, 0 rows affected (1.07 sec)

mysql> INSERT INTO t (i) VALUES(1);
Query OK, 1 row affected (0.09 sec)

mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> SELECT * FROM t WHERE i = 1 LOCK IN SHARE MODE;
+------+
| i    |
+------+
|    1 |
+------+
```

然后，客户B开始一个新事务，新事务是delete表T中的唯一一条数据。

```
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
```

删除操作需要互斥锁 \(X\)，但是互斥锁X和共享锁S是不能相容的。所以删除事务被放到锁请求队列中，客户B阻塞。

最后，客户A也想删除表T中的那条数据：

```
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
```

死锁产生了！因为客户A需要锁X来删除行，而客户B拿着锁X并正在等待客户A释放锁S。看看客户A，B的状态：

* 客户A: 拿着锁S，等待着客户B释放锁X。
* 客户B: 拿着锁X，等待着客户A释放锁S。

发生死锁后，InnoDB会为对一个客户产生错误信息并释放锁。返回给客户的信息：

```
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
```

所以，另一个客户可以正常执行任务。死锁结束。

参考资料：

[http://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks.html](http://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks.html)

[http://www.xaprb.com/blog/2006/08/03/a-little-known-way-to-cause-a-database-deadlock/](http://www.xaprb.com/blog/2006/08/03/a-little-known-way-to-cause-a-database-deadlock/)

