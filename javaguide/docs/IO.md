
### IO Type
BIO,NIO

blocking IO是阻塞式的。read函数必须得到响应后，线程才能够继续往下走。
所以在这种IO下，经常的实践是为每个过来的request开一个新的线程去处理响应。
问题是，当request过多，受限于线程数，服务不过来。性能也不好

NIO是java后面新提供的API。
ServerSocketChannel会注册到selector里面。
然后所有到这个ServerSocketChannel的
这个selector能够监测到来到这个channel的所有事件，比如说有没有新的socket过来，这个是accept。
对于已经是accept的了，就注册read.
然后用户线程只要阻塞在这个selector的select上，就可以阻塞等待自己感兴趣的事件发生。
比如我上次注册的read如果ready了，我就可以被唤醒。

从high level来看，它跟之前的差别是，能够batch观测IO结果。这么一个功能。
有点像生产者消费者的意思。
主线程就是生产者，一直接请求，往池子里扔。
然后应该是有系统线程，是这个池子的消费者，一直去拿里面的东西，检查它的IO状态。
然后它再作为生产者，把状态okay的那些IO往另外一个池子里拿，然后主线程再从这个池子里拿东西。
这时候拿到的就是都是状态成熟可以做IO action的了。就去遍历处理他们。

这里面能够实现这个，最主要的是操作系统层面的IO提供batch观测状态这么一个功能。
像poll，和epoll。一次能

### select, poll, epoll
https://www.cnblogs.com/aspirant/p/9166944.html
java nio的selector，上面这三种作为底层实现，都是可以选择的。
在linux下，NIO可以采用epoll来实现非阻塞，注意，是在linux下：
-Djava.nio.channels.spi.SelectorProvider=sun.nio.ch.EPollSelectorProvider

select和poll差别不大，都是用户要把fd set作为参数传给操作系统。因为都要遍历，所以时间复杂度是o(n)
只是select对于传进来的fd set的大小有限制，应该是因为有一个fd_setsize，bit数有限。
而poll用的是链表，所以大小没有限制。
epoll用的不是遍历，是事件驱动，所以事件复杂度是O(1)
而且它的fdset不是每次都在用户空间和内核空间考来考去，是直接写到内核空间。也省了很多。
也就是fdset它只拷贝一次，而select和poll是次次拷贝。



### zero copy
比如说要读一个文件，写到网络上
普通的是，
文件 -> kernel read buffer -> application space
application space -> socket buffer -> NIC(网卡)
所以这里面有很多拷贝，
第一个kernel read buffer -> application space，
第二个application space -> socket buffer
要注意，对象是OS，它要帮我们做以上两个拷贝
而zero copy，对于同样的情况
要读一个文件，写到网络上
文件 -> kernel read buffer -> NIC
完全不需要拷贝。
应用：kafka。因为kafka就是不断地做以上的操作（读文件写到网络上）

### zero copy的两种技术
mmap和sendfile
sendfile：
DMA从file到kernel buffer
DMA从kernel buffer到NIC
mmap：
用户态和内核态共享数据缓冲区
DMA从file到kernel buffer
用户态直接引用文件句柄（kernel的），把kernel buffer的数据直接DMA传到NIC
java中的实现
```
// 将当前 FileChannel 的字节传输到给定的可写 channel 中
public abstract long transferTo(long position, long count,  WritableByteChannel target) throws IOException;
// 将一个可读 channel 的字节传输到当前 FileChannel中
public abstract long transferFrom(ReadableByteChannel src, long position, long count) throws IOException;
// 对 Channel 做 mmap 映射
public abstract MappedByteBuffer map(MapMode mode, long position, long size) throws IOException;
```
注意，DMA是一种技术，在device和kernel buffer中间的传输，不需要cpu介入






























