### synchronized & Reentrantlock  
Basically they are same. Different is that Reentrantlock has more ability. It can provide:  
1. trylock  
2. reenter mechanism  
3. fair lock  
4. lock interruptibly mechanism.(synchronized needs to wait all the time)  

### volatile
1. visibility
2. happens-before(prevent instruction reordering)
3. can't prevent race condition since it's not atomic instruction  

**Since volatile can't prevnet race condition, when is it enough?**  
When only one thread is write and others are all read. It's enough then.  
Once there's multiple threads to write for a same data, you should use synchronized.

### FixedThreadPool vs CachedThreadPool
FrixedThreadPool: can assign thread queue number, but work queue number is MAX_INT  
CachedThreadPool: thread queue number is MAX_INT, work queue number is 1  
So if tasks too much, one will OOM bcz task queue full, one will OOM bcz not able to create native thread  

In this case, we need to **create threadpoolexecutor by ourselves** 
```java
ThreadPoolExecutor threadPool = new ThreadPoolExecutor( 2, 5, 5, 
    TimeUnit.SECONDS, new ArrayBlockingQueue<>(10), 
    new ThreadFactoryBuilder().setNameFormat("demo-threadpool-%d").get(), 
    new ThreadPoolExecutor.AbortPolicy());
```

### ConnectionPool
**Name convention**
*Pool, *Client, *Connection  
*Pool is for user to get connection. Mostly it's threadsafe. User need to return connction when it's complted using.  
*Client is for user to directly use. User don't need to care about connection. It's automatical maintain by the *Client API. Threadsafe.  
*Connection is not thread safe. And totally maintained by user itself.  
  
If you use nonthreadsafe connection, like Jedis and multiple threads use this connection at the same time. It comes issue.  
Because one Jedis maintain one socket. And socket is not safe to share with multiple threads.  
Fixing by get one connection for each thread by connection pool.  

Close pool vs close connection:  
Pool close is physical close and connection close is return to pool.  

### 锁的升级
偏向锁，轻量锁，重量锁  
偏向锁：  
1. 加锁：看锁标志是否为“01”，CAS在markword里的字段直接填入Thread ID,把”是否偏向“设为1。如果成功就是获取到了，如果不成功就做锁的撤销 
2. 锁的撤销：不会主动释放。而是等其他竞争的线程来触发。并且等到程序到安全点的时候。
    - 如果检查这个threadid已经不在用这个锁了，把threadid去掉
    - 如果还在用，就升级成轻量级锁。把锁标志设为“00”。把mark word拷贝到在用的thread的栈上，作为lock record。然后把mark word上的owner填入这个lock record的指针。

轻量锁:  
把mark word拷贝到自己的栈上。然后CAS把mark word上的owner填入自己栈的lock record的指针。  
成功就获得锁。不成功就自旋等一会儿。等了一定时间还没到，就把这个锁升级成重量级锁。  
  
重量级锁：  
轻量级锁其实还是处在自旋，也就是线程没有挂起。就没有涉及到线程的切换。没有涉及到内核态和用户态的切换。代价就比较小。  
但是重量级锁，利用了monitor，没拿到的线程都要进入wait状态。

A monitor consists of a mutex (lock) object and condition variables.  






















