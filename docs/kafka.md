#  kafka 一个消费者只能u

## kafka是 CAP的哪些？

CAP：一致性、可用性、分区容忍性

**可以是 AP 也可以是 CP**。

Kafka提供了一些配置，用户可以根据具体的业务需求，进行不同的配置，使得Kafka满足AP或者CP

如果要保证**强一致性 C **，使得Kafka满足CP。任意写入一条数据，都需要等到replicate到所有节点之后才返回ack；即使在有主节点宕机的情况下，也能保证数据一致性。

```javascript
replication.factor = 3 //主题副本数
min.insync.replicas = 3 //最少复制的节点
acks = all
```

如果要**保证可用性 A**，使得Kafka满足AP。设置ack=1，当主节点commmit了之后就返回ack；但是有可能会出现主节点宕机数据丢失问题。

```javascript
replication.factor = 3 //主题副本数
min.insync.replicas = 3 //最少复制的节点
acks = 1
```

还有公认比较推荐的一种配置，使 Kafka 满足的是一种**介于AP和CP之间的一种平衡状态**。基于这种配置，损失了一定的 一致性 和 可用性 ，因为，在这种配置下，可以在容忍一个节点（包括主节点）宕机的情况下，仍然保证数据强一致性和整体可用性；但是，有两个节点宕机的情况，就整体不可用了。

```javascript
replication.factor = 3 //主题副本数
min.insync.replicas = 2 //最少复制的节点
acks = all
```



## kafka基础概念

### Kafka 是什么？应用场景有哪些？

Kafka 是一个分布式流式处理平台。

Kafka 主要有两大应用场景：

1. **消息队列**：建立实时流数据管道，以可靠地在系统或应用程序之间获取数据。
2. **数据处理：** 构建实时的流数据处理程序来转换或处理数据流。



### 消息队列对比，kafka的优势？

kafka的优势比如：

**性能最好**：基于 Scala 和 Java 语言开发，设计中**大量使用了批量处理和异步的思想**，最高可以每秒处理千万级别的消息。

**与其他组件的兼容性很好**：Kafka 与周边生态系统的兼容性是最好的没有之一，**尤其在大数据和流计算领域**

早期的kafka是被用来处理日志的，后来随着功能的完善逐渐变成消息队列



### 队列模型了解吗？Kafka 的消息模型？

<img src="kafka.assets/image-20230807091243186.png" alt="image-20230807091243186" style="zoom:50%;" />

**队列模式**：就是在生产者和消费者中间加入了队列，类似生产者和消费者模型，一条消息只能被一个消费者使用，未被消费的消息在队列中保留直到被消费或超时。例：生产者发100条消息，2个消费者平分消息消费。

**kafka的消息模型**：发布订阅模式

发布订阅模式就类似于广播模式，生产者发布一条消息，这条消息就通过主题传递给所有的订阅者，在一条消息广播之后订阅的用户是收不到这条消息的。

> RocketMQ 的消息模型和 Kafka 基本是完全一样的。唯一的区别是 Kafka 中没有队列这个概念，与之对应的是 Partition（分区）。



### kafka关键概念：producer consumer broker topic partition

producer 生产者

consumer 消费者

broker 代理，可以理解为一个kafka节点，一个broker可以包含多个topic

topic 主题，生产者将消息发送到这里，consumer再通过订阅这个主题进行消费，一个topic可以有多个partition

partition 分区 ，一个topic中的partition可以分布在不同的broker上。

group：可以理解为消费者组，将消费者分组可以和topic中的partition进行一一对应

ISR/OSR: ISR就是一个列表，表中存储的是当前能够和leader保持同步的follower的节点，OSR就是没和leader保持同步的follower的节点

### kafka多副本机制，好处？

partition分区中的多个副本之间会有一个leader，其他的都是follower。我们都是把消息发给leader，然后其他的follower到leader上面来拉取数据进行同步。生产者和消费者只和leader交互，follower主要是用来保证存储的安全性，同时当leader挂了之后可以找到合适的follower继任leader。

Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力（负载均衡）。

Partition 可以指定对应的 Replica 数, 这也极大地提高了消息存储的安全性, 提高了容灾能力，不过也相应的增加了所需要的存储空间



### Zookeeper 在 Kafka 中的作用知道吗？

zookeeper主要是为kafka做了下面这些事情：

1.broker注册：在 Zookeeper 上会有一个专门用来进行 Broker 服务器列表记录的节点。每加入一个broker，就会记录一个对应的ip地址和端口

2.topic注册：同一个topic会有很多partition分区，这些分区又会被分布到不同的borker上，分区和broker的对应关系也是通过zookeeper来记录的

3.负载均衡：刚才提到了 Kafka 通过给特定 Topic 指定多个 Partition, 然后各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力。 对于同一个 Topic 的不同 Partition，Kafka 会尽力把这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里面。当 Consumer 消费的时候，Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态负载均衡。



### kafka高性能的原因

1.磁盘顺序读写：保证了消息的堆积

- 顺序读写，磁盘会预读，会在读取的起始地址连续的读取多个页面，主要花费时间在传输上。
- 随机读写，因为数据没有在一起，需要多次寻道和旋转延迟。而这个时间可能是传输时间的许多倍

2.零拷贝：避免cpu将数据从一块存储拷贝到另外一块存储的技术

- 传统的数据复制：
  - 读取磁盘文件数据到内核缓冲区
  - 将内核缓冲区的数据copy到用户缓冲区
  - 将用户缓冲区的数据copy到内核中socket的发送缓冲区
  - 将socket发送缓冲区中的数据发送给网卡，进行传输
- 零拷贝：
  - 磁盘文件->内核空间读取缓冲区->网卡接口->消费者进程 整个过程不会将数据拷贝到用户缓冲区了。

Kafka 零拷贝（Zero-Copy）是一种优化技术，用于减少在数据传输过程中的数据复制操作，从而提高性能和降低内存消耗。

不应用零拷贝的话，从硬盘将数据发送给远程服务器需要以下几个阶段：

磁盘->内核缓冲区->用户缓冲区->内核缓冲区（网卡）->网卡

在通过DMA优化之后，复制过程就可以变成：

磁盘->内核缓冲区->网卡

中间省略了向用户缓冲区复制的过程，从而可以减少数据复制的开销以及线程上下文切换的开销，来提高性能。

3.分区分段+索引：

kafka的message消息实际上是分布式存储在一个一个小的segement中的，每次文件操作也是直接操作的segment，为了进一步的查询优化，kafka又默认为分段后的数据文件建立了索引文件，就是文件系统上的.index文件，这种分区分段+索引的设计，不仅提升了数据读取的效率，同时也提高了数据操作的并行度。

4.批量压缩：多条消息一起压缩，降低带宽

5.批量读写

6.直接操作page cache，而不是jvm，避免了GC耗时及对象创建耗时，且读写速度更高，进程重启、缓存也不会丢失。



### kafka选举机制？

kafka中很多算法都是采用先到先得的机制，kafka中有一个controller的概念，就是在所有的broker中，会选举出一个controller来管理整个集群中的broker topic partition的关联信息。这个controller也会负责leader的选举。当leader宕机之后，kafka会选择ISR列表中的第一个副本变成leader



### kafka 重新分配（Rebalance）

Rebalance 的触发条件有3个。

- 组成员个数发生变化。例如有新的 consumer 实例加入该消费组或者离开组。
- 订阅的 Topic 个数发生变化。
- 订阅 Topic 的分区数发生变化。

Rebalance 发生时，Group 下所有 consumer 实例都会协调在一起共同参与，kafka 能够保证尽量达到最公平的分配。但是 Rebalance 过程对 consumer group 会造成比较严重的影响。在 Rebalance 的过程中 consumer group 下的所有消费者实例都会停止工作，等待 Rebalance 过程完成。



Rebalance 过程分为两步：Join 和 同步。

1. Join 顾名思义就是加入组。这一步中，所有成员都向coordinator发送JoinGroup请求，请求加入消费组。一旦所有成员都发送了JoinGroup请求，coordinator会从中选择一个consumer担任leader的角色，并把组成员信息以及订阅信息发给leader。leader负责消费分配方案的制定。
2. 同步，这一步leader开始分配消费方案，即哪个consumer负责消费哪些topic的哪些partition。一旦完成分配，leader会将这个方案封装进SyncGroup请求中发给coordinator，非leader也会发SyncGroup请求，只是内容为空。coordinator接收到分配方案之后会把方案塞进SyncGroup的response中发给各个consumer。这样组内的所有成员就都知道自己应该消费哪些分区了。

**coordinator**：

Group Coordinator是一个服务，每个Broker在启动的时候都会启动一个该服务。Group Coordinator的作用是用来存储Group的相关Meta信息，并将对应Partition的Offset信息记录到Kafka内置Topic(__consumer_offsets)中。Kafka在0.9之前是基于Zookeeper来存储Partition的Offset信息(consumers/{group}/offsets/{topic}/{partition})，因为ZK并不适用于频繁的写操作，所以在0.9之后通过内置Topic的方式来记录对应Partition的Offset。

每个Group都会选择一个Coordinator来完成自己组内各Partition的Offset信息，选择的规则如下：

- 1，计算Group对应在__consumer_offsets上的Partition
- 2，根据对应的Partition寻找该Partition的leader所对应的Broker，该Broker上的Group Coordinator即就是该Group的Coordinator



**组成员“崩溃”**：

**组成员崩溃和组成员主动离开是两个不同的场景。**因为在崩溃时成员并不会主动地告知coordinator此事，coordinator有可能需要一个完整的session.timeout周期(心跳周期)才能检测到这种崩溃，这必然会造成consumer的滞后。可以说离开组是主动地发起rebalance；而崩溃则是被动地发起rebalance。





### kafka零拷贝原理?

Kafka 零拷贝（Zero-Copy）是一种优化技术，用于减少在数据传输过程中的数据复制操作，从而提高性能和降低内存消耗。

不应用零拷贝的话，从硬盘将数据发送给远程服务器需要以下几个阶段：

磁盘->内核缓冲区->用户缓冲区->内核缓冲区（网卡）->网卡

在通过DMA优化之后，复制过程就可以变成：

磁盘->内核缓冲区->网卡

中间省略了向用户缓冲区复制的过程，从而可以减少数据复制的开销以及线程上下文切换的开销，来提高性能。



### 各大消息队列大对比

吞吐量： kafka（百万） > rocketmq（十万） > rabbitmq（万级）= activemq（万级）

rabbitmq：erlang语言开发，对于java二次开发不是很友好，基于AMQP协议

rocketmq：java实现，方便二次开发。

kafka：高可用、高性能



### kafka poll模式和push模式的区别？

**kafka是拉模式**。

**poll模式**

- 在 Poll 模式下，消费者主动从 Kafka 服务器拉取消息。
- 消费者通过轮询（Polling）机制向 Kafka 服务器请求消息，然后将消息拉取到本地进行处理。
- 这种方式消费者可以根据自己的处理能力来控制消息的获取速率。
- 消费者需要持续发送请求，轮询机制可能会导致一些额外的网络开销。

**push模式**

- 在 Push 模式下，Kafka 服务器将消息直接推送给消费者。
- 消费者在订阅主题时，注册一个回调函数，当有消息到达时，Kafka 会主动调用回调函数将消息推送给消费者。
- 这种方式不需要消费者轮询，可以减少网络开销和资源占用。
- 消费者需要处理消息的分发和并发控制，因此在编码上可能相对复杂一些。

选择 Pull 还是 Push 取决于具体的应用场景和需求：

- Pull 模式适用于消费者需要按照自身处理能力动态拉取消息的情况，可以实现更精细的控制。
- Push 模式适用于消息即时性要求较高的场景，减少了消费者的轮询开销。

## kafka高可用/场景题

### Kafka 如何保证消息的消费顺序？

> kafka的顺序消费基于偏移量和分区实现

kafka只能保证partition中的消息有序，是通过偏移量来实现的。同时kafka还可以通过指定partition或者key的方式把消息发送给指定分区

对于需要保证消费顺序的场景：

1.可以考虑让这个topic中只有一个分区，这样就能够保证顺序消费，不推荐这样设置，本来topic可以对应多个消费者去并发的消费多个分区，如果这样设置的话，就相当于降低了kafka的并发性能。

2.还可以考虑单独为这个业务场景创建一个分区，然后让消费者去消费指定的分区，同时，一个partition只对应一个消费者，这样就能够有效保证消息的顺序消费。

> 实现方式

指定消息生产者将消息发送给某个指定分区：

```java
Properties props = new Properties();
//props配置略....
Producer<String, String> producer = new KafkaProducer<>(props);
ProducerRecord<String, String> record = new ProducerRecord<>(topic, partition, key, value);
producer.send(record);
```

指定消费者去消费指定topic中指定分区的消息：

方式1：通过kafkaListener的方式：

```java
    @KafkaListener(
            groupId = "mallGroup",
            topicPartitions = {
                    @TopicPartition(topic = "maxwell", partitions = { "2" }) // 指定要消费的分区
            }
    )
    public void topicListener1(ConsumerRecord<String, String> record, Acknowledgment item) {
    }
```

方式2：创建消费者对象消费:

```java
// 消费者props配置略
Consumer<String, String> consumer = new KafkaConsumer<>(props);
// 消费者绑定topic和分区
TopicPartition partition = new TopicPartition("my-topic", 0); // 指定要消费的主题和分区
consumer.assign(Collections.singletonList(partition)); // 将分区分配给消费者
// 
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
		// 处理
    }
}
```



### Kafka中重要对象和注解包含哪些参数：

#### @kafkaListener:

- **groupId**：指定消费者组的名称。消费者组是 Kafka 用来协调消费者之间消息分配的关键概念。
- **topics**：要监听的 Kafka 主题名称，可以是字符串数组，指定一个或多个主题。
- **topicPattern**：要监听的主题的正则表达式。可以使用 `topicPattern` 代替 `topics` 来监听匹配正则表达式的主题。在这里可以指定监听指定分区的消息
- **concurrency**：指定消费者的并发度。可以设置为大于 1 的整数，以创建多个消费者实例并并行处理消息。
- **clientIdPrefix**：为消费者生成的客户端 ID 添加前缀，用于区分不同的消费者实例。
- **errorHandler**：指定用于处理消费错误的错误处理器类。

#### KafkaProducer：

- topic partition key value
- **headers**（Headers）：消息的头部信息，用于存储元数据或自定义属性。头部是一个键值对的集合，可以包含任意数量的元数据。
- **timestamp**（Long）：消息的时间戳，用于指示消息的时间。时间戳是一个可选属性，可以为 null。时间戳可以在消息排序、消息过期和其他时间相关的操作中使用。

#### KafkaConsumer:

- key value反序列化的参数
- bootstrap.servers：集群socket
- group.id： 消费者组id
- enable.auto.commit：是否自动提交
- auto.offset.reset：earliest从最早的开始消费 earliest：从最新的开始消费



### kafka消息丢失场景，如何保证消息不丢？

消息丢失要分两种情况，一种是生产者到分区时丢失，一种是分区到消费者时丢失，还有一种可能是kafka弄丢了消息

**生产者到分区时丢失**

对于第一种情况，我们想要判断消息是否发送成功，就要判断消息发送的结果，有两种方式判断：

1.通过Future的get方法获取结果，但是这样也让它变为了同步操作

```java
SendResult<String, Object> sendResult = kafkaTemplate.send(topic, o).get();
if (sendResult.getRecordMetadata() != null) {
  logger.info("生产者成功发送消息到" + sendResult.getProducerRecord().topic() + "-> " + sendRe
              sult.getProducerRecord().value().toString());
}

```

2.通过添加回调函数来做，通过future.addCallback()

```java
ListenableFuture<SendResult<String, Object>> future = kafkaTemplate.send(topic, o);
future.addCallback(result -> logger.info("生产者成功发送消息到topic:{} partition:{}的消息",result.getRecordMetadata().topic(), result.getRecordMetadata().partition()),ex -> logger.error("生产者发送消失败，原因：{}", ex.getMessage()));

```

可以设置 Producer 的`retries `（重试次数），一般是 3 ，这个重试次数是在创建kafka生产者对象的时候设置的，然后要是想保证消息不丢失的话一般会设置稍微大一点。设置完成之后，当出现网络问题之后能够自动重试消息发送，避免消息丢失。还得设置重试间隔，因为间隔太小的话重试的效果就不明显了，网络波动一次你 3 次一下子就重试完了。

**分区到消费者时丢失**

这种有可能是由于消费者自动提交offset，虽然提交了offset但是其实消息因为某些原因并没有被成功消费，就会导致消息丢失

可以通过消费完消息手动提交offset来避免这种情况，但是这样又可能会导致消息重复消费，就需要考虑消费的幂等性问题

**Kafka 弄丢了消息**

kafka是根据一定的消息量或者一个时间间隔进行刷盘的。

因为kafka的partition分区是分leader和follower的，leader和follower之间的数据并不一定是完全一致的，所以当leader宕机了，某个follower接替leader，此时没有同步过来的数据也就丢失了。

解决方案：

刷盘机制：

log.flush.interval.messages：设置多少条消息之后强制刷盘

log.flush.interval.ms：设置每隔多少ms强制刷盘

**几个重要参数：**

设置 acks = all：表示只有所有 ISR 列表的副本全部收到消息时，生产者才会接收到来自服务器的响应. 这种模式是最高级别的，也是最安全的，可以确保不止一个 Broker 接收到了消息. 该模式的延迟会很高. ack = 0 消息发送不需要响应，ack = 1 只等leader接收完就返回

设置replication.factor >= 3：设置的是leader的副本数量

设置min.insync.replicas > 1：一般情况下我们还需要设置 **min.insync.replicas> 1** ，这样配置代表消息至少要被写入到 2 个副本才算是被成功发送。**min.insync.replicas** 的默认值为 1 ，在实际生产中应尽量避免默认值 1。	

设置 unclean.leader.election.enable = false：设置成false之后，不允许选举ISR以外的副本为leader，这样可以提高数据的可靠性。



### kafka重复消费的产生场景，如何避免重复消费？

> 为什么会重复消费？

- 服务端侧已经消费的数据没有成功提交 offset（根本原因）因为默认情况下offset是在下一次去请求kafka消息的时候提交的，如果在下一次去请求kafka消息之前，也就是提交offset之前，消费者宕机了，会存在offset未提交的场景，下次消费者会从未提交的offset处开始消费，这也就导致了重复消费。
- Kafka侧由于服务端处理业务时间长(默认是超过5分钟)或者网络链接等等原因让 Kafka 认为服务假死，触发了分区 rebalance。

> 如何解决？

这个也可以从两个角度来考虑

1. 一个角度就是避免消息重发

**解决方案比如说：**

1.用异步的方式处理消息，缩短消息的消费时长

2.调整消息处理的超时时间

3.减少一次性从broker拉的消息数

4.将自动提交offset改为手动提交offset：

什么时候提交 offset 合适？

1.处理完消息再提交：依旧有消息重复消费的风险，和自动提交一样

2.拉取到消息即提交：会有消息丢失的风险。允许消息延时的场景，一般会采用这种方式。

2. 另一个角度就是通过消费业务的后台幂等性校验来避免重复消费：

例如使用redis和mysql为每一条消息生成一个MD5值，消费消息服务做幂等校验，比如 Redis 的 set、MySQL 的主键等天然的幂等功能。这种方法最有效。

将 enable.auto.commit 参数设置为 false，关闭自动提交，开发者在代码中手动提交 offset。那么这里会有个问题：



### kafka消息发送给消费者处理失败怎么办？

重试机制,设置一个重试次数，每条消息如果消费不成功，就重试指定次数，消费成功才提交偏移量。

```java
KafkaConsumer<String, String> consumer = new KafkaConsumer<>(properties);
        consumer.subscribe(Collections.singletonList(topic));

        int maxRetryAttempts = 3; // 最大重试次数

        while (true) {
            ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
            for (ConsumerRecord<String, String> record : records) {
                boolean success = false;
                int retryCount = 0;

                while (!success && retryCount < maxRetryAttempts) {
                    try {
                        // 处理消息的业务逻辑
                        processMessage(record.value());
                        success = true;
                    } catch (Exception e) {
                        // 处理失败，记录错误信息并增加重试次数
                        System.err.println("Error processing message: " + e.getMessage());
                        retryCount++;
                    }
                }

                if (!success) {
                    // 达到最大重试次数后，将消息标记为无法处理
                    System.err.println("Max retry attempts reached for message: " + record.value());
                } else {
                    // 成功处理消息后手动提交偏移量
                    consumer.commitSync();
                }
            }
        }    
```

> 如果重试了很多次之后，还是不行呢？

那就将失败的消息发送给某个消息队列的分区（也可以理解为kafka版本的死信队列）或者存入数据库当中，再写一个定时任务，隔一段时间再去获取并消费这些消息。





### kafka实现延迟队列的方式：

1.**使用时间戳和流处理**：你可以使用 Kafka Streams 或其他流处理框架来处理消息，并根据消息的时间戳实现延时处理。例如，你可以将消息发送到 Kafka 主题，并在流处理应用程序中等待特定时间后再处理这些消息。这需要你编写自定义逻辑来跟踪消息的时间戳和计算延时。（需要引入包kafka-streams）该方法没去验证

```java
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic", "key", "value");
record.timestamp(System.currentTimeMillis() + 60000); // 设置时间戳为当前时间 + 60秒
producer.send(record);
StreamsBuilder builder = new StreamsBuilder();
KStream<String, String> input = builder.stream("my-topic");

// 过滤出时间戳已到达的消息
KStream<String, String> delayedMessages = input.filter((key, value) -> {
    long timestamp = input.getTimestamp();
    long currentTime = System.currentTimeMillis();
    return (timestamp + 60000) <= currentTime; // 假设 60 秒后为延时消息
});

delayedMessages.foreach((key, value) -> {
    // 在这里处理延时消息
});

KafkaStreams streams = new KafkaStreams(builder.build(), config);
streams.start();
```

2.**定时重新发送**：你可以将需要延时消费的消息发送到 Kafka 主题，并设置它们的时间戳，然后编写一个消费者应用程序来定期轮询主题，检查是否有已经到期的消息，然后将它们消费。这实际上是一种轮询机制，而不是真正的实时延时消费。

```java
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic", "key", "value");
record.timestamp(System.currentTimeMillis()); // 设置时间戳为当前时间
producer.send(record);
Consumer<String, String> consumer = new KafkaConsumer<>(props);
consumer.subscribe(Collections.singletonList("my-topic"));
while (true) {
    ConsumerRecords<String, String> records = consumer.poll(Duration.ofMillis(100));
    for (ConsumerRecord<String, String> record : records) {
        long timestamp = record.timestamp();
        long currentTime = System.currentTimeMillis();
        if ((timestamp + 60000) <= currentTime) { // 假设 60 秒后为延时消息
            // 处理延时消息
        }
    }
}
```

3.**使用工具或库**：有一些 Kafka 的扩展库或工具可以帮助实现延时消费的效果，如 Confluent 的 "Kafka Delayed Message Plugin"。这些工具提供了一些功能，允许你将消息设置为延时消息，并在指定的延时时间后再将其发送到目标主题。这些工具可以简化实现延时消费的过程。

```java
ProducerRecord<String, String> record = new ProducerRecord<>("my-topic", "key", "value");
record.headers().add(new RecordHeader(DelayedMessageHeader.DELAY_MS_HEADER, "60000".getBytes()));
producer.send(record);
```





### 通过@KafkaListener注解中的concurrency 创建多个实例和直接写多个带有@KafkaListener注解的方法有什么区别呢？都是创建多个消费者的方式

两种方式都可以用于创建多个消费者实例，但它们的适用场景略有不同。使用 `concurrency` 属性创建多个实例适合在同一个方法上增加并发性，而直接写多个带有 `@KafkaListener` 注解的方法适合处理不同主题或分区的消息，或者实现不同的逻辑划分。选择哪种方式取决于你的具体需求和架构设计。



### @KafkaListener属于kafka的poll模式还是push模式？

push模式，通过监听器监听kafka的消息，而不需要主动去kafka获取消息。





# 





















# 

# 





# 

# 



# 

# 

# 

# 

# 

# 