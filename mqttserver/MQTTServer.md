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
在Handler内,endpoint实例提供accept方法以响应CONNACK回应远程客户端，