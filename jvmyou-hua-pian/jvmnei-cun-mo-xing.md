# JVM内存模型

## 一、Java内存模型



JVM栈（虚拟机栈）由堆、栈、本地方法栈、方法区等部分组成，逻辑结构图如下所示：

![](/assets/import-04.png)

1） 堆

所有通过new创建的对象的内存都在堆中分配，其大小可以通过-Xmx和-Xms来控制。堆被划分为新生代和旧生代，新生代又被进一步划分为Eden和Survivor区，最后Survivor由From Space和To Space组成，结构图如下所示：

* 新生代。新建的对象都是用新生代分配内存，Eden空间不足的时候，会把存活的对象转移到Survivor中，新生代大小可以由-Xmn来控制，也可以用-XX:SurvivorRatio来控制Eden和Survivor的比例
* 旧生代。用于存放新生代中经过多次垃圾回收仍然存活的对象

2）栈

每个线程执行每个方法的时候都会在栈中申请一个栈帧，每个栈帧包括局部变量区和操作数栈，用于存放此次方法调用过程中的临时变量、参数和中间结果

3）本地方法栈

用于支持native方法的执行，存储了每个native方法调用的状态

4）方法区

存放了要加载的类信息、静态变量、final类型的常量、属性和方法信息。JVM用持久代（Permanet Generation）来存放方法区，可通过-XX:PermSize和-XX:MaxPermSize来指定最小值和最大值

