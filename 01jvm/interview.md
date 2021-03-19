# 面试题
## 1. JVM 分析和调优，有没有自己的案例？
有过一次业务流量高峰，运维反馈在下午2点左右，微服务其中一个模块的某个节点的 cpu 占用100%，导致其中一个节点停止服务，随后整个模块的的服务都出现 cpu 
占用过高的情况，随后我用 jps、jstat 等命令行工具，发现 cpu 占用过高是由于连续的 Full GC 导致的，Full GC 清理不掉内存，当时怀疑是代码中有内存
泄漏，使用 jmap 查看内存中大量存在的对象，分析具体是哪些代码有问题，后来分析代码，感觉不是内存泄漏的问题。再次查看 jstat 发现业务高峰的时候，新生
代的 E 区，内存增加的很快，YGC很频繁，有时候甚至YGC清理完一次后，s0有直接 100%，多余的对象分配到老年代中，所以我想是不是新生代的空间太小导致的，
所以查看了一下 E 区和 S 区的空间发现果然很小，S 区很小的原因是默认 gc 的自适应策略造成的，他会根据运行情况动态调整新生代的大小，E 区和 S 区的比例
以及对象直接晋升到老年代的大小，所以这次开始调整新生代和老年代的比例，增大整个堆内存，不再使用默认的gc，改成 CMS，业务挺过一段时间。
对于 JVM 调优，首先我们应该搞清楚调优的目标，是想降低接口的响应时间还是提高接口的 QPS。JDK 1.8 默认的垃圾收集器是 Parallel Scenvlige，它就是
一个注重吞吐量的 GC，CMS、G1 更注重停顿时间。根据不同的性能要求选择不同的 GC，设置合理的堆内存大小，一般设置为整个机器的70%～75%左右，内存超过16G，
优先使用G1，但是尽量不要使用超过 30G 的堆内存，因为超过 32G 以后，JVM 会关闭压缩指针，意味着指针可能会占用更多的内存，造成资源浪费。开启 GC 日志，
方便出问题时，问题追踪。显式指定堆内存的大小，将 -Xms 和 -Xmx 设置成相同的值，避免堆内存扩容对性能造成影响。显式指定 E 区和 S 区的比例，GC 的线
程数，期望的停顿时间。
