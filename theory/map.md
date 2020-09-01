## 总体
因为所在项目的
## JVM原理
### JMM
栈和堆，还有方法区  
因为JVM其实是计算机的模拟。而计算机主要是CPU，寄存器，和主存构成。  
对每个CPU来说，他都有自己的一组寄存器来存放当前这个线程的所有上下文。  
对应到JVM来说，就是栈。而主存对应到JVM来说，就是堆。  
但是这还是数据区，还有一个是方法区，放代码的地方。  
堆的内存管理是JAVA自己提供的。  
而栈的内存本来就是自动管理的，应该是操作系统直接提供的。比较快，但限制也比较多。比如只能固定大小是栈的宽度。  
有一些问题，比如，静态变量放在哪里。因为静态变量是跟类相挂钩的，所以我觉得是放在方法区。  
**correcting**  
JMM里面，只有栈和堆。 
栈放的是线程的调用栈，还有用到的variable。  
如果这个variable是primitive的，他就直接存在栈里，如果是object的，object在堆里，栈里放object的地址。  
堆则是被创建处理的所有其他的对象。
所以这里重要的一点是，上面说错的，方法去也是堆里面的。  
因为所有的class，在内存中也是class对象，class object，所以当然也是在堆里面。  
但是方法区，是有跟数据区隔开的，因为可能因为他们的生命周期的性质不同的关系。  
分开会比较好管理。  
**correcting again**  
java metaspace是在native/direct memory里，不在stack也不在java heap里。  
但他同样也有GC。 可以设置ratio，超过多少就开始做GC。ratio应该设小一点避免metaspace OOM。  
**field vs local variable**
field is class field or instance field.  
and local variable is inside a method.  
栈里放的是function call和primitive local variable 或者object local variable的reference。
所以field是放在堆上的。
### JMM and hardware memory architecture
**difference between stack and heap**
stack: local variable and function calls  
heap: objects  
那既然每一个线程都有自己的stack并且他们是不互相分享的，为什么还会有多线程的问题呢？
这里要搞清楚一个问题：stack里面的数据确实是不互相分享，也确实每个线程都有自己的一份。但并不是说，线程的数据全在栈上。   
也就是说，栈上的数据只是线程能够访问到的数据的一部分。  
CPU是有自己的寄存器和CPU缓存，并且别人也访问不到。  
但是要注意这中间有两部分。一部分是thread自己的local variable。另一部分是共享的。应该说，除了local variable，其他都是共享的。  
所以共享就会涉及到数据竞争的问题。包括缓存也会涉及到缓存不一致的问题。
### Java GC
算法分类，mark-sweep, mark-copy, mark-compact
mark-copy显然更适合生存的object数量少的情况，mark-sweep更适合生存的object比较多的情况  
所以一个是对应年轻代，一个是对应老年代。
关于mark-compact,跟mark-copy差不多，只是一个牺牲时间，一个牺牲空间。
serial mark-copy, serial mark-sweep
parallel mark-copy, parallel mark-sweep
parallel new, CMS(concurrent mark-sweep)
1. Serial young + Serial old(when JDK first released)
2. Parallel scavenge + Plrallel old(to improve performance)(默认的)
3. Parallel new + CMS(to resolve the STW problem)
新生代普遍用的是copy，老年代普遍用的是compact，除非特殊说明的比如CMS。
内存比例：新生代老年代1：2。老年代更多，可能是由于程序的本质决定。不可能所有的都是短暂的对象。
后面的G1就不是物理的分代了。  
**why is minor gc(young generation gc) a lighter gc than major gc(full gc)**
minor gc虽然说发生在年轻代，但难道只可以针对年轻代扫描吗。都要从根节点扫描起。
而root主要包括：静态对象，当前所有active线程的线程栈内的对象。还有JVM自己持有的对象比如类加载器。
root的标记是指这些对象的标记，然后会顺着这些再标记他们发散出去的对象。
所以说，其实标记这个操作，每次（无论你是什么GC）都是要全部标记一遍的。
而之后，才是，你要做minor GC，那你就扫描年轻代，你要做major gc，你就扫描老年代。或者full gc扫描全堆。
这里的扫描是指，遍历相应的代里的所有object。
所以题目自然。
**what is card table used**
卡表。在老年代里，把老年代的堆分成一个一个的卡，对应这个部分的老年代的堆里是否可能有指向年轻代的对象。是就记为脏。
这个是为了节省minor gc的时间。如果没有这个，那么就要遍历整个老年代，才能标记出所有他指向的年轻代的object。  
但是有一个疑问，无论任何gc，本来就是要从GC root开始标记起。不知道他们这个是在搞什么鬼。
这里的指导思想可能是，从GC root开始标记起，其实就会标记出很多东西来，因为老年代我们的预期里大部分是存活着的。
所以每次这样就很麻烦。干脆我就维护一个关系，把这种“指向”的双向关系记下来。
一方面，在老年代，我记录说，我这个卡(这一小片内存)里面是不是有对年轻代的引用。
一方面，在年轻代里，我记录对我这块region（有很多card）的所有的引用（old card）。然后再去old card里面正向扫描，就知道young的这个region里面哪些object是活着的。
一个问题，既然都记了rset，为什么只是记录old card，而不是干脆直接当场记录我这个young region里面到底哪个是活的？
答案可能是，人家空间换时间，但是有一个程度。你不能全用空间直接做下来啊
**G1 algorithm**
年轻代和老年代杂合。  
Gabage first，垃圾多的块先回收。不管他是什么代。
所以过程就是，无论是谁在分配时碰见了困难，就触发一次GC。
这个GC的过程就是从GC root开始扫描，然后把整个对象图找出来。
然后他可能还维护了一个块的垃圾信息。知道这个块里面有多少垃圾（或者说多少活着的）。然后能够决定回收最多垃圾的那块。
1. 在这种不同的分代策略下，什么时候触发垃圾的回收。
https://www.cnblogs.com/yufengzhang/p/10571081.html
### Java class structure
### Java how to load class
### JVM arguments
### Java core
#### HashMap
1. hash值
    1. hash函数是：(h = key.hashCode()) ^ (h >>> 16)。相当于把高16位的信息也融入进低16位。
    因为这个hash值映射到table index上是通过取模的，所以常常可能只用了低位的信息。 比如如果16个桶，就只用到低4位的信息。
    2. Node有一个字段维护了hash值，应该是空间换时间的做法
    3. hash函数：把任意长度的输入通过散列算法变换成固定长度的输出,不同的输入可能会散列成相同的输出.
2. 扩容
    1. 第一次put的时候会扩容，初始化1<<4 = 16 个。或者是说size * factor > current size会扩容。
    2. 扩容过程
        1. e = oldTab[j]; oldTab[j] = null;
        2. if e.next == null, newTab[e.hash &(newSize - 1)] = e
        3. if e instanceof TreeNode, split Treenode to lo list and hi list, and put to new Table.
        4. if e instanceof Node, split loNode and hiNode based on (e.hash & oldCap), put to new Table.
    3. hashmap死循环：1.7版本
        1. 主要是在扩容的时候因为用的是头插法，顺序会反过来。
        2. A -> B-> C， 假设T1标记了e是A，next是B，然后T2也来，把B->A给做了。然后T1再来，也做了A插入，然后把next标记成B.next，也就是T2做的好事，结果是A，结果这里就有了一个A<->B之间的闭环。再get或者put的时候都会有问题。
    4. hashmap死循环：1.8版本
        1. 在1.8版本，不是用1.7的头差，而是高低链表的尾插，没有了1.7的问题。
        2. 但是在treeify的过程中还是会出现死循环。
3. treeify
    1. length <= threadshold - 1
4. ConcurrentHashMap
    1. 1.7和1.8的性能有很大的不同。
    2. 为什么get不需要加锁
        1. Node的val和next都是用volatile修饰的。也就是说，线程间是有可见性的。
### java并发编程
首先要提到为什么有并发这一说，因为多线程对共享资源的并发处理会引起我们期望外的错误结果。其实呢，他的核心是原子性。也就是说，我在access这个共享变量的时候，别人不能access。
但其实，在单cpu的情况下，多线程并发执行也会有问题。即使说单cpu情况下，多线程也已经是物理上的串行了。所以这个原子性是指，整个事务的原子性。这个事务不管你是多简单，还是多复杂。只要你把他定义成一个事务，也就是你希望一次只能有一个线程在做这个事务，并且要一口气做完。那么你就需要某种机制，来保障这个效果。
最朴素的思想就是，如果我有一把锁，要访问那个资源做一系列操作的时候，我就把这个锁锁上，也就是把这个资源的访问锁上，也即，在我做这个资源访问的时候，没有人能够打断我，也没有人能够碰这个资源。
当我做完了，再把这个锁交出来，给下面一个能够获得这个锁的幸运儿。
okay那么问题来了，怎么样来实现这个“锁”的功能。这个锁其实是一个“排他”的功能。
比如说最简单的，我可以有一个变量，比如是int型，然后它的值，是拥有它的那个线程的threadid。然后它有一个表示没人占用的值，比如-1。
某个thread要去获得这把锁，其实就是去把自己的threadid设置进这个int型的变量。出来了再把它设成-1。
acquireLock：
    if lock == -1:
        lock.threadid = threadid.
    else:
        wait()
但这里有一个问题，你这一段的原子性又怎么保证。。可能多人都去走第一个分支了，结果都去执行那段只能一人执行的代码。
所以这里可以用compareAndSet，UnSafe提供的一个原子接口。compareAndSet(myThreadId, -1)。失败的就自己去wait，成功的就成功拿到。
如果要能够重入的话，就是先加一段，compare(myThreadId),如果是自己的话，就不compareAndSet,直接工作起来了。
所以让我们来看看java里面的实现是不是这样：
synchronized和lock。
##### synchronized
偏向锁。轻量级锁。重量级锁。
这个偏向锁其实应该就是我们上面所说的，把自己的threadId set到这个object的头里面。
但是如果一旦发现，这个操作失败，就升级到轻量级锁。轻量级锁应该是空旋等待这个compareAndSet的操作成功。
然后空旋太久的话，就会升级成重量级锁，就是wait。等那个拿到锁的人成功了后去notify那些在等待的人。
我猜测，这里所有的基础，都是compareAndSet。
#### lock
lock.lock
我感觉好像可以是跟synchronized一样操作的。只是说它多了一些功能。比如可以打断，不等了，还有可以重入（重入应该syn也是可以的）。还有可以实现公平锁和非公平锁。
我觉得就是把syn的那个对象变成lock的这个对象，应该就可以了。但是lock有一个，是需要自己释放锁。这个是需要注意的。这个的次数跟unlock应该也是要对应的。
#### 乐观锁和悲观锁
乐观锁是一种思想，认为大部分情况下数据都不会有问题。
U.compareAndSwap： 成功就true，不成功就false，所以你要自己循环。缺点是不能解决ABA问题。
### java线程池
corePoolSize, maxmiumPoolSize,keepAliveTime, workqueue(blokcing queue), rejectHandler  
线程刚进来，如果size小于corePoolSize，会create一个新线程提交任务，即使当前的pool里已经有空闲的线程。  
这里的指导思想应该是说，这个core本来就是要常驻的，所以就干脆在coresize这个范围内，不管你有没有空闲的，我都create上。  
如果一个任务来的时候，这个已经到达coresize了，就去找空闲的线程，没有空闲的线程，就去放在workqueue里面，等其他的任务执行好，空出资源。  
这里的指导思想应该是，coresize是我想尽力保持的一个量。所以已经排满了但还是有任务进来，就先排队等着。  
然后，如果实在人流太多，等的队列都被挤满了，我就去create新线程。直到到达maxsize。这个maxsize已经是我的极限了。  
所以任务还是多，还是再来，我的workqueue装不下了，也没法create新线程了，所以我就得有一个机制去解决你这个request，这个就是rejecthandler做的事情。  
rejecthandler:
1. r.run 2. throw exception 3. donothing 4. queue.poll; e.execute
### 

## IO框架
BIO, NIO, AIO
### BIO
blocking IO. 阻塞IO。所有的accept，read，write都是阻塞的。所有一般来说用这个的时候都是当有新的请求过来，就新开一个线程去处理这个socket。  
这里有一个很大的问题是请求量一大。就有问题。  
后面是出现了NIO，new IO，Java也提供了NIO的接口。就是这个selector。
把Serversocket和selector绑定，并且注册accept事件。这样事件到达的时候，selector.select会返回。不然线程就会一直阻塞在这里。
这里有几点要注意。
1. selector.select是会一直阻塞的，直到有任意的socket是ready的。
2. select返回后，是没有指明哪些socket是ready的，所有java线程还是要去一个一个遍历检查。
select这个机制是怎么做的呢。其实都是底层的操作系统。有个select函数。
select函数接收的参数是所有你关注的socket的fd。当你调用这个函数的时候，操作系统其实是把你的线程加到这每个socket的等待队列里面。
socket这个结构有三块核心区域读写缓存和等待队列。放到这个等待队列里面，然后也把这个线程从工作队列移除，线程就挂起了。
然后客户端有数据到达传输到kernal内存。就会触发一个中断。中断就是让cpu停下他手头的进程，做我当前这个中断要做的事情。
这个中断就是把数据从kernal内存拷贝到socket缓冲区（数据包因为有port，所以知道是哪个socket的）。
然后把这个socket的等待队列的线程加入到工作队列。反映到java层面就是你这个线程被唤醒了，又有机会获取到cpu的时间片。  
因为select不返回具体的ready的socket，所以java线程接下去就是去做一次遍历所有socket，对那些已经ready的进行处理。
NIO代码：https://www.jianshu.com/p/183163eaac15

===========
NIO，只有一个线程来负责监听所有的IO事件。
主线程可以阻塞地

===========

## 数据库Mysql
1. 返回每科成绩最好的前两名学生。
```
Student, Teacher, Score, Course
select s1.* from Score s1 where
( select count(1) from Score s2 where
    s1.c_id = s2.c_id and s2.s_score >= s1.s_score
)<=2 
order by s1.c_id, s1.s_score desc;
```
## Spark分布式
## 网络基本概念
## 操作系统
进程是操作系统资源分配的最小单位。  
线程是操作系统任务调度的最小单位。
### 进程上下文切换
1. 虚拟地址的切换
2. 当前的CPU寄存器，PC这些东西都要打包放进内存。以备待会儿恢复。
### 线程上下文切换
只需要上面的第二个步骤。
## Linux
## Spark
### narrow vs wide dependency

### 消息中间件怎么保证数据不丢的
1. 弄丢的种类：rabitmq
    1. 生产者弄丢了数据
        1. 传输过程中因为网络问题把数据弄丢了
            1. 开启事务，消息发送到服务端了再返回，不然就回滚事务。缺点是一下这个数据的发送就变阻塞了，吞吐量下降。
            2. 客户端开启confirm模式。每条消息都会有一个id，客户端还是发他的。如果server端收到，会调用
    2. mq自己弄丢了数据
        1. 没开启持久化，那么一旦mq重启，就弄丢了
            1. 
    3. 消费者弄丢了数据
        1. 刚消费到，还没有处理，消费者就挂了。类似于autoack，就会丢失
2. 弄丢的种类: kafka
    1. 生产者弄丢了数据
        1. 生产者没有设置相应的策略，发送过程中丢失数据。
    2. kafka弄丢数据
        1. kafka耨个broker宕机，
    3. 消费者弄丢了数据
        1. 消费者收到数据时自动提交了offset，然后要处理这个消息时自己挂掉了。



https://blog.csdn.net/zengdongwen/article/details/100587102
https://www.jianshu.com/p/e57d9d354faf


### 单例模式和volatile
https://www.cnblogs.com/prayers/p/7468540.html  
几个要点：
1. 不把synchronized加在方法上，粒度太大了，比如getInstance里面如果还需要其他计算，就会blocked住所有的人。所以放在判断myobjct是空之后，这样如果myobject不是空，大家也不需要去等了，可以并行拿。
2. 因为是静态变量，静态方法，所以synchronized锁的是MyObejc.class，一个JVM里全局只有一个。
3. 为什么synchronized里面还要判一次空？因为多个线程同时在等锁的时候，第一个拿到的，等它释放锁后，肯定是myobject已经实例化过了。这样后面已经在等待队列的所有线程，等他们拿到锁时，必须禁止重新实例化。
4. 为什么要加volatile？
    1. 一个是线程间的可见性。在synchronized后。可能你这个变量只在thread1的cache里，thread2看还是null，所以又会去实例化一个。
    2. 一个是指令重排序。在synchronized前。实例化有两步，1）开辟一块内存并且去初始化 2）新建一个变量，变量值去set成刚刚开辟的那块内存的地址。
        1. 可能拿到锁的线程先进行了第二步，而第一步的初始话还没完成。
        2. 造成：thread2在第一个判空处不成立，就直接拿着还没初始化的myobject去用了。引发错误。、
    3. 基于以上两点，用了volitile，都能够避免掉。
5. 为什么要有private的构造函数？禁止他人自实例化，只能通过getInstance的入口。
```java
public class MyObject{
public static volatile MyObject myobject;
private static MyObject(){
}
public static getInstance(){
    if(myobject == null){
        synchronized (MyObject.class){
            if(myobject == null){
                myobject = new MyObject();
            }
        }
    }
    return myobject;
}
}
```

### Spring AOP
把一些系统性的功能抽象起来，避免代码的重复，也能够做到解耦。  
```aidl
class LogAspects{
    public void mybegin(JoinPoint joinpoint){
        System.out.println(jointpoint.getSignature.getName(), jointpoint.getArgs());
    }
}

<bean id = logAspects, class="mypackage.LogAspects">
<aop:aspect ref = logAspects > 
    <aop:pointcut expression="execution(* *.*(. .)) id = "log"" >
    <aop:begin method = "mybegin" point-cut-ref="log"> //the method is the method implemented in Class logAspects
    <aop:afterrunning >
    <aop:afterexception>
    <aop:after>
    <aop:around>
</aop:aspect>
```
Advice, joinpoint, pointcut  
pointcut是你自己规定的某一些你觉得需要有同样的advice的方法
joinpoint是在这个pointcut上的某一个
Advice是在pointcut上都需要做的事情

### Zero Copy
https://developer.ibm.com/technologies/java/articles/j-zerocopy/  

普通的一个IO操作（从磁盘读文件，然后发送到网络的另一端）中间是什么过程呢。  
1. User mode -> Kernal mode: 
    1. user ask kernal to do system call read(file)
    2. data is copied from file to kernal buffer
    3. data is copied from kernal buffer to user buffer
2. Kernal mode -> User mode:
    1. kernal read returns and control goes back to user. 
3. User mode -> kernal mode:
    1. user ask kernal to do socket.send()
    2. data is copied from user buffer to kernal buffer. 
    3. data is copied from kernal buffer to socket buffer
4. Kernal mode -> User mode:
    1. kernal socket.send returns and control goes back to user

零拷贝  
1. User mode -> Kernal mode: 
    1. user ask kernal to do FileChannel.transferTo
    2. data is copied from file to kernal buffer
    3. data is copied from kernal buffer to socket buffer
2. Kernal mode -> User mode:
    1. Kernal tranferTo returns and control goes back to user
```aidl
https://www.cse.iitb.ac.in/~mythili/os/notes/notes-nw-sockets.txt#:~:text=*%20Socket%20buffers%3A%20Every%20socket%20has,buffer%20(in%20user%20space).
* Socket buffers: Every socket has a pair of buffers (memory pages) to store sent and received data. The read system call reads from the socket receive buffer, and copies data from the socket receive buffer (in kernel space) to the user provided buffer (in user space). If there is no data in the socket buffer, read blocks (unless a non blocking option is enabled). 
The write system call copies data from the user provided buffer to the socket transmit buffer, and initiates further processing and transmission of the packet. The write system call may block if there is no space in the socket buffer (unless non blocking option is set). Why would there be no space in the write buffer? For example, the transmission may not be happening fast enough due to TCP congestion control.

```
##  Direct Memory Access(DMA)






















