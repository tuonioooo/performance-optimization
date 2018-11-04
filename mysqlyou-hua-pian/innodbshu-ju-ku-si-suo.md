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



