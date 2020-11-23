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

* 宽窄依赖
    * 父的一个partition如果需要被拆分成多个，跑到不同的子partition中，就是宽依赖
    * 反之，如果父一个partition中的数据没被拆分，还是原封不动跑到另一个partition，就是窄依赖
    * 所以注意，union是窄依赖


* DAG
    * directed acyclic graph(有向无环图)
    * 根据宽窄依赖把DAG划分成不同的stage
    * 规则就是，如果有宽依赖，就要形成不同的stage
    * 因为宽依赖的性质是，它的行为需要parent的全员partition的参与
    * 所以，它一定得“等待”parent全部计算完了，它才可以开始计算
    * 这个“等待”就是stage划分的要义
    * 而窄依赖就是，partition间互不影响，所以你只要这个partition好了，我就可以接下去计算，不管你其他的状态
    * 所以这个stage可以说是，可以一起计算的全部东西，不需要等待谁
    
* 任务集
    * 一次submit就是一个Application
    * 一个action operator就会有一个active job
    * 每到一个shuffle dependency就会划分一个新的stage
    * 每个stage根据result rdd的partition数量，每个partition是一个task
    * 注意是根据result rdd的partition数量，而不是parent rdd
    * 比如reducebykey，如果parent是5个分区，结果是3个分区，就是3个task
    * 因为是从结果出发嘛，去拉取我要的数据来做计算。

* stage数量
    * 1 + shuffle数
    * 因为最后会有一个 resultStage

* cache/persist和checkpoint
    * 两个都是rdd的方法
    * checkpoint是存在hdfs，persist是可以内存也可以磁盘
    * checkpoint在job跑完之后，不会被删，而persisit会
    * checkpoint是reliable的，persist不是
    * checkpoint会切断lineage，persist不会
    * ============
    * 为什么需要
        * 空间换时间呗。提高性能啊，不用每次都算
        

* 分区
    * hash分区和range分区
    * hash是默认的
    * ============
    * 要使用range分区，必须要求数据是有序的

* RDD, Accumulator, Broadcast
    * 一个只读，一个只写。都是共享的
    * broadcast可以减少数据传输
    * accumulator可以做到executor间共享
    

















     