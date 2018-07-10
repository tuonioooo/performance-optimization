# 详细垃圾回收机制

JVM分别对新生代和旧生代采用不同的垃圾回收机制

新生代的GC：

新生代通常存活时间较短，因此基于Copying算法来进行回收，所谓Copying算法就是扫描出存活的对象，并复制到一块新的完全未使用的空间中，对应于新生代，就是在Eden和From Space或To Space之间copy。新生代采用空闲指针的方式来控制GC触发，指针保持最后一个分配的对象在新生代区间的位置，当有新的对象要分配内存时，用于检查空间是否足够，不够就触发GC。当连续分配对象时，对象会逐渐从eden到survivor，最后到旧生代，

用java visualVM来查看，能明显观察到新生代满了后，会把对象转移到旧生代，然后清空继续装载，当旧生代也满了后，就会报outofmemory的异常，如下图所示：

![](/assets/import-07.png)



在执行机制上JVM提供了串行GC（Serial GC）、并行回收GC（Parallel Scavenge）和并行GC（ParNew）

1）串行GC

在整个扫描和复制过程采用单线程的方式来进行，适用于单CPU、新生代空间较小及对暂停时间要求不是非常高的应用上，是client级别默认的GC方式，可以通过-XX:+UseSerialGC来强制指定

2）并行回收GC

在整个扫描和复制过程采用多线程的方式来进行，适用于多CPU、对暂停时间要求较短的应用上，是server级别默认采用的GC方式，可用-XX:+UseParallelGC来强制指定，用-XX:ParallelGCThreads=4来指定线程数

3）并行GC

与旧生代的并发GC配合使用

旧生代的GC：

旧生代与新生代不同，对象存活的时间比较长，比较稳定，因此采用标记（Mark）算法来进行回收，所谓标记就是扫描出存活的对象，然后再进行回收未被标记的对象，回收后对用空出的空间要么进行合并，要么标记出来便于下次进行分配，总之就是要减少内存碎片带来的效率损耗。在执行机制上JVM提供了串行GC（Serial MSC）、并行GC（parallel MSC）和并发GC（CMS），具体算法细节还有待进一步深入研究。

以上各种GC机制是需要组合使用的，指定方式由下表所示：

| 指定方式 | 新生代GC方式 | 旧生代GC方式 |
| :--- | :--- | :--- |
| -XX:+UseSerialGC | 串行GC | 串行GC |
| -XX:+UseParallelGC | 并行回收GC | 并行GC |
| -XX:+UseConeMarkSweepGC | 并行GC | 并发GC |
| -XX:+UseParNewGC | 并行GC | 串行GC |
| -XX:+UseParallelOldGC | 并行回收GC | 并行GC |
| -XX:+ UseConeMarkSweepGC-XX:+UseParNewGC | 串行GC | 并发GC |
|  | 不支持的组合 | 1、-XX:+UseParNewGC -XX:+UseParallelOldGC2、-XX:+UseParNewGC -XX:+UseSerialGC |



