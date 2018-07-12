# 慢SQL监控

**前言：** mysql可以记录用户执行的sql：记录到文件、表格 ，mysql可以定义执行多少时间以上得sql属于慢查询，也会根据配置，记录相关信息到文件、表格

**背景说明：**

公司想监控记录每天执行了哪些sql，哪些sql是慢查询，然后去优化sql

**技术说明：**

其实只要搞清楚了mysql怎样记录执行sql的

怎样记录慢查询的即可

**技术细节：**

* **进入MySql 查询是否开了慢查询**

> show variables like 'slow\_query%'
>
> ![](/assets/import-slowsql-01.png)
>
> 参数说明：
>
> * slow\_query\_log 慢查询开启状态  OFF 未开启 ON 为开启
> * slow\_query\_log\_file 慢查询日志存放的位置（这个目录需要MySQL的运行帐号的可写权限，一般设置为MySQL的数据存放目录）默认为localhost-slow.log

* **查看慢查询超时时间**

> show variables like 'long%';
>
> ![](/assets/import-showsql-02.png)
>
> 参数说明：
>
> * long\_query\_time 查询超过多少秒才记录   默认10秒 修改为1秒
> * set global long\_query\_time=1; 修改之后，先关闭数据库连接，再重新连接，再次查询就可以看到实际上是修改了的。

* **修改方法一（不推荐）**

方法一：优点临时开启慢查询，不需要重启数据库  缺点：MySql 重启慢查询失效

默认情况下slow\_query\_log的值为OFF，表示慢查询日志是禁用的，可以通过设置slow\_query\_log的值来开启，如下所示开启慢查询日志，1表示开启，0表示关闭。

```
mysql> show variables like '%slow_query_log%';

+---------------------+--------------------------------------------+

| Variable_name    | Value                   |

+---------------------+--------------------------------------------+

| slow_query_log   | OFF                    |

| slow_query_log_file | /application/mysql/data/localhost-slow.log |

+---------------------+--------------------------------------------+

2 rows in set (0.01 sec)
```

输入 语句修改（重启后失效，建议在/etc/my.cnf中修改永久生效）

```
mysql> set global slow_query_log=1;

Query OK, 0 rows affected (0.11 sec)
```

再次查看

```
mysql> show variables like '%slow_query_log%';

+---------------------+--------------------------------------------+

| Variable_name    | Value                   |

+---------------------+--------------------------------------------+

| slow_query_log   | ON                     |

| slow_query_log_file | /application/mysql/data/localhost-slow.log |

+---------------------+--------------------------------------------+

2 rows in set (0.00 sec)
```

* **修改方法二**

修改 MySql 慢查询，通过修改my.cnf修改配置参数，设置之后，重启永久生效

```
[root@localhost mysql]# find / -type f -name "my.cnf"
/application/mysql/mysql-test/suite/rpl_ndb/my.cnf
/application/mysql/mysql-test/suite/rpl/extension/bhs/my.cnf
/application/mysql/mysql-test/suite/rpl/my.cnf
/application/mysql/mysql-test/suite/ndb_binlog/my.cnf
/application/mysql/mysql-test/suite/ndb_team/my.cnf
/application/mysql/mysql-test/suite/ndb_rpl/my.cnf
/application/mysql/mysql-test/suite/ndb_big/my.cnf
/application/mysql/mysql-test/suite/federated/my.cnf
/application/mysql/mysql-test/suite/ndb/my.cnf
/application/mysql/my.cnf
```

* vi /application/mysql/my.cnf ，找到 \[mysqld\] 下面添加如下参数：

```
slow_query_log =1

slow_query_log_file=/application/mysql/data/localhost-slow.log

long_query_time = 1
```

修改完重启MySQL

参数说明：

> general\_log=1 \#开启mysql执行sql的日志
>
> general\_log\_file=/log/general.log \#将mysql执行sql日志记录到指定文件中
>
> slow\_query\_log=1 \#开启mysql慢sql的日志
>
> slow\_query\_log\_file=/log/slow.log \#将慢查询日志记录到指定文件中
>
> log\_output=table,File \#日志输出会写表，也会写日志文件，为了便于程序去统计，所以最好写表
>
> long\_query\_time=1 \#设置mysql的慢查询为超过1s的查询
>
> \#如果没有配置general\_log\_file，那么general\_log就只会写表了
>
> \#如果没有配置slow\_query\_log\_file，那么slow\_query\_log就只会写表了

* **查看、测试**

**插入一条测试慢查询**

```
mysql> select sleep(2);

+----------+

| sleep(2) |

+----------+

|    0 |

+----------+

1 row in set (2.00 sec)
```

**查看慢查询日志**

```
[root@localhost data]# cat /application/mysql/data/localhost-slow.log

/application/mysql/bin/mysqld, Version: 5.5.51-log (MySQL Community Server (GPL)). started with:

Tcp port: 3306 Unix socket: /tmp/mysql.sock

Time         Id Command  Argument

/application/mysql/bin/mysqld, Version: 5.5.51-log (MySQL Community Server (GPL)). started with:

Tcp port: 3306 Unix socket: /tmp/mysql.sock

Time         Id Command  Argument

/application/mysql/bin/mysqld, Version: 5.5.51-log (MySQL Community Server (GPL)). started with:

Tcp port: 3306 Unix socket: /tmp/mysql.sock

Time         Id Command  Argument

# Time: 170605 6:37:00

# User@Host: root[root] @ localhost []

# Query_time: 2.000835 Lock_time: 0.000000 Rows_sent: 1 Rows_examined: 0

SET timestamp=1496615820;

select sleep(2);
```

**通过MySQL命令查看有多少慢查询**

```
mysql> show global status like '%Slow_queries%';

+---------------+-------+

| Variable_name | Value |

+---------------+-------+

| Slow_queries | 1   |

+---------------+-------+

1 row in set (0.00 sec)
```

**日志分析工具mysqldumpslow**

在生产环境中，如果要手工分析日志，查找、分析SQL，显然是个体力活，MySQL提供了日志分析工具mysqldumpslow

如：

```

/path/mysqldumpslow -s c -t 10 /database/mysql/slow-log
这会输出记录次数最多的10条SQL语句，其中：

-s, 是表示按照何种方式排序，c、t、l、r分别是按照记录次数、时间、查询时间、返回的记录数来排序，ac、at、al、ar，表示相应的倒叙；
-t, 是top n的意思，即为返回前面多少条的数据；
-g, 后边可以写一个正则匹配模式，大小写不敏感的；
比如
/path/mysqldumpslow -s r -t 10 /database/mysql/slow-log
得到返回记录集最多的10个查询。
/path/mysqldumpslow -s t -t 10 -g “left join” /database/mysql/slow-log
得到按照时间排序的前10条里面含有左连接的查询语句。
```



