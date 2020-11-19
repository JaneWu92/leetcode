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























