# JVM

## 1 Java内存模型
java运行时数据区包括栈、堆、非堆、堆外、jvm自身。
#### 1、栈  
  每个线程的栈内存用Xss来指定，表示jvm允许的栈内存的最大值，默认值是1024个字节，用来存储操作数栈、局部变量表、返回值、Class/method指针等信息。
#### 2、堆  
  堆内存用来存对象实例的，堆内存被所有线程共享，启动时可以用Xms、Xmx分别指定堆内存的最小和最大值，一般把这两个值设置成一样的，
  这样可以减少频繁扩容造成的系统压力，推荐设置成机器内存的70%～80%。堆内存分为新生代和老年代，新生代和老年代的比例是1：2，新生代又分为1个Eden区和2个Survivor区，
  Eden和Survivor的比例是8：1。
#### 3、非堆
  非堆包含Metaspace和CCS和Code Cache。Metaspace包含运行时常量池和方法区。CCS是压缩Class区，当内存小于32G时，默认64位的机器开始压缩指针，CCS叫压缩Class区，
  并不是说对这部分内存进行了压缩，而是对指向该区域的指针进行了压缩，在64位的机器上内存地址只能用64个二进制位来表示，这里的压缩是指可以用32位来表示之前用64位表示的地址值，
  Code Cache用来存储热点方法被JIT编译成机器码后存放机器码的区域。Metaspace上的内存是在类加载的时候被分配的，用来存储Class的元数据。
#### 4、堆外
  NIO的Buffer直接在内存上分配的空间
#### 5、JVM自身使用的内存  
  
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
#### 1、字节码类型

按照性质分成4类
1. 栈操作指令 如iconst iload istore
2. 程序类控制指令 如 if_icmpge
3. 对象操作指令 如 invokevirtual
4. 算术运算和类型转化指令 如 iinc

#### java代码
```java
public class Main{
    public static void main(String[] args) {
        int a = 1 + 2;
        int b = 2 - 1;
        int c = 2 * 2;
        int d = 10 / 2;
        for (int i = 0; i < d; i++) {
            if (i == 0) {
                System.out.println(i);
            }
        }
    }
}
```
---------

#### 编译字节码
```
javac -help 可以列出可能出现的选项 其中-d: 指定放置生成的类文件的位置 -g:var用来生成本地变量表
javac -g:vars -d /Users/kevin/learning/jdk8/target /Users/kevin/learning/jdk8/src/java/Main.java
javap -c -s -v -l /Users/kevin/learning/jdk8/target/Main.class
```
#### 字节码
```
kevindeMBP:java kevin$ javap -v -c -l /Users/kevin/learning/jdk8/target/Main.class
Classfile /Users/kevin/learning/jdk8/target/Main.class
  Last modified 2020-10-21; size 552 bytes
  MD5 checksum b060362ab10cf7d234cea8196e332f99
public class Main
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #5.#24         // java/lang/Object."<init>":()V
   #2 = Fieldref           #25.#26        // java/lang/System.out:Ljava/io/PrintStream;
   #3 = Methodref          #27.#28        // java/io/PrintStream.println:(I)V
   #4 = Class              #29            // Main
   #5 = Class              #30            // java/lang/Object
   #6 = Utf8               <init>
   #7 = Utf8               ()V
   #8 = Utf8               Code
   #9 = Utf8               LocalVariableTable
  #10 = Utf8               this
  #11 = Utf8               LMain;
  #12 = Utf8               main
  #13 = Utf8               ([Ljava/lang/String;)V
  #14 = Utf8               i
  #15 = Utf8               I
  #16 = Utf8               args
  #17 = Utf8               [Ljava/lang/String;
  #18 = Utf8               a
  #19 = Utf8               b
  #20 = Utf8               c
  #21 = Utf8               d
  #22 = Utf8               StackMapTable
  #23 = Class              #17            // "[Ljava/lang/String;"
  #24 = NameAndType        #6:#7          // "<init>":()V
  #25 = Class              #31            // java/lang/System
  #26 = NameAndType        #32:#33        // out:Ljava/io/PrintStream;
  #27 = Class              #34            // java/io/PrintStream
  #28 = NameAndType        #35:#36        // println:(I)V
  #29 = Utf8               Main
  #30 = Utf8               java/lang/Object
  #31 = Utf8               java/lang/System
  #32 = Utf8               out
  #33 = Utf8               Ljava/io/PrintStream;
  #34 = Utf8               java/io/PrintStream
  #35 = Utf8               println
  #36 = Utf8               (I)V
{
  public Main();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=1, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Object."<init>":()V
         4: return
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
            0       5     0  this   LMain;

  public static void main(java.lang.String[]);
    descriptor: ([Ljava/lang/String;)V
    flags: ACC_PUBLIC, ACC_STATIC
    Code:
      stack=2, locals=6, args_size=1 // stack 操作数栈深为2 局部变量表的slot为6 方法入参为1
         0: iconst_3            // 将常量3压入栈
         1: istore_1            // 将栈顶常量3保存进局部变量表slot1中
         2: iconst_1
         3: istore_2
         4: iconst_4
         5: istore_3
         6: iconst_5
         7: istore        4     // 当istore的参数大于3时不再采用istore_n的方式
         9: iconst_0
        10: istore        5
        12: iload         5
        14: iload         4
        16: if_icmpge     38    // 如果一个int类型值大于或者等于另外一个int类型值，则跳转
        19: iload         5
        21: ifne          32    // 如果不等于0，则跳转
        24: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
        27: iload         5
        29: invokevirtual #3                  // Method java/io/PrintStream.println:(I)V
        32: iinc          5, 1 // slot5 + 1
        35: goto          12
        38: return
      LocalVariableTable:
        Start  Length  Slot  Name   Signature
           12      26     5     i   I
            0      39     0  args   [Ljava/lang/String;
            2      37     1     a   I
            4      35     2     b   I
            6      33     3     c   I
            9      30     4     d   I
      StackMapTable: number_of_entries = 3
        frame_type = 255 /* full_frame */
          offset_delta = 12
          locals = [ class "[Ljava/lang/String;", int, int, int, int, int ]
          stack = []
        frame_type = 19 /* same */
        frame_type = 250 /* chop */
          offset_delta = 5
}

```
## 4 类加载器

## 参考资料
1、metaspace讲解： https://stuefe.de/posts/metaspace/what-is-metaspace/ <br>
2、jvm参数官方文档：https://www.oracle.com/java/technologies/javase/vmoptions-jsp.html <br>
3、字节码指令大全：https://www.yuque.com/huaxin803/gb2r7y/gp9xdq#FGtjO