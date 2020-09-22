## JMM

### Java Runtime Data Area
* Type: due to java specification
    * Java Method Area + Java Heap
    * VM Stack + program counter + Native Stack
* Type Detail:
    * Java Method Area
        * class object
        * static varaible
        * constant pool 
* Java constant pool
    * 是放在Method Area里的，也就是线程共享的
    * 是放一些string常量，或者int常量之类的。
    * 常量的意思就是，不会改变。
    * 这些string比如像class的类名之类的这种，要引用到的
* Some points
    * 除了PC，其他的都会有可能OOM。
    * 只有heap是被GC管的
    * 在java里，数组都是对象，都是被分配到堆里的
* 1.7和1.8的不一样
    * 1.7的时候，method area是在permanent这里实现的，也就是在heap里的
    * 1.8的时候移到了direct memory里，大小也就不受jvm heap大小的限制了。而是限制于物理机的内存。

### JVM的参数
* stack
    * -Xss: 每个线程的大小
    * 因为stack和heap不一样，说多少就是多少，没有什么增加的了。所以只有一个参数。
* heap
    * -Xms, -Xmx
    * -Xmn:新生代的大小
* 参数类型
    * 标配参数
        * 从java 1.0到现在13，都在的
        * java -version, java -help
    * X参数
        * -Xint, -Xcomp, -Xmixed
        * 默认是mixed
    * XX参数
        * boolean
            * -XX:+, -XX:-
            * -XX:-PrintGCDetails
            * -XX:+UseParallelGC
        * keyvalue
            * -XX:key=value
            * -XX:MetaspaceSize=128m
            * -XX:MaxTenuringThreshold=15
            * -XX:InitialHeapSize=2048m
        * 查看当前是哪些参数：
            * jps -l 查看现在的进程
            * jinfo -flag PringGCDetails 13632
            * jinfo -flags 13632
* -Xms和-Xmx
    * 等价于-XX:InitialHeapSize和-XX:MaxHeapSize
    * 还是XX参数。
    * 不是X参数，它只是别名
* JVM各项的默认值
    * java -XX:+PrintFlagsInitial： 默认的全部
    * java -XX:+PrintFlagsFinal: 修改以后的全部，用冒号等于跟initial区分
    * java -XX:+PrintCommandLineFlags: 上面的打出全部的，这个只打出自己添加的。
#### 怎么查看JVM系统默认值
* 大概
    * jinfo -flag PringGCDetails 13632
    * jinfo -flags
    * java -XX:+PrintFlagsInitial
    * java -XX:+PrintFlagsFinal
* 最后两个应该是跟着程序一起启动的
#### JVM调参
* xmx和xms设置一致
    * 避免堆动态扩展
    * 既然有这两个值，它当然有它必要的地方。
    * 就比如你的程序有那种比较短的内存飙升期，也就是百分之10的时间你的用的非常高，但是百分九十你用的是低的
    * 这时候你可能就可以借助这个xmx和xms设置的不一致来避免浪费你内存的空间。
    * 如果你内存很充足，浪费没有问题，那就还是调一致。
* xmx和xms默认值
    * 机器物理内存的1/4和1/64
* 常用参数   
    * -Xms
    * -Xmx
    * -Xss
        * 栈的大小
        * 默认是，512k ~ 1024k，根据你的OS会有不同。64bit是1024k
        * -XX:ThreadStackSize
    * -XX:MetaspaceSize
        * 元空间大小。
        * 1.7是永久代，在heap里
        * 1.8是metaspace，在direct memory里，仅受本地内存限制  
        * 默认是 21 m，平台相关。
        * 超过这个limit就会报OOM：Metaspace
        * 也会发生GC。
        * 盲猜是metaspace满了的时候会发生fullgc，也就是连带整个heap都gc了。
        * 或者是老年代满了发生fullgc，然后把metaspace也顺带gc了
    * -XX:SurvivorRatio
        * survivor占young generation的大小比例
        * 默认应该是8：1：1
        * -XX:SurvivorRatio=8 -> 8:1:1
        * -XX:SurvivorRatio=4 -> 4:1:1
        * 越大，那young代实际可放的大小就越小
        * 太小的话，可能年轻代里的promote的age就会被降低，更多的就会被promote到老年代。可能年轻代的作用就不太显现出来。
    * -XX:NewRatio
        * 年轻代的比例。一般是1/3？
        * -XX:NewRatio=2 -> 2：1
        * -XX:NewRatio=4 -> 4：1
        * 这些Ratio都是以自己是1，其他人是多少，来写这个数。
    * -XX:MaxTenuringThreshold
        * promote的age，一般是15
    * -XX:+PrintGCDetails
        * 名称：[该区GC前内存占用 -> 该区GC后内存占用][堆GC前内存占用 -> 堆GC后内存占用]
        * 比如young GC和FullGC
        * youngGC就只有年轻代，FullGC会把年轻代和老年代都打出来，还有metaspace也打出来
    
### JVM 引用
* Type
    * 强软弱虚
* 强引用
    * 默认就是强引用
    * 什么情况下都不会进行GC，内存崩了也不收
* 软引用: soft reference
    * soft referece are guaranteed to be cleared before a JVM throws an OOM error
    * 这个OOM的抛出应该是比如Full GC一次后，它会看这时候老年代空余的空间够不够刚刚触发FullGC的young promote的空间需要大小
    * 如果大于，就okay，如果小于，就会OOM。
    * 而这个soft reference应该就是，一开始FullGC没算上它，但是Full gc后发现计算出来的空间<我的需求，就再去把这些soft回收掉
    * 这时候再看空间，如果够了，就万事大吉，不够，就throw OOM error
    * 用途： cache，尽量多的数据在内存，但是又不想把内存撑爆
    * MyBatis源码大面积地使用了软引用
* 弱引用： weak reference
    * 只要GC，就回收
    * WeakHashMap
        * key是weakreference的，也就是说只要一gc，这个entry就会被
* 虚引用： phantom reference
    * 
#### 引用和回收
* softref是gc后内存还够就不回收，weakref是一有gc就会被回收。
* 这个体现在代码上，或者算法上是什么呢。
* 引用，和堆上内存的那个对象实例。我们在说回收的时候，是在说这两者中的哪一个呢。
* 我理解的应该是，这些什么引用说的都是reference。
* 而堆上的那个对象实例的内存，只有在没有任何引用指向它的时候，它才会被回收。
* 而不是说，我有3个引用同时指向堆内存中的一个对象实例，有任何一个引用设置为null了，GC就会把它指向的堆内存的对象实例回收了。
```aidl
WeakReference<Integer> weakI = new WeakReference<>(new Integer(3));
System.gc();
System.out.println(weakI.get()); // will be null

// or 

Integer i = new Integer(3);
WeakReference<Integer> weakI = new WeakReference<>(i);
i = null;
System.gc();
System.out.println(weakI.get()); // will be null
```

### OOM
* https://www.sohu.com/a/343414554_463994    
* GC几次后会出现OOM？
    * 应该是full gc一次后，发现腾出的空间还不够你要放过来(young promote过来)的东西，就会throw OOM: java heap space
    
    
## GC
### Concurrent Mode failure
https://docs.oracle.com/javase/8/docs/technotes/guides/vm/gctuning/cms.html#concurrent_mode_failure
因为CMS的最后一步，concurrent mark and sweep是跟用户线程一起在跑的，所以如果说application allocate的太快了，
CMS的这最后一步还在做呢，用户线程发现已经allocate失败，就会有concurret mode failure
这时候用户线程就会停下来，直到CMS这最后一步完成，也就是STW了。
* Promotion Failure
就是young的活的要promote到tenured generation发现没空间了，就会报这个，这个就会引发老年代的垃圾回收。

### G1 allocation failure
就是发现没有空的region让我做copy了

    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    