## Code
```aidl

```
## 逻辑
### addCount
* 加数的部分（借鉴LongAdder的部分）
    * 如果counterCells是null，就去cas base
    * 如果counterCells不是null，然后hash匹配到的cell也不空，就去cas cell
    * 如果hash匹配到的cell是空，就去cas拿cellsBusy这个控制权，然后新建cell放进数组。
    * 如果两次或者多次，都还是cas不到相应的cell，就是证明很多人在忙，就去countercell扩容。扩容也有NCPU的限制
* 扩容的部分
    
### LongAdder
* cells是null并且cas(base)成功，return
* cells不是null，并且len>0
    * 如果hash到的那个cell是null
        * cas cellsBusy这个乐观锁
            * 拿到了就去新建一个cell(x)放进数组
            * 没拿到就重新去循环。（因为没拿到就是说明有人现在正在拿着cellsBusy这个锁，要么在做你的“新建cell放进去”，要么。。。）    
    * 如果hash到的那个cell不是null
        * 不是null就证明此cell有人init了，就直接cas加上去。
            * cas成功，return
            * cas不成功，竞争太激烈了。就去扩容。
                * 如果已经超过NCPU了，就不扩容。因为最多也就N个cpu，你再多线程也被物理并发量卡死了。

## points
### cas能保证线程可见性吗
* AtomicLong的incrementAndGet是用CAS加上volatile来保证原子性和可见性的。
### tabAt为什么要用unsafe.cas而不是table[i]
* 为了保证可见性呗。
    * table虽然用了volatile，但是只是这个数组的引用是volatile
    * 数组中的元素，还是non volatile，也就是说，每个线程可能都有自己cache住的数组中的元素的值
    * 如果是table[i]，可能拿的是cache中的值
    * 而unsafe的cas，是以volatile的cas的形式，来操作，就可以直接拿到和写入不会引起多线程问题的值。
### 怎么做到锁元素的
* Node e
* synchronized(e)
### TreeNode Hashmap和Concrrent HashMap实现的不一样
* HashMap
    * TreeNode extends LinkedHashMap.Entry
* ConcurrentHashMap
    * TreeNode extends Node
    
### counterCell和table的操作的锁粒度差别
