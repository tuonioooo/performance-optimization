# JVM内存模型

JVM栈由堆、栈、本地方法栈、方法区等部分组成，逻辑结构图如下所示：

![](/assets/import-04.png)

1）堆

所有通过new创建的对象的内存都在堆中分配，其大小可以通过-Xmx和-Xms来控制。堆被划分为新生代和旧生代，新生代又被进一步划分为Eden和Survivor区，最后Survivor由From Space和To Space组成，结构图如下所示：

* 新生代。新建的对象都是用新生代分配内存，Eden空间不足的时候，会把存活的对象转移到Survivor中，新生代大小可以由-Xmn来控制，也可以用-XX:SurvivorRatio来控制Eden和Survivor的比例
* 旧生代。用于存放新生代中经过多次垃圾回收仍然存活的对象



