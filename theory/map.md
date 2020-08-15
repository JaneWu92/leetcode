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
**synchronized**
偏向锁。轻量级锁。重量级锁。
这个偏向锁其实应该就是我们上面所说的，把自己的threadId set到这个object的头里面。
但是如果一旦发现，这个操作失败，就升级到轻量级锁。轻量级锁应该是空旋等待这个compareAndSet的操作成功。
然后空旋太久的话，就会升级成重量级锁，就是wait。等那个拿到锁的人成功了后去notify那些在等待的人。
我猜测，这里所有的基础，都是compareAndSet。
**lock**
lock.lock
我感觉好像可以是跟synchronized一样操作的。只是说它多了一些功能。比如可以打断，不等了，还有可以重入（重入应该syn也是可以的）。还有可以实现公平锁和非公平锁。
我觉得就是把syn的那个对象变成lock的这个对象，应该就可以了。但是lock有一个，是需要自己释放锁。这个是需要注意的。这个的次数跟unlock应该也是要对应的。
**乐观**
## IO框架
BIO, NIO, AIO
## 数据库Mysql
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
