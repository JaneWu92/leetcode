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



