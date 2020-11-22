* Mapreduce
    * 不好的地方
        * 每一次计算是单独的。
        * 连接单独的计算的介质是文件（落盘夺慢啊）。
        * MR和hadoop紧密耦合。（2.0版本是yarn）
    * =======================
    * 每一次的计算，都是单独的。单独的意思是，从磁盘取，计算，落盘。
    * 如果你要两次计算，你必须第一次计算完，落盘，第二次再计算
    * 所以不适合迭代计算（很多次计算）的场景
    

* 部署模式
    * Spark一共有5种运行模式：Local，Standalone，Yarn-Cluster，Yarn-Client和Mesos。
    * standalone(local)
        * master, worker
    * yarn
        * resource manager, node manager
    * 上面说的，都是资源管理器
  
  
* shuffle
    * 必须是RDD里本来的partition被打乱了。
    * 所以比如union这种，是p1，p2都到另外一个partition去。虽然p1，p2被合起来了，但是p1的数据对于它自己来说，是没有变化的，所以没涉及到shuffle
    
* coalesce
    * 缩减分区
    * 其实用的是合并，比如4->3，用的是把其中两个分区合并了
    * 所以是没有shuffle的
* repartition
    * 有shuffle。
    * 它用的其实就是coalesce的shuffle=true
           
* map和mapPartition的区别
    * map是对每一条数据，都要driver序列化方法过去executor让他执行
    * mapPartition是每个partition才需要序列化一次方法过去
    * 而进程内的通信比进程间的通信快1000倍
    * ===============
    * 我怎么觉得这个每一条数据都要序列化方法过去的这个说法不太靠谱
    * =============
    * 还有一个差别是，map是每条每条，所以处理完可以GC
    * 但是mappartition一定得整个partition处理完了，这个数据才可以被GC，如果太多，比如20G什么的，GC可能会有问题
    * 因为前者没跟partition关联，可以被GC，后者跟partition关联了
    
* groupby
    * 按照元素模2的值进行分组
    * rdd.groupBy(_%2)
    * 结果差不多是Array((0,CompactBuffer(2,4,6)),(1,CompactBuffer(1,3,5)))    

* distinct
    * 有shuffle
    * 有shffle一定要落盘，所以会慢

* shuffle write一定会落盘

* Action operator
    * sc.runJob


















     