# **高性能前缀索引**

有时候需要索引很长的字符列，这会让索引变得大且慢。通常可以索引开始的部分字符，这样可以大大节约索引空间，从而提高索引效率。但这样也会降低索引的选择性。索引的选择性是指不重复的索引值（也称为基数，cardinality\)和数据表的记录总数的比值，范围从1/\#T到1之间。索引的选择性越高则查询效率越高，因为选择性高的索引可以让MySQL在查找时过滤掉更多的行。唯一索引的选择性是1，这是最好的索引选择性，性能也是最好的。

一般情况下某个前缀的选择性也是足够高的，足以满足查询性能。对于BLOB，TEXT，或者很长的VARCHAR类型的列，必须使用前缀索引，因为MySQL不允许索引这些列的完整长度。

诀窍在于要选择足够长的前缀以保证较高的选择性，同时又不能太长（以便节约空间）。前缀应该足够长，以使得前缀索引的选择性接近于索引的整个列。换句话说，前缀的”基数“应该接近于完整的列的”基数“。

为了决定前缀的合适长度，需要找到最常见的值的列表，然后和最常见的前缀列表进行比较。下面的示例是mysql官方提供的示例数据库

下载地址如下：

[http://downloads.mysql.com/docs/sakila-db.zip](http://downloads.mysql.com/docs/sakila-db.zip)

在示例数据库sakila中并没有合适的例子，所以从表city中生成一个示例表，这样就有足够数据进行演示：

1.解压下载的

sakila-db.zip文件

2.使用source命令以及sakila-schema.sql和sakila-data.sql文件来初始化sakila库以及相关表格

```
mysql> select database();                                                           
+------------+
| database() |
+------------+
| sakila     |
+------------+
1 row in set (0.00 sec)

mysql> create table city_demo (city varchar(50) not null);                          

mysql> insert into city_demo (city) select city from city;  --执行两次
Query OK, 600 rows affected (0.02 sec)
Records: 600  Duplicates: 0  Warnings: 0

mysql> update city_demo set city = ( select city from city order by rand() limit 1);
Query OK, 1198 rows affected (0.42 sec)
Rows matched: 1200  Changed: 1198  Warnings: 0

注：因为上述sql语句使用了rand函数，所以每个人的执行结果可以都不一样！
```

首先找到最常见的城市列表：

```
mysql
>
select count(*) as cnt,city from city_demo group by city order by cnt desc limit 10;
+-----+---------------+
| cnt | city          |
+-----+---------------+
|   8 | Dongying      |
|   7 | Omdurman      |
|   6 | Etawah        |
|   6 | Okara         |
|   6 | Tsuyama       |
|   6 | Brindisi      |
|   6 | Kuwana        |
|   6 | Grand Prairie |
|   5 | Fuyu          |
|   5 | Siegen        |
+-----+---------------+
10 rows in set (0.00 sec)
```

现在查找到频繁出现的城市前缀。先从3个前缀字母开始，然后4个，5个，6个：

```
mysql
>
select count(*) as cnt,left(city,3) as pref from city_demo group by pref order by cnt desc limit 10;
+-----+------+
| cnt | pref |
+-----+------+
|  23 | San  |
|  15 | Hal  |
|  14 | Cha  |
|  14 | al-  |
|  12 | Bat  |
|  12 | Kor  |
|  11 | Don  |
|  11 | Shi  |
|  10 | La   |
|  10 | El   |
+-----+------+
10 rows in set (0.00 sec)
```

```
mysql
>
select count(*) as cnt,left(city,4) as pref from city_demo group by pref order by cnt desc limit 10;
+-----+------+
| cnt | pref |
+-----+------+
|  14 | San  |
|   8 | Dong |
|   7 | Iwak |
|   7 | al-Q |
|   7 | Omdu |
|   6 | Kuwa |
|   6 | Tsuy |
|   6 | Brin |
|   6 | Etaw |
|   6 | Okar |
+-----+------+
10 rows in set (0.00 sec)


可以看到3字节检索到的结果与全文检索相差很大，继续增加到4个字节


mysql
>
select count(*) as cnt,left(city,5) as pref from city_demo group by pref order by cnt desc limit 10;
+-----+-------+
| cnt | pref  |
+-----+-------+
|   8 | Dongy |
|   7 | al-Qa |
|   7 | Omdur |
|   6 | Okara |
|   6 | Valle |
|   6 | Grand |
|   6 | Tsuya |
|   6 | Etawa |
|   6 | South |
|   6 | Kuwan |
+-----+-------+
10 rows in set (0.00 sec)
```

```
mysql
>
select count(*) as cnt,left(city,6) as pref from city_demo group by pref order by cnt desc limit 10;
+-----+--------+
| cnt | pref   |
+-----+--------+
|   8 | Dongyi |
|   7 | Omdurm |
|   6 | Okara  |
|   6 | Tsuyam |
|   6 | Valle  |
|   6 | Grand  |
|   6 | Etawah |
|   6 | Brindi |
|   6 | Kuwana |
|   5 | Haldia |
+-----+--------+
10 rows in set (0.01 sec)
```

通过上面改变不同前缀长度发现，当前缀长度为6时，这个前缀的选择性就接近完整咧的选择性了。

当然还有另外更方便的方法，那就是计算完整列的选择性，并使其前缀的选择性接近于完整列的选择性。下面显示如何计算完整列的选择性：

```
mysql
>
select count(distinct city)/count(*) from city_demo;
+-------------------------------+
| count(distinct city)/count(*) |
+-------------------------------+
|                        0.4333 |
+-------------------------------+
1 row in set (0.00 sec)
```

可以在一个查询中针对不同前缀长度的选择性进行计算，这对于大表非常有用，下面给出如何在同一个查询中计算不同前缀长度的选择性：

```
mysql
>
select count(distinct left(city,3))/count(*) as sel3,count(distinct left(city,4))/count(*) as sel4,count(distinct left(city,5))/count(*) as sel5, count(distinct left(city,6))/count(*) as sel6 from city_demo;  
+--------+--------+--------+--------+
| sel3   | sel4   | sel5   | sel6   |
+--------+--------+--------+--------+
| 0.3408 | 0.4100 | 0.4225 | 0.4300 |
+--------+--------+--------+--------+
1 row in set (0.00 sec)
```

可以看见当索引前缀为6时的基数是0.4300，已经接近完整列选择性0.4333。

下面根据找到的索引前缀长度创建前缀索引：

```
mysql> alter table city_demo add key (city(6));
Query OK, 0 rows affected (0.19 sec)
Records: 0  Duplicates: 0  Warnings: 0
```

```
mysql
>
 explain select * from city_demo where city like 'Jin%' \G
*************************** 1. row ***************************
           id: 1
  select_type: SIMPLE
        table: city_demo
   partitions: NULL
         type: range
possible_keys: city
          key: city
      key_len: 8
          ref: NULL
         rows: 4
     filtered: 100.00
        Extra: Using where
1 row in set, 1 warning (0.00 sec)
```

可以看见正确使用刚创建的索引。

**优点**：前缀索引是一种能使索引更小，更快的有效办法

**缺点**：**mysql无法使用其前缀索引做ORDER BY和GROUP BY**，也**无法**使用前缀索引**做覆盖扫描**。

