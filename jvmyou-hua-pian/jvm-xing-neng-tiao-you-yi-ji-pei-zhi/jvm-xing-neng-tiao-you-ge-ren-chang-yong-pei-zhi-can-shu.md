# JVM性能调优——常用配置

#### 查看当前堆内存的使用情况

```text
jmap -heap 17586
```

#### 每过5s查看 进程 10735 gc的回收情况

```text
jstat -gc 10735 5000
```

#### 从内存中导出dump信息到heap.bin中，“pid” 进程标识号

```text
jmap -dump:live,format=b,file=heap.bin pid
```

#### 从内存中导出dump信息到jconsole.dump/hprof中， “pid” 进程标识号

```text
jmap -dump:format=b,file=jconsole.dump pid
jmap -dump:format=b,file=jconsole.hprof pid
```

#### 查看类加载的情况

```text
jmap -clstats pid
```

#### 每隔1秒查看gc的回收的原因

```text
jstat -gccause pid 1000
```

#### 垃圾回收器CMS的JVM参数配置

```text
-Xms4g -Xmx4g -Xmn2g -Xss1024K -XX:MetaspaceSize=512m -XX:MaxMetaspaceSize=1024m -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC -XX:+UseCMSCompactAtFullCollection -XX:CMSInitiatingOccupancyFraction=80
```

#### 目前我使用的GC的JVM参数配置

```text
java -XX:+UseG1GC -Xms4g -Xmx4g -server -XX:MaxMetaspaceSize=2g -XX:SoftRefLRUPolicyMSPerMB=200 -XX:ParallelGCThreads=16 -XX:ConcGCThreads=8 -XX:+HeapDumpOnOutOfMemoryError  -XX:HeapDumpPath=./tmp  -Dapp.path=%BASE_PATH% -Dspring.profiles.active=%APP_PROFILE% -jar app.jar
```

#### 可以通过下面的参数打Heap Dump信息

* -XX:HeapDumpPath
* -XX:+PrintGCDetails
* -XX:+PrintGCTimeStamps
* -Xloggc:/usr/aaa/dump/heap\_trace.txt

    通过下面参数可以控制OutOfMemoryError时打印堆的信息

* -XX:+HeapDumpOnOutOfMemoryError

```text
-server -Xms3G -Xmx3G -Xss256k -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+UseParallelOldGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/aaa/dump -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/usr/aaa/dump/heap_trace.txt -XX:NewSize=1G -XX:MaxNewSize=1G
```

**一个性能较好的web服务器jvm参数配置：**

```text
-server//服务器模式
-Xmx2g //JVM最大允许分配的堆内存，按需分配
-Xms2g //JVM初始分配的堆内存，一般和Xmx配置成一样以避免每次gc后JVM重新分配内存。
-Xmn256m //年轻代内存大小，整个JVM内存=年轻代 + 年老代 + 持久代
-XX:PermSize=128m //持久代内存大小
-Xss256k //设置每个线程的堆栈大小
-XX:+DisableExplicitGC //忽略手动调用GC, System.gc()的调用就会变成一个空调用，完全不触发GC
-XX:+UseConcMarkSweepGC //并发标记清除（CMS）收集器
-XX:+CMSParallelRemarkEnabled //降低标记停顿
-XX:+UseCMSCompactAtFullCollection //在FULL GC的时候对年老代的压缩
-XX:LargePageSizeInBytes=128m //内存页的大小
-XX:+UseFastAccessorMethods //原始类型的快速优化
-XX:+UseCMSInitiatingOccupancyOnly //使用手动定义初始化定义开始CMS收集
-XX:CMSInitiatingOccupancyFraction=70 //使用cms作为垃圾回收使用70％后开始CMS收集
```



