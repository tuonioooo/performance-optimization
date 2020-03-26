# InnoDB数据库死锁

## 场景描述

在update表的时候出现DeadlockLoserDataAccessException异常 \(Deadlock found when trying to get lock; try restarting transaction...\)。

## 问题分析

这个异常并不会影响用户使用，因为数据库遇到死锁会自动回滚并重试。用户的感觉就是操作稍有卡顿。但是监控老是报异常，所以需要解决一下。

## 解决方法

在应用程序中update的地方使用try-catch。

我自己封装了一个函数，如下。

```text
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

## 数据库死锁例子

首先，客户A创建一个表T，并向T中插入一条数据，客户A开始一个select事务，所以拿着共享锁S。

```text
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

```text
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
```

删除操作需要互斥锁 \(X\)，但是互斥锁X和共享锁S是不能相容的。所以删除事务被放到锁请求队列中，客户B阻塞。

最后，客户A也想删除表T中的那条数据：

```text
mysql> START TRANSACTION;
Query OK, 0 rows affected (0.00 sec)

mysql> DELETE FROM t WHERE i = 1;
```

死锁产生了！因为客户A需要锁X来删除行，而客户B拿着锁X并正在等待客户A释放锁S。看看客户A，B的状态：

* 客户A: 拿着锁S，等待着客户B释放锁X。
* 客户B: 拿着锁X，等待着客户A释放锁S。

发生死锁后，InnoDB会为对一个客户产生错误信息并释放锁。返回给客户的信息：

```text
ERROR 1213 (40001): Deadlock found when trying to get lock;
try restarting transaction
```

所以，另一个客户可以正常执行任务。死锁结束。

参考资料：

[http://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks.html](http://dev.mysql.com/doc/refman/5.7/en/innodb-deadlocks.html)

[http://www.xaprb.com/blog/2006/08/03/a-little-known-way-to-cause-a-database-deadlock/](http://www.xaprb.com/blog/2006/08/03/a-little-known-way-to-cause-a-database-deadlock/)

