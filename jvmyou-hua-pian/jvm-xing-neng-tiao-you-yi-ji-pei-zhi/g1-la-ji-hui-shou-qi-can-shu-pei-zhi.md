# G1垃圾回收器参数配置



| 选项/默认值 | 说明 |
| :--- | :--- |
| -XX:+UseG1GC | 使用 G1 \(Garbage First\) 垃圾收集器 |
| -XX:MaxGCPauseMillis=n | 设置最大GC停顿时间\(GC pause time\)指标\(target\). 这是一个软性指标\(soft goal\), JVM 会尽量去达成这个目标. |
| -XX:InitiatingHeapOccupancyPercent=n | 启动并发GC周期时的堆内存占用百分比. G1之类的垃圾收集器用它来触发并发GC周期,基于整个堆的使用率,而不只是某一代内存的使用比. 值为 0 则表示"一直执行GC循环". 默认值为 45. |
| -XX:NewRatio=n | 新生代与老生代\(new/old generation\)的大小比例\(Ratio\). 默认值为 2. |
| -XX:SurvivorRatio=n | eden/survivor 空间大小的比例\(Ratio\). 默认值为 8. |
| -XX:MaxTenuringThreshold=n | 提升年老代的最大临界值\(tenuring threshold\). 默认值为 15. |
| -XX:ParallelGCThreads=n | 设置垃圾收集器在并行阶段使用的线程数,默认值随JVM运行的平台不同而不同. |
| -XX:ConcGCThreads=n | 并发垃圾收集器使用的线程数量. 默认值随JVM运行的平台不同而不同. |
| -XX:G1ReservePercent=n | 设置堆内存保留为假天花板的总量,以降低提升失败的可能性. 默认值是 10. |
| -XX:G1HeapRegionSize=n | 使用G1时Java堆会被分为大小统一的的区\(region\)。此参数可以指定每个heap区的大小. 默认值将根据 heap size 算出最优解. 最小值为 1Mb, 最大值为 32Mb. |

