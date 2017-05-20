# Vert.x MQTT Server

这个组件提供了一个服务器，它能处理远程MQTT服务器连接，通信和信息交换。
它提供了一些API与客户端接接收到raw protocol信息的事件和报露出一些为了发送信息到客户端的特性。
它不是一个功能齐全的broker，但可以用来建立类似的东西或者协议转换。
> 警告：该模块处于技术预览版本， 这意味着它的API可能随着版本变动


## 使用MQTT服务器

该组件还未正式发布到Vertx技术栈中，要使用Vert.x MQTT服务器snapshot版本，增加以下仓库到repositories条目中，之后增加以下依赖到构建描述符中

- Maven(in your pom.xml)
```xml
<repository>
    <id>oss.sonatype.org-snapshot</id>
    <url>https://oss.sonatype.org/content/repositories/snapshots</url>
</repository>
```

```xml
<dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-mqtt-server</artifactId>
    <version>3.4.0-SNAPSHOT</version>
</dependency>
```

- Maven(in your pom.xml)
```xml
maven { url "https://oss.sonatype.org/content/repositories/snapshots" }
```

```xml
compile io.vertx:vertx-mqtt-server:3.4.0-SNAPSHOT
```

## 开始
### 处理客户端连接/断开
这个例子展示了如何处理一个来自远程MQTT客户端的请求，首先，会创建一个`Mqttserver`实例和使用`endpontHandler`方法选定一个处理器用于处理远程客户端发送的CONNECT信息。
`MqttEndpont`实例,会被当做Handler的参数，它携带了所有主要的与CONNECT消息相关联的信息，例如客户端标识符，用户名/密码，【"Will information"】，清除session标志，协议版本和保活超时。
在Handler内,endpoint实例提供accept方法以响应CONNACK回应远程客户端；通过该方式，连接会被建立。最终，服务器通过`listen`方法以默认行为的行为(运行在localhost和默认MQTT端口1883)启动。存在一个相同的方法，允许选定一个Handler去检查是否服务器是否已经正常启动。

```java
MqttServer mqttServer = MqttServer.create(vertx);
mqttServer.endpointHandler(endpoint -> {

  // shows main connect info
  System.out.println("MQTT client [" + endpoint.clientIdentifier() + "] request to connect, clean session = " + endpoint.isCleanSession());

  if (endpoint.auth() != null) {
    System.out.println("[username = " + endpoint.auth().userName() + ", password = " + endpoint.auth().password() + "]");
  }
  if (endpoint.will() != null) {
    System.out.println("[will topic = " + endpoint.will().willTopic() + " msg = " + endpoint.will().willMessage() +
      " QoS = " + endpoint.will().willQos() + " isRetain = " + endpoint.will().isWillRetain() + "]");
  }

  System.out.println("[keep alive timeout = " + endpoint.keepAliveTimeSeconds() + "]");

  // accept connection from the remote client
  endpoint.accept(false);

})
  .listen(ar -> {

    if (ar.succeeded()) {

      System.out.println("MQTT server is listening on port " + ar.result().actualPort());
    } else {

      System.out.println("Error on starting the server");
      ar.cause().printStackTrace();
    }
  });
```
*endpoint*实例提供`disconnectHandler`用于选定一个handler当远程客户端发送 DISCONNECT消息会被调用，该handler没有参数。
```java
endpoint.disconnectHandler(v -> {

  System.out.println("Received disconnect from client");
});
```

### 使用SSL / TLS支持处理客户端连接/断开连接
服务端支持以SSL/TLS方式接受连接请求用来验证和加密。为了做到这一点，`MqttServerOptions`类提供了`setSsl`方法用来设置SSL/TLS的用法(传递`true`作为值)和一些其他提供了服务器验证和私钥(as java key store reference，PEM或PFX格式)。在以下例子，`setKeyCertOptions`方法被用来传递一个PEM格式的证书。该方法需要一个实现了`KeyCertOptions`接口的实例，在这种情况下`PemKeyCertOptions`类被用来提供提供服务器证书和对应`setCertPath`与`setKeyPath`方法的私钥路径。MQTT服务器通常以传递一个Vert.x实例启动和以下的MQTT选项实例作为创建方法。
```java
MqttServerOptions options = new MqttServerOptions()
  .setPort(8883)
  .setKeyCertOptions(new PemKeyCertOptions()
    .setKeyPath("./src/test/resources/tls/server-key.pem")
    .setCertPath("./src/test/resources/tls/server-cert.pem"))
  .setSsl(true);

MqttServer mqttServer = MqttServer.create(vertx, options);
mqttServer.endpointHandler(endpoint -> {

  // shows main connect info
  System.out.println("MQTT client [" + endpoint.clientIdentifier() + "] request to connect, clean session = " + endpoint.isCleanSession());

  if (endpoint.auth() != null) {
    System.out.println("[username = " + endpoint.auth().userName() + ", password = " + endpoint.auth().password() + "]");
  }
  if (endpoint.will() != null) {
    System.out.println("[will topic = " + endpoint.will().willTopic() + " msg = " + endpoint.will().willMessage() +
      " QoS = " + endpoint.will().willQos() + " isRetain = " + endpoint.will().isWillRetain() + "]");
  }

  System.out.println("[keep alive timeout = " + endpoint.keepAliveTimeSeconds() + "]");

  // accept connection from the remote client
  endpoint.accept(false);

})
  .listen(ar -> {

    if (ar.succeeded()) {

      System.out.println("MQTT server is listening on port " + ar.result().actualPort());
    } else {

      System.out.println("Error on starting the server");
      ar.cause().printStackTrace();
    }
  });
```
所有其他与处理端点连接和断开连接相关在没有SSL/TLS支持下被以相同方式被管理。

###处理客户端订阅和退订请求
在客户端和服务端的连接被建立后，客户端可以以指定的主题发送订阅消息。`MqttEndpoint`接口允许使用`subscribeHandler`处理到来的订阅请求。
这样的Handler接受一个`MqttSubscribeMessage`接口的实例，which brings the list of topics with related QoS levels as desired by the client.最终，端点实例提供`subscribeAckowledge`方法以包含了授予QoS级别的相关SUBACK消息回应客户端。
```java
endpoint.subscribeHandler(subscribe -> {

  List<MqttQoS> grantedQosLevels = new ArrayList<>();
  for (MqttTopicSubscription s: subscribe.topicSubscriptions()) {
    System.out.println("Subscription for " + s.topicName() + " with QoS " + s.qualityOfService());
    grantedQosLevels.add(s.qualityOfService());
  }
  // ack the subscriptions request
  endpoint.subscribeAcknowledge(subscribe.messageId(), grantedQosLevels);

});
```
以相同的方式，可以使用`unsubscribeHandler`方法在端点上选定一个handler，当客户端发送UNSUBSCRIBE消息时该handler会被调用。这个handler接受一个实现了`MqttUnsubscribeMessage`接口的实例作为参数，它携带了一个退订列表。最终，端点实例提供`unsubscribeAcknowledge`方法用于以相关的UNSUBACK消息回应客户。
```java
endpoint.unsubscribeHandler(unsubscribe -> {

  for (String t: unsubscribe.topics()) {
    System.out.println("Unsubscription for " + t);
  }
  // ack the subscriptions request
  endpoint.unsubscribeAcknowledge(unsubscribe.messageId());
});
```
###处理客户端发布消息
为了处理远程客户端发布的消息，`MqttEndpoint`接口提供了`publishHandler`方法用于选定handler当客户端发送一个PUBLISH消息时。
这个handler接受一个`MqttPublishMessage`接口的实例作为参数，with the payload, the QoS level, the duplicate and retain flags.

假如QoS级别是0(最多一次)，就没有必要给客户端响应。

假如QoS级别是1(至少一次)，端点需要通过`publishAcknowledge`方法回应一个PUBACK消息

假如QoS级别是2(正好一次)，端点需要使用`publishReceived`方法回应一个PUBREC消息。在这种情况下端点应该处理来自于客户端的PUBREL消息并且(当收到来资源端点的PUBREC消息，远程客户端发送就会发送它)，可以通过`publishReleaseHandler`方法实现。为了关闭QoS级别2传递，端点可以使用`publishComplete`方法用来发送PUBCOMP消息到客户端。

```java
endpoint.publishHandler(message -> {

  System.out.println("Just received message [" + message.payload().toString(Charset.defaultCharset()) + "] with QoS [" + message.qosLevel() + "]");

  if (message.qosLevel() == MqttQoS.AT_LEAST_ONCE) {
    endpoint.publishAcknowledge(message.messageId());
  } else if (message.qosLevel() == MqttQoS.EXACTLY_ONCE) {
    endpoint.publishRelease(message.messageId());
  }

}).publishReleaseHandler(messageId -> {

  endpoint.publishComplete(messageId);
});
```

###发布消息到客户端
通过使用`publish`方法端点可以发布一个消息到远程客户端(发送一个PUBLISH消息)，它使用以下入参，要发布的主题，负载，QoS级别，复制和保留标志。

假如QoS级别是0(最多一次)，端点不会收到任何客户端的响应。

假如QoS级别是1(最多一次)，端点需要去处理来自客户端的PUBACK消息，为了收到最后的确认消息。可以使用`publishAcknowledgeHandler`方法。

假如QoS级别是2(正好一次)，端点需要去处理来自客户端的PUBREC消息。可以通过`publishReceivedHandler`方法来完成该操作。
在该Handler内，端点可以使用`publishRelease`方法响应PUBREL消息给客户端。最后一步是处理来自客户端的PUBCOMP消息；可以使用`publishCompleteHandler`来指定一个handler当收到PUBCOMP消息时候调用。
```java
// just as example, publish a message with QoS level 2
endpoint.publish("my_topic", Buffer.buffer("Hello from the Vert.x MQTT server"), MqttQoS.EXACTLY_ONCE, false, false)

// specifing handlers for handling QoS 1 and 2
endpoint.publishAcknowledgeHandler((messageId: java.lang.Integer) => {

  println(s"Received ack for message = ${messageId}")

}).publishReceivedHandler((messageId: java.lang.Integer) => {

  endpoint.publishRelease(messageId)

}).publishCompleteHandler((messageId: java.lang.Integer) => {

  println(s"Received ack for message = ${messageId}")
})
```
Be notified by client keep alive
底层的MQTT保活机制是由服务器内部处理的。当接收到连接消息，服务器关注在该消息内的保活超时时间，用来检查客户端是否在该超市时间内没有发送任何消息。同时，对于接收到每个PINGREQ消息，服务器会以PINGRESP响应。

即使对于高等级应用不需要处理它，`MqttEndpoint`接口提供`pingHandler`方法用来选定一个handler，当收到来自客户端的PINGREQ消息会被调用。
对于应用程序来说这只是一个通知，客户端并没有发送任何有意义的信息，只是一个用于检测保活的ping消息。在任何情况下，PINGRESP会被服务器自动发送。

### 关闭服务器
`MqttServer`借口提供了`close`方法用于关闭服务器。它停止监听到来的连接和关闭所有远程客户端活跃的连接。该方法是一个异步方法并且有个重载方法提供了选定一个完成handler当服务器真正关闭时进行调用。
```java
mqttServer.closeFuture().onComplete{
  case Success(result) => println("Success")
  case Failure(cause) => println("Failure")
}
```
### 在verticles中自动清理
假如你是在verticles中创建的MQTT服务器，当verticle取消部署这些服务器会被自动关闭。

### 扩展：共享MQTT服务器
与MQTT服务器相关的handler总是在event loop线程中执行。这意味着在一个多核系统中，仅有一个实例被部署，一个核被使用。为了使用更多的核，可以部署更多的MQTT服务器实例
可以通过编程方式实现：
```java
for ( i <- 0 until 10) {

  var mqttServer = MqttServer.create(vertx)
  mqttServer.endpointHandler((endpoint: io.vertx.scala.mqtt.MqttEndpoint) => {
    // handling endpoint
  }).listenFuture().onComplete{
    case Success(result) => println("Success")
    case Failure(cause) => println("Failure")
  }

}
```
或者使用一个verticle指定实例的数量：
```java
var options = DeploymentOptions()
  .setInstances(10)

vertx.deployVerticle("com.mycompany.MyVerticle", options)
```
真正是这样的，即使只有一个MQTT服务器被部署，但到来的连接会被Vert.x使用轮转算法分发到不同的连接handlers在不同的核上执行。
