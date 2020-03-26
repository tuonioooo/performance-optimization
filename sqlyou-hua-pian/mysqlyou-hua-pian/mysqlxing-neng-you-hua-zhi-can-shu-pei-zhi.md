# MySQL性能优化之参数配置

### 1、目的：

通过根据服务器目前状况，修改Mysql的系统参数，达到合理利用服务器现有资源，最大合理的提高MySQL性能。

### 2、服务器参数：

32G内存、4个CPU,每个CPU 8核。

### 3、MySQL目前安装状况。

MySQL目前安装，用的是MySQL默认的最大支持配置。拷贝的是my-huge.cnf.编码已修改为UTF-8.具体修改及安装MySQL,可以参考&lt;&lt;Linux系统上安装MySQL 5.5&gt;&gt;帮助文档。

### 4、修改MySQL配置

打开MySQL配置文件my.cnf

| vi  /etc/my.cnf |
| :--- |


#### 4.1 MySQL非缓存参数变量介绍及修改

**4.1.1修改back\_log参数值:由默认的50修改为500.（每个连接256kb,占用：125M）**

 **back\_log=500**

> back\_log值指出在MySQL暂时停止回答新请求之前的短时间内多少个请求可以被存在堆栈中。也就是说，如果MySql的连接数据达到max\_connections时，新来的请求将会被存在堆栈中，以等待某一连接释放资源，该堆栈的数量即back\_log，如果等待连接的数量超过back\_log，将不被授予连接资源。将会报：unauthenticated user \| xxx.xxx.xxx.xxx \| NULL \| Connect \| NULL \| login \| NULL 的待连接进程时.
>
> back\_log值不能超过TCP/IP连接的侦听队列的大小。若超过则无效，查看当前系统的TCP/IP连接的侦听队列的大小命令：cat /proc/sys/net/ipv4/tcp\_max\_syn\_backlog目前系统为1024。对于Linux系统推荐设置为小于512的整数。
>
> 修改系统内核参数，）[http://www.51testing.com/html/64/n-810764.html](http://www.51testing.com/html/64/n-810764.html)
>
> 查看mysql 当前系统默认back\_log值，命令： show variables like 'back\_log'; 查看当前数量

**4.1.2修改wait\_timeout参数值，由默认的8小时，修改为30分钟。\(本次不用\)**

 **wait\_timeout=1800（单位为妙）**

| 我对wait-timeout这个参数的理解：MySQL客户端的数据库连接闲置最大时间值。说得比较通俗一点，就是当你的MySQL连接闲置超过一定时间后将会被强行关闭。MySQL默认的wait-timeout  值为8个小时，可以通过命令show variables like 'wait\_timeout'查看结果值;。设置这个值是非常有意义的，比如你的网站有大量的MySQL链接请求（每个MySQL连接都是要内存资源开销的 ），由于你的程序的原因有大量的连接请求空闲啥事也不干，白白占用内存资源，或者导致MySQL超过最大连接数从来无法新建连接导致“Too many connections”的错误。在设置之前你可以查看一下你的MYSQL的状态（可用show processlist\)，如果经常发现MYSQL中有大量的Sleep进程，则需要 修改wait-timeout值了。interactive\_timeout：服务器关闭交互式连接前等待活动的秒数。交互式客户端定义为在mysql\_real\_connect\(\)中使用CLIENT\_INTERACTIVE选项的客户端。wait\_timeout:服务器关闭非交互连接之前等待活动的秒数。在线程启动时，根据全局wait\_timeout值或全局 interactive\_timeout值初始化会话wait\_timeout值，取决于客户端类型\(由mysql\_real\_connect\(\)的连接选项CLIENT\_INTERACTIVE定义\).这两个参数必须配合使用。否则单独设置wait\_timeout无效 |
| :--- |


**4.1.3修改max\_connections参数值，由默认的151，修改为3000（750M）。**

 **max\_connections=3000**

| max\_connections是指MySql的最大连接数，如果服务器的并发连接请求量比较大，建议调高此值，以增加并行连接数量，当然这建立在机器能支撑的情况下，因为如果连接数越多，介于MySql会为每个连接提供连接缓冲区，就会开销越多的内存，所以要适当调整该值，不能盲目提高设值。可以过'conn%'通配符查看当前状态的连接数量，以定夺该值的大小。MySQL服务器允许的最大连接数16384；查看系统当前最大连接数：show variables like 'max\_connections'; |
| :--- |


**4.1..4修改max\_user\_connections值，由默认的0，修改为800**

 **max\_user\_connections=800**

| max\_user\_connections是指每个数据库用户的最大连接针对某一个账号的所有客户端并行连接到MYSQL服务的最大并行连接数。简单说是指同一个账号能够同时连接到mysql服务的最大连接数。设置为0表示不限制。目前默认值为：0不受限制。这儿顺便介绍下Max\_used\_connections:它是指从这次mysql服务启动到现在，同一时刻并行连接数的最大值。它不是指当前的连接情况，而是一个比较值。如果在过去某一个时刻，MYSQL服务同时有1000个请求连接过来，而之后再也没有出现这么大的并发请求时，则Max\_used\_connections=1000.请注意与show variables 里的max\_user\_connections的区别。默认为0表示无限大。查看**max\_user\_connections值**show variables like '**max\_user\_connections**'; |
| :--- |


**4.1.5修改thread\_concurrency值，由目前默认的8，修改为64**

 **thread\_concurrency=64**

| thread\_concurrency的值的正确与否, 对mysql的性能影响很大, 在多个cpu\(或多核\)的情况下，错误设置了thread\_concurrency的值, 会导致mysql不能充分利用多cpu\(或多核\), 出现同一时刻只能一个cpu\(或核\)在工作的情况。thread\_concurrency应设为**CPU核数的2倍**. 比如有一个双核的CPU, 那thread\_concurrency  的应该为4; 2个双核的cpu, thread\_concurrency的值应为8.比如：根据上面介绍我们目前系统的配置，可知道为4个CPU,每个CPU为8核，按照上面的计算规则，这儿应为:4\*8\*2=64查看系统当前**thread\_concurrency**默认配置命令： show variables like 'thread\_concurrency'; |
| :--- |


**4.1.6添加skip-name-resolve，默认被注释掉，没有该参数。**

**skip-name-resolve**

| skip-name-resolve：禁止MySQL对外部连接进行DNS解析，使用这一选项可以消除MySQL进行DNS解析的时间。但需要注意，如果开启该选项，则所有远程主机连接授权都要使用IP地址方式，否则MySQL将无法正常处理连接请求！ |
| :--- |


**4.1.7 skip-networking，默认被注释掉。没有该参数。（本次无用）**

 **skip-networking建议被注释掉，不要开启**

| 开启该选项可以彻底关闭MySQL的TCP/IP连接方式，如果WEB服务器是以远程连接的方式访问MySQL数据库服务器则不要开启该选项！否则将无法正常连接！ |
| :--- |


**4.1.8  default-storage-engine\(设置MySQL的默认存储引擎\)**

default-storage-engine= InnoDB\(设置InnoDB类型，另外还可以设置MyISAM类型\)

| 设置创建数据库及表默认存储类型show table status like ‘tablename’显示表的当前存储状态值查看MySQL有哪些存储状态及默认存储状态 show engines;创建表并指定存储类型CREATE TABLE mytable \(id int, title char\(20\)\) ENGINE = INNODB;**修改表存储类型：**  Alter table tableName engine =engineName备注：设置完后把以下几个开启：\# Uncomment the following if you are using InnoDB tablesinnodb\_data\_home\_dir = /var/lib/mysql\#innodb\_data\_file\_path = ibdata1:1024M;ibdata2:10M:autoextend（要注释掉，否则会创建一个新的把原来的替换的。）innodb\_log\_group\_home\_dir = /var/lib/mysql\# You can set ..\_buffer\_pool\_size up to 50 - 80 %\# of RAM but beware of setting memory usage too highinnodb\_buffer\_pool\_size = 1000Minnodb\_additional\_mem\_pool\_size = 20M\# Set ..\_log\_file\_size to 25 % of buffer pool sizeinnodb\_log\_file\_size = 500Minnodb\_log\_buffer\_size = 20Minnodb\_flush\_log\_at\_trx\_commit = 0innodb\_lock\_wait\_timeout = 50设置完后一定记得把MySQL安装目录地址（我们目前是默认安装所以地址/var/lib/mysql/）下的ib\_logfile0和ib\_logfile1删除掉。否则重启MySQL起动失败。 |
| :--- |


#### 4.2 MySQL缓存变量介绍及修改

| 数据库属于IO密集型的应用程序，其主职责就是数据的管理及存储工作。而我们知道，从内存中读取一个数据库的时间是微秒级别，而从一块普通硬盘上读取一个 IO是在毫秒级别，二者相差3个数量级。所以，要优化数据库，首先第一步需要优化的就是IO，尽可能将磁盘IO转化为内存IO。本文先从MySQL数据库 IO相关参数\(缓存参数\)的角度来看看可以通过哪些参数进行IO优化 |
| :--- |


**4.2.1全局缓存**

| 启动MySQL时就要分配并且总是存在的全局缓存。目前有：key\_buffer\_size\(默认值：402653184,即384M\)、innodb\_buffer\_pool\_size\(默认值：134217728即：128M\)、innodb\_additional\_mem\_pool\_size（默认值：8388608即：8M）、innodb\_log\_buffer\_size\(默认值：8388608即：8M\)、query\_cache\_size\(默认值：33554432即：32M\)等五个。总共：560M.这些变量值都可以通过命令如：show variables like '变量名';查看到。 |
| :--- |


**4.2.1.1：key\_buffer\_size,本系统目前为384M,可修改为400M**

```text
key_buffer_size=400M
```

| key\_buffer\_size是用于索引块的缓冲区大小，增加它可得到更好处理的索引\(对所有读和多重写\)，对MyISAM\(MySQL表存储的一种类型，可以百度等查看详情\)表性能影响最大的一个参数。如果你使它太大，系统将开始换页并且真的变慢了。严格说是它决定了数据库索引处理的速度，尤其是索引读的速度。对于内存在4GB左右的服务器该参数可设置为256M或384M.怎么才能知道key\_buffer\_size的设置是否合理呢，一般可以检查状态值Key\_read\_requests和Key\_reads   ，比例key\_reads / key\_read\_requests应该尽可能的低，比如1:100，1:1000 ，1:10000。其值可以用以下命令查得：show status like 'key\_read%';比如查看系统当前key\_read和key\_read\_request值为：+-------------------+-------+\| Variable\_name     \| Value \|+-------------------+-------+\| Key\_read\_requests \| 28535 \|\| Key\_reads         \| 269   \|+-------------------+-------+可知道有28535个请求，有269个请求在内存中没有找到直接从硬盘读取索引.未命中缓存的概率为：0.94%=269/28535\*100%.  一般未命中概率在0.1之下比较好。目前已远远大于0.1，证明效果不好。若命中率在0.01以下，则建议适当的修改key\_buffer\_size值。[http://dbahacker.com/mysql/innodb-myisam-compare\(\[InnoDB与MyISAM的六大区别\]\(http://dbahacker.com/mysql/innodb-myisam-compare\)\)http://kb.cnblogs.com/page/99810/（查看存储引擎介绍）MyISAM、InnoDB、MyISAM](http://dbahacker.com/mysql/innodb-myisam-compare%28[InnoDB与MyISAM的六大区别]%28http://dbahacker.com/mysql/innodb-myisam-compare%29%29http://kb.cnblogs.com/page/99810/（查看存储引擎介绍）MyISAM、InnoDB、MyISAM) Merge引擎、InnoDB、memory\(heap\)、archive |
| :--- |


**4.2.1.2：innodb\_buffer\_pool\_size\(默认128M\)**

innodb\_buffer\_pool\_size=1024M\(1G\)

| innodb\_buffer\_pool\_size:主要针对InnoDB表性能影响最大的一个参数。功能与Key\_buffer\_size一样。InnoDB占用的内存，除innodb\_buffer\_pool\_size用于存储页面缓存数据外，另外正常情况下还有大约8%的开销，主要用在每个缓存页帧的描述、adaptive hash等数据结构，如果不是安全关闭，启动时还要恢复的话，还要另开大约12%的内存用于恢复，两者相加就有差不多21%的开销。假设：12G的innodb\_buffer\_pool\_size，最多的时候InnoDB就可能占用到14.5G的内存。若系统只有16G，而且只运行MySQL，且MySQL只用InnoDB，那么为MySQL开12G，是最大限度地利用内存了。另外InnoDB和 MyISAM 存储引擎不同， MyISAM 的 key\_buffer\_size 只能缓存索引键，而 innodb\_buffer\_pool\_size 却可以缓存数据块和索引键。适当的增加这个参数的大小，可以有效的减少 InnoDB 类型的表的磁盘 I/O 。当我们操作一个 InnoDB 表的时候，返回的所有数据或者去数据过程中用到的任何一个索引块，都会在这个内存区域中走一遭。可以通过 \(Innodb\_buffer\_pool\_read\_requests – Innodb\_buffer\_pool\_reads\) / Innodb\_buffer\_pool\_read\_requests \* 100% 计算缓存命中率，并根据命中率来调整 innodb\_buffer\_pool\_size 参数大小进行优化。值可以用以下命令查得：show status like 'Innodb\_buffer\_pool\_read%';比如查看当前系统中系统中\| Innodb\_buffer\_pool\_read\_requests      \| 1283826 \|\| Innodb\_buffer\_pool\_reads              \| 519     \|+---------------------------------------+---------+其命中率99.959%=（1283826-519）/1283826\*100%  命中率越高越好。 |
| :--- |


**4.2.1.3：innodb\_additional\_mem\_pool\_size\(默认8M\)**

innodb\_additional\_mem\_pool\_size=20M

| innodb\_additional\_mem\_pool\_size 设置了InnoDB存储引擎用来存放数据字典信息以及一些内部数据结构的内存空间大小，所以当我们一个MySQL Instance中的数据库对象非常多的时候，是需要适当调整该参数的大小以确保所有数据都能存放在内存中提高访问效率的。这个参数大小是否足够还是比较容易知道的，因为当过小的时候，MySQL会记录Warning信息到数据库的error log中，这时候你就知道该调整这个参数大小了。查看当前系统mysql的error日志  cat  /var/lib/mysql/机器名.error 发现有很多waring警告。所以要调大为20M.根据MySQL手册，对于2G内存的机器，推荐值是20M。    32G内存的 100M |
| :--- |


**4.2.1.4：innodb\_log\_buffer\_size\(默认8M\)**

innodb\_log\_buffer\_size=20M

| innodb\_log\_buffer\_size  这是InnoDB存储引擎的事务日志所使用的缓冲区。类似于Binlog Buffer，InnoDB在写事务日志的时候，为了提高性能，也是先将信息写入Innofb Log Buffer中，当满足innodb\_flush\_log\_trx\_commit参数所设置的相应条件\(或者日志缓冲区写满\)之后，才会将日志写到文件 \(或者同步到磁盘\)中。可以通过innodb\_log\_buffer\_size 参数设置其可以使用的最大内存空间。   InnoDB 将日志写入日志磁盘文件前的缓冲大小。理想值为 1M 至 8M。大的日志缓冲允许事务运行时不需要将日志保存入磁盘而只到事务被提交\(commit\)。 因此，如果有大的事务处理，设置大的日志缓冲可以减少磁盘I/O。 在 my.cnf中以数字格式设置。默认是8MB，系的如频繁的系统可适当增大至4MB～8MB。当然如上面介绍所说，这个参数实际上还和另外的flush参数相关。一般来说不建议超过32MB注：innodb\_flush\_log\_trx\_commit参数对InnoDB Log的写入性能有非常关键的影响,默认值为1。该参数可以设置为0，1，2，解释如下：0：log buffer中的数据将以每秒一次的频率写入到log file中，且同时会进行文件系统到磁盘的同步操作，但是每个事务的commit并不会触发任何log buffer 到log file的刷新或者文件系统到磁盘的刷新操作;1：在每次事务提交的时候将log buffer 中的数据都会写入到log file，同时也会触发文件系统到磁盘的同步;2：事务提交会触发log buffer到log file的刷新，但并不会触发磁盘文件系统到磁盘的同步。此外，每秒会有一次文件系统到磁盘同步操作。实际测试发现，该值对插入数据的速度影响非常大，设置为2时插入10000条记录只需要2秒，设置为0时只需要1秒，而设置为1时则需要229秒。因此，MySQL手册也建议尽量将插入操作合并成一个事务，这样可以大幅提高速度。根据MySQL手册，在存在丢失最近部分事务的危险的前提下，可以把该值设为0。 |
| :--- |


**4.5.1.5：query\_cache\_size\(默认32M\)**

query\_cache\_size=40M

| query\_cache\_size: 主要用来缓存MySQL中的ResultSet，也就是一条SQL语句执行的结果集，所以仅仅只能针对select语句。当我们打开了 Query Cache功能，MySQL在接受到一条select语句的请求后，如果该语句满足Query Cache的要求\(未显式说明不允许使用Query Cache，或者已经显式申明需要使用Query Cache\)，MySQL会直接根据预先设定好的HASH算法将接受到的select语句以字符串方式进行hash，然后到Query Cache中直接查找是否已经缓存。也就是说，如果已经在缓存中，该select请求就会直接将数据返回，从而省略了后面所有的步骤\(如SQL语句的解析，优化器优化以及向存储引擎请求数据等\)，极大的提高性能。根据MySQL用户手册，使用查询缓冲最多可以达到238%的效率。当然，Query Cache也有一个致命的缺陷，那就是当某个表的数据有任何任何变化，都会导致所有引用了该表的select语句在Query Cache中的缓存数据失效。所以，当我们的数据变化非常频繁的情况下，使用Query Cache可能会得不偿失   Query Cache的使用需要多个参数配合，其中最为关键的是query\_cache\_size和query\_cache\_type，前者设置用于缓存 ResultSet的内存大小，后者设置在何场景下使用Query Cache。在以往的经验来看，如果不是用来缓存基本不变的数据的MySQL数据库，query\_cache\_size一般256MB是一个比较合适的大小。当然，这可以通过计算Query Cache的命中率\(Qcache\_hits/\(Qcache\_hits+Qcache\_inserts\)\*100\)\)来进行调整。 query\_cache\_type可以设置为0\(OFF\)，1\(ON\)或者2\(DEMOND\)，分别表示完全不使用query cache，除显式要求不使用query cache\(使用sql\_no\_cache\)之外的所有的select都使用query cache，只有显示要求才使用query cache\(使用sql\_cache\)。如果Qcache\_lowmem\_prunes的值非常大，则表明经常出现缓冲. 如果Qcache\_hits的值也非常大，则表明查询缓冲使用非常频繁，此时需要增加缓冲大小；根据命中率\(Qcache\_hits/\(Qcache\_hits+Qcache\_inserts\)\*100\)\)进行调整，一般不建议太大，256MB可能已经差不多了，大型的配置型静态数据可适当调大.可以通过命令：show status like 'Qcache\_%';查看目前系统Query catch使用大小\| Qcache\_hits             \| 1892463  \|\| Qcache\_inserts          \| 35627  命中率98.17%=1892463/\(1892463 +35627 \)\*100 |
| :--- |


**4.2.2局部缓存**

| 除了全局缓冲，MySql还会为每个连接发放连接缓冲。个连接到MySQL服务器的线程都需要有自己的缓冲。大概需要立刻分配256K，甚至在线程空闲时，它们使用默认的线程堆栈，网络缓存等。事务开始之后，则需要增加更多的空间。运行较小的查询可能仅给指定的线程增加少量的内存消耗，然而如果对数据表做复杂的操作例如扫描、排序或者需要临时表，则需分配大约read\_buffer\_size，sort\_buffer\_size，read\_rnd\_buffer\_size，tmp\_table\_size 大小的内存空间. 不过它们只是在需要的时候才分配，并且在那些操作做完之后就释放了。有的是立刻分配成单独的组块。tmp\_table\_size 可能高达MySQL所能分配给这个操作的最大内存空间了。注意，这里需要考虑的不只有一点——可能会分配多个同一种类型的缓存，例如用来处理子查询。一些特殊的查询的内存使用量可能更大——如果在MyISAM表上做成批的插入时需要分配 bulk\_insert\_buffer\_size 大小的内存；执行 ALTER TABLE， OPTIMIZE TABLE， REPAIR TABLE 命令时需要分配 myisam\_sort\_buffer\_size 大小的内存。 |
| :--- |


**4.2.2.1：read\_buffer\_size（默认值：2097144即2M）**

read\_buffer\_size=4M

| read\_buffer\_size 是MySql读入缓冲区大小。对表进行顺序扫描的请求将分配一个读入缓冲区，MySql会为它分配一段内存缓冲区。read\_buffer\_size变量控制这一缓冲区的大小。如果对表的顺序扫描请求非常频繁，并且你认为频繁扫描进行得太慢，可以通过增加该变量值以及内存缓冲区大小提高其性能. |
| :--- |


**4.2.2.2：sort\_buffer\_size（默认值：2097144即2M）**

sort\_buffer\_size=4M

| sort\_buffer\_size是MySql执行排序使用的缓冲大小。如果想要增加ORDER BY的速度，首先看是否可以让MySQL使用索引而不是额外的排序阶段。如果不能，可以尝试增加sort\_buffer\_size变量的大小 |
| :--- |


**4.2.2.3: read\_rnd\_buffer\_size\(默认值：8388608即8M\)**

read\_rnd\_buffer\_size=8M

| read\_rnd\_buffer\_size 是MySql的随机读缓冲区大小。当按任意顺序读取行时\(例如，按照排序顺序\)，将分配一个随机读缓存区。进行排序查询时，MySql会首先扫描一遍该缓冲，以避免磁盘搜索，提高查询速度，如果需要排序大量数据，可适当调高该值。但MySql会为每个客户连接发放该缓冲空间，所以应尽量适当设置该值，以避免内存开销过大。 |
| :--- |


**4.2.2.4: tmp\_table\_size\(默认值：8388608 即：16M\)**

tmp\_table\_size=16M

| tmp\_table\_size是MySql的heap （堆积）表缓冲大小。所有联合在一个DML指令内完成，并且大多数联合甚至可以不用临时表即可以完成。大多数临时表是基于内存的\(HEAP\)表。具有大的记录长度的临时表 \(所有列的长度的和\)或包含BLOB列的表存储在硬盘上。如果某个内部heap（堆积）表大小超过tmp\_table\_size，MySQL可以根据需要自动将内存中的heap表改为基于硬盘的MyISAM表。还可以通过设置tmp\_table\_size选项来增加临时表的大小。也就是说，如果调高该值，MySql同时将增加heap表的大小，可达到提高联接查询速度的效果。 |
| :--- |


**4.2.2.5：record\_buffer:（默认值：）**

| record\_buffer每个进行一个顺序扫描的线程为其扫描的每张表分配这个大小的一个缓冲区。如果你做很多顺序扫描，你可能想要增加该值。默认数值是131072\(128K\) |
| :--- |


**4.2.3其它缓存：**

**4.2.3.1：table\_cache\(默认值：512\)**

| TABLE\_CACHE\(5.1.3及以后版本又名TABLE\_OPEN\_CACHE\)**table\_cache**指定表高速缓存的大小。每当MySQL访问一个表时，如果在表缓冲区中还有空间，该表就被打开并放入其中，这样可以更快地访问表内容。通过检查峰值时间的状态值Open\_tables和Opened\_tables，可以决定是否需要增加table\_cache的值。如果你发现open\_tables等于table\_cache，并且opened\_tables在不断增长，那么你就需要增加table\_cache的值了（上述状态值可以使用**SHOW STATUS LIKE ‘Open%tables’**获得）。注意，不能盲目地把table\_cache设置成很大的值。如果设置得太高，可能会造成文件描述符不足，从而造成性能不稳定或者连接失败。SHOW STATUS LIKE 'Open%tables';+---------------+-------+\| Variable\_name \| Value \|+---------------+-------+\| Open\_tables   \| 356   \|\| Opened\_tables \| 0     \|+---------------+-------+2 rows in set \(0.00 sec\)open\_tables表示当前打开的表缓存数，如果执行flush tables操作，则此系统会关闭一些当前没有使用的表缓存而使得此状态值减小；opend\_tables表示曾经打开的表缓存数，会一直进行累加，如果执行flush tables操作，值不会减小。在mysql默认安装情况下，table\_cache的值在2G内存以下的机器中的值默认时256到512，如果机器有4G内存,则默认这个值 是2048，但这决意味着机器内存越大，这个值应该越大，因为table\_cache加大后，使得mysql对SQL响应的速度更快了，不可避免的会产生 更多的死锁（dead lock），这样反而使得数据库整个一套操作慢了下来，严重影响性能。所以平时维护中还是要根据库的实际情况去作出判断，找到最适合你维护的库的 table\_cache值。由于MySQL是多线程的机制,为了提高性能,每个线程都是独自打开自己需要的表的文件描 述符,而不是通过共享已经打开的.针对不同存储引擎处理的方法当然也不一样在myisam表引擎中,数据文件的描述符 \(descriptor\)是不共享的,但是索引文件的描述符却是所有线程共享的.Innodb中和使用表空间类型有关,假如是共享表空间那么实际就一个数 据文件,当然占用的数据文件描述符就会比独立表空间少. mysql手册上给的建议大小 是:table\_cache=`max_connections*nn表示查询语句中最大表数, 还需要为临时表和文件保留一些额外的文件描述符。`这个数据遭到很多质疑,table\_cache够用就好,检查 Opened\_tables值,如果这个值很大,或增长很快那么你就得考虑加大table\_cache了.  table\_cache：所有线程打开的表的数目。增大该值可以增加mysqld需要的文件描述符的数量。默认值是64. |
| :--- |


**4.2.3.2 thread\_cache\_size \(服务器线程缓存\)**

thread\_cache\_size=64

| 默认的thread\_cache\_size=8，但是看到好多配置的样例里的值一般是32，64，甚至是128，感觉这个参数对优化应该有帮助，于是查了下： 根据调查发现以上服务器线程缓存thread\_cache\_size没有进行设置，或者设置过小,这个值表示可以重新利用保存在缓存中线程的数量,当断开连接时如果缓存中还有空间,那么客户端的线程将被放到缓存中,如果线程重新被请求，那么请求将从缓存中读取,如果缓存中是空的或者是新的请求，那么这个线程将被重新创建,如果有很多新的线程，增加这个值可以改善系统性能.通过比较 Connections 和 Threads\_created 状态的变量，可以看到这个变量的作用。\(–&gt;表示要调整的值\)   根据物理内存设置规则如下： 1G —&gt; 8 2G —&gt; 16 3G —&gt; 32     &gt;3G —&gt; 64  mysql&gt; show status like 'thread%'; +——————-+——-+ \| Variable\_name     \| Value \| +——————-+——-+ \| Threads\_cached    \| 0     \|  &lt;—当前被缓存的空闲线程的数量 \| Threads\_connected \| 1     \|  &lt;—正在使用（处于连接状态）的线程 \| Threads\_created   \| 1498  \|  &lt;—服务启动以来，创建了多少个线程 \| Threads\_running   \| 1     \|  &lt;—正在忙的线程（正在查询数据，传输数据等等操作） +——————-+——-+查看开机起来数据库被连接了多少次？mysql&gt; show status like '%connection%'; +———————-+——-+ \| Variable\_name        \| Value \| +———————-+——-+ \| Connections          \| 1504  \|          –&gt;服务启动以来，历史连接数 \| Max\_used\_connections \| 2     \| +———————-+——-+**通过连接线程池的命中率来判断设置值是否合适？命中率超过90%以上,设定合理。** \(Connections -  Threads\_created\) / Connections \* 100 % |
| :--- |


三思而后行！

## 参考

[https://www.2cto.com/database/201711/695306.html](https://www.2cto.com/database/201711/695306.html)

[https://www.cnblogs.com/angryprogrammer/p/6667741.html](https://www.cnblogs.com/angryprogrammer/p/6667741.html)

