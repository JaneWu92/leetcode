### HashMap
````
    Node[] table;
    put(K key, V value){
        idx = hash(key.hashcode);
        putVal(table, idx, key, value);
    }
    hash(hashcode){
        return hashcode >> 16 | hashCode();
    }
    class Node{
        int hash;
        K key;
        V value;
        Node<K, V> next;
    }
    putVal(hash, key, value){
        idx = hash;
        if(table[idx] == null){
            table[idx] = node
        }else if(table[idx] instanceof TreeNode){
            addToTree();
        }else{
            loop until find one euqal key, or add to list tail
            if(size match) treeify();
        }
        ++size;
        if(size > threashold){
            resize();
        }
    }
    resize(){
        newTab = new Node[length << 2];
        for each Node in tab:
            split to hi_list and lo_list base on (length & hash)
            lo_list stay here, hi_list go to cur_idx+length;
    }
````

* 数组里面装的数据结构是Node(或者红黑树树化后是Treenode)
* 结构Node里面除了key，value还有调优后的hashcode。
    * hashcode调优
        * 就是用用高16位异或上低16位，这样信息更多，更不容易冲突。
    * 缓存hashcode
        * 在以下用到hashcode的时候，就可以不用重新计算
            * 扩容，重新计算下标
            * 判断两个元素是否是相同的时候，会先判断hashcode
                * 要equal，hashcode一定要一样。虽然hashcode一样，不一定equal
    * table idx的计算：hashcode | (table_length-1)
        * 位运算更快
        * 这样做的前提是length要是2的倍数（0101 || 1111）
* putVal
        Node是TreeNode的父类
* 树化的阈值是TREEIFY_THRESHOLD = 8
* 判断节点要覆盖(key相同)的逻辑是：
    * hash相等，并且
    * key == or equal
    
    
### ConcurrentHashMap
```
public class Test {
    //注意要加volatile
    volatile Node[] table;
    put(K key, V value){
        idx = hash(key.hashcode);
        putVal(table, idx, key, value);
    }
    hash(hashcode){
        return hashcode >> 16 | hashCode();
    }
    class Node{
        int hash;
        K key;
        V value;
        Node<K, V> next;
    }
    putVal(hash, key, value){
        idx = hash;
        if(table[idx] == null){
            //虽然说table加了volatile，但是只是这个数组类的reference有内存可见性
            //但是它里面的元素是没有内存可见性的。
            //所以用cas其实是保证了原子性和可见性，对于你现在操作的这个table[idx]来说
            casTabAt(table[idx], node);
        }else if(node hash == MOVED){
            helpTransfer();
        }else{
            synchronized (node){
                if (fh >= 0) {
                    same with hashmap
                    loop for finding same node and replace
                    else add to tail
                }else if(is TreeNode){
                    addToTree();
                }
            }
        }
        if need: treeifyBin(tab, i);
        addCount(1L);
    }
    addCount(1L, currentCount){
    }
    transfer(){
        
    }
}

```
* initTable是lazy的，在放进第一个元素的时候触发。
* DEFAULT_CAPACITY = 16
* casTabAt when tab[idx] == null
* 判断是不是链表的标准是fh >= 0
    * 因为在concurrenthashmap里面，有跟hashmap不一样的数据结构TreeBin,然后hash是负数把
* addCount的机制（参见LongAdder）
    * 两个组件。basecount + conterCell数组（就相当于long和long[]）
    * 先CAS去加basecount。如果成功，那就最好，证明此时竞争挺小的，可能就我一个人。
    * fail，就去countercell[]里random选择一个去CAS加。
    * 如果不是这种情况，是涉及总体的，比如countercell[]扩容或者初始化，或者countercell[idx]==null要初始化.（countercell是一个类，维护着一个long）
    * 就用cellbusy自旋锁，锁住全部（我倒想锁单个，没得锁啊，是空啊。就像你要绑绳子在牙齿上，但是你这颗牙齿没有，你只能绑嘴巴）。
    * 关于什么时候要扩容，这个还挺绕的。还没看大明白，指导思想是在竞争太大的时候扩容。
* 问题就是来了，这里的addCount为什么要用CAS cellsbusy而不用synchronized
    * 我的其中一个猜想是为了避开synchronized的wait queue
        ??????????
* 要注意多线程下，条件判断有一个很重要的地方。就是要判断两遍。
    * 在拿锁前判断一变，在拿锁后又要判断一遍。
    * 拿锁前不判断，就是浪费资源。
    * 拿锁后不判断，你会出错。因为在你拿到锁的判断和你拿到锁这中间，已经发生了很多的事情。其他线程可能已经改了你原本的东西。
* 多线程扩容到底怎么实现的。
    * 能想到的就是分治思想。
    * 也就是说，如果我是一个线程扩容，我也是for loop整个table[]，去一个一个bin的做。
    * 所以如果多线程的话呢，就是每人一个。线程A先做idx1，然后线程2过来，它就去拿idx2做，这个时候A做完了，又来拿idx3，然后线程C过来拿idx4做。
    * 这个输入这边是没问题。但是输出这边呢，就是会不会两个线程都往同一个新线程的某个idx上放元素。
    * 不会的。因为hash，要么是在当前位置，要么是在idx+length。因为一个idx只有一个一个线程在做。
    * 请注意，上面说的是一个一个，但实际上default是16，就是一个线程会负责16个槽位。但是其他机制是一样的。
    * 那还有一个问题，如果用户这时有插入或者查询，会是怎么样。
    * 也就是说，扩容时候的锁机制是怎么样的。
    * 照理来说，扩容的时候，用户是不能插入的。果然，扩容的时候，是synchronized (f),锁住头节点。
    * //这里有问题啊。扩容是虽然是一个一个槽位扩，但你扩完了如果有人又往你这个槽位放呢。就是我这个槽位扩完了，其他槽位没扩完啊。所以table的reference还没切过去
    * //哦因为，在扩容的时候，相应bin的头节点会被放入MOVED标记，所以你要放数据的这个线程就会来先帮助扩容。
    * 也就是说，这个粒度还是以bin位单位。就是我扩容做到tab[4],我后面的tab[]都还是可以put的。不受影响。
    * 也就是说，用户put会发生三种情况
        * 在扩容，当前槽位正在被搬运。被在扩容的线程synchronized(f)住，啥都不做，等着
        * 在扩容，当前槽位已经被搬运结束，但其他槽位没结束。发现当前槽位上的是forwarding node，也就是说，我的put操作不行做，因为这个map在扩容，就去帮忙扩容。
        * 无论在不在扩容，这个槽位还没被染指。当前的这个槽位啥事没有（即使其实map在扩容，但暂时还没扩到这个位置），那他就去synchronized(f)，正常操作。
    * get的话，其实除了用了getvolatile，其他什么限制也没有。就正常读
        * 为什么？
        * 因为getvolatile已经能保证可见性。不会读到脏数据
    * 然后如果是碰到forwarding节点。就通过forwarding节点去newTab里面找。
    * 因为new ForwardingNode<K,V>(nextTab)，forwarding里面其实就是放了newTab的地址。 
* 扩容过程
    * 一个一个槽位地去扩容
    * 也是一样，synchronized (f),锁住头节点，然后去高低链分出来，放到new table里
    * 然后把头节点放成forwarding node。
* 正在扩容时发生读写
    * 对写来说，就是上面说的三种情况
    * 对读来说，因为读除了getvolatile，其他不做任何限制。
    * 也就是说，我就正常读，即使你在扩容，也不影响我。
    * 唯一有影响的是你这个槽位已经移到newTab去了（通过forwarding node可知）。
    * 那我就通过forwarding去newTab里find，照样还是读。
* ConcurrentHashMap的get要做同步吗
    * CHM对读其实是没有加锁的。
    * 只用了tabAt来取数据，也就是getObjectVolatile。
    * 只保证了内存的可见性。
    * 也就是说别人其他线程的改动，我能看见。
    * 如果没有这个getObjectVolatile。可能就只读我自己缓存中的数据。是老的。别人更新了我也还没刷新看不见。
    * 所以它这个相当于是只加了写锁，没有加读锁。
* 什么时候需要读锁?
    * 考虑一下ReadWriteLock
    * 这里面的WriteLock是对read也排斥的。
    * 也就是我在写的时候，你也不能读。
    * 但是对CHM来说，它的这个程度还是小一点。就是即使你在写，我也可以读。顶多是你写到一半的时候我去读，我读到之前的值，其实也是可接受的。
* get有加锁吗,需要吗？
    * Node的value和nextNode都是volatile修饰，所以线程间是可见的
    * 这个数组也是加volatile的
    * 也就说，如果数组的地址变了，是线程间可见的
    * 数组中某个元素的value或者next变了，线程间也是可见的。
    * 但是这里还有一个问题，如果我在get的时候直接tab[idx]
        * 这个如果另外一个线程已经tab[idx] = null
        * 我这个线程应该是看不到。
        * 所以这可能是CHM用tabAt(getVolatile)的原因
        * 就会去主存拿，而不是拿自己缓存里面的可能已经老了的数据

#### 数组是放在哪里的
* 数组是个对象。所以在堆里，你可以把Student A[]看作成以下Class As，就好理解了
```
Class As{
    Student a1;
    Student a2;
    Student a3;
    ...
}
```     

### LongAdder
* AtomicLong不好吗？
    * atomiclong是用cas对一个value进行操作，如果竞争太激烈的情况下，效率不高。load无法分散。
    * longadder是将load分散，用一个long数组来进行分流，这样线程很多的情况下，可以把流量分散掉。要用的时候，在同步计算sum
* 大概思路
    * 先去cas basecount
    * 不成功的化就只能去操作Cell[]。
    * Cell[]的操作分两种：涉及局部和涉及整体的。
    * 局部的就是random地选择一个Cell，去cas加一
    * 整体的就是Cell[]还是null，或者Cell[idx]是null，或者要扩容。就得去cas cellBusy
    * 也就是用自旋锁去锁住整个Cell[]

### java的同步机制
* java是通过monitor来实现线程间的同步问题
* monitor是任何object都能充当。主要有两个方面
    * mutual exclusion
        * object locks
    * cooperation
        * object wait/notify method
* monitor的构造
    * lock room, entry set, wait set
* 基本机制
    * lock room只能有一个线程占用
    * 如果有线程要申请锁，就会来到entry set等着lock room里的人出来
    * lock room里面的线程一旦出来，就是release the monitor，所有在entry set里的线程就竞争进入，只有一个会成功
    * lock room里的线程，如果call了wait，就会进入wait set。
    * 可能很多线程都在wait set里面//但是只可能是一个一个进入wait set，因为要进入wait set你首先得在lock room里
    * 只有等另外进入lock room里的某个线程，调用了notify/notify all，被notify的waitset里的线程，才会被移到entry set，有机会重新竞争lock room
    * 注意notify是选择wait set里的一个线程进入entry set
    * 而notify all是选择wait set里的所有线程进入entry set
    * 所以照理来说，一个线程如果是wait的状态，下一个状态应该是要blocked的状态
* wait只能在synchronized块内被调用。
    * 因为wait是release monitor去等待其他线程的notify
    * 你要release不得先已经own了。
* monitor在JVM中的结构和实现
```
ObjectMonitor() {
    _count        = 0; //用来记录该对象被线程获取锁的次数
    _waiters      = 0;
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL; //处于wait状态的线程，会被加入到_WaitSet
    _WaitSetLock  = 0 ;
    _EntryList    = NULL ; //处于等待锁block状态的线程，会被加入到该列表
  }
```

### 单例模式
* 写一个单例模式
```
class MyClass{
    //属性设置成private，因为不想让别人访问
    //设置成static，因为单例，整个JVM只需要有一个
    //要加volatile。避免cpu指令重排，还没实例化成功先赋值了instance的地址。
    //这样其他线程可能就先拿了一个半成品instance
    private static volatile MyClass instance;
    //默认构造函数设置成private
    //使别人没办法直接new
    private MyClass(){
    }
    //设置成static，因为整个函数跟具体的实例没有关系
    public static MyClass getInstance(){
        //先不上锁，判断一下instance是不是可以使用
        //因为使读，不上锁即可以提高性能，又可以当可用就直接用
        if(instance == null){
            //同步锁来保证只有一个线程在做事
            //要锁住class而不是实例，因为我在一个静态方法里面
            synchronized(MyClass.class){
                //里面还要再判一次空，因为在你第一次判空和拿到锁之前，还有好长一段时间
                //可能已经有另外一个线程进来把你的instance已经给初始化了
                //所以同步块外的判断，在同步块里面，永远要重新判断一次
                if(instance == null){
                    instance = new MyClass();
                }
            }
        }
        return instance;
    }

}
```
* 为什么需要单例模式
    * 我觉得应该有两个方面
        * 一个是整个JVM只能有一个
            * 比如一些资源类的
        * 一个是只有一个就够了
            * 比如spring里你配的serviec类，其实就是单例。
            * 为什么单例，因为没有状态，你用一个类就够用了
    * 单例只有一个，必然涉及到共享的问题。多线程下会不会有问题
        * 如果没有状态，就有问题
        * 如果有状态。那你相关的方法一定要是线程安全的
        
* 什么时候用static method
    * 可以用排除法
        * 如果你觉得一个方法，跟一个instance没什么关系。
        * 也就是说，即使你没有任何实例，我这个方法都是有意义的。
        * 那这时候就要考虑用static method

### synchronized和指令重排
* synchonized不能禁止指令重排
* 所以单例模式里instance要加volatile修饰
* synchronized不能防止指令重排，那它使怎么做到保证有序性的
    * 指令重排，是cpu硬件层面的性能优化。
    * 但是要注意，所有的优化，无论怎么重排序，都不能影响单线程程序的执行结果。
    * 这就是as if serial的语义。
    * synchronized是排他锁。所以在synchronized内部，肯定是单线程执行的
    * 所以在单线程下，cpu的重排序，只要是不影响在此单线程内的结果，就都是可以优化的。
    * 所以，它并没有禁止指令重排。因为，它不需要啊（我在单线程的安全范围内，为什么要禁止指令重排）。
    * 举个例子，单例模式中，之所以会出现拿到半成品的结果
    * 就是因为其他的线程的“读”操作，没在synchronized里面。
    * 也就是说，我正经的流程，都在synchronied保护内，是单线程没错。
    * 但是会有很多线程偷偷在外面读这个共享变量（instance）的值
    * 所以我在“as if serial”的指导下的优化，对我自己来说是合理的。但是影响到了你。


### Monitor和Mark Word
* Java object header：Mark Word + klass pointer
* 各自多大呢。从word可以看出，mark word就是一个word，所以如果在32bit机器的话就是32位。
* mark word可能放了：用来做同步的锁的相关信息+GC的信息
* 每个java object都有一个相关联的monitor对象。（至于它怎么做的我就不知道了）
* mark word跟所有锁（偏向锁，轻量级，重量级）都相关
* Monitor只跟重量级锁相关。
* 偏向锁就是先把锁类型设作偏向，然后把自己的线程号cas写到mark word里（这个cas一定要成功，不成功就是有竞争了，就可能要升级到轻量级锁了）


### 锁升级过程
* https://blog.csdn.net/tongdanping/article/details/79647337
* https://blog.csdn.net/weixin_40910372/article/details/107726978


### 同步的一些概念
* wait和sleep的差别
    * wait释放锁，sleep不释放
    * sleep永远是作用在current Thread.它是一个static方法。
        * Thread.sleep
    
* park和wait,和sleep的区别

 
### Synchronized和lock的区别

### AQS是什么东西
* 是java里用来实现锁机制的一个基类
* 主要的结构是，有一个int state。来代表这个锁现在有没有人在占用
//TODO
state, Node队列， 
park， unpark
cas(state)     
     
### 线程
* 线程和进程的区别
    * 线程是调度的最基本单位
    * 进程是资源分配的最基本单位 
* 线程的几种状态
    * new
        * 新建状态。调用start前，只是个对象
    * runable
        * 就绪。等待cpu调度
    * running
        * 正在运行
    * blocked
        * 阻塞。在wait queue里，等待某种条件的满足（比如IO什么的）
    * wait
        * 等待唤醒信号
    * timed_wait
        * 有时间限制的wait
    * terminated
        * 运行结束的线程
* blocked和wait状态有什么不一样
    * 说的是wait是调object.wait后会进入的，就是在等其他线程给信息
    * block是在争抢锁资源的时候
    * wait之后，肯定是要进入block的状态的。但其实没有太明白
    * 参考上面“### java的同步机制”
        
* 创建线程有哪些方法
    * 继承Thread重写run方法
    * 实现Runable接口，然后放进Thread的构造参数里
        * Thread的构造就是有一个Runable target。然后run方法就是调用target.run();    
    * 把FutureTask放进Thread的构造参数里。（因为FutureTask is Runable）
        * 构造FutureTask就得传入一个Callable
        * 最后可以通过FutureTask的get来取得线程运行的结果返回值。
        * FutureTask是怎么来扩展Thread(Runable)的这个框架的
            * FutureTask实现了Runable接口
            * 然后在它ovride的run方法里，调用了FutureTask的field Callable的call方法
            * 然后它把结果set到FutureTask的field outcome里。
            * 这样在调用FutureTask的get方法的时候，就能拿到这个outcome
    * 用线程池来创建。
        * ThreadPoolExecutor x = new ThreadPoolExecutor(corePoolSize,maximumPoolSize, keepAliveTime, TimeUnit.SECONDS, workQueue);
```
//use callable and futuretask to create thread
public class Test {
    public static void main(String[] args) throws ExecutionException, InterruptedException {
        FutureTask<String> f = new FutureTask(new MyCallable());
        Thread t1 = new Thread(f);
        t1.start();
        String s = f.get();
        System.out.println("in main: " + s);
    }
}
class MyCallable implements Callable<String> {
    @Override
    public java.lang.String call() throws Exception {
        System.out.println("asdfsd");
        Thread.sleep(2000);
        return "success";
    }
}
```    

#### 线程池
* 一些重要的类
    * interface Executor
        * 就有一个execute方法
    * interface ExecutorService extends Executor
        * Executor的功能增强
        * submit, shutdown
        * 实现类
            * ThreadPoolExecutor
    * Executors
        * 新建Executors的工厂
        * 主要应该是能产生四种类别的ExecutorService
        * 但阿里巴巴推建不要用这个工厂来建，还是直接自己new ThreadPoolExecutor配置自己的参数比较好。
    * ThreadPoolExecutor 
* 规范的新建线程池方式：
    * ThreadPoolExecutor
        * int corePoolSize
        * int maximumPoolSize
        * long keepAliveTime
        * TimeUnit unit
        * BlockingQueue<Runnable> workQueue
        * ThreadFactory threadFactory
        * RejectedExecutionHandler handler
    * 在还没达到core数，你来几个task我就新建几个thread。因为core的思想就是常驻线程。即使你有idle的可以用，我也不管，我就要建
    * task来了，但是目前的pool里thread已经到达core数了，那我就放到workQueue里。在走廊里排队，等常驻线程里面有人做完，腾出线程来。
    * 如果thread已经到corePoolSize，workQueue也排满了，就新开线程，上限是maxPoolSize
    * 因为常驻数是corePoolSize，所以当workQueue排满去新建线程的时候，等一些任务做完，肯定会出现idle的。
    * 那时候我们就希望能够把超过corePoolSize的部分都回收起来。
    * 这里回收标准就是keepAlivetime,数量>core，idle时间超过keepAlivetime，就把这部分回收。
    * 如果maxPoolSize和workQueue都满了，那就调用rejecthandler来处理接下来进来的task
        * AbortPolicy
        * DiscardPolicy
        * DiscardOldestPolicy
        * CallerRunsPolicy
        * 自定义
    * 主要就是几种：
        * 一种是拒绝，并且给用户exception让他知道这个被reject了
        * 一种是悄无声息，也丢弃了，但是用户不知道
        * 还有也是丢弃，也不告诉用户，但是不是丢弃现在进来的，而是丢弃queue里最老的
        * 不开线程了，我当前这个线程直接去run
        * 自定义。重写rejectedExecution

### java四种线程池

      
### java内存模型
* 有哪几块
    * 线程公有的
        * heap
        * method area
    * 线程私有的
        * stack
        * program counter
        * native stack    
* method area里面有什么
    * method area是存放per-class的东西
    * 比如class
    * 比如static variable refer to 的object。因为也是class级别的。
* object和variable的差别
    * 应该是object是对象
    * 然后variable放的是object的地址把
    * yes！说对了。
                  
### 垃圾回收机制
#### 总的思路
* 通过GC root的扫描，能够扫描到所有目前来说还在被引用（也就是活着）的对象。
* 然后把其它对象给处理掉（也就是让这些dead对象的memory，能够重新用来放置新对象）。       
* 常用算法
    * mark and sweep
        * 会有碎片产生
    * mark and copy
        * 没有碎片，但是空间利用率会低。因为你得划分成两个地方，才能做“copy”操作。也就是要有一个缓冲区，是不能被用的，要被流出来进行下次的copy
    * mark and compact
        * 没有碎片，空间利用率也高。但是可能算法就比较复杂把
* 基于新老年代的特点
    * 新生代是死的多，也就是剩下的多，可以用mark and copy
    * 老年代是活的多，所以一般用mark and sweep。但因为会有碎片，所以可能也用mark and compact
    * mark-copy和mark-compact差不多，只是一个牺牲空间一个牺牲时间
* GC Root是哪一些
    * the objects referred to by variables in stack
    * the objects referred to by variables in native stack
    * static variables in method area
    * 还有一个是method area里的常量。不大理解
* 搭配的发展：
    * sequence mark-copy, sequence mark-compact
    * parallel mark-copy, parallel mark-compact
    * parallel mark-copy, concurrent mark and sweep
    * G1  
* GC的思路
    * 总体思路就是把dead的对下给你
    * 由GC root出发，标记所有的alive object。
    * 所有的alive object被标记后，就意味剩下的东西都是可以被清除掉的
    * 然后你要清除哪一块内存（年轻代，老年代，G1）就可以扫描你的哪一块，然后把daed的清理掉。
    * ???(“清理掉”的具体做法是什么？)从数据结构上来说，怎么样的一块内存才是被”清理“掉的
        * 可能有很多种实现方式。freelist是一种。就是把所有free的memory记在表里面
* GC什么时候被触发
    * 这取决于你的GC算法的特点
    * 比如是parallel mark and compact，因为是STW，又是compact，不需要额外空间和时间，就可以等内存满了再触发
    * 比如CMS，是需要GC和用户线程一起run，所以必然要留一点空间给用户线程做新对象的分配。所以比如要到70%占用就触发之类的
* mark and sweep算法伪代码
```
// mark
for each root in roots
    Mark(root)

Mark(root)
    If markedBit(root) == false then
        markedBit(root) = true
        For each v referenced by root
            Mark(v)

// sweep
Sweep()
//注意这里的each object in heap应该是applicaiton自己维护的放置所有对象的
For each object p in heap
    If markedBit(p) == true then
        markedBit(p) = false
    else
        heap.release(p)
```
* CMS
    * concurrent mark and sweep
    * 步骤:
        * 主要是mark和sweep。然后是针对老年代的。
        * 因为涉及到concurrent，所以要么是mark是c，要么是sweep是c
        * 或者应该说，这里应该先讨论的问题是，当GC和用户线程concurrent work的时候，会出现什么问题
            * 最多就是有浮动垃圾啊。没感觉有太大的问题。只感觉会收的不充分，但是不会出错啊
            * 因为就是会漏标嘛。
            * 哦但是漏标的话，在你做标记的时候，假设内存地址A没标记到，然后当你在做sweep的时候，你就把它当dead清理了。
            * 这样的话就会很危险。
    * 步骤
        * initial mark
            * STW
            * 顺着GC root标记，一旦标记到老年代就停止
            * 就是要把所有老年代的树头标出来
        * concurrent mark
            * GC和application线程并发
            * 根据第一阶段标记出来的所有alive的老年代的头节点，继续递归标记所有alive的对象
            * 对于并发执行下产生的一些新的对象或者新的对象关系，要标记起来。
                * 新生代promote到老年代的对象
                * 直接在老年代分配的对象
                * 老年代的引用关系发生变更
            * 以上这些在这一阶段发生变化的对象所在的card都会被标记为dirty。
        * concurrent preclean
            * GC和application线程并发
            * 重新标记上一阶段的的dirty card
        * remark
            * STW
            * 这部分是重新扫描root。
            * ？？？？？？？？？？
            * 不明白如果这步骤重新扫描了，你前面几步干嘛去了。就完全没必要了
        * concurrent sweep
            * 遍历Ordinary Object Pointer table
            * 这个表里是记录所有的对象
            * 找出dead的，然后它的内存重新加入freelist。
            * 也就说下次这块内存就可以重新使用了
        * concurrent reset   
            * 对所有的markedBit做reset。
            * 相当于重置状态，因为下轮CMS是要用。
* Promote mode failure vs Concurrent Mode Failure
    * Promote mode failure
        * 在进行minor GC的时候
        * survivor已经没空间存放所有copy过来的活着的对象
        * 就触发了promote，把survivor里本身年纪比较大的，移到老年代
        * 但是这时候，发现老年代也没有空间给他了。
        * 这时候会有一个promote mode failure
    * concurrentmode failure
        * 直接分配到老年代的对象发现没有足够大的地方了
        * 从新生代promote过来的对象发现没有地方了。
            * 所以某种程度上来说，promote mode failure会触发concurrent mode failure
        * 这时候就得来一个serial old(mark and compact)做STW的清理

* G1   
    
    
    
    
    
    
    
    
    
    
    
    
    
    



