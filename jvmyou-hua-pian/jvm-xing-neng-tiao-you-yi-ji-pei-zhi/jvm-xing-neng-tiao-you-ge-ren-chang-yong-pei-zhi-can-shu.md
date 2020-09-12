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









