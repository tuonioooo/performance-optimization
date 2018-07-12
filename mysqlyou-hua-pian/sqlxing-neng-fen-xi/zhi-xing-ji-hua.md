# 执行计划

**引言：**

实际项目开发中，由于我们不知道实际查询的时候数据库里发生了什么事情，数据库软件是怎样扫描表、怎样使用索引的，因此，我们能感知到的就只有sql语句运行的时间，在数据规模不大时，查询是瞬间的，因此，在写sql语句的时候就很少考虑到性能的问题。但是随着数据规模增大，如千万、亿的时候，我们运行同样的sql语句时却发现迟迟没有结果，这个时候才知道数据规模已经限制了我们查询的速度。所以，查询优化和索引也就显得很重要了。

**问题：**

当我们在查询前能否预先估计查询究竟要涉及多少行、使用哪些索引、运行时间呢？答案是能的，mysql提供了相应的功能和语法来实现该功能。

**分析：**

MySql提供了EXPLAIN语法用来进行查询分析，在SQL语句前加一个"EXPLAIN"即可。比如我们要分析如下SQL语句：

```
explain select * from table where table.id = 1
```

运行上面的sql语句后你会看到，下面的表头信息：

```
select_type | table | type | possible_keys | key | key_len | ref | rows | Extra
```





字段解释：

<table>
        <tr>
            <td>字段</td>
            <td>含义</td>
        </tr>
        <tr>
            <td>table</td>
            <td>显示这一行的数据是关于哪张表的</td>
        </tr>
        <tr>
            <td colspan="2">type</td>
            <td>system</td>
            <td>表只有一行：system表。这是const连接类型的特殊情况。</td>
            <td>这是重要的列，显示连接使用了何种类型。从最好到最差的连接类型为const、eq_reg、ref、range、indexhe和ALL 说明：不同连接类型的解释（按照效率高低的顺序排序）</td>
        </tr>
    </table>





