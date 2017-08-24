# RocketMQ原理解析

## 专业术语

* Producer

消息生产者，负责产生消息，一般由业务系统负责产生消息。

* Consumer

消息消费者，负责消费消息，一般是后台系统负责异步消费。

* Push Consumer

Consumer 的一种，应用通常向 Consumer 对象注册一个 Listener 接口，一旦收到消息，Consumer 对象立
刻回调 Listener 接口方法。

* Pull Consumer

Consumer 的一种，应用通常主动调用 Consumer 的拉消息方法从 Broker 拉消息，主动权由应用控制。

* Producer Group

一类 Producer 的集合名称，这类 Producer 通常发送一类消息，且发送逻辑一致。

* Consumer Group

一类 Consumer 的集合名称，这类 Consumer 通常消费一类消息，且消费逻辑一致。

* Broker

消息中转角色，负责存储消息，转发消息，一般也称为 Server。在 JMS 规范中称为 Provider。

* 广播消费

一条消息被多个 Consumer 消费，即使这些 Consumer 属于同一个 Consumer Group，消息也会被 Consumer
Group 中的每个 Consumer 都消费一次，广播消费中的 Consumer Group 概念可以认为在消息划分方面无意
义。
在 CORBA Notification 规范中，消费方式都属于广播消费。
在 JMS 规范中，相当于 JMS publish/subscribe model

* 集群消费

一个 Consumer Group 中的 Consumer 实例平均分摊消费消息。例如某个 Topic 有 9 条消息，其中一个
Consumer Group 有 3 个实例（可能是 3 个进程，或者 3 台机器），那么每个实例只消费其中的 3 条消息。
在 CORBA Notification 规范中，无此消费方式。
在 JMS 规范中，JMS point-to-point model 与之类似，但是 RocketMQ 的集群消费功能大等于 PTP 模型。
因为RocketMQ单个Consumer Group内的消费者类似于PTP，但是一个Topic/Queue可以被多个Consumer
Group 消费。

* 顺序消息

消费消息的顺序要同发送消息的顺序一致，在 RocketMQ 中，主要指的是局部顺序，即一类消息为满足顺
序性，必须 Producer 单线程顺序发送，且发送到同一个队列，这样 Consumer 就可以按照 Producer 发送
的顺序去消费消息。

* 普通顺序消息

顺序消息的一种，正常情况下可以保证完全的顺序消息，但是一旦发生通信异常，Broker 重启，由于队列
总数发生变化，哈希取模后定位的队列会变化，产生短暂的消息顺序不一致。
如果业务能容忍在集群异常情况（如某个 Broker 宕机或者重启）下，消息短暂的乱序，使用普通顺序方
式比较合适。

* 严格顺序消息

顺序消息的一种，无论正常异常情况都能保证顺序，但是牺牲了分布式 Failover 特性，即 Broker 集群中只
要有一台机器不可用，则整个集群都不可用，服务可用性大大降低。
如果服务器部署为同步双写模式，此缺陷可通过备机自动切换为主避免，不过仍然会存在几分钟的服务不
可用。（依赖同步双写，主备自动切换，自动切换功能目前还未实现）
目前已知的应用只有数据库 binlog 同步强依赖严格顺序消息，其他应用绝大部分都可以容忍短暂乱序，推
荐使用普通的顺序消息。
*Message Queue在 RocketMQ 中，所有消息队列都是持久化，长度无限的数据结构，所谓长度无限是指队列中的每个存储
单元都是定长，访问其中的存储单元使用 Offset 来访问，offset 为 java long 类型，64 位，理论上在 100
年内不会溢出，所以认为是长度无限，另外队列中只保存最近几天的数据，之前的数据会按照过期时间来
删除。
也可以认为 Message Queue 是一个长度无限的数组，offset 就是下标。



## 消息中间件需要解决哪些问题

### Publish/Subscribe(发布订阅)

发布订阅是消息中间件的最基本功能，也是相对于传统 RPC 通信而言。在此不再详述;

### Message Priority(消息优先级)

规范中描述的优先级是指在一个消息队列中，每条消息都有不同的优先级，一般用整数来描述，优先级高的消
息先投递，如果消息完全在一个内存队列中，那么在投递前可以按照优先级排序，令优先级高的先投递。
由于 RocketMQ 所有消息都是持久化的，所以如果按照优先级来排序，开销会非常大，因此 RocketMQ 没有特
意支持消息优先级，但是可以通过变通的方式实现类似功能，即单独配置一个优先级高的队列，和一个普通优先级
的队列, 将不同优先级发送到不同队列即可。
对于优先级问题，可以归纳为 2 类

* 只要达到优先级目的即可，不是严格意义上的优先级，通常将优先级划分为高、中、低，或者再多几个级别。每个优先级可以用不同的 topic 表示，发消息时，指定不同的 topic 来表示优先级，这种方式可以解决绝大部分的优先级问题，但是对业务的优先级精确性做了妥协。

* 严格的优先级，优先级用整数表示，例如 0 ~ 65535，这种优先级问题一般使用不同 topic 解决就非常不合适。如果要让 MQ 解决此问题，会对 MQ 的性能造成非常大的影响。这里要确保一点，业务上是否确实需要这种严格的优先级，如果将优先级压缩成几个，对业务的影响有多大？


### Message Order

消息有序指的是一类消息消费时，能按照发送的顺序来消费。例如：一个订单产生了 3条消息，分别是订单创建，订单付款，订单完成。消费时，要按照这个顺序消费才能有意义。但是同时订单之间是可以并消费的.RocketMQ 可以严格的保证消息有序。

### Message Filter

* Broker 端消息过滤在 Broker 中，按照 Consumer 的要求做过滤，优点是减少了对于 Consumer 无用消息的网络传输。
  缺点是增加了 Broker 的负担，实现相对复杂。

  * 淘宝 Notify 支持多种过滤方式，包含直接按照消息类型过滤，灵活的语法表达式过滤，几乎可以满足

  最苛刻的过滤需求。

  * 淘宝 RocketMQ 支持按照简单的 Message Tag 过滤，也支持按照 Message Header、body 进行过滤。
  * CORBA Notification 规范中也支持灵活的语法表达式过滤。

* Consumer 端消息过滤
  这种过滤方式可由应用完全自定义实现，但是缺点是很多无用的消息要传输到 Consumer 端。

### Message Persistence

消息中间件通常采用的几种持久化方式：

* 持久化到数据库，例如 Mysql。
* 持久化到 KV 存储，例如 levelDB、伯克利 DB 等 KV 存储系统。
* 文件记录形式持久化，例如 Kafka，RocketMQ


* 对内存数据做一个持久化镜像，例如 beanstalkd，VisiNotify

(1)、(2)、(3)三种持久化方式都具有将内存队列 Buffer 进行扩展的能力，(4)只是一个内存的镜像，作用是当 Broker挂掉重启后仍然能将之前内存的数据恢复出来。
JMS 与 CORBA Notification 规范没有明确说明如何持久化，但是持久化部分的性能直接决定了整个消息中间件
的性能。
RocketMQ 参考了 Kafka 的持久化方式，充分利用 Linux 文件系统内存 cache 来提高性能。

### Message Reliablity

影响消息可靠性的几种情况：

* Broker 正常关闭
* Broker 异常 Crash
* OS Crash
* 机器掉电，但是能立即恢复供电情况。
* 机器无法开机（可能是 cpu、主板、内存等关键设备损坏）
* 磁盘设备损坏。

(1)、(2)、(3)、(4)四种情况都属于硬件资源可立即恢复情况，RocketMQ 在这四种情况下能保证消息不丢，或
者丢失少量数据（依赖刷盘方式是同步还是异步）。
(5)、(6)属于单点故障，且无法恢复，一旦发生，在此单点上的消息全部丢失。RocketMQ 在这两种情况下，通
过异步复制，可保证 99%的消息不丢，但是仍然会有极少量的消息可能丢失。通过同步双写技术可以完全避免单点，
同步双写势必会影响性能，适合对消息可靠性要求极高的场合，例如与 Money 相关的应用。
RocketMQ 从 3.0 版本开始支持同步双写。

### Low Latency Messaging

在消息不堆积情况下，消息到达 Broker 后，能立刻到达 Consumer。
RocketMQ 使用长轮询 Pull 方式，可保证消息非常实时，消息实时性不低于 Push。

### At least Once

是指每个消息必须投递一次
RocketMQ Consumer 先 pull 消息到本地，消费完成后，才向服务器返回 ack，如果没有消费一定不会 ack 消息，所以 RocketMQ 可以很好的支持此特性

### Exactly Only Once

* 发送消息阶段，不允许发送重复的消息。
* 消费消息阶段，不允许消费重复的消息。

只有以上两个条件都满足情况下，才能认为消息是“Exactly Only Once”，而要实现以上两点，在分布式系统环
境下，不可避免要产生巨大的开销。所以 RocketMQ 为了追求高性能，并不保证此特性，要求在业务上进行去重，
也就是说消费消息要做到幂等性。RocketMQ 虽然不能严格保证不重复，但是正常情况下很少会出现重复发送、消
费情况，只有网络异常，Consumer 启停等异常情况下会出现消息重复。
此问题的本质原因是网络调用存在不确定性，即既不成功也不失败的第三种状态，所以才产生了消息重复性问
题。

### Broker 的 Buffer 满了怎么办？

Broker 的 Buffer 通常指的是 Broker 中一个队列的内存 Buffer 大小，这类 Buffer 通常大小有限，如果 Buffer 满
了以后怎么办？
下面是 CORBA Notification 规范中处理方式：

* RejectNewEvents

拒绝新来的消息，向 Producer 返回 RejectNewEvents 错误码。

* 按照特定策略丢弃已有消息
  * AnyOrder - Any event may be discarded on overflow. This is the default setting for this property.
  * FifoOrder - The first event received will be the first discarded.
  * LifoOrder - The last event received will be the first discarded.
  * PriorityOrder - Events should be discarded in priority order, such that lower priority,events will be discarded before higher priority events.
  * DeadlineOrder - Events should be discarded in the order of shortest expiry deadline first.


RocketMQ 没有内存 Buffer 概念，RocketMQ 的队列都是持久化磁盘，数据定期清除。对于此问题的解决思路，RocketMQ 同其他 MQ 有非常显著的区别，RocketMQ 的内存 Buffer 抽象成一个无限长度的队列，不管有多少数据进来都能装得下，这个无限是有前提的，Broker 会定期删除过期的数据，例如Broker 只保存 3 天的消息，那么这个 Buffer 虽然长度无限，但是 3 天前的数据会被从队尾删除。

### 回溯消费

回溯消费是指 Consumer 已经消费成功的消息，由于业务上需求需要重新消费，要支持此功能，Broker 在向
Consumer 投递成功消息后，消息仍然需要保留。并且重新消费一般是按照时间维度，例如由于 Consumer 系统故障，恢复后需要重新消费 1 小时前的数据，那么 Broker 要提供一种机制，可以按照时间维度来回退消费进度。
RocketMQ 支持按照时间回溯消费，时间维度精确到毫秒，可以向前回溯，也可以向后回溯。

### 消息堆积

消息中间件的主要功能是异步解耦，还有个重要功能是挡住前端的数据洪峰，保证后端系统的稳定性，这就要
求消息中间件具有一定的消息堆积能力，消息堆积分以下两种情况：

* 消息堆积在内存 Buffer，一旦超过内存 Buffer，可以根据一定的丢弃策略来丢弃消息，如 CORBA Notification

规范中描述。适合能容忍丢弃消息的业务，这种情况消息的堆积能力主要在于内存 Buffer 大小，而且消息
堆积后，性能下降不会太大，因为内存中数据多少对于对外提供的访问能力影响有限。

* 消息堆积到持久化存储系统中，例如 DB，KV 存储，文件记录形式。

当消息不能在内存 Cache 命中时，要不可避免的访问磁盘，会产生大量读 IO，读 IO 的吞吐量直接决定了消息堆积后的访问能力。
评估消息堆积能力主要有以下四点：

* 消息能堆积多少条，多少字节？即消息的堆积容量。
* 消息堆积后，发消息的吞吐量大小，是否会受堆积影响？
* 消息堆积后，正常消费的 Consumer 是否会受影响？
* 消息堆积后，访问堆积在磁盘的消息时，吞吐量有多大？

###分布式事务

已知的几个分布式事务规范，如 XA，JTA 等。其中 XA 规范被各大数据库厂商广泛支持，如 Oracle，Mysql 等。
其中 XA 的 TM 实现佼佼者如 Oracle Tuxedo，在金融、电信等领域被广泛应用。
分布式事务涉及到两阶段提交问题，在数据存储方面的方面必然需要 KV 存储的支持，因为第二阶段的提交回
滚需要修改消息状态，一定涉及到根据 Key 去查找 Message 的动作。RocketMQ 在第二阶段绕过了根据 Key 去查找
Message 的问题，采用第一阶段发送 Prepared 消息时，拿到了消息的 Offset，第二阶段通过 Offset 去访问消息，
并修改状态，Offset 就是数据的地址。
RocketMQ 这种实现事务方式，没有通过 KV 存储做，而是通过 Offset 方式，存在一个显著缺陷，即通过 Offset
更改数据，会令系统的脏页过多，需要特别关注。



### 定时消息

定时消息是指消息发到 Broker 后，不能立刻被 Consumer 消费，要到特定的时间点或者等待特定的时间后才能
被消费。
如果要支持任意的时间精度，在 Broker 层面，必须要做消息排序，如果再涉及到持久化，那么消息排序要不
可避免的产生巨大性能开销。
RocketMQ 支持定时消息，但是不支持任意时间精度，支持特定的 level，例如定时 5s，10s，1m 等



### 消息重试

Consumer 消费消息失败后，要提供一种重试机制，令消息再消费一次。Consumer 消费消息失败通常可以认为
有以下几种情况

1. 由于消息本身的原因，例如反序列化失败，消息数据本身无法处理（例如话费充值，当前消息的手机号被
   注销，无法充值）等。
   这种错误通常需要跳过这条消息，再消费其他消息，而这条失败的消息即使立刻重试消费，99%也不成功，
   所以最好提供一种定时重试机制，即过 10s 秒后再重试。
2. 由于依赖的下游应用服务不可用，例如 db 连接不可用，外系统网络不可达等。
   遇到这种错误，即使跳过当前失败的消息，消费其他消息同样也会报错。这种情况建议应用 sleep 30s，再
   消费下一条消息，这样可以减轻 Broker 重试消息的压力。