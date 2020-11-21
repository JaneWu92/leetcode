* source: 
    * https://www.confluent.io/blog/exactly-once-semantics-are-possible-heres-how-apache-kafka-does-it/

#### exactly once的难度
* at most once: 生产者不重发
* at least once： 生产者非常乐于重发。但是重发带来的是重复的问题。
* exactly once： 即使重发，message server端也要能够做到识别重复的消息。
* ===========
* 这里面到底是什么问题
    * 网络会crash
    * producer会crash
    * broker（message server）会crash
    * message server是一个集群，它的crash的种类有很多。涉及到你怎么看待这个集群

#### 消息发送过程
* 一条消息的生命过程经历哪些步骤
    * producer发送
    * 经过网络
    * 到达message server
    * message server leader写入磁盘
    * follower们去leader那儿拉取数据
    * 数据经由网络，到达follower
    * follower数据落盘
    * follower发送成功消息给leader
    * leader收到所有follower的成功消息
    * leader发送ack
    * 经过网络
    * ack到达producer
    * producer成功返回这一次发送的call
* 以上的每一步都有可能出错

#### kafka保证exactly once
* 两个机制
    * 单会话和单partition内的幂等性
    * 跨会话和跨partition的事务性
* 单会话和单partition内的幂等性
    * 每个新的producer，都会被server随机分配一个producerid
    * producer发送的每个批次，都会有一个递增的seqid。
    * 上述两个对producer应该都是透明的，是kafka客户端设计维护的。
    * 但有一个问题，加入producer没有收到ack，重发，它的这个是怎么知道seqid应该跟之前一样的
        * kafka client jar肯定有自己的机制。没有ack就没有成功，seqid不会增加的
    * 因为一个partition只会有一个producer去写，所以我判断seqid是很容易的，只要是比我上一个大，就是可接受的。因为它是有序的。
        * 如果消息序号比Broker维护的序号大一以上，说明中间有数据尚未写入，也即乱序，此时Broker拒绝该消息，Producer抛出InvalidSequenceNumber
        * 如果消息序号小于等于Broker维护的序号，说明该消息已被保存，即为重复消息，Broker直接丢弃该消息，Producer抛出DuplicateSequenceNumber
    * 存在的问题
        * 在写多个 Topic-Partition 时，执行的一批写入操作，有可能出现部分 Topic-Partition 写入成功，部分写入失败（比如达到重试次数），这相当于出现了中间的状态
        * Producer 应用中间挂之后再恢复，无法做到 Exactly-Once 语义保证
* 跨会话和跨partition的事务性
* ==============================>???????????????
http://www.jasongj.com/kafka/transaction/
    * 跨会话这个有点小窍门。
    * 在client端自己定义一个transactionid。
    * 这个东西相当于一个身份证，在server那边有记录
    * pid呢，一开始会随机分配一个。并且，server端那边会把他跟你的transactionid绑定。
    * 加入你producer挂了，但是你再起一个，还是用的你本来的transactionid
    * server端就知道你还是刚刚那个人，就会把你之前绑定的那个pid给你。
    * 这个就可以做到跨session
    * ==============
    * 



















