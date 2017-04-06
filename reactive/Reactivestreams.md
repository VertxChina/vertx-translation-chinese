# Vert.x Reactive Streams

- [原文档][1]
- [组件源码][2]

## 中英文对照表

* publisher(n.)：发布者
* subscriber(n.)：订阅者
* read stream(n.)：可读流
* write stream(n.)：可写流
* subscribe(v.)：注册
* back pressure：背压机制

**为了支持在 JVM 上进行非阻塞的带背压机制的异步流处理，[ Reactive Streams ](http://www.reactive-streams.org/)做了一些初创性的工作来提供这样一份标准。**

这个库提供了 Vert.x 上 Reactive Streams 标准的实现。

在处理流式数据方面，Vert.x 有自己的机制；通过这三个类：`io.vertx.core.streams.ReadStream` 、 `io.vertx.core.streams.WriteStream` 和 `io.vertx.core.streams.Pump`，可以在将数据从一个流泵到另一个流时，实现流量控制。更多关于 Vert.x 流方面的信息请查阅 Vert.x Core 部分的手册。

这个库为可读流、可写流都提供了实现，这两者分别扮演了 Reactive Streams 中发布者和订阅者的角色；这使得我们能够以对待 Vert.x 中读写流的方式处理任意 Reactive Streams 的发布者和订阅者。

## 使用 Vert.x Reactive Streams

要使用 Vert.x Reactive Streams 组件，需要在构建描述符中添加如下依赖：

* Maven（在 `pom.xml` 文件中）：

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-reactive-streams</artifactId>
  <version>3.4.1</version>
</dependency>

```

* Gradle（在 `build.gradle` 文件中）：

```groovy
compile 'io.vertx:vertx-reactive-streams:3.4.1'
```

## Reactive Read Stream

我们为 Vert.x 的 `ReadStream` 接口提供了实现类[ `ReactiveReadStream` ](http://vertx.io/docs/apidocs/io/vertx/ext/reactivestreams/ReactiveReadStream.html)，它同样也实现了 Reactive Streams 的`订阅者`角色。

你可以把这个类的实例传递给任意的 Reactive Streams `发布者`（例如来自 Akka 的发布者），随后你就可以像从其他任意的 Vert.x `ReadStream`中一样读取数据（例如使用一个 `Pump` 把数据从这个流泵到一个 `WriteStream`）。

这里有个例子，从某个其他的 Reactive Streams 实现中（例如 Akka）获得一个发布者，并将其数据泵入到服务端的 HTTP 响应体中。其间，背压机制是自动执行的。

```java
ReactiveReadStream<Buffer> rrs = ReactiveReadStream.readStream();

// 在另外的发布者上注册订阅者
otherPublisher.subscribe(rrs);

// 将数据从可读流泵入 HTTP 响应
Pump pump = Pump.pump(rrs, response);

pump.start();
```

## Reactive Write Stream

同样的，我们为 `WriteStream` 接口提供了实现类[ `ReactiveWriteStream` ](http://vertx.io/docs/apidocs/io/vertx/ext/reactivestreams/ReactiveWriteStream.html)，它也是 Reactive Streams 的`发布者`角色的实现。拿到任意的 Reactive Streams `订阅者`（例如来自 Akka 的订阅者）之后，你就可以像处理其他任意的 Vert.x `WriteStream`一样，往其中写入数据（例如使用一个 `Pump` 把从 `ReadStream` 来的数据泵入其中）。

在手动处理 Vert.x 可读流的背压时，你会用到 `pause`，`resume`，`writeQueueFull` 这些方法；它们会在内部被自动转换成 Reactive Streams 中背压机制传播方面的方法（在请求更多数据项时）。

这里有个例子，从其他的 Reactive Streams 实现拿到订阅者之后，将服务端的请求体泵入其中。其间背压机制将自动运行。

```java
ReactiveWriteStream<Buffer> rws = ReactiveWriteStream.writeStream(vertx);

// 在可写流上注册另外的订阅者
rws.subscribe(otherSubscriber);

// 将 HTTP 请求泵入可写流
Pump pump = Pump.pump(request, rws);

pump.start();
```

---

> [原文档](http://vertx.io/docs/vertx-reactive-streams/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-reactive-streams/java/
[2]: https://github.com/vert-x3/vertx-reactive-streams
