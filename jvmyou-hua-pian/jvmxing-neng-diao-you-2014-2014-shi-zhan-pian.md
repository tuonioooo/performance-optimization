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

