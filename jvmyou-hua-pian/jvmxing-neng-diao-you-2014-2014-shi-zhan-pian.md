# JVM性能调优——实战篇

## 一、内部集成构建服务器案例

高性能数据处理的工具应用

服务器配置：1 CPU, 4G MEM, JDK 1.6.X

参数方案：

```
-server -XX:PermSize=196m -XX:MaxPermSize=196m -Xmn320m -Xms768m -Xmx1024m
```

调优说明：

```
-XX:PermSize=196m -XX:MaxPermSize=196m 根据集成构建的特点，大规模的系统编译可能需要加载大量的Java类到内存中，所以预先分配好大量的持久代内存是高效和必要的。
-Xmn320m 遵循年轻代大小为整个堆的3/8原则。
-Xms768m -Xmx1024m 根据系统大致能够承受的堆内存大小设置即可。
```

在64位服务器上运行应用程序，构建执行时，用 jmap -heap 11540 命令观察JVM堆内存状况如下：

```
Attaching to process ID 11540, please wait...
Debugger attached successfully.
Server compiler detected.
JVM version is 20.12-b01
using thread-local object allocation.
Parallel GC with 4 thread(s)
Heap Configuration:
   MinHeapFreeRatio = 40
   MaxHeapFreeRatio = 70
   MaxHeapSize      = 1073741824 (1024.0MB)
   NewSize          = 335544320 (320.0MB)
   MaxNewSize       = 335544320 (320.0MB)
   OldSize          = 5439488 (5.1875MB)
   NewRatio         = 2
   SurvivorRatio    = 8
   PermSize         = 205520896 (196.0MB)
   MaxPermSize      = 205520896 (196.0MB)
Heap Usage:
PS Young Generation
Eden Space:
   capacity = 255852544 (244.0MB)
   used     = 101395504 (96.69828796386719MB)
   free     = 154457040 (147.3017120361328MB)
   39.63044588683081% used
From Space:
   capacity = 34144256 (32.5625MB)
   used     = 33993968 (32.41917419433594MB)
   free     = 150288 (0.1433258056640625MB)
   99.55984397492803% used
To Space:
   capacity = 39845888 (38.0MB)
   used     = 0 (0.0MB)
   free     = 39845888 (38.0MB)
   0.0% used
PS Old Generation
   capacity = 469762048 (448.0MB)
   used     = 44347696 (42.29325866699219MB)
   free     = 425414352 (405.7067413330078MB)
   9.440459523882184% used
PS Perm Generation
   capacity = 205520896 (196.0MB)
   used     = 85169496 (81.22396087646484MB)
   free     = 120351400 (114.77603912353516MB)
   41.440796365543285% used
```

结果是比较健康的。

## 二、-Xmn，-XX:NewSize/-XX:MaxNewSize，-XX:NewRatio 3组参数都可以影响年轻代的大小，混合使用的情况下，优先级是什么？

如下：

1. 高优先级：-XX:NewSize/-XX:MaxNewSize 
2. 中优先级：-Xmn（默认等效  -Xmn=-XX:NewSize=-XX:MaxNewSize=?） 
3. 低优先级：-XX:NewRatio 

## 三、一些常见的示例

**实例1：**

笔者昨日发现部分开发测试机器出现异常：java.lang.OutOfMemoryError: GC overhead limit exceeded，这个异常代表：

GC为了释放很小的空间却耗费了太多的时间，其原因一般有两个：1，堆太小，2，有死循环或大对象；

笔者首先排除了第2个原因，因为这个应用同时是在线上运行的，如果有问题，早就挂了。所以怀疑是这台机器中堆设置太小；

使用ps -ef \|grep "java"查看，发现：

![](http://dl2.iteye.com/upload/attachment/0101/0366/af00bbf9-50dc-3127-a917-e78aced45e01.png)

该应用的堆区设置只有768m，而机器内存有2g，机器上只跑这一个java应用，没有其他需要占用内存的地方。另外，这个应用比较大，需要占用的内存也比较多；

笔者通过上面的情况判断，只需要改变堆中各区域的大小设置即可，于是改成下面的情况：

![](http://dl2.iteye.com/upload/attachment/0101/0368/196da884-1c12-3b4e-9d5a-08300587d5e4.png)

跟踪运行情况发现，相关异常没有再出现；

**实例2：**（[http://www.360doc.com/content/13/0305/10/15643\_269388816.shtml）](http://www.360doc.com/content/13/0305/10/15643_269388816.shtml）)

一个服务系统，**经常出现卡顿，分析原因，发现Full GC时间太长**：

jstat -gcutil:

S0     S1    E     O       P        YGC YGCT FGC FGCT  GCT

12.16 0.00 5.18 63.78 20.32  54   2.047 5     6.946  8.993

分析上面的数据，发现Young GC执行了54次，耗时2.047秒，每次Young GC耗时37ms，在正常范围，**而Full GC执行了5次，耗时6.946秒，每次平均1.389s，数据显示出来的问题是：Full GC耗时较长**，分析该系统的是指发现，NewRatio=9，也就是说，新生代和老生代大小之比为1:9，这就是问题的原因：

1，新生代太小，导致对象提前进入老年代，触发老年代发生Full GC；

2，老年代较大，进行Full GC时耗时较大；

优化的方法是调整NewRatio的值，调整到4，发现Full GC没有再发生，只有Young GC在执行。这就是把对象控制在新生代就清理掉，没有进入老年代（这种做法对一些应用是很有用的，但并不是对所有应用都要这么做）

| 名称 | 主要作用 |
| :--- | :--- |
| jps | JVM Process Status Tool, 显示指定系统内所有的HotSpot虚拟机进程 |
| jstat | JVM Statistics Monitoring Tool , 用于收集HotSpot虚拟机各方面的运行数据 |
| jinfo | Configuration Info for Java，显示虚拟机配置信息 |
| Jmap | Memory Map for Java ，生成虚拟机的内存转储快照（heapdump文件） |
| jhat | JVM Heap Dump Browser，用于分析heapdump文件，它会建立一个HTTP/HTML服务器，让用户在浏览器上看到分析结果。 |
| jstack | Stack Trace for Java, 显示虚拟机的线程快照 |

**实例3：**

一应用在性能测试过程中，发现内存占用率很高，Full GC频繁，使用sudo -u admin -H  jmap -dump:format=b,file=文件名.hprof pid 来dump内存，生成dump文件，并使用Eclipse下的mat差距进行分析，发现：

![](http://dl2.iteye.com/upload/attachment/0101/0370/a61f0f4e-9d89-35a9-b7c9-501b52b2de08.png)

从图中可以看出，这个线程存在问题，队列LinkedBlockingQueue所引用的大量对象并未释放，导致整个线程占用内存高达378m，此时通知开发人员进行代码优化，将相关对象释放掉即可。



实例4：在Tomcat下优化JVM参数

1、 tomcat7安装目录\bin\catalina.bat \(linux修改的是catalina.sh文件\)

添加如下语句：

JAVA\_OPTS=-Djava.awt.headless=true -Dfile.encoding=UTF-8 -server -Xms1024m -Xmx1024m -Xss1m -XX:NewSize=256m -XX:MaxNewSize=512m -XX:PermSize=256M -XX:MaxPermSize=512m

-XX:+DisableExplicitGC

2、查看tomcat的JVM内存

tomcat7中默认没有用户的，我们首先要添加用户有：

修改tomcat7安装目录下\conf\tomcat-users.xml

password是可以自由定义的。 

3、检查webapps下是否有Manager目录，一般发布时我们都把这个目录删除了，现在看来删除早了，在调试期要保留啊！ 

4、访问地址： http://localhost:8080/manager/status 查看内存配置情况，经测试-Xms512m -Xmx512m与-Xms1024m -Xmx1024m内存使用情况不一样，使用1024的时候有一项内存使用99%。所以看来这个设置多少与实际机器有关，需要Manager进行查看后确定。

 5、在启动Tomcat中发现，有同志发布程序时把我们在TOMCAT7中引用的外部JAR包重复发布到LIB目录下了，我们以后在发布时要检查LIB下是不是包括 el-api.jar jsp-api servlet-api,特别注意的是最后一个servlet-api，我发现两个项目都把它拷贝到了LIB目录下！！被我删除了。 6、使用TOMAT的连接池：

说明： maxThreads：最大线程数 300 minSpareThreads：初始化建立的线程数 50 maxThreads：一旦线程超过这个值，Tomcat就会关闭不再需要的线程 maxIdleTime：为最大空闲时间、单位为毫秒。 executor为线程池的名字，对应Executor 中的name属性；Connector 标签中不再有maxThreads的设置。 如果tomcat不使用线程池则基本配置如下：

修改Tomcat的/conf目录下面的server.xml文件，针对端口为8080的连接器添加如下参数：

 1. connectionTimeout：连接失效时间，单位为毫秒、默认为60s、这里设置为30s，如果用户请求在30s内未能进入请求队列，视为本次连接失败。 

2. keepAliveTimeout：连接的存活时间，默认和connectionTimeout一致，这里可以设为15s、这意味着15s之后本次连接关闭. 如果页面需要加载大量图片、js等静态资源，需要将参数适当调大一点、以免多次创建TCP连接。 

3. enableLookups：是否对连接到服务器的远程机器查询其DNS主机名，一般情况下这并不必要，因此设为false即可。 

4. URIEncoding：设置URL参数的编码格式为UTF-8编码，默认为ISO-8859-1编码。 

5. maxHttpHeaderSize：设置HTTP请求、响应的头部内容大小，默认为8192字节\(8k\)，此处设置为32768字节\(32k\)、和Nginx的设置保持一致。 

6. maxThreads：最大线程数、用于处理用户请求的线程数目，默认为200、此处设置为300 

7. acceptCount：用户请求等候队列的大小，默认为100、此处设置为200 Linux系统默认一个进程能够创建的最大线程数为1024、因此对高并发应用需要进行Linux内核调优，至此文件server.xml修改后的内容如下所示：吻 再次登录查看状态，

