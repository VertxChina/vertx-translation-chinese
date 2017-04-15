# Vert.x Kafka Client

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 中英文对照表

- consumer：消费者
- consumer group：消费组（[Kafka中的设计](https://kafka.apache.org/documentation/#intro_consumers)）
- partition：分区（[Kafka中的设计](https://kafka.apache.org/documentation/#intro_topics)）
- producer: 生产者
- offset：偏移量

## 组件介绍

此组件提供了 Kafka Client 的集成，可以以 Vert.x 的方式从 [Apache Kafka](https://kafka.apache.org/) 集群上消费或者发送消息。

对于消费者(consumer)，API以异步的方式订阅消费指定的 topic 以及相关的分区(partition)，或者将消息以 Vert.x Stream 的方式读取（甚至可以支持暂停(pause)和恢复(resume)操作）。  

对于生产者(producer)，API提供发送信息到指定 topic 以及相关的分区(partition)的方法，类似于向 Vert.x Stream 中写入数据。  

> 警告：此组件处于技术预览阶段，因此在之后版本中API可能还会发生一些变更。

## 使用 Vert.x Kafka Client

要使用 Vert.x Kafka Client 组件，需要添加以下依赖：

- Maven（在 `pom.xml`文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-kafka-client</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle（在`build.gradle`文件中）：

```groovy
compile 'io.vertx:vertx-kafka-client:3.4.1'
```

## 创建 Kafka Client

创建 Consumer 和 Producer 以及使用它们的方法其实与原生的 Kafka Client 库非常相似，Vert.x 只是做了一层异步封装。

我们需要对 Consumer 与 Producer 进行一些相关的配置，具体可以参考 Apache Kafka 的官方文档：

- [Consumer Configs](https://kafka.apache.org/documentation/#newconsumerconfigs)
- [Producer Configs](https://kafka.apache.org/documentation/#producerconfigs)

我们可以通过一个 Map 来包装这些配置，然后将其传入到 [`KafkaConsumer`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html) 接口或 [`KafkaProducer`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html) 接口中的 `create` 静态方法里来创建 `KafkaConsumer` 或 `KafkaProducer`：

``` java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
config.put("value.deserializer", "org.apache.kafka.common.serialization.StringDeserializer");
config.put("group.id", "my_group");
config.put("auto.offset.reset", "earliest");
config.put("enable.auto.commit", "false");

// 创建一个Kafka Consumer
KafkaConsumer<String, String> consumer = KafkaConsumer.create(vertx, config);
```

在上面的例子中，我们在创建 [`KafkaConsumer`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html) 实例时传入了一个 Map 实例，用于指定要连接的 Kafka 节点列表（只有一个）以及如何对接收到的消息进行解析以得到 key 与 value。

我们可以用类似的方法来创建 Producer：

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
config.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
config.put("acks", "1");

// 创建一个Kafka Producer
KafkaProducer<String, String> producer = KafkaProducer.create(vertx, config);
```

另外也可以使用 [`Properties`](http://vertx.io/docs/apidocs/java/util/Properties.html) 来代替 Map：

```java
Properties config = new Properties();
config.put(ConsumerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
config.put(ConsumerConfig.KEY_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
config.put(ConsumerConfig.VALUE_DESERIALIZER_CLASS_CONFIG, StringDeserializer.class);
config.put(ConsumerConfig.GROUP_ID_CONFIG, "my_group");
config.put(ConsumerConfig.AUTO_OFFSET_RESET_CONFIG, "earliest");
config.put(ConsumerConfig.ENABLE_AUTO_COMMIT_CONFIG, "false");

KafkaConsumer<String, String> consumer = KafkaConsumer.create(vertx, config);
```  

消息的 key 和 value 的序列化格式也可以作为 [`create`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#create-io.vertx.core.Vertx-java.util.Properties-java.lang.Class-java.lang.Class-) 方法的参数直接传进去，而不是在相关配置中指定：

```java
Properties config = new Properties();
config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
config.put(ProducerConfig.ACKS_CONFIG, "1");

// 注意这里的第三和第四个参数
KafkaProducer<String, String> producer = KafkaProducer.create(vertx, config, String.class, String.class);
```

在这里，我们在创建 [`KafkaProducer`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html) 实例的时候传入了一个 [`Properties`](http://vertx.io/docs/apidocs/java/util/Properties.html) 实例，用于指定要连接的 Kafka 节点列表（只有一个）和消息确认模式。消息 key 和 value 的解析方式作为参数传入 [`KafkaProducer.create`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#create-io.vertx.core.Vertx-java.util.Properties-java.lang.Class-java.lang.Class-) 方法中。

## 消费感兴趣 Topic 的消息并加入消费组

我们可以通过 `KafkaConsumer` 的的 [subscribe](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#subscribe-java.util.Set-) 方法来订阅一个或多个 topic 进行消费，同时加入到某个消费组（consumer group）中（在创建消费者实例时通过配置指定）。当然你需要通过 [`handler`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#handler-io.vertx.core.Handler-) 方法注册一个 `Handler` 来处理接收的消息：

```java
consumer.handler(record -> {
  System.out.println("Processing key=" + record.key() + ",value=" + record.value() +
    ",partition=" + record.partition() + ",offset=" + record.offset());
});

// 订阅多个topic
Set<String> topics = new HashSet<>();
topics.add("topic1");
topics.add("topic2");
topics.add("topic3");
consumer.subscribe(topics);

// 订阅单个主题
consumer.subscribe("a-single-topic");
```

另外如果想知道消息是否成功被消费掉，可以在调用 `subscribe` 方法时绑定一个 `Handler`：

```java
consumer.handler(record -> {
  System.out.println("Processing key=" + record.key() + ",value=" + record.value() +
    ",partition=" + record.partition() + ",offset=" + record.offset());
});

// subscribe to several topics
Set<String> topics = new HashSet<>();
topics.add("topic1");
topics.add("topic2");
topics.add("topic3");

//这里lambda表达式用于接收消息处理结果
consumer.subscribe(topics, ar -> {
  if (ar.succeeded()) {
    System.out.println("subscribed");
  } else {
    System.out.println("Could not subscribe " + ar.cause().getMessage());
  }
});

//这里lambda表达式用于接收消息处理结果
consumer.subscribe("a-single-topic", ar -> {
  if (ar.succeeded()) {
    System.out.println("subscribed");
  } else {
    System.out.println("Could not subscribe " + ar.cause().getMessage());
  }
});
```

由于Kafka的消费者会组成一个消费组(consumer group)，同一个组只有一个消费者可以消费特定的 partition，同时此消费组也可以接纳其他的消费者，这样可以实现 partition 分配给组内其它消费者继续去消费。

如果组内的一个消费者挂了，kafka 集群会自动把 partition 重新分配给组内其他消费者，或者新加入一个消费者去消费对应的 partition。您可以通过 [`partitionsRevokedHandler`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsRevokedHandler-io.vertx.core.Handler-) 和 [`partitionsAssignedHandler`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsAssignedHandler-io.vertx.core.Handler-) 方法在 [`KafkaConsumer`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html) 里注册一个 `Handler` 用于监听对应的 partition 是否被删除或者分配。

```java
consumer.handler(record -> {
  System.out.println("Processing key=" + record.key() + ",value=" + record.value() +
    ",partition=" + record.partition() + ",offset=" + record.offset());
});

// 注册一个用于侦听新分配partition的Handler
consumer.partitionsAssignedHandler(topicPartitions -> {

  System.out.println("Partitions assigned");
  for (TopicPartition topicPartition : topicPartitions) {
    System.out.println(topicPartition.getTopic() + " " + topicPartition.getPartition());
  }
});

// 注册一个用于侦听撤销partition的Handler
consumer.partitionsRevokedHandler(topicPartitions -> {

  System.out.println("Partitions revoked");
  for (TopicPartition topicPartition : topicPartitions) {
    System.out.println(topicPartition.getTopic() + " " + topicPartition.getPartition());
  }
});

// subscribes to the topic
consumer.subscribe("test", ar -> {

  if (ar.succeeded()) {
    System.out.println("Consumer subscribed");
  }
});
```

加入某个 consumer group 的消费者，可以通过 [`unsubscribe`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#unsubscribe--) 方法退出该消费组，从而不再接受到相关消息：

```java
consumer.unsubscribe();
```

当然你也可以在 `unsubscribe` 方法中传入一个 `Handler` 用于监听执行结果状态：

```java
consumer.unsubscribe(ar -> {

  if (ar.succeeded()) {
    System.out.println("Consumer unsubscribed");
  }
});
```

## 从 Topic 的特定分区里接收消息

消费组内的消费者可以消费某个 topic 指定的 partition。如果某个消费者并不属于任何消费组，那么整个程序就不能依赖 Kafka 的 re-balancing 机制去消费消息。

您可以通过 [`assign`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#assign-java.util.Set-io.vertx.core.Handler-) 方法请求分配指定的分区：

```java
consumer.handler(record -> {
  System.out.println("key=" + record.key() + ",value=" + record.value() +
    ",partition=" + record.partition() + ",offset=" + record.offset());
});

//
Set<TopicPartition> topicPartitions = new HashSet<>();
topicPartitions.add(new TopicPartition()
  .setTopic("test")
  .setPartition(0));

// 要求分配到特定的topic以及partitions
consumer.assign(topicPartitions, done -> {

  if (done.succeeded()) {
    System.out.println("Partition assigned");

    // 侦听分配结果
    consumer.assignment(done1 -> {

      if (done1.succeeded()) {

        for (TopicPartition topicPartition : done1.result()) {
          System.out.println(topicPartition.getTopic() + " " + topicPartition.getPartition());
        }
      }
    });
  }
});
```

上面的 [`assignment`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#assignment-io.vertx.core.Handler-) 方法可以列出当前分配的 topic partition。

## 获取 Topic 以及分区信息

您可以通过  [`partitionsFor`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsFor-java.lang.String-io.vertx.core.Handler-) 方法获取指定 topic 的 partition 信息：

```java

consumer.partitionsFor("test", ar -> {
  if (ar.succeeded()) {
    for (PartitionInfo partitionInfo : ar.result()) {
      System.out.println(partitionInfo);
    }
  }
});
```

另外，[`listTopics`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#listTopics-io.vertx.core.Handler-) 方法可以列出消费者下的所有 topic 以及对应的 partition 信息：

```java
consumer.listTopics(ar -> {

  if (ar.succeeded()) {

    Map<String, List<PartitionInfo>> map = ar.result();
    map.forEach((topic, partitions) -> {
      System.out.println("topic = " + topic);
      System.out.println("partitions = " + map.get(topic));
    });
  }
});
```

## 手动提交偏移量

在 Apache Kafka 中，消费者负责处理最新读取消息的偏移量（offset）。Consumer 会在每次从某个 topic partition 中读取一批消息的时候自动执行提交偏移量的操作。需要在创建 `KafkaConsumer` 时将 `enable.auto.commit` 配置项设为 `true` 来开启自动提交。

我们可以通过 [`commit`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#commit-io.vertx.core.Handler-) 方法进行手动提交。手动提交偏移量通常用于确保消息分发的 *at least once* 语义，以确保消息没有被消费前不会执行提交。

```java
consumer.commit(ar -> {

  if (ar.succeeded()) {
    System.out.println("Last read message offset committed");
  }
});
```

## 分区偏移量定位

Apache Kafka 中的消息是按顺序持久化在磁盘上的，所以消费者可以在某个 partition 内部进行偏移量定位(seek)操作，并从任意指定的 topic 以及 partition 位置开始消费消息。我们可以通过 [`seek`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seek-io.vertx.kafka.client.common.TopicPartition-long-) 方法来更改读取位置对应的偏移量：

```java
TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

// 指定offset位置10
consumer.seek(topicPartition, 10, done -> {

  if (done.succeeded()) {
    System.out.println("Seeking done");
  }
});
```

当消费者需要从 Stream 的起始位置读取消息时，可以使用 [seekToBeginning](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToBeginning-io.vertx.kafka.client.common.TopicPartition-) 方法将 `offset` 位置设置到 partition 的起始端：

```java
TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

// 将offset挪到分区起始端
consumer.seekToBeginning(Collections.singleton(topicPartition), done -> {

  if (done.succeeded()) {
    System.out.println("Seeking done");
  }
});
```

最后我们也可以通过 [`seekToEnd`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToEnd-io.vertx.kafka.client.common.TopicPartition-) 方法将 `offset` 位置设置到 partition 的末端：

```java

TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

// 将offset挪到分区末端
consumer.seekToEnd(Collections.singleton(topicPartition), done -> {

  if (done.succeeded()) {
    System.out.println("Seeking done");
  }
});
```

## 偏移量查询

你可以利用 Kafka 0.10.1.1 引入的新的API `beginningOffsets` 来获取给定分区的起始偏移量。这个跟上面的 [`seekToBeginning`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToBeginning-io.vertx.kafka.client.common.TopicPartition-) 方法有一个地方不同：[`beginningOffsets`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#beginningOffsets-java.util.Set-io.vertx.core.Handler-) 方法不会更改 offset 的值，仅仅是读取（只读模式）。

``` java
Set<TopicPartition> topicPartitions = new HashSet<>();
TopicPartition topicPartition = new TopicPartition().setTopic("test").setPartition(0);
topicPartitions.add(topicPartition);

consumer.beginningOffsets(topicPartitions, done -> {
  if(done.succeeded()) {
    Map<TopicPartition, Long> results = done.result();
    results.forEach((topic, beginningOffset) ->
      System.out.println("Beginning offset for topic="+topic.getTopic()+", partition="+
        topic.getPartition()+", beginningOffset="+beginningOffset));
  }
});

// partition offset 查询辅助方法
consumer.beginningOffsets(topicPartition, done -> {
  if(done.succeeded()) {
    Long beginningOffset = done.result();
      System.out.println("Beginning offset for topic="+topicPartition.getTopic()+", partition="+
        topicPartition.getPartition()+", beginningOffset="+beginningOffset);
  }
});
```

与此对应的API还有 [`endOffsets`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#endOffsets-java.util.Set-io.vertx.core.Handler-) 方法，用于获取给定分区末端的偏移量值。与 [`seekToEnd`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToEnd-io.vertx.kafka.client.common.TopicPartition-) 方法相比，[`endOffsets`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#endOffsets-java.util.Set-io.vertx.core.Handler-) 方法不会更改 offset 的值，仅仅是读取（只读模式）。

``` java
Set<TopicPartition> topicPartitions = new HashSet<>();
TopicPartition topicPartition = new TopicPartition().setTopic("test").setPartition(0);
topicPartitions.add(topicPartition);

consumer.endOffsets(topicPartitions, done -> {
  if(done.succeeded()) {
    Map<TopicPartition, Long> results = done.result();
    results.forEach((topic, endOffset) ->
      System.out.println("End offset for topic="+topic.getTopic()+", partition="+
        topic.getPartition()+", endOffset="+endOffset));
  }
});

consumer.endOffsets(topicPartition, done -> {
  if(done.succeeded()) {
    Long endOffset = done.result();
      System.out.println("End offset for topic="+topicPartition.getTopic()+", partition="+
        topicPartition.getPartition()+", endOffset="+endOffset);
  }
});
```

Kafka 0.10.1.1 还提供了一个根据时间戳(timestamp)来定位 offset 的API方法 `offsetsForTimes`，调用此API可以返回大于等于给定时间戳的 offset。因为 Kafka 的 offset 低位就是时间戳，所以 Kafka 很容易定位此类offset。

```java
Map<TopicPartition, Long> topicPartitionsWithTimestamps = new HashMap<>();
TopicPartition topicPartition = new TopicPartition().setTopic("test").setPartition(0);

// 我们只想要60秒之前的消息的offset
long timestamp = (System.currentTimeMillis() - 60000);

topicPartitionsWithTimestamps.put(topicPartition, timestamp);
consumer.offsetsForTimes(topicPartitionsWithTimestamps, done -> {
  if(done.succeeded()) {
    Map<TopicPartition, OffsetAndTimestamp> results = done.result();
    results.forEach((topic, offset) ->
      System.out.println("Offset for topic="+topic.getTopic()+
        ", partition="+topic.getPartition()+"\n"+
        ", timestamp="+timestamp+", offset="+offset.getOffset()+
        ", offsetTimestamp="+offset.getTimestamp()));

  }
});

consumer.offsetsForTimes(topicPartition, timestamp, done -> {
  if(done.succeeded()) {
    OffsetAndTimestamp offsetAndTimestamp = done.result();
      System.out.println("Offset for topic="+topicPartition.getTopic()+
        ", partition="+topicPartition.getPartition()+"\n"+
        ", timestamp="+timestamp+", offset="+offsetAndTimestamp.getOffset()+
        ", offsetTimestamp="+offsetAndTimestamp.getTimestamp());

  }
});
```

## 流量控制

Consumer 可以对消息流进行流量控制。如果我们读到一批消息，需要花点时间进行处理则可以暂时暂停（`pause`）消息的流入（这里实际上是把消息全部缓存到内存里了）；等我们处理了差不多了，可以再继续消费缓存起来的消息（`resume`）。

我们可以利用 [`pause`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#pause--) 方法和 [`resume`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#resume--) 方法来进行流量控制：

```java
TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

//注册一个handler处理进来的消息
consumer.handler(record -> {
  System.out.println("key=" + record.key() + ",value=" + record.value() +
    ",partition=" + record.partition() + ",offset=" + record.offset());

  // 如果我们读到partition0的第5个offset
  if ((record.partition() == 0) && (record.offset() == 5)) {

    // 则暂停读取
    consumer.pause(topicPartition, ar -> {

      if (ar.succeeded()) {

        System.out.println("Paused");

        // 5秒后再恢复,继续读取
        vertx.setTimer(5000, timeId -> {

          // resumi read operations
          consumer.resume(topicPartition);
        });
      }
    });
  }
});
```

## 关闭 Consumer

关闭 Consumer 只需要调用 `close` 方法就可以了，它会自动的关闭与 Kafka 的连接，同时释放相关资源。

由于 `close` 方法是异步的，你并不知道关闭操作什么时候完成或失败，这时你需要注册一个处理器(`Handler`)来监听关闭完成的消息。当关闭操作彻底完成以后，注册的 `Handler` 将会被调用。

```java
consumer.close(res -> {
  if (res.succeeded()) {
    System.out.println("Consumer is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

## 发送消息到某个 Topic

您可以利用 [`write`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#write-io.vertx.kafka.client.producer.KafkaProducerRecord-) 方法来向某个 topic 发送消息(records)。

最简单的发送消息的方式是仅仅指定目的 topic 以及相应的值而省略消息的 key 以及分区。在这种情况下，消息会以轮询(round robin)的方式发送到对应 topic 的所有分区上。

```java
for (int i = 0; i < 5; i++) {

  // 这里指定了topic和 message value,以round robin方式发送的目标partition
  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", "message_" + i);

  producer.write(record);
}
```

您可以通过绑定 `Handler` 来接受发送的结果。这个结果其实就是一些元数据(metadata)，包含消息的 topic、目的分区 (destination partition) 以及分配的偏移量 (assigned offset)。

```java
for (int i = 0; i < 5; i++) {

  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", "message_" + i);

  producer.write(record, done -> {

    if (done.succeeded()) {

      RecordMetadata recordMetadata = done.result();
      System.out.println("Message " + record.value() + " written on topic=" + recordMetadata.getTopic() +
        ", partition=" + recordMetadata.getPartition() +
        ", offset=" + recordMetadata.getOffset());
    }

  });
}
```

如果希望将消息发送到指定的分区，你可以指定分区的标识(identifier)或者设定消息的 key：

```java
for (int i = 0; i < 10; i++) {

  // 这里指定了 partition 为 0
  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", null, "message_" + i, 0);

  producer.write(record);
}
```

因为 Producer 可以使用消息的 key 作为 hash 值来确定 partition，所以我们可以保证所有的消息被发送到同样的 partition 中，并且是有序的。

```java

for (int i = 0; i < 10; i++) {

  // i.e. defining different keys for odd and even messages
  int key = i % 2;

  //这里指明了key,所有的消息将被发送同一个partition.
  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", String.valueOf(key), "message_" + i);

  producer.write(record);
}
```

> 注意：可共享的 Producer 通过 `createShared` 方法创建。它可以在多个 Verticle 实例之间共享，所以相关的配置必须在创建 Producer 的时候定义。

## 共享 Producer

有时候您希望在多个 Verticle 或者 Vert.x Context 下共用一个 Producer。您可以通过 [`KafkaProducer.createShared`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#createShared-io.vertx.core.Vertx-java.lang.String-java.util.Map-) 方法来创建可以在 Verticle 之间安全共享的 `KafkaProducer` 实例：


```java
KafkaProducer<String, String> producer1 = KafkaProducer.createShared(vertx, "the-producer", config);

// 关闭
producer1.close();
```

返回的 `KafkaProducer` 实例将复用相关的资源（如线程、连接等）。使用完 `KafkaProducer` 后，直接调用 `close` 方法关闭即可，相关的资源会自动释放。  

## 关闭 Producer

与关闭 Consumer 类似，关闭 Producer 只需要调用 `close` 方法就可以了，它会自动的关闭与 Kafka 的连接，同时释放所有相关资源。

由于 `close` 方法是异步的，你并不知道关闭操作什么时候完成或失败，这时你需要注册一个处理器(`Handler`)来监听关闭完成的消息。当关闭操作彻底完成以后，注册的 `Handler` 将会被调用。

```java
producer.close(res -> {
  if (res.succeeded()) {
    System.out.println("Producer is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

## 获取 Topic Partition 的相关信息

您可以通过 [`partitionsFor`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#partitionsFor-java.lang.String-io.vertx.core.Handler-) 方法获取指定 topic 的分区信息。

```java
producer.partitionsFor("test", ar -> {

  if (ar.succeeded()) {

    for (PartitionInfo partitionInfo : ar.result()) {
      System.out.println(partitionInfo);
    }
  }
});
```

## 错误处理

您可以利用 [`KafkaProducer#exceptionHandler`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#exceptionHandler-io.vertx.core.Handler-) 方法和 [`KafkaConsumer#exceptionHandler`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#exceptionHandler-io.vertx.core.Handler-) 方法来处理 Kafka 客户端（生产者和消费者）和 Kafka 集群之间的错误（如超时）。比如：

```java
consumer.exceptionHandler(e -> {
  System.out.println("Error = " + e.getMessage());
});
```

## 随 Verticle 自动关闭

如果您是在 Verticle 内部创建的 Consumer 和 Producer，那么当对应 Verticle 被卸载(undeploy)的时候，相关的 Consumer 和 Producer 会自动关闭。

## 使用 Vert.x 自带的序列化与反序列化机制

Vert.x Kafka Client 自带现成的序列化与反序列化机制，可以处理 `Buffer`、`JsonObject` 和 `JsonArray` 等类型。

在 `KafkaConsumer` 里您可以使用 `Buffer`：

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.deserializer", "io.vertx.kafka.client.serialization.BufferDeserializer");
config.put("value.deserializer", "io.vertx.kafka.client.serialization.BufferDeserializer");
config.put("group.id", "my_group");
config.put("auto.offset.reset", "earliest");
config.put("enable.auto.commit", "false");

// 创建一个可以反序列化成jsonObject的consumer.
config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.deserializer", "io.vertx.kafka.client.serialization.JsonObjectDeserializer");
config.put("value.deserializer", "io.vertx.kafka.client.serialization.JsonObjectDeserializer");
config.put("group.id", "my_group");
config.put("auto.offset.reset", "earliest");
config.put("enable.auto.commit", "false");

// 创建一个可以反序列化成jsonArray的consumer.
config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.deserializer", "io.vertx.kafka.client.serialization.JsonArrayDeserializer");
config.put("value.deserializer", "io.vertx.kafka.client.serialization.JsonArrayDeserializer");
config.put("group.id", "my_group");
config.put("auto.offset.reset", "earliest");
config.put("enable.auto.commit", "false");
```

同样在 `KafkaProducer` 中也可以：

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.serializer", "io.vertx.kafka.client.serialization.BufferSerializer");
config.put("value.serializer", "io.vertx.kafka.client.serialization.BufferSerializer");
config.put("acks", "1");

// 创建一个可以序列化成jsonObject的Producer.
config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.serializer", "io.vertx.kafka.client.serialization.JsonObjectSerializer");
config.put("value.serializer", "io.vertx.kafka.client.serialization.JsonObjectSerializer");
config.put("acks", "1");

// 创建一个可以序列化成jsonArray的Producer.
config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.serializer", "io.vertx.kafka.client.serialization.JsonArraySerializer");
config.put("value.serializer", "io.vertx.kafka.client.serialization.JsonArraySerializer");
config.put("acks", "1");
```

您也可以在 `create` 方法里指明序列化与反序列化相关的类。  

比如创建 Consumer 时：

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("group.id", "my_group");
config.put("auto.offset.reset", "earliest");
config.put("enable.auto.commit", "false");

// 创建一个可以反序列化成Buffer的Consumer
KafkaConsumer<Buffer, Buffer> bufferConsumer = KafkaConsumer.create(vertx, config, Buffer.class, Buffer.class);

// 创建一个可以反序列化成JsonObject的Consumer
KafkaConsumer<JsonObject, JsonObject> jsonObjectConsumer = KafkaConsumer.create(vertx, config, JsonObject.class, JsonObject.class);

// 创建一个可以反序列化成JsonArray的Consumer
KafkaConsumer<JsonArray, JsonArray> jsonArrayConsumer = KafkaConsumer.create(vertx, config, JsonArray.class, JsonArray.class);
```

创建 Producer 时：

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("acks", "1");

// 创建一个可以序列化成Buffer的Producer.
KafkaProducer<Buffer, Buffer> bufferProducer = KafkaProducer.create(vertx, config, Buffer.class, Buffer.class);

// 创建一个可以序列化成jsonObject的Producer.
KafkaProducer<JsonObject, JsonObject> jsonObjectProducer = KafkaProducer.create(vertx, config, JsonObject.class, JsonObject.class);

// 创建一个可以序列化成jsonArray的Producer.
KafkaProducer<JsonArray, JsonArray> jsonArrayProducer = KafkaProducer.create(vertx, config, JsonArray.class, JsonArray.class);
```

##  RxJava API

Vert.x Kafka Client 组件也提供Rx风格的API。

> 译者注：此处也可以参考 [Kafka Stream](https://kafka.apache.org/documentation/streams) 相关的 API。

```java
Observable<KafkaConsumerRecord<String, Long>> observable = consumer.toObservable();

observable
  .map(record -> record.value())
  .buffer(256)
  .map(
  list -> list.stream().mapToDouble(n -> n).average()
).subscribe(val -> {

  //获取到一个平均值

});
```

## 流实现与 Kafka 原生对象

如果您希望直接操作原生的 Kafka record，您可以使用原生的 Kafka 流式对象，它可以处理原生 Kafka 对象。

[`KafkaReadStream`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/consumer/KafkaReadStream.html)用于读取 topic partition。它是 [`ConsumerRecord`](http://vertx.io/docs/apidocs/org/apache/kafka/clients/consumer/ConsumerRecord.html) 对象的可读流对象，读到的是 [`ConsumerRecord`](http://vertx.io/docs/apidocs/org/apache/kafka/clients/consumer/ConsumerRecord.html) 对象。

[`KafkaWriteStream`](http://vertx.io/docs/apidocs/io/vertx/kafka/client/producer/KafkaWriteStream.html)用于向某些 topic 中写入数据。它是 [`ProducerRecord`](http://vertx.io/docs/apidocs/org/apache/kafka/clients/producer/ProducerRecord.html) 对象的可写流对象。

API通过这些接口将这些方法展现给用户，其他语言版本也应该类似。

---

> [原文档](http://vertx.io/docs/vertx-kafka-client/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-kafka-client/java/
[2]: https://github.com/vert-x3/vertx-kafka-client
[3]: https://github.com/vert-x3/vertx-examples/tree/master/kafka-examples
