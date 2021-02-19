# JVM

## 1 Java内存模型
java运行时数据区包括栈、堆、非堆、堆外、jvm自身。
- 栈 <br/>
  每个线程的栈内存用Xss来指定，表示jvm允许的栈内存的最大值，默认值是1024个字节，用来存储操作数栈、局部变量表、返回值、Class/method指针等信息。
- 堆  
  堆内存用来存对象实例的，堆内存被所有线程共享，启动时可以用Xms、Xmx分别指定堆内存的最小和最大值，一般把这两个值设置成一样的，
  这样可以减少频繁扩容造成的系统压力，推荐设置成机器内存的70%～80%。堆内存分为新生代和老年代，新生代和老年代的比例是1：2，新生代又分为1个Eden区和2个Survivor区，
  Eden和Survivor的比例是8：1。
- 非堆
  非堆包含Metaspace和CCS和Code Cache。Metaspace包含运行时常量池和方法区。CCS是压缩Class区，当内存小于32G时，默认64位的机器开始压缩指针，CCS叫压缩Class区，
  并不是说对这部分内存进行了压缩，而是对指向该区域的指针进行了压缩，在64位的机器上内存地址只能用64个二进制位来表示，这里的压缩是指可以用32位来表示之前用64位表示的地址值，
  Code Cache用来存储热点方法被JIT编译成机器码后存放机器码的区域。Metaspace上的内存是在类加载的时候被分配的，用来存储Class的元数据。
- 堆外
  NIO的Buffer直接在内存上分配的空间
- JVM自身使用的内存  
  
## 2 GC

#### 1、串行GC
使用单个线程进行垃圾回收，**新生代**使用**标记-复制算法**，**老年代**使用**标记-整理算法**，回收的效率没有并行gc高

#### 2、并行GC
使用cpu核心数的线程进行垃圾回收，相比串行gc用多个线程并行进行垃圾回收。**新生代**的并行gc有两个都采用的是**标记-复制算法**，一个是ParNew，它的特点是串行gc的多线程版本，另一个是Parallel Scavenge收集器，它与ParNew的一个重要区别是自适应的调节策略，它是一个注重吞吐量的gc

参数配置：<br>
-XX:+UseParallelGC jdk8默认的垃圾收集器，新生代使用Parallel Scavenge，老年代使用Parallel Old<br>
-XX:GCTimeRatio 垃圾收集时间占总时间的比率，默认99<br>
-XX:+UseAdaptiveSizePolicy 开启自适应调节策略，不需要指定新生代的大小(-Xmn)、Eden与Survivor区的比例(-XX:SurvivorRatio)、晋升老年代对象大小(-XX:PretenureSizeThreshold)等细节参数

#### 3、CMS
老年代收集器CMS是一个采用**标记清除算法**并且注重停顿时间的的收集器，适用于web服务器等注重请求低延迟的场景。它的特点是除了初始标记阶段和最终标记阶段外，其他阶段gc的回收线程和用户线程是并发进行的，所以cms能做到低停顿时间。cms并发标记时的线程数是cpu核心数的1/4

参数配置：
-XX:+UseConcMarkSweepGC 开启cms进行垃圾回收，默认新生代gc适用的是ParNew

#### 4、G1
G1是CMS的改进版本，为了实现高吞吐、减少内存碎片、停顿时间可控为目标来设计的。G1将整个堆默认分成2048个region，进行回收时会将region按预估的回收时间排序，只清理用户设置的gc停顿时间内的region

参数配置：<br>
-XX:+UseG1GC 开启G1进行垃圾回收，jdk9默认使用G1 <br>
-XX:G1HeapRegionSize 设置region的大小，<br>
-XX:G1NewSizePercent 设置新生代初始占比，默认5%<br>
-XX:G1MaxNewSizePercent 设置新生代最大占比 默认60%<br>
-XX:MaxGCPauseMills 最大gc停顿时间<br>

## 3 字节码

## 4 类加载器

## 参考资料
metaspace讲解： https://stuefe.de/posts/metaspace/what-is-metaspace/ <br>
jvm参数官方文档：https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html