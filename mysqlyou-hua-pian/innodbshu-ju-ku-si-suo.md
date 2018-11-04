# InnoDB数据库死锁

## 场景描述

在update表的时候出现DeadlockLoserDataAccessException异常 \(Deadlock found when trying to get lock; try restarting transaction...\)。



## 问题分析

这个异常并不会影响用户使用，因为数据库遇到死锁会自动回滚并重试。用户的感觉就是操作稍有卡顿。但是监控老是报异常，所以需要解决一下。





## 解决方法

在应用程序中update的地方使用try-catch。

我自己封装了一个函数，如下。

