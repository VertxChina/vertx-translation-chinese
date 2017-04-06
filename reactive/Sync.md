# Vert.x Sync

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 中英文对照表

* bytecode instrumentation：字节码修改/增强
* kernel thread：内核线程

##  简介

Vert.x Sync 是一组工具集，其特点是在不阻塞内核线程的同时，允许用户以同步的方式接收事件、执行异步操作。

比起很多历史遗留的应用系统，Vert.x 的一个关键优点是完全非阻塞(于内核线程而言) —— 这使它用少量的内核线程就可以处理大量的并发(例如，处理很多的连接、消息之类)，具有优异的可扩展性。

Vert.x 非阻塞性的特性产生了异步的 API 。异步 API 可以有多种风格，包括回调、Promise和Rx风格。Vert.x 在绝大多数地方使用回调(尽管它也支持Rx)。

某些情况下，使用异步 API 编程比直接使用同步 API 要更具挑战，特别是当你有好几个操作要按顺序完成时。同时，使用异步 API 时，错误的传递也会变得更复杂。

Vert.x Sync 可以让你在熟悉的同步风格下继续使用异步 API 。

在此，通往自由之路的功臣乃 `Fiber`(译者注：这个词国内有译作纤程，类似协程-coroutine)。Fiber 是超轻量级的线程，并不是对应于底层的那种内核线程，它们被阻塞时不会导致内核线程也被阻塞。

Vert.x 借助[ Quasar ](http://docs.paralleluniverse.co/quasar/)库来实现 Fiber。

> 注意：Vert.x Sync 当前只适用于 Java 。


## SyncVerticle

要使用 Vert.x Sync 库，你的代码需要继承 `io.vertx.ext.sync.SyncVerticle` 类，并重载`start()`方法和`stop()`方法(`stop` 非必需)。

这些方法还必须加上 `@Suspendable` 注解。

写好的 Sync Verticle，其部署方法和其他 Verticle 完全一样。


## Instrumentation

Vert.x 用到了 Quasar 库，这个库借助字节码增强(bytecode instrumentation)的技术实现了 Fiber。字节码增强工作是在运行时使用 Java Agent 技术完成的。

为了使这个特性正常工作，需要在启动 JVM 时指定 quasar-core jar包为 Java Agent jar 包：

```shell
-javaagent:/path/to/quasar/core/quasar-core.jar
```

如果你用的是 `vertx` 命令行工具，可以在执行 `vertx` 前设置环境变量 `ENABLE_VERTX_SYNC_AGENT` 为 `ture`，这样可以启用 Agent 的配置。

你也可以使用 [quasar-maven-plugin](https://github.com/vy/quasar-maven-plugin) 达成离线增强(offline instrumentation，指非运行时织入字节码)的效果。更多细节请参考 [Quasar 官方文档](http://docs.paralleluniverse.co/quasar/)。

## 获得一次性的异步操作结果

在Vert.x 的领域里，很多操作都会接受一个 `Handler<AsyncResult<T>>` 作为最后的参数，例如用 Vert.x 的 Mongo 客户端执行一次查询或者发送一个 Event Bus 消息然后拿到回应。

Vert.x Sync 可以让你用同步的方式得到这种一次性的异步操作的结果。这是通过调用 [`Sync.awaitResult`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/Sync.html#awaitResult-java.util.function.Consumer-) 方法完成的。调用这个方法时，需将想要执行的异步操作包装成 [`Consumer`](http://vertx.io/docs/apidocs/java/util/function/Consumer.html) 的形式。Handler 参数会在运行时传给此 `Consumer` 。

看下面的例子：

```java
EventBus eb = vertx.eventBus();

// Send a message and get the reply synchronously

Message<String> reply = awaitResult(h -> eb.send("someaddress", "ping", h));

System.out.println("Received reply " + reply.body());
```

上面的例子中，在回应返回前，Fiber 会一直被阻塞住，而内核线程不会。


## 获得一次性的事件

Vert.x Sync 也能以同步的方式获得一次性的事件，例如定时器的触发，或者 end handler（译者注：关于 end handler 的例子可以参见 Vert.x Core 文档中 HTTP 服务器与客户端 一节）的执行。这是通过 [`Sync.awaitEvent`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/Sync.html#awaitEvent-java.util.function.Consumer-) 方法完成成的。

看下面的例子：

```java
long tid = awaitEvent(h -> vertx.setTimer(1000, h));

System.out.println("Timer has now fired");
```

## 事件流

很多时候，Vert.x 的 `Handler` 接收到的是事件流，例如 Event Bus 消息的消费者(Consumer)、HTTP 服务端里的 HTTP 服务端请求(server request)。

Vert.x Sync 使你能以同步的方式从这种流中接收事件。

你需要一个同时实现了 [`Handler`](http://vertx.io/docs/apidocs/io/vertx/core/Handler.html) 和 [`Receiver`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/Receiver.html) 接口的 [`HandlerReceiverAdaptor`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/HandlerReceiverAdaptor.html) 类实例。[`Sync.streamAdaptor`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/Sync.html#streamAdaptor--) 方法可以创建这样一个实例。

你可以把它当成一个普通的 Handler ，然后可以用实现自 [`Receiver`](http://vertx.io/docs/apidocs/io/vertx/ext/sync/Receiver.html) 接口的方法来同步地接收事件。

下面是一个 Event Bus 消息消费者的例子：

```java
EventBus eb = vertx.eventBus();

HandlerReceiverAdaptor<Message<String>> adaptor = streamAdaptor();

eb.<String>consumer("some-address").handler(adaptor);

// Receive 10 messages from the consumer:
for (int i = 0; i < 10; i++) {

  Message<String> received1 = adaptor.receive();

  System.out.println("got message: " + received1.body());

}
```

## 使用 `FiberHandler`

如果你想在一般的 Handler 中使用 Fiber —— 例如 HTTP 服务端的请求处理器（request handler） ，那得首先把这个一般的 Handler 转换为 Fiber Handler 。

Fiber Handler 会在 Fiber 里运行那个一般的 Handler 。

看例子：

```java
EventBus eb = vertx.eventBus();

vertx.createHttpServer().requestHandler(fiberHandler(req -> {

  // Send a message to address and wait for a reply
  Message<String> reply = awaitResult(h -> eb.send("some-address", "blah", h));

  System.out.println("Got reply: " + reply.body());

  // Now end the response
  req.response().end("blah");

})).listen(8080, "localhost");
```

## 更多示例

在 [Examples Repository](https://github.com/vert-x3/vertx-examples/tree/master/sync-examples) 这里有一些示例，展示了 vertx-sync 的用法。

---

> [原文档](http://vertx.io/docs/vertx-sync/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-sync/java/
[2]: https://github.com/vert-x3/vertx-sync
[3]: https://github.com/vert-x3/vertx-examples/tree/master/sync-examples
