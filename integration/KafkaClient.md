# Vert.x Kafka Client

这个组件包装Kafka的Client以Vert.x的方式从[Apache Kafka](https://kafka.apache.org/)集群上消费或者发送消息.  

作为Consumer,API以异步的方式订阅消费指定的topic以及相关的partition，或者将消息转换成Vert.x stream的方式,从而支持pause和resume.  

作为Producer,API提供发送信息到指定topic以及分区的方法,且可以包装成stream.  

WARNING: 此组件处于技术预览阶段,API可能会发生一些变更.  

== 使用 Vert.x Kafka client
由于Kafka组件还有没有正式发布，所以要想尝鲜的话需要自己手动添加 _仓库_ 以及相关的 _依赖_ 到你的构建工具配置文件里.  

=== Maven 
``` html
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-kafka-client</artifactId>
  <version>3.4.1</version>
</dependency>
```  

=== Gradle  
```
compile io.vertx:vertx-kafka-client:3.4.1
```  

== 创建Kafka Client
创建consumers和producers以及使用他们的方法其实与原生的Kafka Client非常相似,Vert.x只是做了一层异步封装.  
kafka的consumer与producer需要设置一些配置文件,具体可以参考Kafka官方文档,[consumer](:https://kafka.apache.org/documentation/#newconsumerconfigs),[producer](https://kafka.apache.org/documentation/#producerconfigs),我们可以通过一个Map来包装这些配置，然后传入到相应的静态*create*方法里,[KafkaConsumer](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html[KafkaConsumer) [KafkaProducer](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html).  

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
上面的例子,[../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html](KafkaConsumer) 展示了如何通过map来包装一个kafka的配置并且实例化一个Consumer节点，这些参数指定了也指明了如何反序列化消息的key与value.  
同样的Producer也可以这样创建.  

```java
Map<String, String> config = new HashMap<>();
config.put("bootstrap.servers", "localhost:9092");
config.put("key.serializer", "org.apache.kafka.common.serialization.StringSerializer");
config.put("value.serializer", "org.apache.kafka.common.serialization.StringSerializer");
config.put("acks", "1");

// 创建一个Kafka Producer
KafkaProducer<String, String> producer = KafkaProducer.create(vertx, config);
```

另外也可以使用[Properties](../../apidocs/java/util/Properties.html)  
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

消息的key,value序列化格式也可以作为create方法的第三个,第四个参数直接传进去，而不是设置在Properties里.
```java
Properties config = new Properties();
config.put(ProducerConfig.BOOTSTRAP_SERVERS_CONFIG, "localhost:9092");
config.put(ProducerConfig.ACKS_CONFIG, "1");

// 注意这里的第三,四个参数
KafkaProducer<String, String> producer = KafkaProducer.create(vertx, config, String.class, String.class);
```
这里有一个[KafkaProducer](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html)例子使用了[Properties](../../apidocs/java/util/Properties.html)作为参数---构建了一个特定kafka节点,并且设置了消息确认模式以及key,value的反序列化方式[KafkaProducer.create](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#create-io.vertx.core.Vertx-java.util.Properties-java.lang.Class-java.lang.Class).  

== 通过Consumer消费Topic
通过Consumer的[subscribe](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#subscribe-java.util.Set)方法,我们可以订阅一个或多个topic进行消费,当然了你需要注册一个[Handler](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#handler-io.vertx.core.Handler)来处理消息,这个很Vert.x  

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
另外如果想知道消息是否成功被消费掉,可以再加一个接受 *AsyncResult* 参数的Handler.
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

由于Kafka的消费者会组成一个组,同一个组只有一个consumer可以消费特定的partition,同时此consumer组也可以连入其他的consumer,这样可以实现partitions分配给组内其他consumer继续去消费.  

如果组内的一个consumer挂了,kafka集群会自动把partitions重新分配给组内其他consumer,或者新加入一个consumer去消费该partitions.你可以在[KafkaConsumer](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html)里注册一个Handler用于侦听partitions是否被删除或者分配.[partitionsRevokedHandler](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsRevokedHandler-io.vertx.core.Handler),[partitionsAssignedHandler](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsAssignedHandler-io.vertx.core.Handler).
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

加入consumer组的consumer,可以随时退出该组,通过[unsubscribe](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#unsubscribe)方法避免消费分配给该组的消息.
```java
consumer.unsubscribe();

// 侦听执行结果
consumer.unsubscribe(ar -> {

  if (ar.succeeded()) {
    System.out.println("Consumer unsubscribed");
  }
});
```

== 从topic的特定partitions里接受消息
组内的consumer可以消费topic指定的partition,如果一个consumer并不是属于任何consumer组,则不能依赖kafka的re-balancing机制去消费消息.[assign](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#assign-java.util.Set-io.vertx.core.Handler)API可以指定特定的topic以及partition.
````java
----
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
上面的[assignment](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#assignment-io.vertx.core.Handler-[assignment)方法可以列出当前分配的Topic以及partitions.

== 获取Topic以及partition信息
调用[partitionsFor](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#partitionsFor-java.lang.String-io.vertx.core.Handler)方法,可以获取指定topic的partitions信息

```java

consumer.partitionsFor("test", ar -> {
  if (ar.succeeded()) {
    for (PartitionInfo partitionInfo : ar.result()) {
      System.out.println(partitionInfo);
    }
  }
});
```
[listTopics](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#listTopics-io.vertx.core.Handler)可以列出consumer下的所有topic,以及对应的partitions信息

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

==手动提交偏移量
Kafka的consumer可以手动提交最新读取消息的offset.  

如果希望consumer在读取topic一批消息后自动提交offset的话,需要在创建consumer时设置`enable.auto.commit`为 `true`.  
手动[提交](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#commit-io.vertx.core.Handler)offset通常用于消息_at least once (至少一次)_分发,确保消息没有被消费前不会执行提交.

```java
//手动执行commit
consumer.commit(ar -> {

  if (ar.succeeded()) {
    System.out.println("Last read message offset committed");
  }
});
```

== 从topic指定的位置开始消费
Apache Kafka的消息不同于传统的消息队列,消息是按顺序持久化在磁盘上的,所以可以从指定的topic以及partition位置开始消费过去的消息.可以通过[seek](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seek-io.vertx.kafka.client.common.TopicPartition-long)方法设置offset位置.

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
当consumer需要从stream的开始处读取消息时,可以使用[seekToBeginning](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToBeginning-io.vertx.kafka.client.common.TopicPartition).  

```java
TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

// 从partition的开始处重新读取
consumer.seekToBeginning(Collections.singleton(topicPartition), done -> {

  if (done.succeeded()) {
    System.out.println("Seeking done");
  }
});
```

最后我们也可以用[seekToEnd](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToEnd-io.vertx.kafka.client.common.TopicPartition)去设置从stream的末尾处开始读取消息.

```java

TopicPartition topicPartition = new TopicPartition()
  .setTopic("test")
  .setPartition(0);

// 从partition的末尾处开始去读
consumer.seekToEnd(Collections.singleton(topicPartition), done -> {

  if (done.succeeded()) {
    System.out.println("Seeking done");
  }
});
```

== Offset lookup
Kafka 0.10.1.1 有了新的API *beginningOffsets*,这个跟上面的[seekToBeginning](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToBeginning-io.vertx.kafka.client.common.TopicPartition)有一个地方不同,虽然他们都会从offset的开始处消费消息,但是*beginningOffsets*不会更改offset,你可以想象成只读模式.  

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

// 消费单个topic以及partition
consumer.beginningOffsets(topicPartition, done -> {
  if(done.succeeded()) {
    Long beginningOffset = done.result();
      System.out.println("Beginning offset for topic="+topicPartition.getTopic()+", partition="+
        topicPartition.getPartition()+", beginningOffset="+beginningOffset);
  }
});
```

与此对应的API还有*endOffsets*,从partition的末尾开始消费,但是不会修改offset,这个也跟[seekToEnd](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#seekToEnd-io.vertx.kafka.client.common.TopicPartition)不一样.  

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

Kafka 0.10.1.1还提供了一个根据Timestamp来定位offset的API *offsetsForTimes*, 调用此API可以返回 *>=* 给定timestamp的offset,因为kafka的offset的低位就是timestamp,所以kafka很容易定位此类offset.

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
== 流控
Consumer可以对消息流进行流控,如果我们读到一批消息,需要花点时间进行处理则可以暂时*pause*消息的流入(这里实际上是把消息全部缓存到内存里了),等我们处理了差不多了,可以再通过*resume*继续消费缓存起来的消息.具体可以参考[pause](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html),[resume](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html).

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

== 关闭Consumer
关闭Consumer只需要调用*close*方法就可以了,他会自动的关闭与kafka的连接同时释放相关资源.  
由于close行为是异步,你并不知道什么时候close行为已经完成或者失败了,这时你需要注册一个handler来侦听关闭完成的行为.

```java
consumer.close(res -> {
  if (res.succeeded()) {
    System.out.println("Consumer is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

== 发送消息到一个Topic
使用[KafkaProducer](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#write-io.vertx.kafka.client.producer.KafkaProducerRecord)发送一个消息(records)到一个指定的topic.  
这是发送消息最简单的方式,这里忽略了key和partition,kafka内部默认消息以*round robin*方式发送到topic的所有partitions上.  

```java

for (int i = 0; i < 5; i++) {

  // 这里指定了topic和 message value,以round robin方式发送的目标partition
  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", "message_" + i);

  producer.write(record);
}
```
你可以通过一个handler来接受发送的结果,这个结果其实就是一个*metadata*,包含了topic,partition和offset.

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
如果希望将消息发送到指定的partition,你可以指定其partition的标识或者设定消息的key.

```java
for (int i = 0; i < 10; i++) {

  // 这里指定了partition为0
  KafkaProducerRecord<String, String> record =
    KafkaProducerRecord.create("test", null, "message_" + i, 0);

  producer.write(record);
}
```

因为producer可以使用消息的key作为hash值来确定partition,所以我们可以保证所有的消息被发送到同样的partition并且是有序的.

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

注意: 可共享的Producer通过`createShared`创建,他可以在多个verticle实例之间公用,所以相关的配置也在创建的Producer的时候定义.
== 共享Producer
有时候你希望在多个verticle或者contexts下共用一个Producer.你可以通过[KafkaProducer.createShared](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#createShared-io.vertx.core.Vertx-java.lang.String-java.util.Map)来创建,这个方法返回的Producer可以安全的用于多个Verticle.


```java
KafkaProducer<String, String> producer1 = KafkaProducer.createShared(vertx, "the-producer", config);

// 关闭
producer1.close();
```

返回KafkaProducer将复用所有的thread,connection等相关资源.使用完后,直接调用`close`方法关闭即可,相关的资源会自动释放.  

== 关闭Producer
同样的如果你比较关心Producer关闭行为是否正在的完成了,关闭的时候是否发生了异常(因为关闭行为是异步的),此时你就需要注册一个侦听关闭行为的Handler.

```java
producer.close(res -> {
  if (res.succeeded()) {
    System.out.println("Producer is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

== 获取Topic partition相关信息
调用[partitionsFor](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#partitionsFor-java.lang.String-io.vertx.core.Handler)获取指定topic的partitions信息.

```java
producer.partitionsFor("test", ar -> {

  if (ar.succeeded()) {

    for (PartitionInfo partitionInfo : ar.result()) {
      System.out.println(partitionInfo);
    }
  }
});
```

== 错误处理
通过注册一个`exceptionHandler`可以用来处理consumer或者producer发生的异常比如 timeout 异常.[consumer](../../apidocs/io/vertx/kafka/client/consumer/KafkaConsumer.html#exceptionHandler-io.vertx.core.Handler),[producer](../../apidocs/io/vertx/kafka/client/producer/KafkaProducer.html#exceptionHandler-io.vertx.core.Handler).

```java

consumer.exceptionHandler(e -> {
  System.out.println("Error = " + e.getMessage());
});
```

== 随verticles自动销毁
如果在verticles内部创建consumer或者producer,那么当vertcle执行`undeployed`的时候，相关的consumer和producer会自动关闭.  

== 使用Vert.x序列化与反序列化
Vert.x Kafka Client 自带序列化与反序列化,比如buffer, jsonObject,jsonArray.  
在consumer里可以使用Buffer.

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

同意的Producer也可以.

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

也可以在`create`方法里指明序列化与反序列化.  

比如创建Consumer时.

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

创建Producer时

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

== RxJava API
Kafka Client也提供Rx风格的API. `(译者注:这个也可以看看kafka-stream)`

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

== 流实现与kafka原生对象
如果你希望直接操作原生的kafka records,你可以使用原生的kafka流式对象.[KafkaReadStream](../../apidocs/io/vertx/kafka/client/consumer/KafkaReadStream.html)用于读取topic以及partition,他读到的是一个流对象[ConsumerRecord](../../apidocs/org/apache/kafka/clients/consumer/ConsumerRecord.html)对象.[KafkaWriteStream](../../apidocs/io/vertx/kafka/client/producer/KafkaWriteStream.html)用于写消息到topic,写的是一个[ProducerRecord](../../apidocs/org/apache/kafka/clients/producer/ProducerRecord.html)这也是一个流对象.  

API通过这些接口直接暴露给用户,其他语言实现也应该类似.

