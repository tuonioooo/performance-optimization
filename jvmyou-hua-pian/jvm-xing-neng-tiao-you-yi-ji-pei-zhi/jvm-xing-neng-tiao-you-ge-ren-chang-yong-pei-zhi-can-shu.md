# JVM性能调优——个人常用配置参数

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







