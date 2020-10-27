# Mysql的联合索引

**一、什么是联合索引**

  两个或更多个列上的索引被称作联合索引，联合索引又叫复合索引。对于复合索引:Mysql从左到右的使用索引中的字段，一个查询可以只使用索引中的一部份，但只能是最左侧部分。例如索引是key index \(a,b,c\). 可以支持a \| a,b\| a,b,c 3种组合进行查找，但不支持 b,c进行查找 .当最左侧字段是常量引用时，索引就十分有效。

**二、命名规则**

1、需要加索引的字段，要在where条件中  
 2、数据量少的字段不需要加索引  
 3、如果where条件中是OR关系，加索引不起作用  
 4、符合最左原则

**三、创建索引**

在执行CREATE TABLE语句时可以创建索引，也可以单独用CREATE INDEX或ALTER TABLE来为表增加索引。

**【1】ALTER TABLE**

ALTER TABLE用来创建普通索引、UNIQUE索引或PRIMARY KEY索引。  
 例如：  
 ALTER TABLE table\_name ADD INDEX index\_name \(column\_list\)  
 ALTER TABLE table\_name ADD UNIQUE \(column\_list\)  
 ALTER TABLE table\_name ADD PRIMARY KEY \(column\_list\)  
   其中table\_name是要增加索引的表名，column\_list指出对哪些列进行索引，多列时各列之间用逗号分隔。索引名index\_name可选，缺省时，MySQL将根据第一个索引列赋一个名称。另外，ALTER TABLE允许在单个语句中更改多个表，因此可以在同时创建多个索引。

**【2】CREATE INDEX**

CREATE INDEX可对表增加普通索引或UNIQUE索引。  
 例如：  
 CREATE INDEX index\_name ON table\_name \(column\_list\)  
 CREATE UNIQUE INDEX index\_name ON table\_name \(column\_list\)  
 table\_name、index\_name和column\_list具有与ALTER TABLE语句中相同的含义，索引名不可选。另外，不能用CREATE INDEX语句创建PRIMARY KEY索引。

**四、索引类型**

  在创建索引时，可以规定索引能否包含重复值。如果不包含，则索引应该创建为PRIMARY KEY或UNIQUE索引。对于单列惟一性索引，这保证单列不包含重复的值。对于多列惟一性索引，保证多个值的组合不重复。  
   PRIMARY KEY索引和UNIQUE索引非常类似。  
   事实上，PRIMARY KEY索引仅是一个具有名称PRIMARY的UNIQUE索引。这表示一个表只能包含一个PRIMARY KEY，因为一个表中不可能具有两个同名的索引。  
   下面的SQL语句对students表在sid上添加PRIMARY KEY索引。  
     ALTER TABLE students ADD PRIMARY KEY \(sid\)

**五、删除索引**

  可利用ALTER TABLE或DROP INDEX语句来删除索引。类似于CREATE INDEX语句，DROP INDEX可以在ALTER TABLE内部作为一条语句处理，语法如下。  
 DROP INDEX index\_name ON talbe\_name  
 ALTER TABLE table\_name DROP INDEX index\_name  
 ALTER TABLE table\_name DROP PRIMARY KEY  
   其中，前两条语句是等价的，删除掉table\_name中的索引index\_name。

第3条语句只在删除PRIMARY KEY索引时使用，因为一个表只可能有一个PRIMARY KEY索引，因此不需要指定索引名。如果没有创建PRIMARY KEY索引，但表具有一个或多个UNIQUE索引，则MySQL将删除第一个UNIQUE索引。  
 如果从表中删除了某列，则索引会受到影响。对于多列组合的索引，如果删除其中的某列，则该列也会从索引中删除。如果删除组成索引的所有列，则整个索引将被删除。

**六、什么情况下使用索引**

【1】为了快速查找匹配WHERE条件的行。  
 【2】为了从考虑的条件中消除行。  
 【3】如果表有一个multiple-column索引，任何一个索引的最左前缀可以通过使用优化器来查找行。  
 【4】查询中与其它表关联的字，字段常常建立了外键关系  
 【5】查询中统计或分组统计的字段  
 select max\(hbs\_bh\) from zl\_yhjbqk  
 select qc\_bh,count\(\*\) from zl\_yhjbqk group by qc\_bh

**七、注意事项**

1，创建索引

对于查询占主要的应用来说，索引显得尤为重要。很多时候性能问题很简单的就是因为我们忘了添加索引而造成的，或者说没有添加更为有效的索引导致。如果不加  
 索引的话，那么查找任何哪怕只是一条特定的数据都会进行一次全表扫描，如果一张表的数据量很大而符合条件的结果又很少，那么不加索引会引起致命的性能下降。但是也不是什么情况都非得建索引不可，比如性别可能就只有两个值，建索引不仅没什么优势，还会影响到更新速度，这被称为过度索引。

2，复合索引

比如有一条语句是这样的：select \* from users where area=’beijing’ and age=22;

如果我们是在area和age上分别创建单个索引的话，由于mysql查询每次只能使用一个索引，所以虽然这样已经相对不做索引时全表扫描提高了很多效率，但是如果在area、age两列上创建复合索引的话将带来更高的效率。如果我们创建了\(area, age,salary\)的复合索引，那么其实相当于创建了\(area,age,salary\)、\(area,age\)、\(area\)三个索引，这被称为最佳左前缀特性。  
 因此我们在创建复合索引时应该将最常用作限制条件的列放在最左边，依次递减。

3，索引不会包含有NULL值的列

只要列中包含有NULL值都将不会被包含在索引中，复合索引中只要有一列含有NULL值，那么这一列对于此复合索引就是无效的。所以我们在数据库设计时不要让字段的默认值为NULL。

4，使用短索引

对串列进行索引，如果可能应该指定一个前缀长度。例如，如果有一个CHAR\(255\)的 列，如果在前10 个或20 个字符内，多数值是惟一的，那么就不要对整个列进行索引。短索引不仅可以提高查询速度而且可以节省磁盘空间和I/O操作。

5，排序的索引问题

mysql查询只使用一个索引，因此如果where子句中已经使用了索引的话，那么order by中的列是不会使用索引的。因此数据库默认排序可以符合要求的情况下不要使用排序操作；尽量不要包含多个列的排序，如果需要最好给这些列创建复合索引。

6，like语句操作

一般情况下不鼓励使用like操作，如果非使用不可，如何使用也是一个问题。like “%aaa%” 不会使用索引，而like “aaa%”可以使用索引。

7，不要在列上进行运算

8，不使用NOT IN操作

NOT IN操作不会使用索引将进行全表扫描。NOT IN可以用NOT EXISTS代替  
  


