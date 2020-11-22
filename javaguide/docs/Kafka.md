消息中间件
有两种形式：peer to peer + pub and sub
kafka is pub and sub
topic, broker
broker是跟server差不多。因为kafka是分布式的容错的。所以它会有很多broker
（要容错，只能通过分布式，没分布的话，一个挂就全部挂）
topic，是一个主题，可以有很多人来往里pub，也可以有很多的subscriber。
关于offset，是可以设置的，可以从头开始，也可以就保持你自己上次sub到的地方。
这个offset在kafka后面的版本，是放在kafka里的（之前的版本是在zookeeper里）
关于里面消息帮你保留的时间。默认是168小时，七天，也就是一周。
关于partition。每个topic都可以设置partition，是为了分流。比如一个topic有三个partition，那你就可以有三个consumer来消费。
consumer里有一个group的概念，在一个group里的，是去分销某个topic。
容易想见，如果consumer<partition，那就是一个consumer会消费多个parititon
consumer>partition，因为每个partition只能被一个消费者消费，就会有空闲的消费者出现。
一个consumer消费多个parition的时候，顺序是不能保证的，单个partition内部的顺序可以保证，但是partition间的顺序不能保证。
consumer和partition的分配。大概有几种，一种是range分，就是10/3=3余1，那就C1消费123，C2，456，C3，789，剩下的一个，再按顺序给C1
一种是roundrobin，先把partition的分区名按hashcode排序，然后轮流给C1,C2,C3.
kafka有副本的概念。比如设replicate数是4，意味着每个topic都会有四份。要注意不同的副本要在不同的broker上。
因为，副本是用来容错的。你放在一个broker上，要死一起死了，有什么用。
既然是副本，就要提到kafka是主备的配置。replicate是4，只有一个leader，其他3个都是follower。
意味着，publish或者subscribe，都是跟leader交互。follower是不提供服务的。
然后还有一点，partition可能最好也在不同的broker，因为partition的好处也有是减轻broker的load负担的。分在不同的broker上，broker的负担也比较小。
然后producer在做publish到一个topic，因为有多个partition，它的publish到partition的顺序也是不定的。
就是可能p1发一下，p2发一下，p1再发一下，p3再发一下。
偏移量的话，没有全局offset，只有每个partition自己的局部offset。所以没有全局有序，只有区内有序。
follow和leader之间的同步，是follow的拉取。
文件存储系统
topicA-0文件夹
topicA-1文件夹
0000000.log
0000000.index
0000456.log   //456是这个log的起始偏移量
00004546.index  //索引文件里存的是每条消息的偏移量
因为index存的是偏移量。所以数据的大小是能够定下来的。也就是跟一个数组一样，我能够在O(1)时间定位到
0000.index, 0000.log一个是索引，一个是数据
（kafka里面，数据就是用log这个概念来称呼的）
然后这里面，因为7天之后会删掉。
所以注意一个概念，就是这个时间，是publish的时间，不管你有多少个consumer在消费，我按publish时间保存的这些数据，7天之后就没了
所以consumer如果7天过后再连上来消费，就会发现自己消费不到了。
数据是1个G一个文件。超过一个G就会生成一个新文件来存放。


分区策略
partition => 方便水平扩展。可以提高并发。
ProducerRecord(String topic, Integer partition,...)
ProducerRecord(String topic, String key, String value)
ProducerRecord(String topic, String value)
一，自己指定分区
二，有key的，就用key的hash值去模分区数
三，没有key，就roundroubin，一个一个去放。但是谁是第一个？random


数据可靠性
很简单，就是在我server端收到消息后，给client发ack
client收到ack，就知道数据安全到达，没收到，就认为数据丢了，就重发。
那，kafka有leader和follower，就存在一个问题，我的follower怎么办，也要等他们全部同步完再ack吗。
因为kafka要容错，所以肯定不能是leader收到，不管follower就直接ack
因为如果这样，那leader ack后挂掉，数据就没了。
所以一定也要等follower ack。
有两种方案，一种是，全数sync后再ack，一种是，半数后就ack。
那这两种有什么差别？
比如说，如果我要容忍3个节点出错。
全数ack：只要有 3 + 1 个副本
半数ack：3 + 3 + 1。
也就是相当于，有7个球，3黑4白，你随便拿走三个，我剩下的四个都肯定有一个是白的。
这里这个白的就是有同步ack的节点，也就是数据没丢失的节点。
这个的指导思想就是说，你要容错，允许人挂掉，那么你一定要保证你有没挂掉的并且是数据完备的。
这个数据完备就是由ack来指导。
ack的是收到数据的。那如果leader ack后挂了，它肯定就要去找follower里面也ack的还没挂的节点。
如果我很悲观，觉得我很多节点都会挂，那么我一定要尽量保证，我ack覆盖多一点follower。

但是kafka选的是ISR这个机制，就是不是半数，也不是全数
ISR：insync replica set
就是和leader保持同步的follower集合。
在这个队列里的条件是满足replica.lag.time.max.ms的
当这个队列里的follower都同步完之后，leader才会给client发sync。
然后leader发生故障，就从ISR里选leader

ack参数配置
publisher层面
0：不等待ack。消息一到broker，就return。publsher收到return就结束这一条
1：只等leader的ack
注意这里。broker接收到，和ack间，还有一段时间。
第一种的话，如果broker在期间挂掉。其实消息还没persist或者说正常全部处理。publisher就会return。也就丢了数据
等leader ack，至少能保证，leader是数据完备的
-1：leader和follower（ISR里的）全部落盘成功才返回。


ack = -1 重复案例
follower已经同步数据成功，但总体ack还没发，这时候leader挂了
然后重新选举leader，那新的leader是不知道挂了的那个leader上一个ack还没发的
所以它也不会去修复。然后client端因为没收到ack就会再发一条，就重复了

丢数据问题
ISR里只有leader，leader ack后挂了，数据也还没同步到其他副本里面。
这种情况下就丢数据的。但是比较极端。因为ISR里面正常不会只有leader。也就是其他副本broker时延也太长了



副本数据的一致性
high watermark。
就是副本种的最小的log end offset
high watermark之前的数据才对消费者可见
然后选新leader的时候，发一个消息，让大家都截取到high watermark

At Least Once : ack = -1     全部副本都要同步
At Most Once  ： ack = 0     什么ack，我不管。leader和follower都不等



offset记录
group+topic+partition

 
kafka为什么consumer是zoopkeeper，priducer是brokerlist


kafka高效读取
1. 分区（集群）
2. 磁盘顺序读写（单台）
3. zero copy


kafka事务

kafka支持幂等性
ProducerID + topic + sequenceID来标识唯一的一条数据
所以如果发重了，broker端是知道的，就不会再写入
以上其实是针对于server端挂了的场景。
但是，如果你producer挂了，那producerid就是新的了。这个幂等性就没用了
就是只通过这个幂等性来保持exactly once就没用了
但是有一个疑问是，这个sequenceid，kafka server怎么样快速查到，你这个在我这里面有没有


* consumer
是while(true)主动去拉取的，需要设一个拉取的时间间隔
offset有一些概念
一个是latest，一个是earliest，一个是你上次记录在案的一个offset
可以设置offset.reset，然后定义是latest还是earliest
但是这个reset生效，只在你没有valid的offset的时候（一开始没有offset，或者你的offset是已经被删掉了的）

* 自动提交和手动提交
    * 自动提交的问题
        * 如果设置时间太短，1s提交一次，我收到的这个batch，我还没处理完呢，我挂了，但是你刚刚已经帮我提交了。我再起来的时候，server不会再给我发送之前那些数据了。
        * 时间设置得太长，10s提交一次，我收到好几个batch，都处理完了（写mysql比如），但是你还没给我提交。然后我挂了，我再起来的时候，server把我已经处理完的那些数据又发个我了。
        * 总的问题还是，这个提交，要跟我的处理同步。我处理完了，server就要知道我处理完了，我没处理完，server就该知道我没处理完
        * 所以手动提交就能够做到这点。
    * 手动提交
        * 但是手动也会遇到问题啊。在一个batch里面，假设我处理一半后挂了，是没提交。但是我起来后，还是会重复处理我刚刚已经处理过的1/3 batch
        * 所以差别应该是说，手动提交能把这个误差范围控制在一个batch的处理时间之内，而已
    * 其实这种问题，归根结底就是“原子性”的问题。
        * 你一个操作，要具有原子性。也就是不可分割，要么全做，要么全不做。
        * 要解决就是用事务，把这一组操作“原子性”圈起来，别无他法。
        * kafka里提供的是，自定义存储offset。但是具体的机制我现在不太了解。
        
   
* ISR
就是in sync replica set。
在kafka的同步机制中，不是全部follower都要同步
而是valid的follower要同步。
因为你想，如果我10个副本，有一个非常之慢，然后我每次发消息给leader完，要等你这个非常慢的也同步完，才ack，那不是很费性能吗
所以它维护了一个in sync replica set，就是只有符合我的要求的（比如速度比较快的，才是我的ack必须集合）
然后这个ISR是会动态维护更新的。慢的踢出去，快的加上来


* 拦截器，序列化器，分区器
* 为什么需要，又为什么要这个顺序
* 拦截器就是统一做一点事情，序列化器就是因为要落盘，分区器可以自定义分区
* 拦截器放前面，因为相当于剪枝
* 序列化在分区前，可能是因为，分区需要依靠你序列化后的信息。


* 生产者的整体结构，使用了几个线程
两个线程，main和sender线程。
就是生产者消费者模型，中间的那个容器是RecorderAccumulator
main线程经历三个：拦截器，序列化器，分区器，就放入RA
然后有一个另外的sender线程，从RA中拿数据，去send

* 消息漏消费和重复消费
在于你consumer那边commit的时刻跟处理的真实情况的时刻。
consumer如果接收到100条，要等全处理完才commit
然后处理完50条后挂了，再起来，还是要处理100条，就重复消费了
漏消费比如说按时间去commit，commit的太早了，我刚收到100条，还没处理，你commit了，然后我挂了
我再起来后，就丢失了这些数据


* kafka topic的分区数可不可以增减，为什么
可以增加，然后动态去调整消费者和生产者
不可减少。因为你减少后，数被你删掉的数据怎么处理

* kafka有内部topic吗
有，比如consumer_offset


* 分区分配
range和roundrobin
range就是正常的对每个topic来说
roundrobin是为了尽量让所有的consumer能分到平均的partition数
所以是把所有的topic和partition排序，然后分配给所有消费者
这样每个consumer的partition差别数最多只是1


* kafka日志目录结构
.log,.index
log存的应该是offset+数据
index存的应该是offset+数据在log file中的偏移量
感觉这样的话才make sense。但是这样的话，全部都是常数时间啊
哪有网上说的什么二分查找
==============澄清一下
.index。人家是index，不是一一map。
```
00001.index

offset: 28 position: 4362
offset: 80 position: 12990
offset: 160 position: 25809

00001.log
message: sf1  offset:28
message: sf2  offset:50
message: sf34  offset:80
message: sf24  offset:90
message: sf4  offset:160
```
只有有序并且一一map的，才是常数时间。
如果只是有序，就是用二分查找。


* kafka controller
就是broker集群里面临时选出的一个老大，跟zookeeper打交道的。
更新元数据什么的

* kafka的选举
    * controller和leader
    * controller
        * 抢zookeeper里的一个资源。就跟分布式锁差不多
    * leader
        * 从ISR中选一个出来


* kafka的高性能
    * 分布式
    * 顺序写磁盘
    * 零拷贝

* kafka消息积压怎么办
    * 加分区加消费者数（消费者不变，你多加分区也不变）
    * 或者是，提高每批次拉取的数量。max.poll.records。默认是500


