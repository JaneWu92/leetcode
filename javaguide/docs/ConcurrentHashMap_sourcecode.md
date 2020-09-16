## Code
```aidl

```
## 逻辑
### putVal
* table是null，去initTable()
* table不空，hash到的那个bucket是空，就cas把新的node放进去
* hash到的bucket不空，但是是forwardingNode，去helpTransfer()
* hash到的bucket不空，就锁住这个bucket里的节点，synchronized(f)
    * 如果是链表(hash>0)
        * 如果找到key相同的，直接把newVal替换进去
        * 如果没找到key相同的，新建node加在尾node的next里。
        * 注意以上不需要任何什么原子性，可见性和顺序性的保证，因为已经是在synchronied(f)的包围下。什么都有了。
    * 如果是TreeBin
        * 就插到红黑树里，然后调整平衡
*addCount()
    * size+1 和 扩容控制
### addCount
* 加数的部分（借鉴LongAdder的部分）
    * 如果counterCells是null，就去cas base
    * 如果counterCells不是null，然后hash匹配到的cell也不空，就去cas cell
    * 如果hash匹配到的cell是空，就去cas拿cellsBusy这个控制权，然后新建cell放进数组。
    * 如果两次或者多次，都还是cas不到相应的cell，就是证明很多人在忙，就去countercell扩容。扩容也有NCPU的限制
* 扩容的部分
    * 会先计算resizeStamp = Integer.numberOfLeadingZeros(n) | (1 << (RESIZE_STAMP_BITS - 1))。
        * 这是一个1000 0000 0001 1011，高位的1就是要搞成负数，低位的1 1011，是n的leadingZero的个数。
        * 这是一个戳，就是你在扩容的每个size里，都会是不同的（n不同）。
    * 然后根据sizeCtl的值：
        * 大于0，就代表没人在transfer，就变成-2，然后去transfer
        * 小于0，就代表有人在transfer，
            * 条件符合transfer资格，就再-1，然后去transfer
            * 不符合资格，。。不知道这里什么情况是不符合资格
    * transfer()
### transfer
* 通过这个transferIndex全局变量来标记如果有下一个线程，应该从哪个段位开始去迁移（从后往前，每线程一次16个bucket）
* 然后在当前的自己的16个bucket里，一个一个去遍历
* 如果是null，就cas tab[i]一个forwardingNode
* 如果不是null，就把当前tab[i]这个位置上的Node锁住，synchronized(f)，然后去做转移操作。
    * 如果是链表，也就是hash>0。因为TreeBin的hash是-2。
        * 还是用高低位的方法，取出高低链，用unsafe的putObjectVolatile去放在nextTable
        * 把forwardNode放在table[i],unsafe
    * 如果instanceOf TreeBin
         * 也是去遍历treeBin（只是不知道treebin具体怎么也是和链表一样next遍历的），分高低链，和上面链表一样。

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
* ========上面说的都错了=============
* cas能保证线程可见性，人家用的都是setObjectVolatile，不管你变量加没加volatile
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
