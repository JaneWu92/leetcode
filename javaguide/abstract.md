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
    }
    putVal(hash, key, value){
        idx = hash;
        if(table[idx] == null){
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
    * 如果不是这种情况，是涉及总体的，比如countercell[]扩容或者初始化，或者countercell[idx]==null要初始化.
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
    * 也就是说，这个粒度还是以bin位单位。就是我扩容做到tab[4],我后面的tab[]都还是可以put的。不受影响。
    * 也就是说，用户put会发生三种情况
        * 被在扩容的线程synchronized(f)住，啥都不做，等着
        * 发现当前槽位上的是forwarding node，也就是说，我的put操作不行做，因为这个map在扩容，就去帮忙扩容
        * 当前的这个槽位啥事没有（即使其实map在扩容，但暂时还没扩到这个位置），那他就去synchronized(f)，正常操作。
    * get的话，其实除了用了getvolatile，其他什么限制也没有。就正常读
    * 然后如果是碰到forwarding节点。就通过forwarding节点去newTab里面找。
    * 因为new ForwardingNode<K,V>(nextTab)，forwarding里面其实就是放了newTab的地址。 
* 扩容过程
    * 一个一个槽位地去扩容
    * 也是一样，synchronized (f),锁住头节点，然后去高低链分出来，放到new table里
    * 然后把头节点放成forwarding node。
    * 本来以为这个forwarding node是 
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
??????????????????????????????
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
 
### 锁升级过程
感觉这个锁的机制还不是太懂。monitor exit，monitor enter到底是什么
      
### Synchronized和lock的区别

### AQS是什么东西
     
     
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
   ?????????????????
        
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
    
    
    
    
    
    
    
    
    
    
    
    
    
    
    



