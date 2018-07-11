# JVM性能调优——理论篇

### ![](/assets/import-optimize.png)

### 一、JVM何时会抛出OutOfMemoryException？

* JVM98%的时间都花费在内存回收
* 每次回收的内存小于2%

  满足这两个条件将触发OutOfMemoryException，这将会留给系统一个微小的间隙以做一些Down之前的操作，比如手动打印Heap Dump。

> **注意：并不是内存被耗空的时候才抛出**

### 二、内存泄漏及解决方法

1.系统崩溃前的一些现象：

* 每次垃圾回收的时间越来越长，由之前的10ms延长到50ms左右，FullGC的时间也有之前的0.5s延长到4、5s
* FullGC的次数越来越多，最频繁时隔不到1分钟就进行一次FullGC
* 年老代的内存越来越大并且每次FullGC后年老代没有内存被释放

  之后系统会无法响应新的请求，逐渐到达OutOfMemoryError的临界值。

2.生成堆的dump文件

通过JMX的MBean生成当前的Heap信息，大小为一个3G（整个堆的大小）的hprof文件，如果没有启动JMX可以通过Java的jmap命令来生成该文件。

3.分析dump文件

下面要考虑的是如何打开这个3G的堆信息文件，显然一般的Window系统没有这么大的内存，必须借助高配置的Linux。当然我们可以借助X-Window把Linux上的图形导入到Window。我们考虑用下面几种工具打开该文件：

1. Visual VM
2. IBM HeapAnalyzer
3. JDK 自带的Hprof工具

   使用这些工具时为了确保加载速度，建议设置最大内存为6G。使用后发现，这些工具都无法直观地观察到内存泄漏，Visual VM虽能观察到对象大小，但看不到调用堆栈；HeapAnalyzer虽然能看到调用堆栈，却无法正确打开一个3G的文件。因此，我们又选用了Eclipse专门的静态内存分析工具：Mat。

4.分析内存泄漏

通过Mat我们能清楚地看到，哪些对象被怀疑为内存泄漏，哪些对象占的空间最大及对象的调用关系。针对本案，在ThreadLocal中有很多的JbpmContext实例，经过调查是JBPM的Context没有关闭所致。

另，通过Mat或JMX我们还可以分析线程状态，可以观察到线程被阻塞在哪个对象上，从而判断系统的瓶颈。

5.回归问题

Q：为什么崩溃前垃圾回收的时间越来越长？

A:根据内存模型和垃圾回收算法，垃圾回收分两部分：内存标记、清除（复制），标记部分只要内存大小固定时间是不变的，变的是复制部分，因为每次垃圾回收都有一些回收不掉的内存，所以增加了复制量，导致时间延长。所以，垃圾回收的时间也可以作为判断内存泄漏的依据

Q：为什么Full GC的次数越来越多？

A：因此内存的积累，逐渐耗尽了年老代的内存，导致新对象分配没有更多的空间，从而导致频繁的垃圾回收

Q:为什么年老代占用的内存越来越大？

A:因为年轻代的内存无法被回收，越来越多地被Copy到年老代

### 三、性能调优

除了上述内存泄漏外，我们还发现CPU长期不足3%，系统吞吐量不够，针对8core×16G、64bit的Linux服务器来说，是严重的资源浪费。

在CPU负载不足的同时，偶尔会有用户反映请求的时间过长，我们意识到必须对程序及JVM进行调优。从以下几个方面进行：

* **线程池**：解决用户响应时间长的问题
* **连接池**
* **JVM启动参数**：调整各代的内存比例和垃圾回收算法，提高吞吐量
* **程序算法**：改进程序逻辑算法提高性能

**  1.Java线程池（java.util.concurrent.ThreadPoolExecutor）**

大多数JVM6上的应用采用的线程池都是JDK自带的线程池，之所以把成熟的Java线程池进行罗嗦说明，是因为该线程池的行为与我们想象的有点出入。Java线程池有几个重要的配置参数：

* corePoolSize：核心线程数（最新线程数）
* maximumPoolSize：最大线程数，超过这个数量的任务会被拒绝，用户可以通过RejectedExecutionHandler接口自定义处理方式
* keepAliveTime：线程保持活动的时间
* workQueue：工作队列，存放执行的任务

  Java线程池需要传入一个Queue参数（workQueue）用来存放执行的任务，而对Queue的不同选择，线程池有完全不同的行为：

* SynchronousQueue： 一个无容量的等待队列，一个线程的insert操作必须等待另一线程的remove操作，采用这个Queue线程池将会为每个任务分配一个新线程

* LinkedBlockingQueue ： 无界队列，采用该Queue，线程池将忽略 maximumPoolSize参数，仅用corePoolSize的线程处理所有的任务，未处理的任务便在LinkedBlockingQueue中排队

* ArrayBlockingQueue： 有界队列，在有界队列和 maximumPoolSize的作用下，程序将很难被调优：更大的Queue和小的maximumPoolSize将导致CPU的低负载；小的Queue和大的池，Queue就没起动应有的作用。

其实我们的要求很简单，希望线程池能跟连接池一样，能设置最小线程数、最大线程数，当最小数&lt;任务&lt;最大数时，应该分配新的线程处理；当任务&gt;最大数时，应该等待有空闲线程再处理该任务。

但线程池的设计思路是，任务应该放到Queue中，当Queue放不下时再考虑用新线程处理，如果Queue满且无法派生新线程，就拒绝该任务。设计导致“先放等执行”、“放不下再执行”、“拒绝不等待”。所以，根据不同的Queue参数，要提高吞吐量不能一味地增大maximumPoolSize。

当然，要达到我们的目标，必须对线程池进行一定的封装，幸运的是ThreadPoolExecutor中留了足够的自定义接口以帮助我们达到目标。我们封装的方式是：

* 以SynchronousQueue作为参数，使maximumPoolSize发挥作用，以防止线程被无限制的分配，同时可以通过提高maximumPoolSize来提高系统吞吐量
* 自定义一个RejectedExecutionHandler，当线程数超过maximumPoolSize时进行处理，处理方式为隔一段时间检查线程池是否可以执行新Task，如果可以把拒绝的Task重新放入到线程池，检查的时间依赖keepAliveTime的大小。

**  2.连接池（org.apache.commons.dbcp.BasicDataSource）**

在使用org.apache.commons.dbcp.BasicDataSource的时候，因为之前采用了默认配置，所以当访问量大时，通过JMX观察到很多Tomcat线程都阻塞在BasicDataSource使用的Apache ObjectPool的锁上，直接原因当时是因为BasicDataSource连接池的最大连接数设置的太小，默认的BasicDataSource配置，仅使用8个最大连接。

我还观察到一个问题，当较长的时间不访问系统，比如2天，DB上的Mysql会断掉所以的连接，导致连接池中缓存的连接不能用。为了解决这些问题，我们充分研究了BasicDataSource，发现了一些优化的点：

* Mysql默认支持100个链接，所以每个连接池的配置要根据集群中的机器数进行，如有2台服务器，可每个设置为60
* initialSize：参数是一直打开的连接数
* minEvictableIdleTimeMillis：该参数设置每个连接的空闲时间，超过这个时间连接将被关闭
* timeBetweenEvictionRunsMillis：后台线程的运行周期，用来检测过期连接
* maxActive：最大能分配的连接数
* maxIdle：最大空闲数，当连接使用完毕后发现连接数大于maxIdle，连接将被直接关闭。只有initialSize &lt;x &lt;maxIdle的连接将被定期检测是否超期。这个参数主要用来在峰值访问时提高吞吐量。
* initialSize是如何保持的？经过研究代码发现，BasicDataSource会关闭所有超期的连接，然后再打开initialSize数量的连接，这个特性与minEvictableIdleTimeMillis、timeBetweenEvictionRunsMillis一起保证了所有超期的initialSize连接都会被重新连接，从而避免了Mysql长时间无动作会断掉连接的问题。

**  3.JVM参数**

在JVM启动参数中，可以设置跟内存、垃圾回收相关的一些参数设置，默认情况不做任何设置JVM会工作的很好，但对一些配置很好的Server和具体的应用必须仔细调优才能获得最佳性能。通过设置我们希望达到一些目标：

* **GC的时间足够的小**
* **GC的次数足够的少**
* **发生Full GC的周期足够的长**

  前两个目前是相悖的，要想GC时间小必须要一个更小的堆，要保证GC次数足够少，必须保证一个更大的堆，我们只能取其平衡。

  （1）针对JVM堆的设置，一般可以通过-Xms -Xmx限定其最小、最大值，**为了防止垃圾收集器在最小、最大之间收缩堆而产生额外的时间，我们通常把最大、最小设置为相同的值**  
   （2）**年轻代和年老代将根据默认的比例（1：2）分配堆内存**，可以通过调整二者之间的比率NewRadio来调整二者之间的大小，也可以针对回收代，比如年轻代，通过 -XX:newSize -XX:MaxNewSize来设置其绝对大小。同样，为了防止年轻代的堆收缩，我们通常会把-XX:newSize -XX:MaxNewSize设置为同样大小

  （3）年轻代和年老代设置多大才算合理？这个我问题毫无疑问是没有答案的，否则也就不会有调优。我们观察一下二者大小变化有哪些影响

* **更大的年轻代必然导致更小的年老代，大的年轻代会延长普通GC的周期，但会增加每次GC的时间；小的年老代会导致更频繁的Full GC**

* **更小的年轻代必然导致更大年老代，小的年轻代会导致普通GC很频繁，但每次的GC时间会更短；大的年老代会减少Full GC的频率**

* 如何选择应该依赖应用程序  
  **对象生命周期的分布情况**  
  ：如果应用存在大量的临时对象，应该选择更大的年轻代；如果存在相对较多的持久对象，年老代应该适当增大。但很多应用都没有这样明显的特性，在抉择时应该根据以下两点：（A）本着Full GC尽量少的原则，让年老代尽量缓存常用对象，JVM的默认比例1：2也是这个道理 （B）通过观察应用一段时间，看其他在峰值时年老代会占多少内存，在不影响Full GC的前提下，根据实际情况加大年轻代，比如可以把比例控制在1：1。但应该给年老代至少预留1/3的增长空间

  （4）**在配置较好的机器上（比如多核、大内存），可以为年老代选择并行收集算法**： **-XX:+UseParallelOldGC** ，默认为Serial收集

  （5）线程堆栈的设置：每个线程默认会开启1M的堆栈，用于存放栈帧、调用参数、局部变量等，对大多数应用而言这个默认值太了，一般256K就足用。理论上，在内存不变的情况下，减少每个线程的堆栈，可以产生更多的线程，但这实际上还受限于操作系统。

  （4）可以通过下面的参数打Heap Dump信息

* -XX:HeapDumpPath

* -XX:+PrintGCDetails

* -XX:+PrintGCTimeStamps

* -Xloggc:/usr/aaa/dump/heap\_trace.txt

  通过下面参数可以控制OutOfMemoryError时打印堆的信息

* -XX:+HeapDumpOnOutOfMemoryError

  请看一下一个时间的Java参数配置：（服务器：Linux 64Bit，8Core×16G）

**JAVA\_OPTS="$JAVA\_OPTS -server -Xms3G -Xmx3G -Xss256k -XX:PermSize=128m -XX:MaxPermSize=128m -XX:+UseParallelOldGC -XX:+HeapDumpOnOutOfMemoryError -XX:HeapDumpPath=/usr/aaa/dump -XX:+PrintGCDetails -XX:+PrintGCTimeStamps -Xloggc:/usr/aaa/dump/heap\_trace.txt -XX:NewSize=1G -XX:MaxNewSize=1G"**

经过观察该配置非常稳定，每次普通GC的时间在10ms左右，Full GC基本不发生，或隔很长很长的时间才发生一次

通过分析dump文件可以发现，每个1小时都会发生一次Full GC，经过多方求证，只要在JVM中开启了JMX服务，JMX将会1小时执行一次Full GC以清除引用，关于这点请参考附件文档。

4.程序算法调优：本次不作为重点

### 四、**调优方法的原则**

1、多数的Java应用不需要在服务器上进行GC优化；

2、多数导致GC问题的Java应用，都不是因为我们参数设置错误，而是代码问题；

3、在应用上线之前，先考虑将机器的JVM参数设置到最优（最适合）；

4、减少创建对象的数量；

5、减少使用全局变量和大对象；

6、GC优化是到最后不得已才采用的手段；

7、在实际使用中，分析GC情况优化代码比优化GC参数要多得多；

### **五、GC优化的目的有两个**

**1、将转移到老年代的对象数量降低到最小；**

**2、减少full GC的执行时间；**

为了达到上面的目的，一般地，你需要做的事情有：[详细垃圾回收机制](/jvmyou-hua-pian/xiang-xi-la-ji-hui-shou-ji-zhi.md)——减少GC开销的措施



### JVM参数总结

Xms 是指设定程序启动时占用内存大小。一般来讲，大点，程序会启动的快一点，但是也可能会导致机器暂时间变慢。

Xmx 是指设定程序运行期间最大可占用的内存大小。如果程序运行需要占用更多的内存，超出了这个设置值，就会抛出OutOfMemory异常。

Xss 是指设定每个线程的堆栈大小。这个就要依据你的程序，看一个线程大约需要占用多少内存，可能会有多少线程同时运行等。

以上三个参数的设置都是默认以Byte为单位的，也可以在数字后面添加\[k/K\]或者\[m/M\]来表示KB或者MB。而且，超过机器本身的内存大小也是不可以的，否则就等着机器变慢而不是程序变慢了。

> ```
> -Xms 为jvm启动时分配的内存，比如-Xms200m，表示分配200M
> -Xmx 为jvm运行过程中分配的最大内存，比如-Xms500m，表示jvm进程最多只能够占用500M内存
> -Xss 为jvm启动的每个线程分配的内存大小，默认JDK1.4中是256K，JDK1.5+中是1M
> ```

Total Memory -Xms -Xmx -Xss Spare Memory JDK Thread Count  
1024M 256M 256M 256K 768M 1.4 3072  
1024M 256M 256M 256K 768M 1.5 768

上面的表格只是大致的估计了下在特定内存条件下可以在java中创建的最大线程数。随着-Xmx的加大，空闲的内存数就更少，那么可以创建的线程也就更少，同时在JDK1.4和1.5版本不同下，可创建的线程数也会根据每个线程的内存大小不同而不同。

```
其实只要我们了解了JVM的内存大小指定以及java中线程的内存模型，基本上我们就可以很好的控制如何在java中使用线程和避免内存溢出或错误的问题了。

最近在网上看到一些人讨论到java.lang.Runtime类中的 freeMemory(), totalMemory(), maxMemory()这几个方法的一些问题，很多人感到很疑惑，为什么，在java程序刚刚启动起来的时候freeMemory()这个方法返回的只有一两兆字节，而随着java程序往前运行，创建了不少的对象，freeMemory()这个方法的返回有时候不但没有减少，反而会增加。这些人对 freeMemory()这个方法的意义应该有一些误解，他们认为这个方法返回的是操作系统的剩余可用内存，其实根本就不是这样的。这三个方法反映的都是 java这个进程的内存情况，跟操作系统的内存根本没有关系。下面结合totalMemory(), maxMemory()一起来解释。

maxMemory()这个方法返回的是java虚拟机（这个进程）能构从操作系统那里挖到的最大的内存，以字节为单位，如果在运行java程序的时候，没有添加-Xmx参数，那么就是64兆，也就是说maxMemory()返回的大约是64*1024*1024字节，这是java虚拟机默认情况下能从操作系统那里挖到的最大的内存。如果添加了-Xmx参数，将以这个参数后面的值为准，例如java -cp you_classpath -Xmx512m your_class，那么最大内存就是512*1024*1024字节。

totalMemory()这个方法返回的是java虚拟机现在已经从操作系统那里挖过来的内存大小，也就是java虚拟机这个进程当时所占用的所有内存。如果在运行java的时候没有添加-Xms参数，那么，在java程序运行的过程的，内存总是慢慢的从操作系统那里挖的，基本上是用多少挖多少，直到挖到maxMemory()为止，所以totalMemory()是慢慢增大的。如果用了-Xms参数，程序在启动的时候就会无条件的从操作系统中挖 -Xms后面定义的内存数，然后在这些内存用的差不多的时候，再去挖。

freeMemory()是什么呢，刚才讲到如果在运行java的时候没有添加-Xms参数，那么，在java程序运行的过程的，内存总是慢慢的从操作系统那里挖的，基本上是用多少挖多少，但是java虚拟机100％的情况下是会稍微多挖一点的，这些挖过来而又没有用上的内存，实际上就是 freeMemory()，所以freeMemory()的值一般情况下都是很小的，但是如果你在运行java程序的时候使用了-Xms，这个时候因为程序在启动的时候就会无条件的从操作系统中挖-Xms后面定义的内存数，这个时候，挖过来的内存可能大部分没用上，所以这个时候freeMemory()可能会有些大。

堆大小设置
JVM 中最大堆大小有三方面限制：相关操作系统的数据模型（32-bt还是64-bit）限制；系统的可用虚拟内存限制；系统的可用物理内存限制。32位系统下，一般限制在1.5G~2G；64为操作系统对内存无限制。我在Windows Server 2003 系统，3.5G物理内存，JDK5.0下测试，最大可设置为1478m。
典型设置：
    java -Xmx3550m -Xms3550m -Xmn2g -Xss128k
    - Xmx3550m ：设置JVM最大可用内存为3550M。
    -Xms3550m ：设置JVM促使内存为3550m。此值可以设置与-Xmx相同，以避免每次垃圾回收完成后JVM重新分配内存。
    -Xmn2g ：设置年轻代大小为2G。整个堆大小=年轻代大小 + 年老代大小 + 持久代大小 。持久代一般固定大小为64m，所以增大年轻代后，将会减小年老代大小。此值对系统性能影响较大，Sun官方推荐配置为整个堆的3/8。
    -Xss128k ：设置每个线程的堆栈大小。JDK5.0以后每个线程堆栈大小为1M，以前每个线程堆栈大小为256K。更具应用的线程所需内存大小进行调整。在相同物理内存下，减小这个值能生成更多的线程。但是操作系统对一个进程内的线程数还是有限制的，不能无限生成，经验值在3000~5000左右。
    java -Xmx3550m -Xms3550m -Xss128k -XX:NewRatio=4 -XX:SurvivorRatio=4 -XX:MaxPermSize=16m -XX:MaxTenuringThreshold=0
    -XX:NewRatio=4 :设置年轻代（包括Eden和两个Survivor区）与年老代的比值（除去持久代）。设置为4，则年轻代与年老代所占比值为1：4，年轻代占整个堆栈的1/5
    -XX:SurvivorRatio=4 ：设置年轻代中Eden区与Survivor区的大小比值。设置为4，则两个Survivor区与一个Eden区的比值为2:4，一个Survivor区占整个年轻代的1/6
    -XX:MaxPermSize=16m :设置持久代大小为16m。
    -XX:MaxTenuringThreshold=0 ：设置垃圾最大年龄。如果设置为0的话，则年轻代对象不经过Survivor区，直接进入年老代 。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间 ，增加在年轻代即被回收的概论。
回收器选择
JVM给了三种选择：串行收集器、并行收集器、并发收集器 ，但是串行收集器只适用于小数据量的情况，所以这里的选择主要针对并行收集器和并发收集器。默认情况下，JDK5.0以前都是使用串行收集器，如果想使用其他收集器需要在启动时加入相应参数。JDK5.0以后，JVM会根据当前系统配置 进行判断。
    吞吐量优先 的并行收集器
    如上文所述，并行收集器主要以到达一定的吞吐量为目标，适用于科学技术和后台处理等。
    典型配置 ：
        java -Xmx3800m -Xms3800m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20
        -XX:+UseParallelGC ：选择垃圾收集器为并行收集器。 此配置仅对年轻代有效。即上述配置下，年轻代使用并发收集，而年老代仍旧使用串行收集。
        -XX:ParallelGCThreads=20 ：配置并行收集器的线程数，即：同时多少个线程一起进行垃圾回收。此值最好配置与处理器数目相等。
        java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:ParallelGCThreads=20 -XX:+UseParallelOldGC
        -XX:+UseParallelOldGC ：配置年老代垃圾收集方式为并行收集。JDK6.0支持对年老代并行收集。
        java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100
        -XX:MaxGCPauseMillis=100 : 设置每次年轻代垃圾回收的最长时间，如果无法满足此时间，JVM会自动调整年轻代大小，以满足此值。
        java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseParallelGC -XX:MaxGCPauseMillis=100 -XX:+UseAdaptiveSizePolicy
        -XX:+UseAdaptiveSizePolicy ：设置此选项后，并行收集器会自动选择年轻代区大小和相应的Survivor区比例，以达到目标系统规定的最低相应时间或者收集频率等，此值建议使用并行收集器时，一直打开。
    响应时间优先 的并发收集器
    如上文所述，并发收集器主要是保证系统的响应时间，减少垃圾收集时的停顿时间。适用于应用服务器、电信领域等。
    典型配置 ：
        java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:ParallelGCThreads=20 -XX:+UseConcMarkSweepGC -XX:+UseParNewGC
        -XX:+UseConcMarkSweepGC ：设置年老代为并发收集。测试中配置这个以后，-XX:NewRatio=4的配置失效了，原因不明。所以，此时年轻代大小最好用-Xmn设置。
        -XX:+UseParNewGC :设置年轻代为并行收集。可与CMS收集同时使用。JDK5.0以上，JVM会根据系统配置自行设置，所以无需再设置此值。
        java -Xmx3550m -Xms3550m -Xmn2g -Xss128k -XX:+UseConcMarkSweepGC -XX:CMSFullGCsBeforeCompaction=5 -XX:+UseCMSCompactAtFullCollection
        -XX:CMSFullGCsBeforeCompaction ：由于并发收集器不对内存空间进行压缩、整理，所以运行一段时间以后会产生“碎片”，使得运行效率降低。此值设置运行多少次GC以后对内存空间进行压缩、整理。
        -XX:+UseCMSCompactAtFullCollection ：打开对年老代的压缩。可能会影响性能，但是可以消除碎片
辅助信息
JVM提供了大量命令行参数，打印信息，供调试使用。主要有以下一些：
    -XX:+PrintGC
    输出形式：[GC 118250K->113543K(130112K), 0.0094143 secs]

                    [Full GC 121376K->10414K(130112K), 0.0650971 secs]
    -XX:+PrintGCDetails
    输出形式：[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]

                    [GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]
    -XX:+PrintGCTimeStamps -XX:+PrintGC：PrintGCTimeStamps可与上面两个混合使用
    输出形式：11.851: [GC 98328K->93620K(130112K), 0.0082960 secs]
    -XX:+PrintGCApplicationConcurrentTime: 打印每次垃圾回收前，程序未中断的执行时间。可与上面混合使用
    输出形式：Application time: 0.5291524 seconds
    -XX:+PrintGCApplicationStoppedTime ：打印垃圾回收期间程序暂停的时间。可与上面混合使用
    输出形式：Total time for which application threads were stopped: 0.0468229 seconds
    -XX:PrintHeapAtGC :打印GC前后的详细堆栈信息
    输出形式：
    34.702: [GC {Heap before gc invocations=7:
    def new generation   total 55296K, used 52568K [0x1ebd0000, 0x227d0000, 0x227d0000)
    eden space 49152K, 99% used [0x1ebd0000, 0x21bce430, 0x21bd0000)
    from space 6144K, 55% used [0x221d0000, 0x22527e10, 0x227d0000)
    to   space 6144K,   0% used [0x21bd0000, 0x21bd0000, 0x221d0000)
    tenured generation   total 69632K, used 2696K [0x227d0000, 0x26bd0000, 0x26bd0000)
    the space 69632K,   3% used [0x227d0000, 0x22a720f8, 0x22a72200, 0x26bd0000)
    compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
       the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
        ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
        rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
    34.735: [DefNew: 52568K->3433K(55296K), 0.0072126 secs] 55264K->6615K(124928K)Heap after gc invocations=8:
    def new generation   total 55296K, used 3433K [0x1ebd0000, 0x227d0000, 0x227d0000)
    eden space 49152K,   0% used [0x1ebd0000, 0x1ebd0000, 0x21bd0000)
    from space 6144K, 55% used [0x21bd0000, 0x21f2a5e8, 0x221d0000)
    to   space 6144K,   0% used [0x221d0000, 0x221d0000, 0x227d0000)
    tenured generation   total 69632K, used 3182K [0x227d0000, 0x26bd0000, 0x26bd0000)
    the space 69632K,   4% used [0x227d0000, 0x22aeb958, 0x22aeba00, 0x26bd0000)
    compacting perm gen total 8192K, used 2898K [0x26bd0000, 0x273d0000, 0x2abd0000)
       the space 8192K, 35% used [0x26bd0000, 0x26ea4ba8, 0x26ea4c00, 0x273d0000)
        ro space 8192K, 66% used [0x2abd0000, 0x2b12bcc0, 0x2b12be00, 0x2b3d0000)
        rw space 12288K, 46% used [0x2b3d0000, 0x2b972060, 0x2b972200, 0x2bfd0000)
    }
    , 0.0757599 secs]
    -Xloggc:filename :与上面几个配合使用，把相关日志信息记录到文件以便分析。
常见配置汇总
    堆设置
        -Xms :初始堆大小
        -Xmx :最大堆大小
        -XX:NewSize=n :设置年轻代大小
        -XX:NewRatio=n: 设置年轻代和年老代的比值。如:为3，表示年轻代与年老代比值为1：3，年轻代占整个年轻代年老代和的1/4
        -XX:SurvivorRatio=n :年轻代中Eden区与两个Survivor区的比值。注意Survivor区有两个。如：3，表示Eden：Survivor=3：2，一个Survivor区占整个年轻代的1/5
        -XX:MaxPermSize=n :设置持久代大小
    收集器设置
        -XX:+UseSerialGC :设置串行收集器
        -XX:+UseParallelGC :设置并行收集器
        -XX:+UseParalledlOldGC :设置并行年老代收集器
        -XX:+UseConcMarkSweepGC :设置并发收集器
    垃圾回收统计信息
        -XX:+PrintGC
        -XX:+PrintGCDetails
        -XX:+PrintGCTimeStamps
        -Xloggc:filename
    并行收集器设置
        -XX:ParallelGCThreads=n :设置并行收集器收集时使用的CPU数。并行收集线程数。
        -XX:MaxGCPauseMillis=n :设置并行收集最大暂停时间
        -XX:GCTimeRatio=n :设置垃圾回收时间占程序运行时间的百分比。公式为1/(1+n)
    并发收集器设置
        -XX:+CMSIncrementalMode :设置为增量模式。适用于单CPU情况。
        -XX:ParallelGCThreads=n :设置并发收集器年轻代收集方式为并行收集时，使用的CPU数。并行收集线程数。
```













