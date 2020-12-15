# Vert.x Core 文档手册

## 中英对照表

* Client：客户端
* Server：服务器
* Primitive：基本（描述类型）
* Writing：编写（有些地方译为开发）
* Fluent：流式的
* Reactor：反应器，Multi-Reactor即多反应器
* Options：配置项，作为参数时候翻译成选项
* Context：上下文环境
* Undeploy：撤销（反部署，对应部署）
* Unregister：注销（反注册，对应注册）
* Destroyed：销毁
* Handler/Handle：处理器/处理，有些特定处理器未翻译，如Completion Handler等。
* Block：阻塞
* Out of Box：标准环境（开箱即用）
* Timer：计时器
* Event Loop Pool：事件轮询线程池，大部分地方未翻译
* Worker Pool：工作者线程池，大部分地方未翻译
* Sender：发送者
* Consumer：消费者
* Receiver/Recipient：接收者
* Entry：条目（一条key=value的键值对）
* Map：动词翻译成 “映射”，名词为数据结构未翻译
* Logging：动词翻译成 “记录”，名词翻译成日志器
* Trust Store：受信存储
* Frame：帧
* Event Bus：事件总线
* Buffer：缓冲区（一些地方使用的 Vert.x 中的 `Buffer` 类则不翻译）
* Chunk：块（HTTP 数据块，分块传输、分块模式中会用到）
* Pump：泵（平滑流式数据读入内存的机制，防止一次性将大量数据读入内存导致内存溢出）
* Header：请求/响应头
* Body：请求/响应体（有些地方翻译成请求/响应正文）
* Pipe：管道
* Round-Robin：轮询
* Application-Layer Protocol Negotiation：ALPN，应用层协议协商
* Wire：报文
* Flush：刷新（指将缓冲区中已有的数据一次性压入，用这种方式清空缓冲区，传统上翻译成刷新）
* Cipher Suite：密码套件
* Datagram：数据报
* Socket：套接字（有些地方未翻译，直接用的 Socket）
* Multicast：多播（组播）
* Concurrent Composition：并发合并
* High Availability：高可用性
* Multiplexing：多路复用
* Fail-Over：故障转移
* Hops：跳数（一台路由器/主机到另外一台路由器/主机所经过的路由器的数量，经过路由转发次数越多，跳数越大）
* Launcher：启动器

> 请注意：Vert.x 和 `Vertx` 的区别：文中所有 Vert.x 概念使用标准单词 Vert.x，而 `Vertx` 通常表示Java中的类 `io.vertx.core.Vertx`。

---
## 组件介绍

**Vert.x 的核心 Java API 被我们称为 Vert.x Core**。

[Github仓库](https://github.com/eclipse/vert.x)

Vert.x Core 提供了下列功能:

* 编写 TCP 客户端和服务端
* 编写支持 WebSocket 的 HTTP 客户端和服务端
* 事件总线
* 共享数据 —— 本地的Map和分布式集群Map
* 周期性、延迟性动作
* 部署和撤销 Verticle 实例
* 数据报套接字
* DNS客户端
* 文件系统访问
* 高可用性
* 本地传输
* 集群

Vert.x Core中的功能相当底层，不包含诸如数据库访问、授权或高层Web应用的功能。您可以在**Vert.x ext** （扩展包）（译者注：Vert.x的扩展包是Vert.x的子项目集合，类似[Web](https://vertx.io/docs/vertx-web/java/)、[Web Client](https://vertx.io/docs/vertx-web-client/java/)、[Databases](https://vertx.io/docs/#databases)等）中找到这些功能。

**Vert.x Core** 小而轻，您可以只使用您需要的部分，它可整体嵌入现存应用中。Vert.x没有强制要求使用特定的方式构造应用。

Vert.x也支持在其他语言中使用Vert.x Core，而且在使用诸如 JavaScript 或 Ruby 等语言编写Vert.x代码时，无需直接调用 Java的API；毕竟不同的语言有不同的代码风格，若强行让 Ruby 开发人员遵循 Java 的代码风格会很怪异，所以我们根据 Java API 自动生成了适应不同语言代码风格的 API。

如果您在使用 Maven 或 Gradle（译者注：两种常用的项目构建工具），将以下依赖项添加到项目描述文件的 `dependencies` 节点即可使用 **Vert.x Core** 的API：

* Maven（您的 `pom.xml` 中）

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-core</artifactId>
  <version>4.0.0</version>
</dependency>
```

* Gradle（您的 `build.gradle` 中）

```gradle
dependencies {
    compile 'io.vertx:vertx-core:4.0.0'
}
```

接下来讨论 Vert.x Core 的概念和特性。

---
## 故事从 Vert.x 开始

使用Vert.x进行开发离不开 [`Vertx`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html) 对象。它是 Vert.x 的控制中心，也是您做几乎一切事情的基础，包括创建客户端和服务器、获取事件总线的引用、设置定时器等等。

那么如何获取它的实例呢？

如果您用嵌入方式使用Vert.x，可通过以下代码创建实例：

```java
Vertx vertx = Vertx.vertx();
```

> 请注意：*大部分应用将只会需要一个Vert.x实例，但如果您有需要也可创建多个Vert.x实例，如：隔离的事件总线或不同组的客户端和服务器。*

### 创建 Vertx 对象时指定配置项

如果缺省的配置不适合您，可在创建 `Vertx` 对象的同时指定配置项：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().setWorkerPoolSize(40));
```

[`VertxOptions`](https://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html)对象有很多配置，包括集群、高可用、池大小等。在Javadoc中描述了所有配置的细节。

### 创建集群模式的 Vert.x 对象

如果您想创建一个集群模式的 `Vertx` 对象（参考 [Event Bus](#Event-Bus) 章节了解更多事件总线集群细节），那么通常情况下您将需要使用另一种异步的方式来创建 `Vertx` 对象。

不同的 Vert.x 实例组成一个集群需要一些时间（也许是几秒钟）,在这段时间内，我们不想去阻塞调用线程，所以我们将结果异步返回给您。

> 译者注：这里给个示例：
```java
// 注意要添加对应的集群管理器依赖，详情见集群管理器章节
VertxOptions options = new VertxOptions();
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result(); // 获取到了集群模式下的 Vertx 对象
    // 做一些其他的事情
  } else {
    // 获取失败，可能是集群管理器出现了问题
  }
});
```

---

## 是流式的吗？

您也许注意到前边的例子里使用了一个流式（Fluent）的API。

一个流式的API表示将多个方法的调用链在一起。例如：

```java
request.response().putHeader("Content-Type", "text/plain").write("some text").end();
```

这是贯穿 Vert.x API 中的一个通用模式，所以请适应这种代码风格。

流式调用可以让代码更为简洁。当然，Vert.x并不强制您用这种方式书写代码，如果您更倾向于用以下非流式编码，您可以忽略它：

```java
HttpServerResponse response = request.response();
response.putHeader("Content-Type", "text/plain");
response.write("some text");
response.end();
```

---

## Don’t call us, we’ll call you

Vert.x 的 API 大部分都是事件驱动的。这意味着当您感兴趣的事情发生时，会以事件的形式发送给您。

以下是一些事件的例子：

* 触发一个计时器
* Socket 收到了一些数据
* 从磁盘中读取了一些数据
* 发生了一个异常
* HTTP 服务器收到了一个请求

Vert.x API调用您提供的处理器来处理事件。例如每隔一秒发送一个事件的计时器：

```java
vertx.setPeriodic(1000, id -> {
  // This handler will get called every second
  // 这个处理器将会每隔一秒被调用一次
  System.out.println("timer fired!");
});
```

又比如收到一个HTTP请求：

```java
server.requestHandler(request -> {
  // This handler will be called every time an HTTP request is received at the server
  // 服务器每次收到一个HTTP请求时这个处理器将被调用
  request.response().end("hello world!");
});
```

稍后当Vert.x有事件要传给您的处理器时，它会 **异步地** 调用这个处理器。

由此，下面会引入Vert.x中一些重要的概念。

---

## 不要阻塞我！

Vert.x中的几乎所有API都不会阻塞调用线程，除了个别特例（如以 "Sync" 结尾的某些文件系统操作）。

可以立即提供结果的API会立即返回，否则您需要提供一个处理器（`Handler`）来接收稍后回调的事件。

因为Vert.x API不会阻塞线程，所以通过Vert.x您可以只使用少量的线程来处理大量的并发。

当使用传统的阻塞式API做以下操作时，调用线程可能会被阻塞：

* 从 Socket 中读取数据
* 写数据到磁盘
* 发送消息给接收者并等待回复
* 其他很多情况

在上述情况下，线程在等待处理结果时它不能做任何事，此时这些线程并无实际用处。这意味着如果使用阻塞式API处理大量并发，需要大量线程来防止应用程序停止运转，而这些线程使用的内存（例如它们的栈）和线程上下文切换开销很可观。这意味着，阻塞式的方式对于现代应用程序所需要的并发级别来说是难于扩展的。

---

## Reactor 模式和 Multi-Reactor 模式

我们前边提过 Vert.x 的 API 都是事件驱动的，当有事件时 Vert.x 会将事件传给处理器来处理。在多数情况下，Vert.x使用被称为 **Event Loop** 的线程来调用您的处理器。由于Vert.x或应用程序的代码块中没有阻塞，**Event Loop** 可以在事件到达时快速地分发到不同的处理器中。由于没有阻塞，Event Loop 可在短时间内分发大量的事件。例如，一个单独的 **Event Loop** 可以非常迅速地处理数千个 HTTP 请求。

我们称之为 [Reactor 模式](https://en.wikipedia.org/wiki/Reactor_pattern)（译者注：Reactor Pattern 翻译成了[反应器模式](https://zh.wikipedia.org/wiki/%E5%8F%8D%E5%BA%94%E5%99%A8%E6%A8%A1%E5%BC%8F)）。

您之前也许听说过它，例如 Node.js 实现了这种模式。

在一个标准的Reactor实现中，有 **一个独立的 Event Loop** 会循环执行，处理所有到达的事件并传递给处理器处理。单一线程的问题在于它在任意时刻只能运行在一个核上，如果您希望单线程反应器应用（如您的 Node.js 应用）扩展到多核服务器上，则需要启动并且管理多个不同的进程。

Vert.x的工作方式有所不同。每个 `Vertx` 实例维护的是 **多个Event Loop 线程**。默认情况下，我们会根据机器上可用的核数量来设置 Event Loop 的数量，您亦可自行设置。这意味着 Vertx 进程能够在您的服务器上扩展，与 Node.js 不同。

我们将这种模式称为 **Multi-Reactor 模式**（多反应器模式），区别于单线程的 Reactor 模式（反应器模式）。

> 请注意：*即使一个 `Vertx` 实例维护了多个 Event Loop，任何一个特定的处理器永远不会被并发执行。大部分情况下（除了 [Worker Verticle](https://vertx.io/docs/vertx-core/java/#worker_verticles) 以外）它们总是在同一个 Event Loop 线程中被调用。*

---

## 黄金法则：不要阻塞Event Loop

尽管Vert.x 的 API 都是非阻塞式的，且不会阻塞 Event Loop，但是用户编写的处理器中可能会阻塞 Event Loop。如果这样做，该 Event Loop 在被阻塞时就不能做任何事情；如果您阻塞了 `Vertx` 实例中的所有 Event Loop，那么您的应用就会完全停止！

所以不要这样做！**这是一个警告!**

这些阻塞做法包括：

* `Thead.sleep()`
* 等待一个锁
* 等待一个互斥信号或监视器（例如同步的代码块）
* 执行一个长时间数据库操作并等待其结果
* 执行一个复杂的计算，占用了可感知的时长
* 在循环语句中长时间逗留

如果上述任何一种情况停止了 Event Loop 并占用了 **显著执行时间** ，那您应该去面壁（译者注：原文此处为 Naughy Step，英国父母会在家里选择一个角落作为小孩罚站或静坐的地方，被称为 naughty corner 或 naughty step），等待下一步的指示。

所以，什么是 **显著执行时间** ？

您要等多久？它取决于您的应用程序和所需的并发数量。

如果您只有单个 Event Loop，而且您希望每秒处理10000个 HTTP 请求，很明显的是每一个请求处理时间不可以超过0.1毫秒，所以您不能阻塞任何过多（大于0.1毫秒）的时间。

**这个数学题并不难，将留给读者作为练习。**

如果您的应用程序没有响应，可能这是一个迹象，表明您在某个地方阻塞了Event Loop。为了帮助您诊断类似问题，若 Vert.x 检测到 Event Loop 有一段时间没有响应，将会自动记录这种警告。若您在日志中看到类似警告，那么您需要检查您的代码。比如：

```
Thread vertx-eventloop-thread-3 has been blocked for 20458 ms
```

Vert.x 还将提供堆栈跟踪，以精确定位发生阻塞的位置。

如果想关闭这些警告或更改设置，您可以在创建 `Vertx` 对象之前在 [`VertxOptions`](https://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html) 中完成此操作。

---

## Future的异步结果

Vert.x 4使用future承载异步结果。 

异步的方法会返回一个[`Future`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html)对象，其包含成功或失败的异步结果。

我们不能直接操作future的异步结果，而应该设置future的handler；当future执行完毕，结果可用时，会调用handler进行处理。

```java
FileSystem fs = vertx.fileSystem();

Future<FileProps> future = fs.props("/my_file.txt");

future.onComplete((AsyncResult<FileProps> ar) -> {
  if (ar.succeeded()) {
    FileProps props = ar.result();
    System.out.println("File size = " + props.size());
  } else {
    System.out.println("Failure: " + ar.cause().getMessage());
  }
});
```

> 请注意：Vert.x 3的API只提供了回调模式；为了减少从Vert.x 3迁移到Vert.x 4的工作量，Vert.x 4为每个异步方法都保留了回调版本。如上面样例代码的`props`方法，提供了带回调参数的版本[`props`](https://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html#props-java.lang.String-io.vertx.core.Handler-)

---

## Future组合

[`compose`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html#compose-java.util.function.Function-) 方法作用于顺序组合 future：  

- 若当前future成功，执行`compose`方法指定的方法，该方法返回新的future；当返回的新future完成时，future组合成功；
- 若当前future失败，则future组合失败。


```java
FileSystem fs = vertx.fileSystem();

Future<Void> future = fs
  .createFile("/foo")
  .compose(v -> {
     // createFile文件创建完成后执行
    return fs.writeFile("/foo", Buffer.buffer());
  })
  .compose(v -> {
    // writeFile文件写入完成后执行
    return fs.move("/foo", "/bar");
  });
```

这里例子中，有三个操作被串起来了：

1. 一个文件被创建（`createFile`）
2. 一些东西被写入到文件（`writeFile`）
3. 文件被移走（`move`）

如果这三个步骤全部成功，则最终的 `Future`（`future`）会是成功的；其中任何一步失败，则最终 `Future` 就是失败的。

除了上述方法，[`Future`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html)还提供了更多方法：`map`，`recover`，`otherwise`，以及`flatMap`（等同`compose`方法）。

---

## Future协作

Vert.x 中的 [`Future`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html) 支持协调多个Future，支持并发组合（并行执行多个异步调用）和顺序组合（依次执行异步调用）。

> 译者注：Vert.x 中的 [`Future`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html) 即异步开发模式中的 Future/Promise 模式的实现。

[`CompositeFuture.all`](https://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#all-io.vertx.core.Future-io.vertx.core.Future-) 方法接受多个 `Future` 对象作为参数（最多6个，或者传入 `List`）。当所有的 `Future` 都成功完成，该方法将返回一个*成功的* `Future`；当任一个 `Future` 执行失败，则返回一个*失败的* `Future`：

```java
Future<HttpServer> httpServerFuture = httpServer.listen();

Future<NetServer> netServerFuture = netServer.listen();

CompositeFuture.all(httpServerFuture, netServerFuture).onComplete(ar -> {
  if (ar.succeeded()) {
    // 所有服务器启动完成
  } else {
    // 有一个服务器启动失败
  }
});
```

所有被合并的 `Future` 中的操作同时运行。当组合的处理操作完成时，该方法返回的 `Future` 上绑定的处理器（[`Handler`](https://vertx.io/docs/apidocs/io/vertx/core/Handler.html)）会被调用。只要有一个操作失败（其中的某一个 `Future` 的状态被标记成失败），则返回的 `Future` 会被标记为失败。如果所有的操作都成功，则返回的 `Future` 将会成功完成。

您可以传入一个 `Future` 列表（可能为空）：

```java
CompositeFuture.all(Arrays.asList(future1, future2, future3));
```

 `all` 方法的合并会等待所有的 Future 成功执行（或任一失败），而`any` 方法的合并会等待第一个成功执行的Future。[`CompositeFuture.any`](https://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#any-io.vertx.core.Future-io.vertx.core.Future-) 方法接受多个 `Future` 作为参数（最多6个，或传入 `List`）。当任意一个 `Future` 成功得到结果，则该 `Future` 成功；当所有的 `Future` 都执行失败，则该 `Future` 失败。

```java
CompositeFuture.any(future1, future2).onComplete(ar -> {
  if (ar.succeeded()) {
    // 至少一个成功
  } else {
    // 所有的都失败
  }
});
```

它也可使用 `Future` 列表传参：

```java
CompositeFuture.any(Arrays.asList(f1, f2, f3));
```

`join` 方法的合并会等待所有的 `Future` 完成，无论成败。[`CompositeFuture.join`](https://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#join-io.vertx.core.Future-io.vertx.core.Future-) 方法接受多个 `Future` 作为参数（最多6个），并将结果归并成一个 `Future` 。当全部 `Future` 成功执行完成，得到的 `Future` 是成功状态的；当至少一个 `Future` 执行失败时，得到的 `Future` 是失败状态的。

```java
CompositeFuture.join(future1, future2, future3).onComplete(ar -> {
  if (ar.succeeded()) {
    // 所有都成功
  } else {
    // 至少一个失败
  }
});
```

它也可使用 `Future` 列表传参：

```java
CompositeFuture.join(Arrays.asList(future1, future2, future3));
```

### 兼容CompletionStage

JDK的`CompletionStage`接口用于组合异步操作，Vert.x的`Future` API可兼容`CompletionStage`。

我们可以用[`toCompletionStage`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html#toCompletionStage--)方法将Vert.x的`Future`对象转为`CompletionStage`对象，如：

```java
Future<String> future = vertx.createDnsClient().lookup("vertx.io");
future.toCompletionStage().whenComplete((ip, err) -> {
  if (err != null) {
    System.err.println("Could not resolve vertx.io");
    err.printStackTrace();
  } else {
    System.out.println("vertx.io => " + ip);
  }
});
```

相应地，可使用[`Future.fromCompletionStage`](https://vertx.io/docs/apidocs/io/vertx/core/Future.html#fromCompletionStage-java.util.concurrent.CompletionStage-)方法将`CompletionStage`对象转为Vert.x的`Future`对象。`Future.fromCompletionStage`有两个重载方法：

1. 第一个重载方法只接收一个`CompletionStage`参数，会在执行`CompletionStage`实例的线程中调用`Future`的方法；
2. 第二个重载方法额外多接收一个[`Context`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html)参数，会在Vert.x的Context中调用`Future`的方法。

> 重要提示：由于Vert.x的`Future`通常会与Vert.x的代码、库以及客户端等一起使用，为了与Vert.x的线程模型更好地配合，大部分场景下应使用`Future.fromCompletionStage(CompletionStage, Context)`方法。

下面的例子展示了如何将`CompletionStage`对象转为Vert.x的`Future`对象，这里选择使用Vert.x的Context执行:

```java
Future.fromCompletionStage(completionStage, vertx.getOrCreateContext())
  .flatMap(str -> {
    String key = UUID.randomUUID().toString();
    return storeInDb(key, str);
  })
  .onSuccess(str -> {
    System.out.println("We have a result: " + str);
  })
  .onFailure(err -> {
    System.err.println("We have a problem");
    err.printStackTrace();
  });
```

---

## Verticle

Vert.x 通过开箱即用的方式提供了一个简单便捷的、可扩展的、类似 [Actor Model](https://en.wikipedia.org/wiki/Actor_model) 的部署和并发模型机制。您可以用此模型机制来保管您自己的代码组件。

**这个模型是可选的，Vert.x 并不强制使用这种方式创建应用程序。**

这个模型并不是严格的 Actor 模式实现，但它确实有相似之处，特别是在并发、扩展性和部署等方面。

使用该模型，需要将应用代码编写成多个 **Verticle**。

Verticle 是由 Vert.x 部署和运行的代码块。默认情况一个 Vert.x 实例维护了N（默认情况下N = CPU核数 x 2）个 Event Loop 线程。Verticle 实例可使用任意 Vert.x 支持的编程语言编写，而且一个简单的应用程序也可以包含多种语言编写的 Verticle。

您可以将 Verticle 想成 [Actor Model](https://en.wikipedia.org/wiki/Actor_model) 中的 Actor。

一个应用程序通常是由在同一个 Vert.x 实例中同时运行的许多 Verticle 实例组合而成。不同的 Verticle 实例通过向 [Event Bus](#event_bus) 上发送消息来相互通信。

### 编写 Verticle

Verticle 的实现类必须实现 [`Verticle`](https://vertx.io/docs/apidocs/io/vertx/core/Verticle.html) 接口。

如果您喜欢的话，可以直接实现该接口，但是通常直接从抽象类 [`AbstractVerticle`](https://vertx.io/docs/apidocs/io/vertx/core/AbstractVerticle.html) 继承更简单。

这儿有一个例子：

```java
public class MyVerticle extends AbstractVerticle {

  // Called when verticle is deployed
  // Verticle部署时调用
  public void start() {
  }

  // Optional - called when verticle is undeployed
  // 可选 - Verticle撤销时调用
  public void stop() {
  }

}
```

通常您需要像上边例子一样重写 `start` 方法。

当 Vert.x 部署 Verticle 时，它的 `start` 方法将被调用，这个方法执行完成后 Verticle 就变成已启动状态。

您同样可以重写 `stop` 方法，当Vert.x 撤销一个 Verticle 时它会被调用，这个方法执行完成后 Verticle 就变成已停止状态了。

### Verticle 异步启动和停止

有些时候您的 Verticle 启动会耗费一些时间，您想要在这个过程做一些事，并且您做的这些事并不想等到Verticle部署完成过后再发生。如：您想在 `start` 方法中部署其他的 Verticle。

您不能在您的 `start` 方法中阻塞等待其他的 Verticle 部署完成，这样做会破坏 [黄金法则](#黄金法则不要阻塞event-loop)。

所以您要怎么做？

您可以实现 **异步版本** 的 `start` 方法来实现，它接收一个 `Future` 参数。方法执行完时，Verticle 实例**并没有**部署好（状态不是 deployed）。当所有您需要做的事（如：启动HTTP服务）完成后，就可以调用 `Future` 的 `complete`（或 `fail` ）方法来标记启动完成或失败了。

这儿有一个例子：

```java
public class MyVerticle extends AbstractVerticle {

 private HttpServer server;

 public void start(Future<Void> startFuture) {
   server = vertx.createHttpServer().requestHandler(req -> {
     req.response()
       .putHeader("content-type", "text/plain")
       .end("Hello from Vert.x!");
     });

   // Now bind the server:
   server.listen(8080, res -> {
     if (res.succeeded()) {
       startFuture.complete();
     } else {
       startFuture.fail(res.cause());
     }
   });
 }
}
```

同样的，这儿也有一个异步版本的 `stop` 方法，如果您想做一些耗时的 Verticle 清理工作，您可以使用它。

```java
public class MyVerticle extends AbstractVerticle {

  public void start() {
    // 做一些事
  }

  public void stop(Future<Void> stopFuture) {
    obj.doSomethingThatTakesTime(res -> {
      if (res.succeeded()) {
        stopFuture.complete();
      } else {
        stopFuture.fail();
      }
    });
  }
}
```

> 请注意：在Verticle中启动的HTTP服务，无需在`stop`方法中手动停止；Vert.x在撤销Verticle时会自动停止运行中的服务。

### Verticle 种类

这儿有两种 Verticle：

* **Stardand Verticle**：这是最常用的一类 Verticle —— 它们永远运行在 Event Loop 线程上。更多讨论详见稍后的章节。
* **Worker Verticle**：这类 Verticle 会运行在 Worker Pool 中的线程上。一个实例绝对不会被多个线程同时执行。

### Standard Verticle

当 Standard Verticle 被创建时，它会被分派给一个 Event Loop 线程，并在这个 Event Loop 中执行它的 `start` 方法。当您在一个 Event Loop 上调用了 Core API 中的方法并传入了处理器时，Vert.x 将保证用与调用该方法时相同的 Event Loop 来执行这些处理器。

这意味着我们可以保证您的 Verticle 实例中 **所有的代码都是在相同Event Loop中执行**（只要您不创建自己的线程来调用它！）

同样意味着您可以将您的应用中的所有代码用单线程方式编写，让 Vert.x 去考虑线程和扩展问题。您不用再考虑 synchronized 和 volatile 的问题，也可以避免传统的多线程应用经常会遇到的竞态条件和死锁的问题。

### Worker Verticle

Worker Verticle 和 Standard Verticle 很像，但它并不是由一个 Event Loop 来执行，而是由Vert.x中的 Worker Pool 中的线程执行。

Worker Verticle 设计用于调用阻塞式代码，它不会阻塞任何 Event Loop。

如果您不想使用 Worker Verticle 来运行阻塞式代码，您还可以在一个Event Loop中直接使用 [内联阻塞式代码](#运行阻塞式代码)。

您需要通过 [`setWorker`](https://vertx.io/docs/apidocs/io/vertx/core/DeploymentOptions.html#setWorker-boolean-) 方法来将 Verticle 部署成一个 Worker Verticle：

```java
DeploymentOptions options = new DeploymentOptions().setWorker(true);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

在 Vert.x 中，Worker Verticle 实例绝对不会同时被多个线程执行，但它可以在不同时间由不同线程执行。

### 编程方式部署Verticle

部署Verticle可以使用任意一个 [`deployVerticle`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#deployVerticle-io.vertx.core.Verticle-) 方法，并传入一个 Verticle 名称或Verticle 实例。

> 请注意：通过 Verticle **实例** 来部署 Verticle 仅限Java语言。

```java
Verticle myVerticle = new MyVerticle();
vertx.deployVerticle(myVerticle);
```

您同样可以指定 Verticle 的 **名称** 来部署它。这个 Verticle 的名称会用于查找实例化 Verticle 的特定 [`VerticleFactory`](https://vertx.io/docs/apidocs/io/vertx/core/spi/VerticleFactory.html)。

不同的 Verticle Factory 可用于实例化不同语言的 Verticle，也可用于其他目的，例如加载服务、运行时从Maven中获取Verticle实例等。因此可以部署任何使用Vert.x支持的语言编写的Verticle。

下面的例子展示了如何部署多个不同语言编写的 Verticle ：

```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle");

// 部署JavaScript的Verticle
vertx.deployVerticle("verticles/myverticle.js");

// 部署Ruby的Verticle
vertx.deployVerticle("verticles/my_verticle.rb");
```

### Verticle名称到Factory的映射规则

使用名称部署Verticle时，会通过名称来选择一个用于实例化 Verticle 的 Verticle Factory。

Verticle 名称可以增加一个以冒号结尾的前缀，这个前缀用于查找Factory，如：

```
js:foo.js // 使用JavaScript的Factory
groovy:com.mycompany.SomeGroovyCompiledVerticle // 用Groovy的Factory
service:com.mycompany:myorderservice // 用Service的Factory
```

如果不指定前缀，Vert.x将根据Verticle名称的后缀来查找对应Factory，如：

```
foo.js // 将使用JavaScript的Factory
SomeScript.groovy // 将使用Groovy的Factory
```

若前缀后缀都没指定，Vert.x将假定Verticle名称是一个Java 全限定类名（FQCN），并尝试实例化它。

### 如何定位Verticle Factory？

大部分Verticle Factory会从 classpath 中加载，并在 Vert.x 启动时注册。

您同样可以使用编程的方式去注册或注销Verticle Factory：通过 [`registerVerticleFactory`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#registerVerticleFactory-io.vertx.core.spi.VerticleFactory-) 方法和 [`unregisterVerticleFactory`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#unregisterVerticleFactory-io.vertx.core.spi.VerticleFactory-) 方法。

### 等待部署完成

Verticle是异步部署的，换而言之，可能在 `deploy` 方法调用返回后一段时间才会完成部署。

如果您想要在部署完成时收到通知，则可以指定一个完成处理器：

```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", res -> {
  if (res.succeeded()) {
    System.out.println("Deployment id is: " + res.result());
  } else {
    System.out.println("Deployment failed!");
  }
});
```

如果部署成功，这个完成处理器的结果中将会包含部署ID的字符串。这个部署ID可以用于撤销部署。

### 撤销Verticle

我们可以通过 [`undeploy`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#undeploy-java.lang.String-) 方法来撤销部署好的 Verticle。

撤销操作也是异步的，因此若您想要在撤销完成后收到通知，则可以指定另一个完成处理器：

```java
vertx.undeploy(deploymentID, res -> {
  if (res.succeeded()) {
    System.out.println("Undeployed ok");
  } else {
    System.out.println("Undeploy failed!");
  }
});
```

### 设置 Verticle 实例数量

使用名称部署 Verticle 时，可以指定需要部署的 Verticle 实例的数量。

```java
DeploymentOptions options = new DeploymentOptions().setInstances(16);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

这个功能对于跨多核扩展时很有用。例如，您有一个带Web服务的Verticle需要部署在多核的机器上，您可以部署多个实例来利用所有的核。

### 向 Verticle 传入配置

可在部署时传给 Verticle 一个 JSON 格式的配置

```java
JsonObject config = new JsonObject().put("name", "tim").put("directory", "/blah");
DeploymentOptions options = new DeploymentOptions().setConfig(config);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

传入之后，这个配置可以通过 [`Context`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html) 对象或使用 [`config`](https://vertx.io/docs/apidocs/io/vertx/core/AbstractVerticle.html#config--) 方法访问。这个配置会以 JSON 对象（`JsonObject`）的形式返回，因此您可以用下边代码读取数据：

```java
System.out.println("Configuration: " + config().getString("name"));
```

### 在 Verticle 中访问环境变量

环境变量和系统属性可以直接通过 Java API 访问：

```java
System.getProperty("prop");
System.getenv("HOME");
```

### 高可用性

Verticle可以启用高可用方式（HA）部署。在这种方式下，当其中一个部署在 Vert.x 实例中的 Verticle 突然挂掉，这个 Verticle 可以在集群环境中的另一个 Vert.x 实例中重新部署。

若要启用高可用方式运行一个 Verticle，仅需要追加 `-ha` 参数：

```
vertx run my-verticle.js -ha
```

当启用高可用方式时，不需要追加 `-cluster` 参数。

关于高可用的功能和配置的更多细节可参考 [高可用和故障转移](#高可用和故障转移) 章节。

### 从命令行运行Verticle

您可以在 Maven 或 Gradle 项目中以正常方式添加 Vert.x Core 为依赖，在项目中直接使用 Vert.x。

您也可以从命令行直接运行 Vert.x 的 Verticle。

为此，您需要下载并安装 Vert.x 的发行版，并且将安装的 `bin` 目录添加到您的 `PATH` 环境变量中，并确保您的 `PATH` 中设置了Java 8的JDK环境。

> 请注意：* 在`PATH`设置JDK是为了支持Java代码的运行时编译（on the fly compilation）。*

现在您可以使用 `vertx run` 命令运行Verticle了，这儿是一些例子：

```
# 运行JavaScript的Verticle
vertx run my_verticle.js

# 运行Ruby的Verticle
vertx run a_n_other_verticle.rb

# 使用集群模式运行Groovy的Verticle
vertx run FooVerticle.groovy -cluster
```

您甚至可以不必编译 Java 源代码，直接运行它：

```
vertx run SomeJavaSourceFile.java
```

Vert.x 在运行Java 源代码文件之前将执行运行时编译，这对于快速原型制作和演示很有用，而且不需要配置 Maven 或 Gradle 就能跑起来！

欲了解有关在命令行执行 `vertx` 可用的各种选项完整信息，可以直接在命令行键入 `vertx` 查看帮助。

### 退出 Vert.x 环境

Vert.x 实例维护的线程不是守护线程，因此它们会阻止JVM退出。

如果您通过嵌入式的方式使用 Vert.x 并且完成了操作，您可以调用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#close--) 方法关闭它。这将关闭所有内部线程池并关闭其他资源，允许JVM退出。

### Context 对象

当 Vert.x 传递一个事件给处理器或者调用 [`Verticle`](https://vertx.io/docs/apidocs/io/vertx/core/Verticle.html) 的 `start` 或 `stop` 方法时，它会关联一个 `Context` 对象来执行。通常来说这个 `Context` 会是一个 **Event Loop Context**，它绑定到了一个特定的 Event Loop 线程上。所以在该 `Context` 上执行的操作总是在同一个 Event Loop 线程中。对于运行内联的阻塞代码的 Worker Verticle 来说，会关联一个 Worker Context，并且所有的操作运都会运行在 Worker 线程池的线程上。

> 译者注：每个 `Verticle` 在部署的时候都会被分配一个 `Context`（根据配置不同，可以是Event Loop Context 或者 Worker Context），之后此 `Verticle` 上所有的普通代码都会在此 `Context` 上执行（即对应的 Event Loop 或Worker 线程）。一个 `Context` 对应一个 Event Loop 线程（或 Worker 线程），但一个 Event Loop 可能对应多个 `Context`。

您可以通过 [`getOrCreateContext`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#getOrCreateContext--) 方法获取 `Context` 实例：

```java
Context context = vertx.getOrCreateContext();
```

若已经有一个 `Context` 和当前线程关联，那么它直接重用这个 `Context` 对象，如果没有则创建一个新的。您可以检查获取的 `Context` 的类型：

```java
Context context = vertx.getOrCreateContext();
if (context.isEventLoopContext()) {
  System.out.println("Context attached to Event Loop");
} else if (context.isWorkerContext()) {
  System.out.println("Context attached to Worker Thread");
} else if (! Context.isOnVertxThread()) {
  System.out.println("Context not attached to a thread managed by vert.x");
}
```

当您获取了这个 `Context` 对象，您就可以在 `Context` 中异步执行代码了。换句话说，您提交的任务将会在同一个 `Context` 中运行：

```java
vertx.getOrCreateContext().runOnContext(v -> {
  System.out.println("This will be executed asynchronously in the same context");
});
```

当在同一个 `Context` 中运行了多个处理函数时，可能需要在它们之间共享数据。 `Context` 对象提供了存储和读取共享数据的方法。举例来说，它允许您将数据传递到 [`runOnContext`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html#runOnContext-io.vertx.core.Handler-) 方法运行的某些操作中：

```java
final Context context = vertx.getOrCreateContext();
context.put("data", "hello");
context.runOnContext((v) -> {
  String hello = context.get("data");
});
```

您还可以通过 [`config`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html#config--) 方法访问 Verticle 的配置信息。查看 [向 Verticle 传入配置](#向-verticle-传入配置) 章节了解更多配置信息。

### 执行周期性/延迟性操作

在 Vert.x 中，延迟执行或定期执行操作很常见。

在 Standard Verticle 中您不能直接让线程休眠以引入延迟，因为它会阻塞 Event Loop 线程。取而代之是使用 Vert.x 定时器。定时器可以是一次性或周期性的，两者我们都会讨论到。

#### 一次性计时器

一次性计时器会在一定延迟后调用一个 Event Handler，以毫秒为单位计时。

您可以通过 [`setTimer`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setTimer-long-io.vertx.core.Handler-) 方法传递延迟时间和一个处理器来设置计时器的触发。

```java
long timerID = vertx.setTimer(1000, id -> {
  System.out.println("And one second later this is printed");
});

System.out.println("First this is printed");
```

返回值是一个唯一的计时器id，该id可用于之后取消该计时器，这个计时器id会传入给处理器。

#### 周期性计时器

您同样可以使用 [`setPeriodic`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setPeriodic-long-io.vertx.core.Handler-) 方法设置一个周期性触发的计时器。第一次触发之前同样会有一段设置的延时时间。

`setPeriodic` 方法的返回值也是一个唯一的计时器id，若之后该计时器需要取消则使用该id。传给处理器的参数也是这个唯一的计时器id。

请记住这个计时器将会定期触发。如果您的定时任务会花费大量的时间，则您的计时器事件可能会连续执行甚至发生更坏的情况：重叠。这种情况，您应考虑使用 [`setTimer`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setTimer-long-io.vertx.core.Handler-) 方法，当任务执行完成时设置下一个计时器。

```java
long timerID = vertx.setPeriodic(1000, id -> {
  System.out.println("And every second this is printed");
});

System.out.println("First this is printed");
```

#### 取消计时器

指定一个计时器id并调用 [`cancelTimer`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#cancelTimer-long-) 方法来取消一个周期性计时器。如：

```java
vertx.cancelTimer(timerID);
```

#### Verticle 中自动清除定时器

如果您在 Verticle 中创建了计时器，当这个 Verticle 被撤销时这个计时器会被自动关闭。

### Verticle Worker Pool

Verticle 使用 Vert.x 中的 Worker Pool 来执行阻塞式行为，例如 [`executeBlocking`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-) 或 Worker Verticle。

可以在部署配置项中指定不同的 Worker 线程池：

```java
vertx.deployVerticle("the-verticle", new DeploymentOptions().setWorkerPoolName("the-specific-pool"));
```

---

## Event Bus

Event Bus 是 Vert.x 的神经系统。

每一个 Vert.x 实例都有一个单独的 Event Bus 实例。您可以通过 `Vertx` 实例的 [`eventBus`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#eventBus--) 方法来获得对应的 `EventBus` 实例。

应用中的不同组成部分可以通过 Event Bus 相互通信，您无需关心它们由哪一种语言实现，也无需关心它们是否在同一个 Vert.x 实例中。

您甚至可以通过桥接的方式让浏览器中运行的多个JavaScript客户端在同一个 Event Bus 上相互通信。

Event Bus构建了一个跨越多个服务器节点和多个浏览器的分布式点对点消息系统。

Event Bus支持发布/订阅、点对点、请求-响应的消息传递方式。

Event Bus的API很简单。基本上只涉及注册处理器、注销处理器以及发送和发布(publish)消息。

先来看一些基本概念和理论。

### 基本概念

#### 寻址

消息的发送目标被称作 **地址(address)**。

Vert.x的地址格式并不花哨。Vert.x中的地址就是一个简单的字符串，任何字符串都合法。不过还是建议使用某种规范来进行地址的命名。例如使用点号(`.`)来划分命名空间。

一些合法的地址形如：`europe.news.feed1`、`acme.games.pacman`、`sausages`以及`X`。

#### 处理器

消息需由处理器（`Handler`）来接收。您需要将处理器注册在某个地址上。

同一个地址可以注册许多不同的处理器。

一个处理器也可以注册在多个不同的地址上。

#### 发布/订阅消息

Event Bus支持 **发布(publish)消息** 功能。

消息将被发布到一个地址上。发布意味着信息会被传递给所有注册在该地址上的处理器。

即我们熟悉的 **发布/订阅** 消息传递模式。

#### 点对点消息传递 与 请求-响应消息传递

Event Bus也支持 **点对点** 消息模式。

消息将被发送到一个地址上，Vert.x仅会把消息发给注册在该地址上的处理器中的其中一个。

若这个地址上注册有不止一个处理器，那么Vert.x将使用 **不严格的轮询算法** 选择其中一个。

点对点消息传递模式下，可在消息发送的时候指定一个应答处理器（可选）。

当接收者收到消息并且处理完成时，它可以选择性地回复该消息。若回复，则关联的应答处理器将会被调用。

当发送者收到应答消息时，发送者还可以继续回复这个“应答”，这个过程可以不断重复。通过这种方式可以在两个不同的 Verticle 之间建立一个对话窗口。

这也是一个常见的消息传递模式：**请求-响应** 模式。

#### 尽力传输

Vert.x会尽它最大努力去传递消息，并且不会主动丢弃消息。这种方式称为 **尽力传输(Best-effort delivery)**。

但是，当 Event Bus 发生故障时，消息可能会丢失。

若您的应用关心消息丢失，那么您应当编写具有幂等性的处理器，并且您的发送者应当在故障恢复后重试。

> 译者注：RPC通信通常情况下有三种语义：**at least once**、**at most once** 和 **exactly once**。不同语义情况下要考虑的情况不同。

> 本小节中文档建议开发者通过重试来实现at least once语义，并通过幂等设计来规避重复接收消息的影响。  

#### 消息类型

Vert.x 默认允许任何基本/简单类型、`String` 类型、 [`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html) 类型的值作为消息发送。

不过在 Vert.x 中更规范且更通用的做法是使用 [JSON](https://json.org/) 格式来发送消息。

对于 Vert.x 支持的所有语言来说，JSON都是非常容易创建、读取和解析的，因此JSON已经成为了Vert.x中的通用语(*lingua franca*)。

但是若您不想用 JSON，我们也不强制您使用它。

Event Bus 非常灵活，您可以通过自定义 [`codec`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageCodec.html) 来实现任何类型对象在 Event Bus 上的传输。

### Event Bus API

下面我们来看一下 API。

#### 获取Event Bus

您可以通过下面的代码获取 Event Bus 的引用：

```java
EventBus eb = vertx.eventBus();
```

每一个 Vertx.x 实例仅有一个 Event Bus 实例。

#### 注册处理器

最简单的注册处理器的方式是使用 [consumer](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#consumer-java.lang.String-io.vertx.core.Handler-) 方法，这儿有个例子：

```java
EventBus eb = vertx.eventBus();

eb.consumer("news.uk.sport", message -> {
  System.out.println("I have received a message: " + message.body());
});
```

当消息达到您的处理器时，该消息会被放入 [`message`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html) 参数进行处理器的调用。

调用 `consumer` 方法会返回一个 [`MessageConsumer`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html) 对象。

该对象后续可用于注销处理器，或者流式地处理该对象。

您也可以使用 [`consumer`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#consumer-java.lang.String-io.vertx.core.Handler-) 方法直接返回一个不带处理器的 `MessageConsumer`，之后再在这个返回的对象上设置处理器。如：

```java
EventBus eb = vertx.eventBus();

MessageConsumer<String> consumer = eb.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
});
```

在向集群模式下的 Event Bus 注册处理器时，注册信息会花费一些时间才能传播到集群中的所有节点。

若您希望在完成注册后收到通知，您可以在 `MessageConsumer` 对象上注册一个 [`completion handler`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html#completionHandler-io.vertx.core.Handler-)。

```java
consumer.completionHandler(res -> {
  if (res.succeeded()) {
    System.out.println("The handler registration has reached all nodes");
  } else {
    System.out.println("Registration failed!");
  }
});
```

#### 注销处理器

您可以通过 [`unregister()`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html#unregister--) 方法来注销处理器。

若您在使用集群模式的 Event Bus，注销处理器的动作会花费一些时间在节点中传播。若您想在完成后收到通知，可以使用[`unregister(handler)`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html#unregister-io.vertx.core.Handler-) 方法注册回调：

```java
consumer.unregister(res -> {
  if (res.succeeded()) {
    System.out.println("The handler un-registration has reached all nodes");
  } else {
    System.out.println("Un-registration failed!");
  }
});
```

#### 发布消息

发布消息很简单，只需使用 [`publish`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#publish-java.lang.String-java.lang.Object-) 方法指定一个地址去发布即可。

```java
eventBus.publish("news.uk.sport", "Yay! Someone kicked a ball");
```

这个消息将会传递给所有在地址 `news.uk.sport` 上注册过的处理器。

#### 发送消息

在对应地址上注册过的所有处理器中，仅一个处理器能够接收到发送的消息。这是一种点对点消息传递模式。Vert.x 使用不严格的轮询算法来选择绑定的处理器。

您可以使用 [`send`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#send-java.lang.String-java.lang.Object-) 方法来发送消息：

```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball");
```

#### 设置消息头

在 Event Bus 上发送的消息可包含头信息。您可以在发送或发布(publish)时提供一个 [`DeliveryOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html) 来指定头信息。例如：

```java
DeliveryOptions options = new DeliveryOptions();
options.addHeader("some-header", "some-value");
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball", options);
```

#### 消息顺序

Vert.x会按照发送者发送消息的顺序，将消息以同样的顺序传递给处理器。

#### 消息对象

您在消息处理器中接收到的对象的类型是 [`Message`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html)。

消息的 [`body`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#body--) 对应发送或发布(publish)的对象。

消息的头信息可以通过 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#headers--) 方法获取。

#### 应答消息/发送回复

当使用 [`send`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#send-java.lang.String-java.lang.Object-) 方法发送消息时，Event Bus会尝试将消息传递到注册在Event Bus上的 [`MessageConsumer`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html)中。

某些情况下，发送者可以通过 `请求/响应` 模式来得知消费者已经收到并 *处理* 了该消息。

消费者可以通过调用 [`reply`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#reply-java.lang.Object-) 方法来应答这个消息，确认该消息已被处理。

此时，它会将一个应答消息返回给发送者并调用发送者的应答处理器。

看这个例子会更清楚：

接收者：

```java
MessageConsumer<String> consumer = eventBus.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
  message.reply("how interesting!");
});
```

发送者：

```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball across a patch of grass", ar -> {
  if (ar.succeeded()) {
    System.out.println("Received reply: " + ar.result().body());
  }
});
```

在应答的消息体中可以包含一些有用的信息。

“处理中”的实际含义应当由应用程序来定义。这完全取决于消费者如何执行，Event Bus 对此并不关心。

一些例子：

* 一个简单地实现了返回当天时间的服务，在应答的消息里会包含当天时间信息。
* 一个实现了持久化队列的消息消费者，可以回复`true`来表示消息已成功持久化到存储设备中，或回复`false`表示失败。
* 一个处理订单的消息消费者可以使用`true`确认这个订单已经成功处理并且可以从数据库中删除。

#### 带超时的发送

当发送带有应答处理器的消息时，可以在 [`DeliveryOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html) 中指定一个超时时间。

如果在这个时间之内没有收到应答，则会以“失败的结果”为参数调用应答处理器。

默认超时是 **30 秒**。

#### 发送失败

消息发送可能会因为其他原因失败，包括：

* 没有可用的处理器来接收消息
* 接收者调用了 [`fail`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#fail-int-java.lang.String-) 方法显式声明失败

发生这些情况时，应答处理器将会以这些异常失败结果为参数进行调用。

#### 消息编解码器

您可以在 Event Bus 中发送任何对象，只需为这个对象类型注册一个编解码器 [`message codec`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageCodec.html) 即可。

每个消息编解码器都有一个名称，您需要在发送或发布消息时通过 [`DeliveryOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html) 来指定：

```java
eventBus.registerCodec(myCodec);

DeliveryOptions options = new DeliveryOptions().setCodecName(myCodec.name());

eventBus.send("orders", new MyPOJO(), options);
```

若您希望某个类总是使用特定的编解码器，那么您可以为这个类注册默认编解码器。这样您就不需要在每次发送的时候使用 [`DeliveryOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html) 来指定了：

```java
eventBus.registerDefaultCodec(MyPOJO.class, myCodec);

eventBus.send("orders", new MyPOJO());
```

您可以通过 [`unregisterCodec`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#unregisterCodec-java.lang.String-) 方法注销某个消息编解码器。

消息编解码器的编码输入和解码输出不一定使用同一个类型。例如您可以编写一个编解码器来发送 MyPOJO 类的对象，但是当消息发送给处理器后解码成 MyOtherPOJO 对象。

#### 集群模式的 Event Bus

Event Bus 不仅仅只存在于单个 Vert.x 实例中。将网络上不同的 Vert.x 实例组合成集群，就可以在这些实例间形成一个单一的、分布式的Event Bus。

#### 通过代码的方式启用集群模式

若您用编程的方式创建 Vert.x 实例（`Vertx`），则可以通过将 Vert.x 实例配置成集群模式来获取集群模式的Event Bus：

```java
VertxOptions options = new VertxOptions();
Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

您需要确保在您的 classpath 中（或构建工具的依赖中）包含 [`ClusterManager`](https://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html) 的实现类，如默认的 `HazelcastClusterManager`。

#### 通过命令行启用集群模式

您可以通过以下命令以集群模式运行 Vert.x 应用：

```
vertx run my-verticle.js -cluster
```

### Verticle 中的自动清理

若您在 Verticle 中注册了 Event Bus 的处理器，那么这些处理器在 Verticle 被撤销（undeploy）的时候会自动被注销。

### 配置 Event Bus

Event Bus 是可配置的，这对于以集群模式运行的 Event Bus 来说非常有用。Event Bus 使用 TCP 连接发送和接收消息，因此可以通过 [`EventBusOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBusOptions.html) 对TCP连接进行全面的配置。由于 Event Bus 既可以用作客户端又可以用作服务端，因此这些配置近似于 [`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html) 和 [`NetServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)。

```java
VertxOptions options = new VertxOptions()
    .setEventBusOptions(new EventBusOptions()
        .setSsl(true)
        .setKeyStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
        .setTrustStoreOptions(new JksOptions().setPath("keystore.jks").setPassword("wibble"))
        .setClientAuth(ClientAuth.REQUIRED)
    );

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

上边代码段描述了如何在Event Bus中使用SSL连接替换明文的TCP连接。

> **警告：** 若要在集群模式下保证安全性，您 **必须** 将集群管理器配置成加密的或者加强安全规则。参考集群管理器的文档获取更多细节。

Event Bus 的配置需要在集群的所有节点中保持一致。

[`EventBusOptions`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBusOptions.html)还允许您指定 Event Bus 是否运行在集群模式下，以及它的端口和主机信息（译者注：host，这里指网络socket绑定的地址）。

在容器中使用时，您还可以配置公共主机和端口号：

```java
VertxOptions options = new VertxOptions()
    .setEventBusOptions(new EventBusOptions()
        .setClusterPublicHost("whatever")
        .setClusterPublicPort(1234)
    );

Vertx.clusteredVertx(options, res -> {
  if (res.succeeded()) {
    Vertx vertx = res.result();
    EventBus eventBus = vertx.eventBus();
    System.out.println("We now have a clustered event bus: " + eventBus);
  } else {
    System.out.println("Failed: " + res.cause());
  }
});
```

> 译者注：setClusterPublicHost 和 setClusterPublicPort 的功能在原文档上描述得不清晰，但是API文档上有详细描述。  
> 在某些容器、云环境等场景下，本节点监听的地址，和其他节点连接本节点时使用的地址，是不同的。这种情况下则可以利用上面两个配置区分监听地址和公开暴露的地址。  

## JSON

和其他一些语言不同，Java 没有对 JSON 做原生支持（first class support），因此我们提供了两个类，以便在 Vert.x 应用中更方便地处理 JSON。

### JSON 对象

[`JsonObject`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html) 类用来描述JSON对象。

一个JSON 对象基本上只是一个 Map 结构。它具有字符串的键，值可以是任意一种JSON 支持的类型（如 string, number, boolean）。

JSON 对象也支持 `null` 值。

#### 创建 JSON 对象

可以使用默认构造函数创建空的JSON对象。

您可以通过一个 JSON 格式的字符串创建JSON对象：

```java
String jsonString = "{\"foo\":\"bar\"}";
JsonObject object = new JsonObject(jsonString);
```

您可以根据Map创建JSON对象：

```java
Map<String, Object> map = new HashMap<>();
map.put("foo", "bar");
map.put("xyz", 3);
JsonObject object = new JsonObject(map);
```

#### 将键值对放入 JSON 对象

使用[`put`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html#put-java.lang.String-java.lang.Enum-) 方法可以将值放入到JSON对象里。

这个API是流式的，因此这个方法可以被链式地调用。

```java
JsonObject object = new JsonObject();
object.put("foo", "bar").put("num", 123).put("mybool", true);
```

#### 从 JSON 对象获取值

您可使用 `getXXX` 方法从JSON对象中获取值。例如：

```java
String val = jsonObject.getString("some-key");
int intVal = jsonObject.getInteger("some-other-key");
```

#### JSON 对象和 Java 对象间的映射

您可以根据 Java 对象的字段创建一个JSON 对象，如下所示：

```java
// TODO
```

你可以根据一个 JSON 对象来实例化一个Java 对象并填充字段值。如下所示：

```java
request.bodyHandler(buff -> {
  JsonObject jsonObject = buff.toJsonObject();
  User javaObject = jsonObject.mapTo(User.class);
});
```

请注意上述代码直接使用了 Jackson 的 `ObjectMapper#convertValue()` 来执行映射。关于字段和构造函数的可见性的影响、对象引用的序列化和反序列化的问题等等可参考 Jackson 的文档获取更多信息。

在最简单的情况下，如果 Java 类中所有的字段都是 `public`（或者有 `public` 的 getter/setter）时，并且有一个 `public` 的默认构造函数（或不定义构造函数），`mapFrom` 和 `mapTo` 都应该成功。

只要不存在对象的循环引用，嵌套的 Java 对象就可以和嵌套的 JSON 对象相互序列化/反序列化。

#### 将 JSON 对象编码成字符串

您可使用 [`encode`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html#encode--) 方法将一个对象编码成字符串格式。

> 译者注：如要得到更优美、格式化的字符串，可以使用 [`encodePrettily`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html#encodePrettily--) 方法。

### JSON 数组

[`JsonArray`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html) 类用来描述 JSON数组。

一个JSON 数组是一个值的序列（值的类型可以是 string、number、boolean 等）。

JSON 数组同样可以包含 `null` 值。

#### 创建 JSON 数组

可以使用默认构造函数创建空的JSON数组。

您可以根据JSON格式的字符串创建一个JSON数组：

```java
String jsonString = "[\"foo\",\"bar\"]";
JsonArray array = new JsonArray(jsonString);
```

#### 将数组项添加到JSON数组

您可以使用 [`add`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html#add-java.lang.Enum-) 方法添加数组项到JSON数组中：

```java
JsonArray array = new JsonArray();
array.add("foo").add(123).add(false);
```

#### 从 JSON 数组中获取值

您可使用 `getXXX` 方法从JSON 数组中获取值。例如：

```java
String val = array.getString(0);
Integer intVal = array.getInteger(1);
Boolean boolVal = array.getBoolean(2);
```

#### 将 JSON 数组编码成字符串

您可以使用 [`encode`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html#encode--) 将一个 `JsonArray` 编码成字符串格式。

#### 创建任意类型的 JSON

创建 JSON 对象或数组的前提是，你需要事先已知其输入是合法的字符串。

> 译者注：这里说的“合法”指的是，你在使用`JsonObject`时，需要事先知道构造函数输入的字符串是一个json object `{...}`，同理，使用`JsonArray`时，字符串需要是一个json array `[...]`，否则即使输入了一个规范的Json字符串，也没有办法成功解析。  

当你不确定字符串是否合法时，你应当转而使用 [Json.decodeValue](https://vertx.io/docs/apidocs/io/vertx/core/json/Json.html#decodeValue-java.lang.String-) 方法。

```java
Object object = Json.decodeValue(arbitraryJson);
if (object instanceof JsonObject) {
  // 是一个合法的json对象
} else if (object instanceof JsonArray) {
  // 是一个合法的json数组
} else if (object instanceof String) {
  // 是一个合法的字符串
} else {
  // 以此类推...
}
```

## Json 指针（Json Pointers）

Vert.x 提供了一个 [Json指针 RFC6901](https://tools.ietf.org/html/rfc6901) 的实现。无论是查询还是写入，你都可以使用Json指针来完成。你可以基于字符串、URI，或者通过手动追加路径(path)的方式，来构建 [JsonPointer](https://vertx.io/docs/apidocs/io/vertx/core/json/pointer/JsonPointer.html) 对象：

```java
JsonPointer pointer1 = JsonPointer.from("/hello/world");
// 手动构造一个Json指针
JsonPointer pointer2 = JsonPointer.create()
  .append("hello")
  .append("world");
```

在初始化Json指针后，你可以使用 [queryJson](https://vertx.io/docs/apidocs/io/vertx/core/json/pointer/JsonPointer.html#queryJson-java.lang.Object-) 方法做查询，也可以使用 [writeJson](https://vertx.io/docs/apidocs/io/vertx/core/json/pointer/JsonPointer.html#writeJson-java.lang.Object-java.lang.Object-) 方法修改JSON的值。

```java
// 查询JsonObject
Object result1 = objectPointer.queryJson(jsonObject);
// 查询JsonArray
Object result2 = arrayPointer.queryJson(jsonArray);
// 从开头写入JsonObject
objectPointer.writeJson(jsonObject, "new element");
// 从开头写入JsonArray
arrayPointer.writeJson(jsonArray, "new element");
```

你可以将Vert.x的Json指针功能应用在任何类型的对象上，只需实现一个自定义的 [JsonPointerIterator](https://vertx.io/docs/apidocs/io/vertx/core/json/pointer/JsonPointerIterator.html) 即可。

## Buffer

在 Vert.x 内部，大部分数据被重新组织（shuffle，表意为洗牌）成 `Buffer` 格式。

`Buffer` 是一个可以被读取或写入的，包含0个或多个字节的序列，并且能够根据写入的字节自动扩容。您也可以将 `Buffer` 想象成一个智能的字节数组。

### 创建 Buffer

可以使用静态方法 [`Buffer.buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#buffer--) 来创建 `Buffer`。

`Buffer` 可以从字符串或字节数组初始化，或者直接创建空的 `Buffer`。

这儿有一些创建 `Buffer` 的例子。

创建一个空的 `Buffer`：

```java
Buffer buff = Buffer.buffer();
```

从字符串创建一个 `Buffer`，这个 `Buffer` 中的字符会以 UTF-8 格式编码：

```java
Buffer buff = Buffer.buffer("some string");
```

从字符串创建一个 `Buffer`，这个字符串会以指定的编码方式编码，例如：

```java
Buffer buff = Buffer.buffer("some string", "UTF-16");
```

从字节数组 `byte[]` 创建 `Buffer`：

```java
byte[] bytes = new byte[] {1, 3, 5};
Buffer buff = Buffer.buffer(bytes);
```

创建一个指定初始大小的 `Buffer`。若您知道您的 `Buffer` 会写入一定量的数据，您可以在创建 `Buffer` 时指定它的大小，使这个 `Buffer` 在初始化时就分配了更多的内存，比数据写入时重新调整大小效率更高。

注意以这种方式创建的 `Buffer` 是 **空的**。它不会创建一个填满了 0 的Buffer。代码如下：

```java
Buffer buff = Buffer.buffer(10000);
```

> 译者注：这里说的“空的”、“不会填满0”，指的是buffer内部的游标会从头开始，并不是在说内存布局。这种实现方式和使用直觉是一致的，只不过明确通过文档进行描述有点奇怪。

### 向Buffer写入数据

向 `Buffer` 写入数据的方式有两种：追加和随机访问。任何一种情况下 `Buffer` 都会自动进行扩容，所以你不会在使用 `Buffer` 时遇到 `IndexOutOfBoundsException`。

#### 追加到Buffer

您可以使用 `appendXXX` 方法追加数据到 `Buffer`。`Buffer` 类提供了追加各种不同类型数据的追加写入方法。

因为 `appendXXX` 方法的返回值就是 Buffer 自身，所以它可以链式地调用:

```java
Buffer buff = Buffer.buffer();

buff.appendInt(123).appendString("hello\n");

socket.write(buff);
```

#### 随机访问写Buffer

您还可以指定一个索引值，通过 `setXXX` 方法写入数据到 `Buffer`。`setXXX` 也为各种不同数据类型提供了对应的方法。所有的 set 方法都会将索引值作为第一个参数 —— 这表示 `Buffer` 中开始写入数据的位置。

`Buffer` 始终根据需要进行自动扩容。

```java
Buffer buff = Buffer.buffer();

buff.setInt(1000, 123);
buff.setString(0, "hello");
```

### 从Buffer中读取

可使用 `getXXX` 方法从 Buffer 中读取数据，`getXXX` 为各种不同数据类型提供了对应的方法，这些方法的第一个参数是 `Buffer` 中待获取的数据的索引位置。

```java
Buffer buff = Buffer.buffer();
for (int i = 0; i < buff.length(); i += 4) {
  System.out.println("int value at " + i + " is " + buff.getInt(i));
}
```

### 使用无符号数

可使用 `getUnsignedXXX`、`appendUnsignedXXX` 和 `setUnsignedXXX` 方法将无符号数从 `Buffer` 中读取或追加/设置到 `Buffer` 里。这对于实现一个致力于优化带宽占用的网络协议的编解码器是非常有用的。

下边例子中，值 200 被设置到了仅占用一个字节的特定位置：

```java
Buffer buff = Buffer.buffer(128);
int pos = 15;
buff.setUnsignedByte(pos, (short) 200);
System.out.println(buff.getUnsignedByte(pos));
```

控制台中显示 `200`。

### Buffer长度

可使用 [`length`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#length--) 方法获取Buffer长度，Buffer的长度值是Buffer中包含的字节的最大索引 + 1。

#### 拷贝Buffer

可使用 [`copy`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#copy--) 方法创建一个Buffer的副本。

#### 裁剪Buffer

裁剪得到的Buffer是完全依赖于原始Buffer的一个新的Buffer，换句话说，它不会对 `Buffer` 中的数据做拷贝。使用 [`slice`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#slice--) 方法裁剪一个Buffer。

#### Buffer 重用

将Buffer写入到一个Socket或其他类似位置后，Buffer就不可被重用了。

## 编写 TCP 服务端和客户端

Vert.x允许您很容易编写非阻塞的TCP客户端和服务器。

### 创建 TCP 服务端

最简单地使用所有默认配置项创建 TCP 服务端的方式如下：

```java
NetServer server = vertx.createNetServer();
```

### 配置 TCP 服务端

若您不想使用默认配置，可以在创建时通过传入一个 [`NetServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html) 实例来配置服务器：

```java
NetServerOptions options = new NetServerOptions().setPort(4321);
NetServer server = vertx.createNetServer(options);
```

### 启动服务端监听

要告诉服务端监听传入的请求，您可以使用其中一个 [`listen`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#listen--) 方法。

让服务器监听配置项指定的主机和端口：

```java
NetServer server = vertx.createNetServer();
server.listen();
```

或在调用 `listen` 方法时指定主机和端口号，忽略配置项中的配置：

```java
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost");
```

默认主机名是 `0.0.0.0`，它表示：监听所有可用地址。默认端口号是 `0`，这也是一个特殊值，它告诉服务器随机选择并监听一个本地没有被占用的端口。

实际的绑定也是异步的，因此服务器在调用了 `listen` 方法的一段时间之后才会实际开始监听。若您希望在服务器实际监听时收到通知，您可以在调用 `listen` 方法时提供一个处理器。例如：

```java
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening!");
  } else {
    System.out.println("Failed to bind!");
  }
});
```

### 监听随机端口

若设置监听端口为`0`，服务器将随机寻找一个没有使用的端口来监听。

可以调用 [`actualPort`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#actualPort--) 方法来获得服务器实际监听的端口：

```java
NetServer server = vertx.createNetServer();
server.listen(0, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening on actual port: " + server.actualPort());
  } else {
    System.out.println("Failed to bind!");
  }
});
```

### 接收传入连接的通知

若您想要在连接创建完时收到通知，则需要设置一个 [`connectHandler`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#connectHandler-io.vertx.core.Handler-)：

```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  // 在这里处理传入连接
});
```

当连接成功时，您可以在回调函数中处理得到的 [`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 实例。这是一个代表了实际连接的套接字接口，它允许您读取和写入数据、以及执行各种其他操作，如关闭 Socket。

### 从Socket读取数据

您可以在Socket上调用 [`handler`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#handler-io.vertx.core.Handler-) 方法来设置用于读取数据的处理器。

每次 Socket 接收到数据时，会以 [`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html) 对象为参数调用处理器。

```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  socket.handler(buffer -> {
    System.out.println("I received some bytes: " + buffer.length());
  });
});
```

#### 向Socket中写入数据

您可使用 [`write`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#write-io.vertx.core.buffer.Buffer-) 方法写入数据到Socket：

```java
Buffer buffer = Buffer.buffer().appendFloat(12.34f).appendInt(123);
socket.write(buffer);

// 以UTF-8的编码方式写入一个字符串
socket.write("some data");

// 以指定的编码方式写入一个字符串
socket.write("some data", "UTF-16");
```

写入操作是异步的，可能调用 `write` 方法返回过后一段时间才会发生。

### 关闭处理器

若您想要在 Socket 关闭时收到通知，可以设置一个 [`closeHandler`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#closeHandler-io.vertx.core.Handler-)：

```java
socket.closeHandler(v -> {
  System.out.println("The socket has been closed");
});
```

### 处理异常

您可以设置一个 [`exceptionHandler`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#exceptionHandler-io.vertx.core.Handler-) 用以在发生任何异常的时候接收异常信息。

### Event Bus 写处理器

每个 Socket 会自动在Event Bus中注册一个处理器，当这个处理器中收到任意 `Buffer` 时，它会将数据写入到 Socket。

这意味着您可以通过向这个地址发送 `Buffer` 的方式，从不同的 Verticle 甚至是不同的 Vert.x 实例中向指定的 Socket 发送数据。

处理器的地址由 [`writeHandlerID`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#writeHandlerID--) 方法提供。

### 本地和远程地址

您可以通过 [localAddress](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#localAddress--)方法获取 [NetSocket](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 的本地地址，通过 [remoteAddress](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#remoteAddress--) 方法获取 [NetSocket](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 的远程地址（即连接的另一端的地址）。

### 发送文件或 Classpath 中的资源

您可以直接通过 [sendFile](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#sendFile-java.lang.String-) 方法将文件和 classpath 中的资源写入Socket。这种做法是非常高效的，它可以被操作系统内核直接处理。

请阅读 [从 Classpath 访问文件](#从classpath访问文件) 章节了解类路径的限制或禁用它。

### 流式的Socket

[`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 接口继承了 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 和 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 接口，因此您可以将它套用（pump）到其他的读写流上。

有关更多信息，请参阅 [流和管道](#流) 章节。

### 升级到 SSL/TLS 连接

一个非SSL/TLS连接可以通过[`upgradeToSsl`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#upgradeToSsl-io.vertx.core.Handler-)方法升级到SSL/TLS连接。

必须为服务器或客户端配置SSL/TLS才能正常工作。请参阅[SSL/TLS](https://vertx.io/docs/vertx-core/java/#ssl)章节来获取详细信息。

### 关闭 TCP 服务端

您可以调用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#close--) 方法关闭服务端。关闭操作将关闭所有打开的连接并释放所有服务端资源。

关闭操作也是异步的，可能直到方法调用返回过后一段时间才会实际关闭。若您想在实际关闭完成时收到通知，那么您可以传递一个处理器。

当关闭操作完成后，绑定的处理器将被调用：

```java
server.close(res -> {
  if (res.succeeded()) {
    System.out.println("Server is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

### Verticle中的自动清理

若您在 Verticle 内创建了 TCP 服务端和客户端，它们将会在Verticle 撤销时自动被关闭。

### 扩展 - 共享 TCP 服务端

任意一个 TCP 服务端中的处理器总是在相同的 Event Loop 线程上执行。这意味着如果您在多核的服务器上运行，并且只部署了一个实例，那么您的服务器上最多只能使用一个核。

为了利用更多的服务器核，您将需要部署更多的服务器实例。您可以在代码中以编程方式实例化更多（Server的）实例：

```java
for (int i = 0; i < 10; i++) {
  NetServer server = vertx.createNetServer();
  server.connectHandler(socket -> {
    socket.handler(buffer -> {
	    //仅回传数据
      socket.write(buffer);
    });
  });
  server.listen(1234, "localhost");
}
```

如果您使用的是 Verticle，您可以通过在命令行上使用 `-instances` 选项来简单部署更多的服务器实例：

```
vertx run com.mycompany.MyVerticle -instances 10
```

或者使用编程方式部署您的 Verticle 时：

```java
DeploymentOptions options = new DeploymentOptions().setInstances(10);
vertx.deployVerticle("com.mycompany.MyVerticle", options);
```

一旦您这样做，您将发现echo服务器在功能上与之前相同，但是服务器上的所有核都可以被利用，并且可以处理更多的工作。

在这一点上，您可能会问自己：**如何让多台服务器在同一主机和端口上侦听？尝试部署一个以上的实例时真的不会遇到端口冲突吗？**

*Vert.x在这里有一点魔法。*

当您在与现有服务器相同的主机和端口上部署另一个服务器实例时，实际上它并不会尝试创建在同一主机/端口上侦听的新服务器实例。

相反，它内部仅仅维护一个服务器实例。当传入新的连接时，它以轮询的方式将其分发给任意一个连接处理器处理。

因此，Vert.x TCP 服务端可以水平扩展到多个核，并且每个实例保持单线程环境不变。

### 创建 TCP 客户端

使用所有默认选项创建 TCP 客户端的最简单方法如下：

```java
NetClient client = vertx.createNetClient();
```

### 配置 TCP 客户端

如果您不想使用默认值，则可以在创建实例时传入 [`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html) 给客户端：

```java
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
```

### 创建连接

您可以使用 [`connect`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClient.html#connect-int-java.lang.String-io.vertx.core.Handler-) 方法创建到服务器的连接。请指定服务器的端口和主机，以及用于处理 [`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html) 的处理器。当连接成功或失败时处理器会被调用。

```java
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
client.connect(4321, "localhost", res -> {
  if (res.succeeded()) {
    System.out.println("Connected!");
    NetSocket socket = res.result();
  } else {
    System.out.println("Failed to connect: " + res.cause().getMessage());
  }
});
```

### 配置连接重试

可以将客户端配置为在无法连接的情况下自动重试。这是通过 [`setReconnectInterval`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#setReconnectInterval-long-) 和 [`setReconnectAttempts`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#setReconnectAttempts-int-) 方法配置的。

> *注意：目前如果连接失效，Vert.x将不尝试重新连接。重新连接尝试和时间间隔仅适用于创建初始连接。*

```java
NetClientOptions options = new NetClientOptions().
  setReconnectAttempts(10).
  setReconnectInterval(500);

NetClient client = vertx.createNetClient(options);
```

默认情况下，多个连接尝试是被禁用的。

### 记录网络活动

网络活动可以被记录下来，用于调试：

```java
NetServerOptions options = new NetServerOptions().setLogActivity(true);

NetServer server = vertx.createNetServer(options);
```

对于客户端：

```java
NetClientOptions options = new NetClientOptions().setLogActivity(true);

NetClient client = vertx.createNetClient(options);
```

Netty 使用 `DEBUG` 级别和 `io.netty.handler.logging.LoggingHandler` 名称来记录网络活动。使用网络活动记录时，需要注意以下几点：

* 日志的记录是由Netty而不是Vert.x的日志来执行
* 这个功能不能用于生产环境

您应该阅读 [Netty 日志记录](#netty-日志记录) 章节来了解详细信息。

### 配置服务端和客户端以使用SSL/TLS

TCP 客户端和服务端可以通过配置来使用 [TLS（传输层安全性协议）](https://zh.wikipedia.org/wiki/%E5%82%B3%E8%BC%B8%E5%B1%A4%E5%AE%89%E5%85%A8%E5%8D%94%E8%AD%B0)。早期版本的TLS被称为SSL。

无论是否使用SSL/TLS，服务器和客户端的API都是相同的。通过创建客户端/服务器时使用的 [`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html) / [`NetServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html) 来启用TLS/SSL。

#### 在服务端启用SSL/TLS

您需要设置 [`ssl`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html#setSsl-boolean-) 配置项来启用 SSL/TLS。默认是禁用的。

#### 指定服务端的密钥/证书

SSL/TLS 服务端通常向客户端提供证书，以便验证服务端的身份。

可以通过以下几种方式为服务端配置证书/密钥：

第一种方法是指定包含证书和私钥的Java密钥库位置。可以使用 JDK 附带的 [keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html) 实用程序来管理Java密钥存储。

还应提供密钥存储的密码：

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setKeyStoreOptions(
  new JksOptions().
    setPath("/path/to/your/server-keystore.jks").
    setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
```

或者，您可以自己读取密钥库到一个`Buffer`，并将它直接提供给 `JksOptions`：

```java
Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-keystore.jks");
JksOptions jksOptions = new JksOptions().
  setValue(myKeyStoreAsABuffer).
  setPassword("password-of-your-keystore");
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(jksOptions);
NetServer server = vertx.createNetServer(options);
```

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)，通常为`.pfx`或`.p12`扩展名）也可以用与JKS密钥存储相似的方式加载：

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setPfxKeyCertOptions(
  new PfxOptions().
    setPath("/path/to/your/server-keystore.pfx").
    setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-keystore.pfx");
PfxOptions pfxOptions = new PfxOptions().
  setValue(myKeyStoreAsABuffer).
  setPassword("password-of-your-keystore");
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setPfxKeyCertOptions(pfxOptions);
NetServer server = vertx.createNetServer(options);
```

另外一种分别提供服务器私钥和证书的方法是使用`.pem`文件。

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setPemKeyCertOptions(
  new PemKeyCertOptions().
    setKeyPath("/path/to/your/server-key.pem").
    setCertPath("/path/to/your/server-cert.pem")
);
NetServer server = vertx.createNetServer(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myKeyAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-key.pem");
Buffer myCertAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-cert.pem");
PemKeyCertOptions pemOptions = new PemKeyCertOptions().
  setKeyValue(myKeyAsABuffer).
  setCertValue(myCertAsABuffer);
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setPemKeyCertOptions(pemOptions);
NetServer server = vertx.createNetServer(options);
```

请记住pem的配置和私钥是不加密的。

#### 指定服务器信任

SSL/TLS 服务端可以使用证书颁发机构来验证客户端的身份。

证书颁发机构可通过多种方式为服务端配置。

可使用JDK随附的[keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html)实用程序来管理Java 受信存储。

还应提供受信存储的密码：

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setTrustStoreOptions(
    new JksOptions().
      setPath("/path/to/your/truststore.jks").
      setPassword("password-of-your-truststore")
  );
NetServer server = vertx.createNetServer(options);
```

或者您可以自己读取受信存储到`Buffer`，并将它直接提供：

```java
Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.jks");
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setTrustStoreOptions(
    new JksOptions().
      setValue(myTrustStoreAsABuffer).
      setPassword("password-of-your-truststore")
  );
NetServer server = vertx.createNetServer(options);
```

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)，通常为`.pfx`或`.p12`扩展名）也可以用与JKS密钥存储相似的方式加载：

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setPfxTrustOptions(
    new PfxOptions().
      setPath("/path/to/your/truststore.pfx").
      setPassword("password-of-your-truststore")
  );
NetServer server = vertx.createNetServer(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.pfx");
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setPfxTrustOptions(
    new PfxOptions().
      setValue(myTrustStoreAsABuffer).
      setPassword("password-of-your-truststore")
  );
NetServer server = vertx.createNetServer(options);
```

另一种提供服务器证书颁发机构的方法是使用一个 `.pem` 文件列表。

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setPemTrustOptions(
    new PemTrustOptions().
      addCertPath("/path/to/your/server-ca.pem")
  );
NetServer server = vertx.createNetServer(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myCaAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/server-ca.pfx");
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setClientAuth(ClientAuth.REQUIRED).
  setPemTrustOptions(
    new PemTrustOptions().
      addCertValue(myCaAsABuffer)
  );
NetServer server = vertx.createNetServer(options);
```

#### 客户端启用SSL/TLS

客户端也可以轻松地配置为SSL。使用SSL和使用标准套接字具有完全相同的API。

若要启用 `NetClient` 上的SSL，可调用函数 `setSSL(true)`。

#### 客户端受信配置

若客户端将 [`trustAll`](https://vertx.io/docs/apidocs/io/vertx/core/net/ClientOptionsBase.html#setTrustAll-boolean-) 设置为 `true`，则客户端将信任所有服务端证书。连接仍然会被加密，但这种模式很容易受到中间人攻击。即您无法确定您正连接到谁，请谨慎使用。默认值为`false`。

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustAll(true);
NetClient client = vertx.createNetClient(options);
```

若客户端没有设置[`trustAll`](https://vertx.io/docs/apidocs/io/vertx/core/net/ClientOptionsBase.html#setTrustAll-boolean-)，则必须配置客户端受信存储，并且受信客户端应该包含服务器的证书。

默认情况下，客户端禁用主机验证。要启用主机验证，请在客户端上设置使用的算法（目前仅支持HTTPS和LDAPS）：

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setHostnameVerificationAlgorithm("HTTPS");
NetClient client = vertx.createNetClient(options);
```

和服务器配置相同，也可通过以下几种方式配置受信客户端：

第一种方法是指定包含证书颁发机构的Java受信库的位置。

它只是一个标准的Java密钥存储，与服务器端的密钥存储相同。通过在[`jsk options`](https://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html)上使用[`path`](https://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html#setPath-java.lang.String-)设置客户端受信存储位置。如果服务器在连接期间提供不在客户端受信存储中的证书，则尝试连接将不会成功。

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(
    new JksOptions().
      setPath("/path/to/your/truststore.jks").
      setPassword("password-of-your-truststore")
  );
NetClient client = vertx.createNetClient(options);
```

它也支持`Buffer`的配置：

```java
Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.jks");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(
    new JksOptions().
      setValue(myTrustStoreAsABuffer).
      setPassword("password-of-your-truststore")
  );
NetClient client = vertx.createNetClient(options);
```

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)，通常为`.pfx`或`.p12`扩展名）也可以用与JKS密钥存储相似的方式加载：

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPfxTrustOptions(
    new PfxOptions().
      setPath("/path/to/your/truststore.pfx").
      setPassword("password-of-your-truststore")
  );
NetClient client = vertx.createNetClient(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/truststore.pfx");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPfxTrustOptions(
    new PfxOptions().
      setValue(myTrustStoreAsABuffer).
      setPassword("password-of-your-truststore")
  );
NetClient client = vertx.createNetClient(options);
```

另一种提供服务器证书颁发机构的方法是使用一个`.pem`文件列表。

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPemTrustOptions(
    new PemTrustOptions().
      addCertPath("/path/to/your/ca-cert.pem")
  );
NetClient client = vertx.createNetClient(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myTrustStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/ca-cert.pem");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPemTrustOptions(
    new PemTrustOptions().
      addCertValue(myTrustStoreAsABuffer)
  );
NetClient client = vertx.createNetClient(options);
```

#### 指定客户端的密钥/证书

如果服务器需要客户端认证，那么当连接时，客户端必须向服务器提供自己的证书。可通过以下几种方式配置客户端：

第一种方法是指定包含密钥和证书的Java 密钥库的位置，它只是一个常规的Java 密钥存储。使用[jks options](https://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html)上的功能路径设置客户端密钥库位置。

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setKeyStoreOptions(
  new JksOptions().
    setPath("/path/to/your/client-keystore.jks").
    setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-keystore.jks");
JksOptions jksOptions = new JksOptions().
  setValue(myKeyStoreAsABuffer).
  setPassword("password-of-your-keystore");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setKeyStoreOptions(jksOptions);
NetClient client = vertx.createNetClient(options);
```

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)，通常为`.pfx`或`.p12`扩展名）也可以用与JKS密钥存储相似的方式加载：

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setPfxKeyCertOptions(
  new PfxOptions().
    setPath("/path/to/your/client-keystore.pfx").
    setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myKeyStoreAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-keystore.pfx");
PfxOptions pfxOptions = new PfxOptions().
  setValue(myKeyStoreAsABuffer).
  setPassword("password-of-your-keystore");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPfxKeyCertOptions(pfxOptions);
NetClient client = vertx.createNetClient(options);
```

另一种单独提供服务器私钥和证书的方法是使用 `.pem` 文件。

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setPemKeyCertOptions(
  new PemKeyCertOptions().
    setKeyPath("/path/to/your/client-key.pem").
    setCertPath("/path/to/your/client-cert.pem")
);
NetClient client = vertx.createNetClient(options);
```

也支持通过 `Buffer` 来配置：

```java
Buffer myKeyAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-key.pem");
Buffer myCertAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/client-cert.pem");
PemKeyCertOptions pemOptions = new PemKeyCertOptions().
  setKeyValue(myKeyAsABuffer).
  setCertValue(myCertAsABuffer);
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPemKeyCertOptions(pemOptions);
NetClient client = vertx.createNetClient(options);
```

请记住 `pem` 的配置和私钥是不加密的。

#### 用于测试和开发目的的自签名证书

> 提醒：不要在生产设置中使用，这里生成的密钥非常不安全。

在运行单元/集成测试或是运行开发版的应用程序时都经常需要自签名证书。

[`SelfSignedCertificate`](https://vertx.io/docs/apidocs/io/vertx/core/net/SelfSignedCertificate.html) 可用于提供自签名PEM证书，并可以提供 [`KeyCertOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/KeyCertOptions.html)和 [`TrustOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/TrustOptions.html) 配置：

```java
SelfSignedCertificate certificate = SelfSignedCertificate.create();

NetServerOptions serverOptions = new NetServerOptions()
  .setSsl(true)
  .setKeyCertOptions(certificate.keyCertOptions())
  .setTrustOptions(certificate.trustOptions());

NetServer server = vertx.createNetServer(serverOptions)
  .connectHandler(socket -> socket.write("Hello!").end())
  .listen(1234, "localhost");

NetClientOptions clientOptions = new NetClientOptions()
  .setSsl(true)
  .setKeyCertOptions(certificate.keyCertOptions())
  .setTrustOptions(certificate.trustOptions());

NetClient client = vertx.createNetClient(clientOptions);
client.connect(1234, "localhost", ar -> {
  if (ar.succeeded()) {
    ar.result().handler(buffer -> System.out.println(buffer));
  } else {
    System.err.println("Woops: " + ar.cause().getMessage());
  }
});
```

客户端也可配置为信任所有证书：

```java
NetClientOptions clientOptions = new NetClientOptions()
  .setSsl(true)
  .setTrustAll(true);
```

自签名证书也适用于其他基于TCP的协议，如HTTPS：

```java
SelfSignedCertificate certificate = SelfSignedCertificate.create();

vertx.createHttpServer(new HttpServerOptions()
  .setSsl(true)
  .setKeyCertOptions(certificate.keyCertOptions())
  .setTrustOptions(certificate.trustOptions()))
  .requestHandler(req -> req.response().end("Hello!"))
  .listen(8080);
```

#### 待撤销证书颁发机构

可以通过配置证书吊销列表（CRL）来吊销不再被信任的证书机构。[`crlPath`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#addCrlPath-java.lang.String-)配置了使用的CRL：

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(trustOptions).
  addCrlPath("/path/to/your/crl.pem");
NetClient client = vertx.createNetClient(options);
```

也支持通过`Buffer`来配置：

```java
Buffer myCrlAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/crl.pem");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(trustOptions).
  addCrlValue(myCrlAsABuffer);
NetClient client = vertx.createNetClient(options);
```

#### 配置密码套件

默认情况下，TLS配置将使用运行Vert.x的JVM 密码套件，该密码套件可以配置一套启用的密码：

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions).
  addEnabledCipherSuite("ECDHE-RSA-AES128-GCM-SHA256").
  addEnabledCipherSuite("ECDHE-ECDSA-AES128-GCM-SHA256").
  addEnabledCipherSuite("ECDHE-RSA-AES256-GCM-SHA384").
  addEnabledCipherSuite("CDHE-ECDSA-AES256-GCM-SHA384");
NetServer server = vertx.createNetServer(options);
```

密码套件可在[`NetServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)或[`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)配置项中指定。

#### 配置TLS协议版本

默认情况下，TLS配置将使用以下协议版本：SSLv2Hello、TLSv1、TLSv1.1 和 TLSv1.2。 协议版本可以通过显式添加启用协议进行配置：

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions).
  addEnabledSecureTransportProtocol("TLSv1.1").
  addEnabledSecureTransportProtocol("TLSv1.2");
NetServer server = vertx.createNetServer(options);
```

协议版本可在[`NetServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)或[`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)配置项中指定。

#### SSL引擎

引擎实现可以配置为使用 OpenSSL 而不是JDK实现（来支持SSL）。 OpenSSL提供比JDK引擎更好的性能和CPU使用率、以及JDK版本独立性。

引擎选项可使用：

* 当[`getSslEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#getSslEngineOptions--)被设置时，使用该选项
* 否则使用[`JdkSSLEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/JdkSSLEngineOptions.html)

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions);

// Use JDK SSL engine explicitly
// 显式使用JDK SSL引擎
options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions).
  setJdkSslEngineOptions(new JdkSSLEngineOptions());

// Use OpenSSL engine
// 使用OpenSSL引擎
options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions).
  setOpenSslEngineOptions(new OpenSSLEngineOptions());
```

#### 应用层协议协商

ALPN(Application-Layer Protocol Negotiation)是应用层协议协商的TLS扩展，它被HTTP/2使用：在TLS握手期时，客户端给出其接受的应用协议列表，之后服务器使用它所支持的协议响应。

标准的Java 8不支持ALPN，所以ALPN应该通过其他方式启用：

* OpenSSL支持
* Jetty-ALPN支持

引擎选项可使用:

* 当 [`getSslEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#getSslEngineOptions--) 被设置时，使用该选项
* JDK中ALPN可用时使用 [`JdkSSLEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/JdkSSLEngineOptions.html)
* OpenSSL中ALPN可用时使用 [`OpenSSLEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/OpenSSLEngineOptions.html)
* 否则失败

#### OpenSSL ALPN支持

OpenSSL提供了原生的ALPN支持。

OpenSSL需要配置 [`setOpenSslEngineOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#setOpenSslEngineOptions-io.vertx.core.net.OpenSSLEngineOptions-) 并在类路径上使用 [netty-tcnative](http://netty.io/wiki/forked-tomcat-native.html) 的jar库。依赖于tcnative的实现它需要OpenSSL安装在您的操作系统中。

#### Jetty-ALPN支持

Jetty-ALPN是一个小型的jar，它覆盖了几种Java 8发行版用以支持ALPN。

JVM必须将 `alpn-boot-${version}.jar` 放在它的 `boot classpath` 中启动：

```
-Xbootclasspath/p:/path/to/alpn-boot${version}.jar
```

其中 `${version}` 取决于JVM的版本，如 *OpenJDK 1.8.0u74* 中的 *8.1.7.v20160121*。这个完整列表可以在 [Jetty-ALPN](http://www.eclipse.org/jetty/documentation/current/alpn-chapter.html) 页面上找到。

这种方法主要缺点是ALPN的实现版本依赖于JVM的版本。为了解决这个问题，可以使用 [Jetty ALPN agent](https://github.com/jetty-project/jetty-alpn-agent)。agent是一个JVM代理，它会为运行它的JVM选择正确的ALPN版本：

```
-javaagent:/path/to/alpn/agent
```

### 客户端连接使用代理

[`NetClient`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClient.html) 支持HTTP/1.x *CONNECT*、*SOCKS4a* 或 *SOCKS5* 代理。

代理可以在 [`NetClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html) 内设置 [`ProxyOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/ProxyOptions.html) 来配置代理类型、主机名、端口、可选的用户名和密码。

以下是一个例子：

```java
NetClientOptions options = new NetClientOptions()
  .setProxyOptions(new ProxyOptions().setType(ProxyType.SOCKS5)
    .setHost("localhost").setPort(1080)
    .setUsername("username").setPassword("secret"));
NetClient client = vertx.createNetClient(options);
```

DNS 解析总是在代理服务器上完成解析，为了实现 SOCKS4 客户端的功能，需要先在本地解析 DNS 地址。

## 编写 HTTP 服务端和客户端

Vert.x 允许您轻松编写非阻塞的 HTTP 客户端和服务端。

Vert.x 支持 HTTP/1.0、HTTP/1.1 和 HTTP/2 协议。

用于 HTTP 的基本 API 对 HTTP/1.x 和 HTTP/2 是相同的，特定的API功能也可用于处理 HTTP/2 协议。

### 创建 HTTP 服务端

使用所有默认选项创建 HTTP 服务端的最简单方法如下：

```java
HttpServer server = vertx.createHttpServer();
```

### 配置 HTTP 服务端

若您不想用默认值，可以在创建服务器时传递一个 [`HttpServerOptions`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html) 实例给它：

```java
HttpServerOptions options = new HttpServerOptions().setMaxWebsocketFrameSize(1000000);

HttpServer server = vertx.createHttpServer(options);
```

### 配置 HTTP/2 服务端

Vert.x支持 TLS `h2`和TCP `h2c`之上的 HTTP/2 协议。

* `h2` 表示使用了TLS的应用层协议协商(ALPN)协议来协商的 HTTP/2 协议
* `h2c` 表示在TCP层上使用明文形式的 HTTP/2 协议，这样的连接是使用 HTTP/1.1升级 请求或者直接建立

要处理 h2 请求，你必须调用 [`setUseAlpn`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setUseAlpn-boolean-) 方法来启用TLS：

```java
HttpServerOptions options = new HttpServerOptions()
    .setUseAlpn(true)
    .setSsl(true)
    .setKeyStoreOptions(new JksOptions().setPath("/path/to/my/keystore"));

HttpServer server = vertx.createHttpServer(options);
```

ALPN是一个TLS的扩展，它在客户端和服务器开始交换数据之前协商协议。

不支持ALPN的客户端仍然可以执行经典的SSL握手。

通常情况，ALPN会对 `h2` 协议达成一致，尽管服务器或客户端决定了仍然使用 HTTP/1.1 协议。

要处理 `h2c` 请求，TLS必须被禁用，服务器将升级到 HTTP/2 以满足任何希望升级到 HTTP/2 的 HTTP/1.1 请求。它还将接受以 `PRI*HTTP/2.0\r\nSM\r\n` 开始的`h2c`直接连接。

> 警告：大多数浏览器不支持 `h2c`，所以在建站时，您应该使用 `h2` 而不是 `h2c`。

当服务器接受 HTTP/2 连接时，它会向客户端发送其初始设置。定义客户端如何使用连接，服务器的默认初始设置为：

* [`getMaxConcurrentStreams`](https://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html#getMaxConcurrentStreams--)：按照 HTTP/2 RFC建议推荐值为100
* 其他默认的 HTTP/2 的设置

> 请注意：**Worker Verticle 和 HTTP/2 不兼容。**

### 记录服务端网络活动

为了进行调试，可记录网络活动。

```java
HttpServerOptions options = new HttpServerOptions().setLogActivity(true);

HttpServer server = vertx.createHttpServer(options);
```

详细说明请参阅 [记录网络活动](#记录网络活动) 章节。

### 开启服务端监听

要告诉服务器监听传入的请求，您可以使用其中一个 [`listen`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#listen--) 方法。

在配置项中告诉服务器监听指定的主机和端口：

```java
HttpServer server = vertx.createHttpServer();
server.listen();
```

或在调用 `listen` 方法时指定主机和端口号，这样就忽略了配置项（中的主机和端口）：

```java
HttpServer server = vertx.createHttpServer();
server.listen(8080, "myhost.com");
```

默认主机名是`0.0.0.0`，它表示：监听所有可用地址；默认端口号是`80`。

实际的绑定也是异步的，因此服务器也许并没有在调用 `listen` 方法返回时监听，而是在一段时间过后才监听。

若您希望在服务器实际监听时收到通知，您可以向 `listen` 提供一个处理器。例如：

```java
HttpServer server = vertx.createHttpServer();
server.listen(8080, "myhost.com", res -> {
  if (res.succeeded()) {
    System.out.println("Server is now listening!");
  } else {
    System.out.println("Failed to bind!");
  }
});
```

### 收到传入请求的通知

若您需要在收到请求时收到通知，则需要设置一个 [`requestHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#requestHandler-io.vertx.core.Handler-)：

```java
HttpServer server = vertx.createHttpServer();
server.requestHandler(request -> {
  // 在这里处理请求
});
```

### 处理请求

当请求到达时，Vert.x 会像对应的处理函数传入一个 [`HttpServerRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html) 实例并调用请求处理函数，此对象表示服务端 HTTP 请求。当请求的头信息被完全读取时会调用该请求处理器。

如果请求包含请求体，那么该请求体将在请求处理器被调用后的某个时间到达服务器。

服务请求对象允许您检索 [`uri`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uri--)，[`path`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#path--)，[`params`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--) 和 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--) 等其他信息。

每一个服务请求对象和一个服务响应对象绑定，您可以用 [`response`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#response--) 方法获取一个 [`HttpServerResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html) 对象的引用。

这是服务器处理请求并回复 “hello world” 的简单示例。

```java
vertx.createHttpServer().requestHandler(request -> {
  request.response().end("Hello world");
}).listen(8080);
```

#### 请求版本

在请求中指定的 HTTP 版本可通过 [`version`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#version--) 方法获取。

#### 请求方法

使用 [`method`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#method--) 方法读取请求中的 HTTP Method（即GET、POST、PUT、DELETE、HEAD、OPTIONS等）。

#### 请求URI

使用 [`uri`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uri--) 方法读取请求中的URI路径。

请注意，这是在HTTP 请求中传递的实际URI，它总是一个相对的URI。

这个URI是在 [Section 5.1.2 of the HTTP specification - Request-URI](http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html) 中定义的。

#### 请求路径

使用 [`path`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#path--) 方法读取URI中的路径部分。

例如，请求的URI为：

```
a/b/c/page.html?param1=abc&param2=xyz
```

路径部分应该是：

```
/a/b/c/page.html
```

#### 请求查询

使用[`query`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#query--)读取URI中的查询部分。

例如，请求的URI为：

```
a/b/c/page.html?param1=abc&param2=xyz
```

查询部分应该是：

```
param1=abc&param2=xyz
```

#### 请求头部

使用 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--) 方法获取HTTP 请求中的请求头部信息。

这个方法返回一个 [`MultiMap`](https://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html) 实例。它像一个普通的Map或Hash，并且它还允许同一个键支持多个值 —— 因为HTTP允许同一个键支持多个请求头的值。

它的键值不区分大小写，这意味着您可以执行以下操作：

```java
MultiMap headers = request.headers();

// Get the User-Agent:
// 读取User-Agent
System.out.println("User agent is " + headers.get("user-agent"));

// You can also do this and get the same result:
// 这样做可以得到和上边相同的结果
System.out.println("User agent is " + headers.get("User-Agent"));
```

#### 请求主机

使用 [`host`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#host--) 方法返回 HTTP 请求中的主机名。

对于 HTTP/1.x 请求返回请求头中的 `host` 值，对于 HTTP/1 请求则返回伪头中的`:authority`的值。

#### 请求参数

您可以使用 [`params`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--) 方法返回HTTP请求中的参数信息。像 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--) 方法一样它也会返回一个 [`MultiMap`](https://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html) 实例，因为可以有多个具有相同名称的参数。

请求参数在请求URI的 path 部分之后，例如URI是：

```
/page.html?param1=abc&param2=xyz
```

那么参数将包含以下内容：

```
param1: 'abc'
param2: 'xyz'
```

请注意，这些请求参数是从请求的 URI 中解析读取的，若您已经将表单属性存放在请求体中发送出去，并且该请求为 `multi-part/form-data` 类型请求，那么它们将不会显示在此处的参数中。

#### 远程地址

可以使用 [`remoteAddress`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#remoteAddress--) 方法读取请求发送者的地址。

#### 绝对URI

HTTP 请求中传递的URI通常是相对的，若您想要读取请求中和相对URI对应的绝对URI，可调用 [`absoluteURI`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#absoluteURI--) 方法。

#### 结束处理器

当整个请求（包括任何正文）已经被完全读取时，请求中的 [`endHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#endHandler-io.vertx.core.Handler-) 方法会被调用。

#### 请求体中读取数据

HTTP请求通常包含我们需要读取的主体。如前所述，当请求头部达到时，请求处理器会被调用，因此请求对象在此时没有请求体。

这是因为请求体可能非常大（如文件上传），并且我们不会在内容发送给您之前将其全部缓冲存储在内存中，这可能会导致服务器耗尽可用内存。

要接收请求体，您可在请求中调用 [`handler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#handler-io.vertx.core.Handler-) 方法设置一个处理器，每次请求体的一小块数据收到时，该处理器都会被调用。以下是一个例子：

```java
request.handler(buffer -> {
  System.out.println("I have received a chunk of the body of length " + buffer.length());
});
```

传递给处理器的对象是一个 [`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)，当数据从网络到达时，处理器可以多次被调用，这取决于请求体的大小。

在某些情况下（例：若请求体很小），您将需要将这个请求体聚合到内存中，以便您可以按照下边的方式进行聚合：

```java
Buffer totalBuffer = Buffer.buffer();

request.handler(buffer -> {
  System.out.println("I have received a chunk of the body of length " + buffer.length());
  totalBuffer.appendBuffer(buffer);
});

request.endHandler(v -> {
  System.out.println("Full body received, length = " + totalBuffer.length());
});
```

这是一个常见的情况，Vert.x为您提供了一个 [`bodyHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#bodyHandler-io.vertx.core.Handler-) 方法来执行此操作。当所有请求体被收到时，`bodyHandler` 绑定的处理器会被调用一次：

```java
request.bodyHandler(totalBuffer -> {
  System.out.println("Full body received, length = " + totalBuffer.length());
});
```

#### Pumping 请求

请求对象实现了 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 接口，因此您可以将请求体读取到任何 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 实例中。

详细请参阅 [流和管道](#流) 章节。

#### 处理 HTML 表单

您可使用 `application/x-www-form-urlencoded` 或 `multipart/form-data` 这两种 **content-type** 来提交 HTML 表单。

对于使用 URL 编码过的表单，表单属性会被编码在URL中，如同普通查询参数一样。

对于 multipart 类型的表单，它会被编码在请求体中，而且在整个请求体被完全读取之前它是不可用的。Multipart 表单还可以包含文件上传。

若您想要读取 multipart 表单的属性，您应该告诉 Vert.x 您会在读取任何正文 **之前** 调用 [`setExpectMultipart(true)`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#setExpectMultipart-boolean-) 方法，然后在整个请求体都被读取后，您可以使用 [`formAttributes`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#formAttributes--) 方法来读取实际的表单属性。

```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.endHandler(v -> {
    // 请求体被完全读取，所以直接读取表单属性
    MultiMap formAttributes = request.formAttributes();
  });
});
```

#### 处理文件上传

Vert.x 可以处理以 multipart 编码形式上传的的文件。

要接收文件，您可以告诉 Vert.x 使用 multipart 表单，并对请求设置 [`uploadHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uploadHandler-io.vertx.core.Handler-)。

当服务器每次接收到上传请求时，该处理器将被调用一次。

传递给处理器的对象是一个 [`HttpServerFileUpload`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html) 实例。

```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.uploadHandler(upload -> {
    System.out.println("Got a file upload " + upload.name());
  });
});
```

上传的文件可能很大，我们不会在单个缓冲区中包含整个上传的数据，因为这样会导致内存耗尽。相反，上传数据是以块的形式被接收的：

```java
request.uploadHandler(upload -> {
  upload.handler(chunk -> {
    System.out.println("Received a chunk of the upload of length " + chunk.length());
  });
});
```

上传对象实现了 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 接口，因此您可以将请求体读取到任何 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 实例中。详细说明请参阅 [流和管道（泵）](#流) 章节。

若您只是想将文件上传到服务器的某个磁盘，可以使用 [`streamToFileSystem`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html#streamToFileSystem-java.lang.String-) 方法：

```java
request.uploadHandler(upload -> {
  upload.streamToFileSystem("myuploads_directory/" + upload.filename());
});
```

> 警告：确保您检查了生产系统的文件名，以避免恶意客户将文件上传到文件系统中的任意位置。有关详细信息，参阅 [安全说明](https://vertx.io/docs/vertx-core/java/#_security_notes)。

#### 处理压缩体

Vert.x 可以处理在客户端通过 *deflate* 或 *gzip* 算法压缩过的请求体信息。

若要启用解压缩功能则您要在创建服务器时调用 [`setDecompressionSupported`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setDecompressionSupported-boolean-) 方法设置配置项。默认情况下解压缩是被禁用的。

#### 接收自定义 HTTP/2 帧

HTTP/2 是用于 HTTP 请求/响应模型的包含各种帧的一种帧协议，该协议允许发送和接收其他类型的帧。

若要接收自定义帧(frame)，您可以在请求中使用 [`customFrameHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#customFrameHandler-io.vertx.core.Handler-)，每次当自定义的帧数据到达时，这个处理器会被调用。这而是一个例子：

```java
request.customFrameHandler(frame -> {
  System.out.println("Received a frame type=" + frame.type() +
      " payload" + frame.payload().toString());
});
```

HTTP/2 帧不受流量控制限制 —— 当接收到自定义帧时，不论请求是否暂停，自定义帧处理器都将立即被调用。

#### 非标准的 HTTP 方法

[`OTHER`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpMethod.html#OTHER) HTTP 方法可用于非标准方法，在这种情况下，[`rawMethod`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#rawMethod--) 方法返回客户端发送的实际 HTTP 方法。

### 发回响应

服务器响应对象是一个 [`HttpServerResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html) 实例，它可以从 `request` 对应的 [`response`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#response--) 方法中读取。

您可以使用响应对象回写一个响应到 HTTP客户端。

#### 设置状态码和消息

默认的 HTTP 状态响应码为 `200`，表示 `OK`。

可使用 [`setStatusCode`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#setStatusCode-int-) 方法设置不同状态代码。

您还可用 [`setStatusMessage`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#setStatusMessage-java.lang.String-) 方法指定自定义状态消息。

若您不指定状态信息，将会使用默认的状态码响应。

> 注意：对于 HTTP/2 中的状态不会在响应中描述 —— 因为协议不会将消息发送回客户端。

#### 向 HTTP 响应写入数据

想要将数据写入 HTTP Response，您可使用任意一个 [`write`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#write-io.vertx.core.buffer.Buffer-) 方法。

它们可以在响应结束之前被多次调用，它们可以通过以下几种方式调用：

对用单个缓冲区：

```java
HttpServerResponse response = request.response();
response.write(buffer);
```

写入字符串，这种请求字符串将使用 UTF-8 进行编码，并将结果写入到报文中。

```java
HttpServerResponse response = request.response();
response.write("hello world!");
```

写入带编码方式的字符串，这种情况字符串将使用指定的编码方式编码，并将结果写入到报文中。

```java
HttpServerResponse response = request.response();
response.write("hello world!", "UTF-16");
```

响应写入是异步的，并且在写操作进入队列之后会立即返回。

若您只需要将单个字符串或 `Buffer` 写入到HTTP 响应，则可使用 [`end`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#end-java.lang.String-) 方法将其直接写入响应中并发回到客户端。

第一次写入操作会触发响应头的写入，因此，若您不使用HTTP 分块，那么必须在写入响应之前设置 `Content-Length` 头，否则会太迟。若您使用 HTTP 分块则不需要担心这点。

#### 完成 HTTP 响应

一旦您完成了 HTTP 响应，可调用 [`end`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#end-java.lang.String-) 将其发回客户端。

这可以通过几种方式完成：

没有参数，直接结束响应，发回客户端：

```java
HttpServerResponse response = request.response();
response.write("hello world!");
response.end();
```

您也可以和调用`write`方法一样传 `String` 或 `Buffer` 给 `end` 方法。这种情况，它和先调用带 `String` 或 `Buffer` 参数的 `write` 方法，之后调用无参 `end` 方法一样。例如：

```java
HttpServerResponse response = request.response();
response.end("hello world!");
```

#### 关闭底层连接

您可以调用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#close--) 方法关闭底层的TCP 连接。

当响应结束时，Vert.x 将自动关闭非 keep-alive 的连接。

默认情况下，Vert.x 不会自动关闭 keep-alive 的连接，若您想要在一段空闲时间之后让 Vert.x 自动关闭 keep-alive 的连接，则使用[`setIdleTimeout`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setIdleTimeout-int-) 方法进行配置。

HTTP/2 连接在关闭响应之前会发送 `GOAWAY` 帧。

#### 设置响应头

HTTP 响应头可直接添加到 HTTP 响应中，通常直接操作 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#headers--)：

```java
HttpServerResponse response = request.response();
MultiMap headers = response.headers();
headers.set("content-type", "text/html");
headers.set("other-header", "wibble");
```

或您可使用 [`putHeader`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#putHeader-java.lang.String-java.lang.String-) 方法：

```java
HttpServerResponse response = request.response();
response.putHeader("content-type", "text/html").putHeader("other-header", "wibble");
```

响应头必须在写入响应正文消息之前进行设置。

#### 分块 HTTP 响应和附加尾部

Vert.x 支持 [分块传输编码(HTTP Chunked Transfer Encoding)](http://en.wikipedia.org/wiki/Chunked_transfer_encoding)。

这允许HTTP 响应体以块的形式写入，通常在响应体预先不知道尺寸、需要将很大响应正文以流式传输到客户端时使用。

您可以通过如下方式开启分块模式：

```java
HttpServerResponse response = request.response();
response.setChunked(true);
```

默认是不分块的，当处于分块模式，每次调用任意一个 [write](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#write-io.vertx.core.buffer.Buffer-) 方法将导致新的 HTTP 块被写出。

在分块模式下，您还可以将响应的HTTP 响应附加尾部(trailers)写入响应，这种方式实际上是在写入响应的最后一块。

> 注意：分块响应在 HTTP/2 流中无效。

若要向响应添加尾部，则直接添加到 [trailers](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#trailers--) 里。

```java
HttpServerResponse response = request.response();
response.setChunked(true);
MultiMap trailers = response.trailers();
trailers.set("X-wibble", "woobble").set("X-quux", "flooble");
```

或者调用 [`putTrailer`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#putTrailer-java.lang.String-java.lang.String-) 方法：

```java
HttpServerResponse response = request.response();
response.setChunked(true);
response.putTrailer("X-wibble", "woobble").putTrailer("X-quux", "flooble");
```

#### 直接从磁盘或 Classpath 读文件

若您正在编写一个Web 服务端，一种从磁盘中读取并提供文件的方法是将文件作为 [`AsyncFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html) 对象打开并其传送到HTTP 响应中。

或您可以使用 [`readFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html#readFile-java.lang.String-io.vertx.core.Handler-) 方法一次性加载它，并直接将其写入响应。

或者，Vert.x 提供了一种方法，允许您在一个操作中将文件从磁盘或文件系统中读取并提供给HTTP 响应。若底层操作系统支持，这会导致操作系统不通过用户空间复制而直接将文件内容中字节数据从文件传输到Socket。这是使用 [`sendFile`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#sendFile-java.lang.String-) 方法完成的，对于大文件处理通常更有效，而这个方法对于小文件可能很慢。

这儿是一个非常简单的 Web 服务器，它使用 `sendFile` 方法从文件系统中读取并提供文件：

```java
vertx.createHttpServer().requestHandler(request -> {
  String file = "";
  if (request.path().equals("/")) {
    file = "index.html";
  } else if (!request.path().contains("..")) {
    file = request.path();
  }
  request.response().sendFile("web/" + file);
}).listen(8080);
```

发送文件是异步的，可能在调用返回一段时间后才能完成。如果要在文件写入时收到通知，可以在 `sendFile` 方法中设置一个处理器。

请阅读 [从 Classpath 访问文件](#从classpath访问文件) 章节了解类路径的限制或禁用它。

> 注意：若在 HTTPS 协议中使用 `sendFile` 方法，它将会通过用户空间进行复制，因为若内核将数据直接从磁盘复制到 Socket，则不会给我们任何加密的机会。

> 警告：若您要直接使用 Vert.x 编写 Web 服务器，请注意，您想提供文件和类路径之外访问的位置 —— 用户是无法直接利用路径访问的。更安全的做法是使用Vert.x Web替代。

当需要提供文件的一部分，从给定的字节开始，您可以像下边这样做：

```java
vertx.createHttpServer().requestHandler(request -> {
  long offset = 0;
  try {
    offset = Long.parseLong(request.getParam("start"));
  } catch (NumberFormatException e) {
    // 异常处理
  }

  long end = Long.MAX_VALUE;
  try {
    end = Long.parseLong(request.getParam("end"));
  } catch (NumberFormatException e) {
    // 异常处理
  }

  request.response().sendFile("web/mybigfile.txt", offset, end);
}).listen(8080);
```

若您想要从偏移量开始发送文件直到尾部，则不需要提供长度信息，这种情况下，您可以执行以下操作：

```java
vertx.createHttpServer().requestHandler(request -> {
  long offset = 0;
  try {
    offset = Long.parseLong(request.getParam("start"));
  } catch (NumberFormatException e) {
    //异常处理
  }

  request.response().sendFile("web/mybigfile.txt", offset);
}).listen(8080);
```

#### Pumping 响应

服务端响应 `HttpServerResponse` 也是一个 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 实例，因此您可以从任何 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 向其泵送数据，如 [`AsyncFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)、[`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)、[`WebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html) 或 [`HttpServerRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html)。

这儿有一个例子，它回应了任何 PUT 方法的响应中的请求体，它为请求体使用了 Pump，所以即使 HTTP 请求体很大并填满了内存，任何时候它依旧会工作：

```java
vertx.createHttpServer().requestHandler(request -> {
  HttpServerResponse response = request.response();
  if (request.method() == HttpMethod.PUT) {
    response.setChunked(true);
    Pump.pump(request, response).start();
    request.endHandler(v -> response.end());
  } else {
    response.setStatusCode(400).end();
  }
}).listen(8080);
```

#### 写入 HTTP/2 帧

HTTP/2 是用于 HTTP 请求/响应模型的包含各种帧的一种帧协议，该协议允许发送和接收其他类型的帧。

要发送这样的帧，您可以在响应中使用 [`writeCustomFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#writeCustomFrame-int-int-io.vertx.core.buffer.Buffer-) 方法，以下是一个例子：

```java
int frameType = 40;
int frameStatus = 10;
Buffer payload = Buffer.buffer("some data");

// 向客户端发送一帧
response.writeCustomFrame(frameType, frameStatus, payload);
```

这些帧被立即发送，并且不受流程控制的影响——当这样的帧被发送到那里时，可以在其他的 `DATA` 帧之前完成。

#### 流重置

HTTP/1.x 不允许请求或响应流执行清除重置，如当客户端上传的资源已经存在于服务器上，服务器就需要接受整个响应。

HTTP/2 在请求/响应期间随时支持流重置：

```java
request.response().reset();
```

默认的 `NO_ERROR(0)` 错误代码会发送，您也可以发送另外一个错误代码：

```java
request.response().reset(8);
```

HTTP/2 规范中定义了可用的 [错误码](http://httpwg.org/specs/rfc7540.html#ErrorCodes) 列表：

若使用了 [request handler](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#exceptionHandler-io.vertx.core.Handler-) 和 [response handler](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#exceptionHandler-io.vertx.core.Handler-) 两个处理器过后，在流重置完成时您将会收到通知：

```java
request.response().exceptionHandler(err -> {
  if (err instanceof StreamResetException) {
    StreamResetException reset = (StreamResetException) err;
    System.out.println("Stream reset " + reset.getCode());
  }
});
```

#### 服务器推送

服务器推送(Server Push)是 HTTP/2 支持的一个新功能，可以为单个客户端请求并行发送多个响应。

当服务器处理请求时，它可以向客户端推送请求/响应：

```java
HttpServerResponse response = request.response();

// 推送main.js到客户端
response.push(HttpMethod.GET, "/main.js", ar -> {

  if (ar.succeeded()) {

    // 服务器准备推送响应
    HttpServerResponse pushedResponse = ar.result();

    // 发送main.js响应
    pushedResponse.
        putHeader("content-type", "application/json").
        end("alert(\"Push response hello\")");
  } else {
    System.out.println("Could not push client resource " + ar.cause());
  }
});

// 发送请求的资源内容
response.sendFile("<html><head><script src=\"/main.js\"></script></head><body></body></html>");
```

当服务器准备推送响应时，推送响应处理器会被调用，并会发送响应。

推送响应处理器客户能会接收到失败，如：客户端可能取消推送，因为它已经在缓存中包含了 `main.js`，并不在需要它。

您必须在响应结束之前调用 [`push`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#push-io.vertx.core.http.HttpMethod-java.lang.String-java.lang.String-io.vertx.core.Handler-) 方法，但是在推送响应过后依然可以写响应。

### HTTP 压缩

Vert.x 支持 HTTP 压缩。

这意味着在响应发送回客户端之前，您可以将响应体自动压缩。

若客户端不支持HTTP 压缩，则它可以发回没有压缩过的请求。

这允许它同时处理支持HTTP 压缩的客户端和不支持的客户端。

要启用压缩，可以使用 [`setCompressionSupported`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionSupported-boolean-) 方法进行配置。默认情况下，未启用压缩。

当启用HTTP 压缩时，服务器将检查客户端请求头中是否包含了 `Accept-Encoding` 并支持常用的 deflate 和 gzip 压缩算法。Vert.x 两者都支持。若找到这样的请求头，服务器将使用所支持的压缩算法之一自动压缩响应正文并发送回客户端。

注意：压缩可以减少网络流量，但是CPU密集度会更高。

为了解决后边一个问题，Vert.x也允许您调整原始的 gzip/deflate 压缩算法的 “压缩级别” 参数

压缩级别允许根据所得数据的压缩比和压缩/解压的计算成本来配置 gzip/deflate 算法。

压缩级别是从 1 到 9 的整数值，其中 1 表示更低的压缩比但是最快的算法，9 表示可用的最大压缩比但比较慢的算法。

使用高于 1-2 的压缩级别通常允许仅仅保存一些字节大小 —— 它的增益不是线性的，并取决于要压缩的特定数据 —— 但它可以满足服务器所要求的CPU周期的不可控的成本（注意现在Vert.x不支持任何缓存形式的响应数据，如静态文件，因此压缩是在每个请求体生成时进行的）,它可生成压缩过的响应数据、并对接收的响应解码（膨胀）—— 和客户端使用的方式一致，这种操作随着压缩级别的增长会变得更加倾向于CPU密集型。

默认情况下 —— 如果通过 [`setCompressionSupported`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionSupported-boolean-) 方法启用压缩，Vert.x 将使用 *6* 作为压缩级别，但是该参数可通过 [`setCompressionLevel`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionLevel-int-) 方法来更改。

### 创建 HTTP 客户端

您可通过以下方式创建一个具有默认配置的 [`HttpClient`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 实例：

```java
HttpClient client = vertx.createHttpClient();
```

若您想要配置客户端选项，可按以下方式创建：

```java
HttpClientOptions options = new HttpClientOptions().setKeepAlive(false);
HttpClient client = vertx.createHttpClient(options);
```

Vert.x 支持基于 TLS `h2` 和 TCP `h2c` 的 HTTP/2 协议。

默认情况下，HTTP 客户端会发送 HTTP/1.1 请求。若要执行 HTTP/2 请求，则必须调用 [`setProtocolVersion`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setProtocolVersion-io.vertx.core.http.HttpVersion-) 方法将版本设置成 [HTTP_2](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpVersion.html#HTTP_2)。

对于 `h2` 请求，必须使用应用层协议协商(ALPN)启用TLS：

```java
HttpClientOptions options = new HttpClientOptions().
    setProtocolVersion(HttpVersion.HTTP_2).
    setSsl(true).
    setUseAlpn(true).
    setTrustAll(true);

HttpClient client = vertx.createHttpClient(options);
```

对于 `h2c` 请求，TLS必须禁用，客户端将执行 HTTP/1.1 请求并尝试升级到 HTTP/2：

```java
HttpClientOptions options = new HttpClientOptions().setProtocolVersion(HttpVersion.HTTP_2);

HttpClient client = vertx.createHttpClient(options);
```

`h2c` 连接也可以直接建立，如连接可以使用前文提到的方式创建，当 [`setHttp2ClearTextUpgrade`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2ClearTextUpgrade-boolean-) 选项设置为 `false` 时：建立连接后，客户端将发送 HTTP/2 连接前缀，并期望从服务端接收相同的连接偏好。

HTTP 服务端可能不支持 HTTP/2，当响应到达时，可以使用 [`version`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#version--) 方法检查响应实际HTTP版本。

当客户端连接到 HTTP/2 服务端时，它将向服务端发送其[初始设置](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#getInitialSettings--)。设置定义服务器如何使用连接、客户端的默认初始设置是由 HTTP/2 RFC定义的。

### 记录客户端网络活动

为了进行调试，可以记录网络活动：

```java
HttpClientOptions options = new HttpClientOptions().setLogActivity(true);
HttpClient client = vertx.createHttpClient(options);
```

详情请参阅 [记录网络活动](#记录网络活动) 章节。

### 发出请求

HTTP 客户端是很灵活的，您可以通过各种方式发出请求。

通常您希望使用 HTTP 客户端向同一个主机/端口发送很多请求。为避免每次发送请求时重复设主机/端口，您可以为客户端配置默认主机/端口：

```java
HttpClientOptions options = new HttpClientOptions().setDefaultHost("wibble.com");
// Can also set default port if you want...
// 若您想可设置默认端口
HttpClient client = vertx.createHttpClient(options);
client.getNow("/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

或者您发现自己使用相同的客户端向不同主机的主机/端口发送大量请求，则可以在发出请求时简单指定主机/端口：

```java
HttpClient client = vertx.createHttpClient();

// 指定端口和主机名
client.getNow(8080, "myserver.mycompany.com", "/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// 这次使用默认端口80和指定的主机名
client.getNow("foo.othercompany.com", "/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

用客户端发出请求的所有不同方式都支持这两种指定主机/端口的方法。

#### 无请求体的简单请求

通常，您想发出没有请求体的HTTP 请求，这种情况通常如HTTP GET、OPTIONS 和 HEAD 请求。

使用 Vert.x HTTP Client 执行这种请求最简单的方式是使用加了 `Now` 后缀的请求方法，如 [`getNow`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#getNow-io.vertx.core.http.RequestOptions-io.vertx.core.Handler-)。这些方法会创建HTTP 请求，并在单个方法调用中发送它，而且允许您提供一个处理器，当HTTP 响应发送回来时调用该处理器来处理响应结果。

```java
HttpClient client = vertx.createHttpClient();

// 发送GET请求
client.getNow("/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// 发送HEAD请求
client.headNow("/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

#### 发送通用请求

有时您在运行时不知道发送请求的 HTTP 方法，对于这种情况，我们提供通用请求方法 [`request`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#request-io.vertx.core.http.HttpMethod-io.vertx.core.http.RequestOptions-)，允许您在运行时指定 HTTP 方法：

```java
HttpClient client = vertx.createHttpClient();

client.request(HttpMethod.GET, "some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end();

client.request(HttpMethod.POST, "foo-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end("some-data");
```

#### 写请求体

有时您想要发送一个包含了请求体的请求，或者也许您想要在发送请求之前写入头部到请求中。

为此，您可以调用其中一个指定的请求方法，如 [`post`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#post-io.vertx.core.http.RequestOptions-) 或一个其他通用请求方法，如 [`request`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#request-io.vertx.core.http.HttpMethod-io.vertx.core.http.RequestOptions-)。这些方法都不会立即发送请求，而是返回一个 [`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html) 实例，它可以用来写数据到请求体和请求头。

这儿有一些写入请求体的 POST 请求例子：

```java
HttpClient client = vertx.createHttpClient();

HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// 现在准备请求的一些东西
request.putHeader("content-length", "1000");
request.putHeader("content-type", "text/plain");
request.write(body);

// 确保请求完成后结束
request.end();

// 或使用Fluent的API
client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-length", "1000").putHeader("content-type", "text/plain").write(body).end();

// 或事情更简单
client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-type", "text/plain").end(body);
```

可以用UTF-8编码方式编码字符串和以指定方式编码编码字符串、或写`Buffer`的方法：

```java
// 写UTF-8编码的字符串
request.write("some data");

// 写指定编码的字符串
request.write("some other data", "UTF-16");

// 写Buffer
Buffer buffer = Buffer.buffer();
buffer.appendInt(123).appendLong(245l);
request.write(buffer);
```

若您仅需要写单个字符串或 `Buffer` 到HTTP请求中，您可以直接调用 `end` 函数完成写入和请求的发送操作。

```java
request.end("some simple data");

// 在单次调用中写Buffer并结束请求（直接发送）
Buffer buffer = Buffer.buffer().appendDouble(12.34d).appendLong(432l);
request.end(buffer);
```

当您写入请求时，第一次调用 `write` 方法将先将请求头写入到请求报文中。

实际写入操作是异步的，它可能在调用返回一段时间后才发生。

带请求体的非分块 HTTP 请求需要提供 `Content-Length` 头。因此，若您不使用 HTTP 分块，则必须在写入请求之前设置`Content-Length`头，否则会出错。

若您在调用其中一个 `end` 方法处理 String 或 Buffer，在写入请求体之前，Vert.x 将自动计算并设置 `Content-Length`。若您在使用HTTP 分块模式，则不需要 `Content-Length` 头，因此您不必先计算大小。

#### 写请求头

您可以直接使用 MultiMap 结构的 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#headers--) 来设置请求头：

```java
MultiMap headers = request.headers();
headers.set("content-type", "application/json").set("other-header", "foo");
```

这个headers是一个 [`MultiMap`](https://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html) 的实例，它提供了添加、设置、删除条目的操作。HTTP Header允许一个特定的键包含多个值。

您也可以使用 [`putHeader`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#putHeader-java.lang.String-java.lang.String-) 方法编写头文件：

```java
request.putHeader("content-type", "application/json").putHeader("other-header", "foo");
```

若您想写入请求头，则您必须在写入任何请求体之前这样做来设置请求头。

#### 非标准的HTTP 方法

[`OTHER`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpMethod.html#OTHER) HTTP方法用于非标准HTTP方法，当使用此方法时，必须使用 [`setRawMethod`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setRawMethod-java.lang.String-) 方法来设置需要发送到服务器的raw方法。

#### 发送 HTTP 请求

一旦完成了 HTTP 请求的准备工作，您必须调用其中一个 [`end`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#end-java.lang.String-) 方法来发送该请求（结束请求）。

结束一个请求时，若请求头尚未被写入，会导致它们被写入，并且请求被标记成完成的。

请求可以通过多种方式结束。无参简单结束请求的方式如：

```java
request.end();  
```

或可以在调用 `end` 方法时提供 String 或 Buffer，这个和先调用带 String/Buffer 参数的 `write` 方法之后再调用无参 `end` 方法一样：

```java
request.end("some-data");

// 使用buffer结束
Buffer buffer = Buffer.buffer().appendFloat(12.3f).appendInt(321);
request.end(buffer);
```

#### 分块 HTTP 请求

Vert.x 支持 [HTTP Chunked Transfer Encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding) 请求。

这允许使用块方式写入HTTP 请求体，这个在请求体比较大需要流式发送到服务器，或预先不知道大小时很常用。

您可使用 [`setChuncked`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setChunked-boolean-) 将HTTP 请求设置成分块模式。

在分块模式下，每次调用 `write` 方法将导致新的块被写入到报文，这种模式中，无需先设置请求头中的 `Content-Length`。

```java
request.setChunked(true);

// Write some chunks
// 写一些块
for (int i = 0; i < 10; i++) {
  request.write("this-is-chunk-" + i);
}

request.end();
```

#### 请求超时

您可使用 [`setTimeout`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setTimeout-long-)设置一个特定 HTTP 请求的超时时间。

若请求在超时期限内未返回任何数据，则异常将会被传给异常处理器（若提供），并且请求将会被关闭。

#### 处理异常

您可以通过在 [`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html) 实例中设置异常处理器来处理请求时发生的异常：

```java
HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
request.exceptionHandler(e -> {
  System.out.println("Received exception: " + e.getMessage());
  e.printStackTrace();
});
```

这种处理器不处理需要在 [`HttpClientResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html) 中处理的非 2xx 响应：

```java
HttpClientRequest request = client.post("some-uri", response -> {
  if (response.statusCode() == 200) {
    System.out.println("Everything fine");
    return;
  }
  if (response.statusCode() == 500) {
    System.out.println("Unexpected behavior on the server side");
    return;
  }
});
request.end();
```

> 重要提示：一系列的 `XXXNow` 方法均不接收异常处理器做为参数。

#### 客户端请求中指定处理器

不像在调用中提供响应处理器来创建客户端请求对象，相反您可以当请求创建时不提供处理器、稍后在请求对象中调用 [`handler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#handler-io.vertx.core.Handler-) 来设置。如：

```java
HttpClientRequest request = client.post("some-uri");
request.handler(response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

#### 使用流式请求

[`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html) 实例实现了 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 接口，这意味着您可以从任何 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 实例将数据泵入请求中。

例如，您可以将磁盘上的文件直接泵送到HTTP 请求体中，如下所示：

```java
request.setChunked(true);
Pump pump = Pump.pump(file, request);
file.endHandler(v -> request.end());
pump.start();
```

#### 写 HTTP/2 帧

HTTP/2 是用于 HTTP 请求/响应模型的具有各种帧的一个帧协议，该协议允许发送和接收其他类型的帧。

要发送这样的帧，您可以使用 [`write`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#write-io.vertx.core.buffer.Buffer-) 方法写入请求，以下是一个例子：

```java
int frameType = 40;
int frameStatus = 10;
Buffer payload = Buffer.buffer("some data");

// Sending a frame to the server
// 发送一帧到服务器
request.writeCustomFrame(frameType, frameStatus, payload);
```

#### 流重置

HTTP/1.x 不允许请求或响应流进行重置，如当客户端上传了服务器上存在的资源时，服务器依然要接收整个响应。

HTTP/2 在请求/响应期间随时支持流重置：

```java
request.reset();
```

默认情况，发送 `NO_ERROR(0)`错误代码，可发送另一个代码：

```java
request.reset(8);
```

HTTP/2规范定义了可使用的 [错误码](http://httpwg.org/specs/rfc7540.html#ErrorCodes) 列表。

若使用了 [request handler](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#exceptionHandler-io.vertx.core.Handler-) 和 [response handler](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#exceptionHandler-io.vertx.core.Handler-) 两个处理器过后，在流重置完成时您将会收到通知。

```java
request.exceptionHandler(err -> {
  if (err instanceof StreamResetException) {
    StreamResetException reset = (StreamResetException) err;
    System.out.println("Stream reset " + reset.getCode());
  }
});
```

### 处理 HTTP 响应

您可以在请求方法中指定处理器或通过 [`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html) 对象直接设置处理器来接收到 [`HttpClientResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html) 的实例。

您可以通过 [`statusCode`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#statusCode--) 和 [`statusMessage`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#statusMessage--) 方法从响应中查询响应的状态码和状态消息：

```java
client.getNow("some-uri", response -> {
  // the status code - e.g. 200 or 404
  // 状态代码,如:200、404
  System.out.println("Status code is " + response.statusCode());

  // the status message e.g. "OK" or "Not Found".
  // 状态消息,如:OK、Not Found
  System.out.println("Status message is " + response.statusMessage());
});
```

#### 使用流式响应

[`HttpClientResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html) 实例也是一个 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 实例，这意味着您可以泵送数据到任何 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 实例。

#### 响应头和尾

HTTP 响应可包含头信息。您可以使用 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#headers--) 方法来读取响应头。该方法返回的对象是 一个 [`MultiMap`](https://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html) 实例，因为 HTTP 响应头中单个键可以关联多个值。

```java
String contentType = response.headers().get("content-type");
String contentLength = response.headers().get("content-lengh");
```

分块 HTTP 响应还可以包含响应尾(trailer) —— 这实际上是在发送响应体的最后一个（数据）块。

您可使用 [`trailers`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#trailers--)方法读取响应尾，尾数据也是一个 [`MultiMap`](https://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)。

#### 读取请求体

当从报文中读取到响应头时，响应处理器就会被调用。如果收到的HTTP 响应包含响应体（正文），它可能会在响应头被读取后的某个时间以分片的方式到达。在调用响应处理器之前，我们不要等待所有的响应体到达，因为它可能非常大而要等待很长时间、又或者会花费大量内存。

当响应体的某部分（数据）到达时，[`handler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#handler-io.vertx.core.Handler-) 方法绑定的回调函数将会被调用，其中传入的 [`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html) 中包含了响应体的这一分片（部分）内容：

```java
client.getNow("some-uri", response -> {

  response.handler(buffer -> {
    System.out.println("Received a part of the response body: " + buffer);
  });
});
```

若您知道响应体不是很大，并想在处理之前在内存中聚合所有响应体数据，那么您可以自己聚合：

```java
client.getNow("some-uri", response -> {

  // Create an empty buffer
  // 创建空的缓冲区
  Buffer totalBuffer = Buffer.buffer();

  response.handler(buffer -> {
    System.out.println("Received a part of the response body: " + buffer.length());

    totalBuffer.appendBuffer(buffer);
  });

  response.endHandler(v -> {
    // Now all the body has been read
    // 现在所有的响应体都读取了
    System.out.println("Total response body length is " + totalBuffer.length());
  });
});
```

或者当响应已被完全读取时，您可以使用 `bodyHandler` 方法以便读取整个响应体：

```java
client.getNow("some-uri", response -> {

  response.bodyHandler(totalBuffer -> {
    // Now all the body has been read
    // 现在所有的响应体都读取了
    System.out.println("Total response body length is " + totalBuffer.length());
  });
});
```

#### 响应完成处理器

当整个响应体被完全读取或者无响应体的响应头被完全读取时，响应的 [`endHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#endHandler-io.vertx.core.Handler-) 就会被调用。

#### 从响应中读取Cookie

您可以通过 [`cookies`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#cookies--) 方法从响应中获取 Cookie 列表。或者您可以在响应中自己解析`Set-Cookie`头。

#### 30x 重定向处理器

客户端可配置成遵循HTTP 重定向：当客户端接收到 `301`、`302`、`303` 或 `307` 状态代码时，它遵循由 `Location` 响应头提供的重定向，并且响应处理器将传递重定向响应以替代原始响应。

这有个例子：

```java
client.get("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).setFollowRedirects(true).end();
```

重定向策略如下：

* 当接收到 `301`、`302` 或 `303` 状态代码时，使用 GET 方法执行重定向
* 当接收到 `307` 状态代码时，使用相同的 HTTP 方法和缓存的请求体执行重定向

> 警告：随后的重定向会缓存请求体。

默认情况最大的重定向数为 `16`，您可使用 [`setMaxRedirects`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxRedirects-int-) 方法设置。

```java
HttpClient client = vertx.createHttpClient(
    new HttpClientOptions()
        .setMaxRedirects(32));

client.get("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).setFollowRedirects(true).end();
```

没有放之四海而皆准的策略，缺省的重定向策略可能不能满足您的需要。默认重定向策略可使用自定义实现更改：

```java
client.redirectHandler(response -> {

  // Only follow 301 code
  // 仅仅遵循301状态代码
  if (response.statusCode() == 301 && response.getHeader("Location") != null) {

    // Compute the redirect URI
    // 计算重定向URI
    String absoluteURI = resolveURI(response.request().absoluteURI(), response.getHeader("Location"));

    // Create a new ready to use request that the client will use
    // 创建客户端将使用的新的可用请求
    return Future.succeededFuture(client.getAbs(absoluteURI));
  }

  // We don't redirect
  // 我们不需要重定向
  return null;
});
```
这个策略将会处理接收到的原始 [`HttpClientResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html)，并返回 `null` 或 `Future<HttpClientRequest>` 。

* 当返回的是 `null` 时，处理原始响应
* 当返回的是 `Future` 时，请求将在它成功完成后发送
* 当返回的是 `Future` 时，请求失败时调用设置的异常处理器

返回的请求必须是未发送的，这样原始请求处理器才会被发送而且客户端之后才能发送请求。大多数原始请求设置将会传播（拷贝）到新请求中：

* 请求头，除非您已经设置了一些头（包括 [`setHost`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setHost-java.lang.String-)）
* 请求体，除非返回的请求使用了 `GET` 方法
* 响应处理器
* 请求异常处理器
* 请求超时

#### 100-Continue 处理

根据 [HTTP/1.1 规范](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html)，一个客户端可以设置请求头 `Expect: 100-Continue`，并且在发送剩余请求体之前先发送请求头。然后服务器可以通过回复临时响应状态 `Status: 100 (Continue)` 来告诉客户端可以发送请求的剩余部分。

这里的想法是允许服务器在发送大量数据之前授权、接收/拒绝请求，若请求不能被接收，则发送大量数据信息会浪费带宽，并将服务器绑定在读取即将丢弃的无用数据中。

Vert.x 允许您在客户端请求对象中设置一个 [`continueHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#continueHandler-io.vertx.core.Handler-)。它将在服务器发回一个状态 `Status: 100 (Continue)` 时被调用, 同时也表示（客户端）可以发送请求的剩余部分。通常将其与 [`sendHead`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#sendHead--) 结合起来发送请求的头信息。

以下是一个例子：

```java
HttpClientRequest request = client.put("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

request.putHeader("Expect", "100-Continue");

request.continueHandler(v -> {
  // OK to send rest of body
  // 可发送请求体剩余部分
  request.write("Some data");
  request.write("Some more data");
  request.end();
});
```

在服务端，Vert.x HTTP Server可配置成接收到 `Expect: 100-Continue` 头时自动发回 `100 Continue` 临时响应信息。这个可通过 [`setHandle100ContinueAutomatically`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setHandle100ContinueAutomatically-boolean-) 方法来设置。

若您想要决定是否手动发送持续响应，那么此属性可设置成`false`（默认值），然后您可以通过检查头信息并且调用 [`writeContinue`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#writeContinue--) 方法让客户端持续发送请求体：

```java
httpServer.requestHandler(request -> {
  if (request.getHeader("Expect").equalsIgnoreCase("100-Continue")) {

    // Send a 100 continue response
    // 发送100 Continue持续响应
    request.response().writeContinue();

    // The client should send the body when it receives the 100 response
    // 当客户端收到100响应代码则可以发送剩余请求体
    request.bodyHandler(body -> {
      // Do something with body
    });

    request.endHandler(v -> {
      request.response().end();
    });
  }
});
```

您也可以通过直接发送故障状态代码来拒绝该请求：这种情况下，请求体应该被忽略或连接应该被关闭（`100-Continue` 是一个性能提示，并不是逻辑协议约束）：

```java
httpServer.requestHandler(request -> {
  if (request.getHeader("Expect").equalsIgnoreCase("100-Continue")) {

    //
    boolean rejectAndClose = true;
    if (rejectAndClose) {

      // Reject with a failure code and close the connection
      // this is probably best with persistent connection
      // 使用失败码拒绝并关闭这个连接，这可能是长连接最好的
      request.response()
          .setStatusCode(405)
          .putHeader("Connection", "close")
          .end();
    } else {

      // Reject with a failure code and ignore the body
      // this may be appropriate if the body is small
      // 使用失败码拒绝忽略请求体，若体很小，这是适用的
      request.response()
          .setStatusCode(405)
          .end();
    }
  }
});
```

#### 客户端推送

服务器推送(Server Push)是 HTTP/2 的一个新功能，它可以为单个客户端并行发送多个响应。

可以在接收服务器推送的请求/响应的请求上设置一个推送处理器：

```java
HttpClientRequest request = client.get("/index.html", response -> {
  // Process index.html response
  // 处理index.html响应
});

// Set a push handler to be aware of any resource pushed by the server
// 设置一个推送处理器来感知服务器推送的任何资源
request.pushHandler(pushedRequest -> {

  // A resource is pushed for this request
  // 为当前请求推送资源
  System.out.println("Server pushed " + pushedRequest.path());

  // Set an handler for the response
  // 为响应设置处理器
  pushedRequest.handler(pushedResponse -> {
    System.out.println("The response for the pushed request");
  });
});

// End the request
// 结束请求
request.end();
```

若客户端不想收到推送请求，它可重置流：

```java
request.pushHandler(pushedRequest -> {
  if (pushedRequest.path().equals("/main.js")) {
    pushedRequest.reset();
  } else {
    // Handle it
    // 处理逻辑
  }
});
```

若没有设置任何处理器时，任何被推送的流将被客户端自动重置流（错误代码 `8`）。

#### 接收自定义 HTTP/2 帧

HTTP/2 是用于 HTTP 请求/响应模型的具有各种帧的一个帧协议，该协议允许发送和接收其他类型的帧。

要接收自定义帧，您可以在请求中使用 `customFrameHandler`，每次自定义帧到达时就会调用它。以下是一个例子：

```java
response.customFrameHandler(frame -> {

  System.out.println("Received a frame type=" + frame.type() +
      " payload" + frame.payload().toString());
});
```

### 客户端启用压缩

HTTP 客户端支持开箱即用的 HTTP 压缩功能。这意味着客户端可以让远程服务器知道它支持压缩，并且能处理压缩过的响应体（数据）。

HTTP 服务端可以自由地使用自己支持的压缩算法之一进行压缩，也可以在不压缩的情况下将响应体发回。所以这仅仅是 HTTP 服务端的一个可能被随意忽略的提示。

要告诉服务器当前客户端支持哪种压缩，则它（请求头）将包含一个 `Accept-Encoding` 头，其值为可支持的压缩算法，（该值可）支持多种压缩算法。这种情况 Vert.x 将添加以下头：

```java
Accept-Encoding: gzip, deflate
```

服务器将从其中（算法）选择一个，您可以通过服务器发回的响应中响应头 `Content-Encoding` 来检测服务器是否适应这个正文。

若响应体通过 `gzip` 压缩，它将包含例如下边的头：

```java
Content-Encoding: gzip
```

创建客户端时可使用 [`setTryUseCompression`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setTryUseCompression-boolean-) 设置配置项启用压缩。

默认情况压缩被禁用。

### HTTP/1.x Pooling 和 Keep alive

HTTP 的 Keep Alive 允许单个 HTTP 连接用于多个请求。当您向同一台服务器发送多个请求时，可以更加有效使用连接。

对于 HTTP/1.x 版本，HTTP 客户端支持连接池，它允许您重用请求之间的连接。

为了连接池（能）工作，配置客户端时，keep alive 必须通过 [`setKeepAlive`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setKeepAlive-boolean-) 方法设置成`true`。默认值为`true`。

当 keep alive 启用时，Vert.x 将为每一个发送的 HTTP/1.0 请求添加一个 `Connection: Keep-Alive` 头。当 keep alive 禁用时，Vert.x 将为每一个 HTTP/1.1 请求添加一个 `Connection: Close` 头 —— 表示在响应完成后连接将被关闭。

可使用 [`setMaxPoolSize`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxPoolSize-int-) 方法为每个服务器配置连接池的最大连接数。

当启用连接池创建请求时，若存在少于已经为服务器创建的最大连接数，Vert.x 将创建一个新连接，否则直接将请求添加到队列中。

Keep Alive的连接将不会被客户端自动关闭，要关闭它们您可以关闭客户端实例。

或者，您可使用[`setIdleTimeout`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setIdleTimeout-int-)设置空闲时间——在设置的时间内然后没使用的连接将被关闭。请注意空闲超时值以秒为单位而不是毫秒。

### HTTP/1.1 Pipe-lining

客户端还支持连接上的请求管道(pipeline)。

管道意味着在返回一个响应之前，在同一个连接上发送另一个请求，管道不适合所有请求。

若要启用管道，必须调用[`setPipelining`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setPipelining-boolean-)方法，默认管道是禁止的。

当启用管道时，请求可以不等待以前的响应返回而写入到连接。

单个连接的管道请求限制数由 [`setPipeliningLimit`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setPipeliningLimit-int-) 方法设置，此选项定义了发送到服务器的等待响应的最大请求数。这个限制可以确保和同一个服务器的连接分发到客户端的公平性。

### HTTP/2 多路复用

HTTP/2 提倡使用服务器的单一连接，默认情况下，HTTP 客户端针对每个服务器都使用单一连接，同样服务器上的所有流都会复用到对应连接中。

当客户端需要使用连接池并使用超过一个连接时，则可使用[`setHttp2MaxPoolSize`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2MaxPoolSize-int-) 设置。

当您希望限制每个连接的多路复用流数量而使用连接池而不是单个连接时，可使用 [`setHttp2MultiplexingLimit`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2MultiplexingLimit-int-) 设置。

```java
HttpClientOptions clientOptions = new HttpClientOptions().
    setHttp2MultiplexingLimit(10).
    setHttp2MaxPoolSize(3);

// Uses up to 3 connections and up to 10 streams per connection
// 每个连接最多可用三个连接，每个连接可连接10个流
HttpClient client = vertx.createHttpClient(clientOptions);
```

连接的复用限制是在客户端上设置限制单个连接的流数量，如果服务器使用 [`SETTINGS_MAX_CONCURRENT_STREAMS`](https://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html#setMaxConcurrentStreams-long-) 设置了下限，则有效值可以更低。

HTTP/2 连接不会被客户端自动关闭，若要关闭它们，可以调用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#close--) 来关闭客户端实例。

或者，您可以使用[`setIdleTimeout`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setIdleTimeout-int-)设置空闲时间——这个时间内没有使用的任何连接将被关闭，注意，空闲时间以秒为单位，不是毫秒。

### HTTP 连接

[`HttpConnection`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html) 接口提供了处理HTTP 连接事件、生命周期、设置的API。

HTTP/2 实现了完整的 [`HttpConnection`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html) API。

HTTP/1.x 实现了 [`HttpConnection`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html) 中的部分API：仅关闭操作，实现了关闭处理器和异常处理器。该协议并不提供其他操作的语义。

#### 服务端连接

[`connection`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#connection--) 方法会返回服务器上的请求连接：

```java
HttpConnection connection = request.connection();
```

可以在服务器上设置连接处理器，任意连接传入时可得到通知：

```java
HttpServer server = vertx.createHttpServer(http2Options);

server.connectionHandler(connection -> {
  System.out.println("A client connected");
});
```

#### 客户端连接

[`connection`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#connection--) 方法会返回客户端上的连接请求：

```java
HttpConnection connection = request.connection();
```

可以在请求上设置连接处理器在连接发生时通知：

```java
request.connectionHandler(connection -> {
  System.out.println("Connected to the server");
});
```

#### 连接配置

HTTP/2 由 [`Http2Settings`](https://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html) 数据对象来配置。

每个 Endpoint 都必须遵守连接另一端的发送设置。

当建立连接时，客户端和服务器交换初始配置，初始设置由客户端上的 [`setInitialSettings`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setInitialSettings-io.vertx.core.http.Http2Settings-) 和服务器上的 [`setInitialSettings`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setInitialSettings-io.vertx.core.http.Http2Settings-) 方法配置。

连接建立后可随时更改设置：

```java
connection.updateSettings(new Http2Settings().setMaxConcurrentStreams(100));
```

由于远程方应该确认接收者的配置更新，也有可能在回调中接收确认通知：

```java
connection.updateSettings(new Http2Settings().setMaxConcurrentStreams(100), ar -> {
  if (ar.succeeded()) {
    System.out.println("The settings update has been acknowledged ");
  }
});
```

相反，在收到新的远程设置时会通知 [`remoteSettingsHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#remoteSettingsHandler-io.vertx.core.Handler-)：

```java
connection.remoteSettingsHandler(settings -> {
  System.out.println("Received new settings");
});
```

> 请注意：此功能仅适用于 HTTP/2 协议。

#### 连接 Ping

HTTP/2 连接 ping 对于确定连接往返时间或检查连接有效性很有用：[`ping`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#ping-io.vertx.core.buffer.Buffer-io.vertx.core.Handler-) 发送 `PING` 帧到远端：

```java
Buffer data = Buffer.buffer();
for (byte i = 0;i < 8;i++) {
  data.appendByte(i);
}
connection.ping(data, pong -> {
  System.out.println("Remote side replied");
});
```

当接收到 `PING` 帧时，Vert.x 将自动发送确认，可设置处理器当收到 ping 帧时发送通知调用处理器：

```java
connection.pingHandler(ping -> {
  System.out.println("Got pinged by remote side");
});
```

处理器只是接到通知，确认被发送，这个功能旨在基于 HTTP/2 协议之上实现。

> 请注意：此功能仅适用于 HTTP/2 协议。

#### 连接关闭/GOAWAY

调用 [`shutdown`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdown--) 方法将发送 `GOAWAY` 帧到远程的连接，要求其停止创建流：客户端将停止发送新请求，并且服务器将停止推送响应。发送 `GOAWAY` 帧后，连接将等待一段时间（默认为30秒），直到所有当前流关闭和连接关闭。

```java
connection.shutdown();
```

[`shutdownHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdownHandler-io.vertx.core.Handler-) 通知何时关闭所有流，连接尚未关闭。

有可能只需发送 `GOAWAY` 帧，和关闭主要的区别在于它将只是告诉远程连接停止创建新流，而没有计划关闭连接：

```java
connection.goAway(0);
```

相反，也可以在收到 `GOAWAY` 时收到通知：

```java
connection.goAwayHandler(goAway -> {
  System.out.println("Received a go away frame");
});
```

当所有当前流已经关闭并且可关闭连接时，[`shutdownHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdownHandler-io.vertx.core.Handler-) 将被调用：

```java
connection.goAway(0);
connection.shutdownHandler(v -> {

  // All streams are closed, close the connection
  // 所有流被关闭，连接也关闭
  connection.close();
});
```

当接收到 `GOAWAY` 时也适用。

> 请注意：此功能仅适用于HTTP/2协议。

#### 连接关闭

您可以通过 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#close--) 方法关闭连接：

* 对于 HTTP/1.x 来说，它会关闭底层的 Socket
* 对于 HTTP/2 来说，它将执行无延迟关闭，`GOAWAY` 帧将会在连接关闭之前被发送

连接关闭时 [`closeHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#closeHandler-io.vertx.core.Handler-) 将发出通知。

### HttpClient 使用说明

`HttpClient`可以在一个 Verticle 中使用或者嵌入使用。

在 Verticle 中使用时，Verticle **应该使用自己的客户端实例**。

一般来说，不应该在不同的 Vert.x 上下文环境之间共享客户端，因为它可能导致不可预知的意外。例如：保持活动连接将在打开连接的请求上下文环境调用客户端处理器，后续请求将使用相同上下文环境。

当这种情况发生时，Vert.x会检测到并记录下边警告：

```
Reusing a connection with a different context: an HttpClient is probably shared between different Verticles
```

`HttpClient` 可以嵌套在非 Vert.x 线程中，如单元测试或纯Java的 `main` 线程中：客户端处理器将被不同的Vert.x 线程和上下文调用，这样的上下文会根据需要创建。对于生产环境，不推荐这样使用。

### 水平扩展 - 服务端共享

当多个 HTTP 服务端在同一个端口上监听时，Vert.x 会使用轮询策略来管理请求处理。

我们用 Verticle 来创建 HTTP 服务端，如：

**io.vertx.examples.http.sharing.HttpServerVerticle**

```java
vertx.createHttpServer().requestHandler(request -> {
  request.response().end("Hello from server " + this);
}).listen(8080);
```

这个服务正在监听 `8080` 端口。所以，当这个 Verticle 被实例化多次，如运行以下命令：

```
vertx run io.vertx.examples.http.sharing.HttpServerVerticle -instances 2
```

将会发生什么？如果两个 Verticle 都绑定到同一个端口，您将收到一个 Socket 异常。幸运的是，Vert.x 可以为您处理这种情况。在与现有服务端相同的主机和端口上部署另一个服务器时，实际上并不会尝试创建在同一主机/端口上监听的新服务端，它只绑定一次到Socket，当接收到请求时，会按照轮询策略调用服务端的请求处理函数。

我们现在想象一个客户端，如下：

```java
vertx.setPeriodic(100, (l) -> {
  vertx.createHttpClient().getNow(8080, "localhost", "/", resp -> {
    resp.bodyHandler(body -> {
      System.out.println(body.toString("ISO-8859-1"));
    });
  });
});
```

Vert.x 将请求顺序委托给其中一个服务器：

```
Hello from i.v.e.h.s.HttpServerVerticle@1
Hello from i.v.e.h.s.HttpServerVerticle@2
Hello from i.v.e.h.s.HttpServerVerticle@1
Hello from i.v.e.h.s.HttpServerVerticle@2
...
```

因此，服务器可直接扩展可用的核，而每个 Vert.x 中的 Verticle 实例仍然严格使用单线程，您不需要像编写负载均衡器那样使用任何特殊技巧去编写，以便在多核机器上扩展服务器。

### 使用 HTTPS

Vert.x 的 HTTP 服务端和客户端可以配置成和网络服务器完全相同的方式使用 HTTPS。

有关详细信息，请参阅 [配置网络服务器以使用 SSL](https://vertx.io/docs/vertx-core/java/#ssl) 章节。

SSL可以通过每个请求的 [`RequestOptions`](https://vertx.io/docs/apidocs/io/vertx/core/http/RequestOptions.html) 来启用/禁用，或在指定模式时调用 [`requestAbs`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#requestAbs-io.vertx.core.http.HttpMethod-java.lang.String-)：

```java
client.getNow(new RequestOptions()
    .setHost("localhost")
    .setPort(8080)
    .setURI("/")
    .setSsl(true), response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

[`setSsl`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setSsl-boolean-) 设置将用作客户端默认配置。

[`setSsl`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setSsl-boolean-) 将覆盖默认客户端设置：

* 即使客户端配置成使用 SSL/TLS，该值设置成`false`将禁用SSL/TLS。
* 即使客户端配置成不使用 SSL/TLS，该值设置成`true`将启用SSL/TLS，实际的客户端SSL/TLS（如受信、密钥/证书、密码、ALPN 等）将被重用。

同样，[`requestAbs`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#requestAbs-io.vertx.core.http.HttpMethod-java.lang.String-) 方法也会（在调用时）覆盖默认客户端设置。

### WebSocket

[WebSocket](http://en.wikipedia.org/wiki/WebSocket) 是一种Web技术，可以在 HTTP 服务端和 HTTP 客户端（通常是浏览器）之间实现全双工 Socket 连接。

Vert.x HTTP 客户端和服务端都支持 WebSocket。

#### 服务端 WebSocket

在服务端处理 WebSocket 有两种方法。

- WebSocket Handler

第一种方法需要在服务端实例上提供一个 [`websocketHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#websocketHandler-io.vertx.core.Handler-)。

当对服务端创建 WebSocket 连接时，Vert.x 将向 `Handler`传入一个 [`ServerWebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html) 实例，在其中去处理它。

```java
server.websocketHandler(websocket -> {
  System.out.println("Connected!");
});
```

您可以调用[`reject`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#reject--) 方法来拒绝一个 WebSocket。

```java
server.websocketHandler(websocket -> {
  if (websocket.path().equals("/myapi")) {
    websocket.reject();
  } else {
    // Do something
    // 做一些事
  }
});
```

- 转换到 WebSocket

处理 WebSocket 的第二种方法是处理从客户端发送的HTTP升级请求，调用服务器请求对象的 [`upgrade`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#upgrade--)方法：

```java
server.requestHandler(request -> {
  if (request.path().equals("/myapi")) {

    ServerWebSocket websocket = request.upgrade();
    // Do something
    // 做一些事
  } else {
    // Reject
    // 拒绝
    request.response().setStatusCode(400).end();
  }
});
```

- 服务端 WebSocket

[`ServerWebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html) 实例能够让您读取在WebSocket 握手中的HTTP 请求的 [`headers`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#headers--)、[`path`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#path--)、[`query`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#query--) 和 [`URI`](https://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#uri--)。

#### 客户端 WebSocket

Vert.x 的 [`HttpClient`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 支持 WebSocket。

您可以调用其中任意一个 [`websocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#websocket-io.vertx.core.http.RequestOptions-io.vertx.core.Handler-) 方法创建 WebSocket 连接到服务端，并提供回调函数。

当连接建立时，处理器将被调用并且传入 [`WebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html)实例：

```java
client.websocket("/some-uri", websocket -> {
  System.out.println("Connected!");
});
```

#### 向 WebSocket 写入消息

若您想将一个 WebSocket 消息写入 WebSocket，可使用[`writeBinaryMessage`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeBinaryMessage-io.vertx.core.buffer.Buffer-) 方法或 [`writeTextMessage`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeTextMessage-java.lang.String-) 方法来执行该操作：

```java
Buffer buffer = Buffer.buffer().appendInt(123).appendFloat(1.23f);
websocket.writeBinaryMessage(buffer);

// Write a simple text message
// 写一个简单文本消息
String message = "hello";
websocket.writeTextMessage(message);
```

若WebSocket 消息大于使用[`setMaxWebsocketFrameSize`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxWebsocketFrameSize-int-) 设置的W ebSocket 的帧的最大值，则Vert.x在将其发送到报文之前将其拆分为多个WebSocket 帧。

#### 向 WebSocket 写入帧

WebSocket 消息可以由多个帧组成，在这种情况下，第一帧是二进制或文本帧（text | binary），后边跟着零个或多个 *连续* 帧。

消息中的最后一帧标记成 *final*。

要发送多个帧组成的消息，请使用 [`WebSocketFrame.binaryFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#binaryFrame-io.vertx.core.buffer.Buffer-boolean-)、[`WebSocketFrame.textFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#textFrame-java.lang.String-boolean-) 或 [`WebSocketFrame.continuationFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#continuationFrame-io.vertx.core.buffer.Buffer-boolean-) 方法创建帧，并使用 [`writeFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFrame-io.vertx.core.http.WebSocketFrame-) 方法将其写入WebSocket。

以下是二进制帧的示例：

```java
WebSocketFrame frame1 = WebSocketFrame.binaryFrame(buffer1, false);
websocket.writeFrame(frame1);

WebSocketFrame frame2 = WebSocketFrame.continuationFrame(buffer2, false);
websocket.writeFrame(frame2);

// Write the final frame
// 写最终帧
WebSocketFrame frame3 = WebSocketFrame.continuationFrame(buffer2, true);
websocket.writeFrame(frame3);
```

许多情况下，您只需要发送一个包含了单个最终帧的 WebSocket 消息，因此我们提供了 [`writeFinalBinaryFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFinalBinaryFrame-io.vertx.core.buffer.Buffer-) 和 [`writeFinalTextFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFinalTextFrame-java.lang.String-) 这两个快捷方法。

下边是示例：

```java
websocket.writeFinalTextFrame("Geronimo!");

// Send a websocket messages consisting of a single final binary frame:
// 发送由单个最终二进制帧组成的websocket消息：
Buffer buff = Buffer.buffer().appendInt(12).appendString("foo");

websocket.writeFinalBinaryFrame(buff);
```

#### 从 WebSocket 读取帧

要 从WebSocket 读取帧，您可以使用[`frameHandler`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#frameHandler-io.vertx.core.Handler-) 方法。

当帧到达时，会传入一个[`WebSocketFrame`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html)实例给帧处理器，并调用它，例如：

```java
websocket.frameHandler(frame -> {
  System.out.println("Received a frame of size!");
});
```

#### 关闭 WebSocket

处理完成之后，请使用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketBase.html#close--) 方法关闭 WebSocket 连接。

#### 流式 WebSocket

[WebSocket](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html) 实例也是[ReadStream](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)和[WriteStream](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 的实现类，因此可以和泵(pump)一起使用。

当使用 WebSocket 作为可写流或可读流时，它只能用于不分割多个帧的二进制帧一起使用的 WebSocket 连接。

### 使用 HTTP/HTTPS 连接代理

HTTP 客户端支持通过HTTP 代理（如Squid）或 *SOCKS4a* 或 *SOCKS5* 代理访问 HTTP/HTTPS 的 URL。CONNECT 协议使用 HTTP/1.x，但可以连接到 HTTP/1.x 和 HTTP/2 服务器。

到 `h2c`（未加密HTTP/2服务器）的连接可能不受 HTTP 代理支持，因为代理仅支持 HTTP/1.1。

您可以通过 [`HttpClientOptions`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html) 中的 [`ProxyOptions`](https://vertx.io/docs/apidocs/io/vertx/core/net/ProxyOptions.html) 对象配置来配置代理（包括代理类型、主机名、端口和可选用户名和密码）。

以下是使用 HTTP 代理的例子：

```java
HttpClientOptions options = new HttpClientOptions()
    .setProxyOptions(new ProxyOptions().setType(ProxyType.HTTP)
        .setHost("localhost").setPort(3128)
        .setUsername("username").setPassword("secret"));
HttpClient client = vertx.createHttpClient(options);
```

当客户端连接到HTTP URL时，它连接到代理服务器，并在HTTP请求中提供完整URL（`GET http://www.somehost.com/path/file.html HTTP/1.1`）。

当客户端连接到HTTPS URL时，它要求代理使用 CONNECT 方法创建到远程主机的通道。

对于 SOCKS5 代理：

```java
HttpClientOptions options = new HttpClientOptions()
    .setProxyOptions(new ProxyOptions().setType(ProxyType.SOCKS5)
        .setHost("localhost").setPort(1080)
        .setUsername("username").setPassword("secret"));
HttpClient client = vertx.createHttpClient(options);
```

DNS 解析会一直在代理服务器上执行。为了实现 SOCKS4 客户端的功能，需要先在本地解析 DNS 地址。

#### Verticle 中自动清理

如果您是在 Verticle 内部创建的 HTTP 服务端和客户端，则在撤销该Verticle时，它们将自动关闭。

## 使用 Vert.x 共享数据

共享数据（Shared Data）包含的功能允许您可以安全地在应用程序的不同部分之间、同一 Vert.x 实例中的不同应用程序之间或集群中的不同 Vert.x 实例之间安全地共享数据。

共享数据包括本地共享Map、分布式、集群范围Map、异步集群范围锁和异步集群范围计数器。

> 重要提示：分布式数据结构的行为取决于您使用的集群管理器，网络分区面临的备份（复制）和行为由集群管理器和它的配置来定义。请参阅集群管理器文档以及底层框架手册。

### 本地共享Map

本地共享Map [`LocalMap`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/LocalMap.html) 允许您在同一个 Vert.x 实例中的不同 Event Loop（如不同的 Verticle 中）之间安全共享数据。

本地共享Map仅允许将某些数据类型作为键值和值，这些类型必须是不可变的，或可以像[`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)那样复制某些其他类型。在后一种情况中，键/值将被复制，然后再放到Map中。

这样，我们可以确保在Vert.x应用程序不同线程之间 **没有共享访问可变状态**，因此您不必担心需要通过同步访问来保护该状态。

以下是使用 `LocalMap` 的示例：

```java
SharedData sd = vertx.sharedData();

LocalMap<String, String> map1 = sd.getLocalMap("mymap1");

// String是不可变的，所以不需要复制
map1.put("foo", "bar"); // Strings are immutable so no need to copy

LocalMap<String, Buffer> map2 = sd.getLocalMap("mymap2");

// Buffer将会在添加到Map之前拷贝
map2.put("eek", Buffer.buffer().appendInt(123)); // This buffer will be copied before adding to map

// Then... in another part of your application:
// 之后，您的应用另外一部分
map1 = sd.getLocalMap("mymap1");

String val = map1.get("foo");

map2 = sd.getLocalMap("mymap2");

Buffer buff = map2.get("eek");
```

### 集群范围异步Map

集群范围异步Map(Cluster-wide asynchronous maps)允许从集群的任何节点将数据放到 Map 中，并从任何其他节点读取。

这使得它们对于托管Vert.x Web应用程序的服务器场中的会话状态存储非常有用。

您可以使用 [`getClusterWideMap`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getClusterWideMap-java.lang.String-io.vertx.core.Handler-) 方法获取[`AsyncMap`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html)的实例。

获取Map的过程是异步的，返回结果可以传给您指定的处理器中。以下是一个例子：

```java
SharedData sd = vertx.sharedData();

sd.<String, String>getClusterWideMap("mymap", res -> {
  if (res.succeeded()) {
    AsyncMap<String, String> map = res.result();
  } else {
    // Something went wrong!
    // 出现一些错误
  }
});
```

#### 将数据放入Map

您可以使用 [`put`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html#put-java.lang.Object-java.lang.Object-io.vertx.core.Handler-) 方法将数据放入Map。`put`方法是异步的，一旦完成它会通知处理器：

```java
map.put("foo", "bar", resPut -> {
  if (resPut.succeeded()) {
    // Successfully put the value
    // 成功放入值
  } else {
    // Something went wrong!
    // 出了些问题
  }
});
```

#### 从Map读取数据

您可以使用 [`get`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html#get-java.lang.Object-io.vertx.core.Handler-) 方法从Map读取数据。`get` 方法也是异步的，一段时间过后它会通知处理器并传入结果。

```java
map.get("foo", resGet -> {
  if (resGet.succeeded()) {
    // Successfully got the value
    // 成功读取值
    Object val = resGet.result();
  } else {
    // Something went wrong!
    // 出了些问题
  }
});
```

#### 其他Map操作

您还可以从异步Map中删除条目、清除Map、读取它的大小。

有关更多信息，请参阅 [API 文档](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html)。

### 集群范围锁

集群范围锁（[`Lock`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html)）允许您在集群中获取独占锁 —— 当您想要在任何时间只在集群一个节点上执行某些操作或访问资源时，这很有用。

集群范围锁具有异步API，它和大多数等待锁释放的阻塞调用线程的API锁不相同。

可使用 [`getLock`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getLock-java.lang.String-io.vertx.core.Handler-) 方法获取锁。

它不会阻塞，但当锁可用时，将 [`Lock`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html) 的实例传入处理器并调用它，表示您现在拥有该锁。

若您拥有的锁没有其他调用者，集群上的任何地方都可以获得该锁。

当您用完锁后，您可以调用 [`release`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html#release--) 方法来释放它，以便另一个调用者可获得它。

```java
sd.getLock("mylock", res -> {
  if (res.succeeded()) {
    // Got the lock!
    // 获得锁
    Lock lock = res.result();

    // 5 seconds later we release the lock so someone else can get it
	// 5秒后我们释放该锁其他人可以得到它
    vertx.setTimer(5000, tid -> lock.release());

  } else {
    // Something went wrong
	// 出了些问题
  }
});
```

您可以为锁设置一个超时，若在超时时间期间无法获取锁，将会进入失败状态，处理器会去处理对应的异常：

```java
sd.getLockWithTimeout("mylock", 10000, res -> {
  if (res.succeeded()) {
    // Got the lock!
	// 获得锁
    Lock lock = res.result();

  } else {
    // Failed to get lock
	// 锁获取失败
  }
});
```

### 集群范围计数器

很多时候我们需要在集群范围内维护一个原子计数器。

您可以用 [`Counter`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/Counter.html) 来做到这一点。

您可以通过 [`getCounter`](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getCounter-java.lang.String-io.vertx.core.Handler-) 方法获取一个实例：

```java
sd.getCounter("mycounter", res -> {
  if (res.succeeded()) {
    Counter counter = res.result();
  } else {
    // Something went wrong!
	// 出了些问题
  }
});
```

一旦您有了一个实例，您可以获取当前的计数，以原子方式递增、递减，并使用各种方法添加一个值。

有更多信息，请参阅 [API 文档](https://vertx.io/docs/apidocs/io/vertx/core/shareddata/Counter.html)。

## 访问文件系统

Vert.x 中的 [`FileSystem`](https://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html) 对象提供了许多操作文件系统的方法。

每个Vert.x 实例有一个文件系统对象，您可以使用 [`fileSystem`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#fileSystem--) 方法获取它。

每个操作都提供了阻塞和非阻塞版本，其中非阻塞版本接受一个处理器 `Handler`，当操作完成或发生错误时调用该处理器。

以下是文件异步拷贝的示例：

```java
FileSystem fs = vertx.fileSystem();

// Copy file from foo.txt to bar.txt
// 从foo.txt拷贝到bar.txt
fs.copy("foo.txt", "bar.txt", res -> {
    if (res.succeeded()) {
        // Copied ok!
		// 拷贝完成
    } else {
        // Something went wrong
		// 出了些问题
    }
});
```

阻塞版本的方法名为 `xxxBlocking`，它要么返回结果或直接抛出异常。很多情况下，一些潜在的阻塞操作可以快速返回（这取决于操作系统和文件系统），这就是我们为什么提供它。但是强烈建议您在 Event Loop 中使用它之前测试使用它们究竟需要耗费多长时间，以避免打破黄金法则。

以下是使用阻塞 API的拷贝示例：

```java
FileSystem fs = vertx.fileSystem();

// Copy file from foo.txt to bar.txt synchronously
// 同步拷贝从foo.txt到bar.txt
fs.copyBlocking("foo.txt", "bar.txt");
```

Vert.x 文件系统支持诸如 copy、move、truncate、chmod 和许多其他文件操作。我们不会在这里列出所有内容，请参考 [API 文档](https://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html) 获取完整列表。

让我们看看使用异步方法的几个例子：

```java
Vertx vertx = Vertx.vertx();

// Read a file
// 读取文件
vertx.fileSystem().readFile("target/classes/readme.txt", result -> {
    if (result.succeeded()) {
        System.out.println(result.result());
    } else {
        System.err.println("Oh oh ..." + result.cause());
    }
});

// Copy a file
// 拷贝文件
vertx.fileSystem().copy("target/classes/readme.txt", "target/classes/readme2.txt", result -> {
    if (result.succeeded()) {
        System.out.println("File copied");
    } else {
        System.err.println("Oh oh ..." + result.cause());
    }
});

// Write a file
// 写文件
vertx.fileSystem().writeFile("target/classes/hello.txt", Buffer.buffer("Hello"), result -> {
    if (result.succeeded()) {
        System.out.println("File written");
    } else {
        System.err.println("Oh oh ..." + result.cause());
    }
});

// Check existence and delete
// 检测存在以及删除
vertx.fileSystem().exists("target/classes/junk.txt", result -> {
    if (result.succeeded() && result.result()) {
        vertx.fileSystem().delete("target/classes/junk.txt", r -> {
            System.out.println("File deleted");
        });
    } else {
        System.err.println("Oh oh ... - cannot delete the file: " + result.cause());
    }
});
```

### 异步文件访问

Vert.x提供了异步文件访问的抽象，允许您操作文件系统上的文件。

您可以像下边代码打开一个 [`AsyncFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html) ：

```java
OpenOptions options = new OpenOptions();
fileSystem.open("myfile.txt", options, res -> {
    if (res.succeeded()) {
        AsyncFile file = res.result();
    } else {
        // Something went wrong!
		// 出了些问题
    }
});
```

`AsyncFile` 实现了 `ReadStream` 和 `WriteStream` 接口，因此您可以将文件和其他流对象配合 *泵* 工作，如 `NetSocket`、HTTP 请求和响应和 WebSocket 等。

它们还允许您直接读写。

#### 随机访问写

要使用`AsyncFile`进行随机访问写，请使用 [`write`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#write-io.vertx.core.buffer.Buffer-long-io.vertx.core.Handler-) 方法。

这个方法的参数有：

* `buffer`：要写入的缓冲
* `position`：一个整数指定在文件中写入缓冲的位置，若位置大于或等于文件大小，文件将被扩展以适应偏移的位置
* `handler`：结果处理器

这是随机访问写的示例：

```java
Vertx vertx = Vertx.vertx();
vertx.fileSystem().open("target/classes/hello.txt", new OpenOptions(), result -> {
    if (result.succeeded()) {
        AsyncFile file = result.result();
        Buffer buff = Buffer.buffer("foo");
        for (int i = 0; i < 5; i++) {
            file.write(buff, buff.length() * i, ar -> {
                if (ar.succeeded()) {
                    System.out.println("Written ok!");
                    // etc
					// 等
                } else {
                    System.err.println("Failed to write: " + ar.cause());
                }
            });
        }
    } else {
        System.err.println("Cannot open file " + result.cause());
    }
});
```

#### 随机访问读

要使用`AsyncFile`进行随机访问读，请使用 [`read`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#read-io.vertx.core.buffer.Buffer-int-long-int-io.vertx.core.Handler-) 方法。

该方法的参数有：

* `buffer`：读取数据的 Buffer
* `offset`：读取数据将被放到 Buffer 中的偏移量
* `position`：从文件中读取数据的位置
* `length`：要读取的数据的字节数
* `handler`：结果处理器

一下是随机访问读的示例：

```java
Vertx vertx = Vertx.vertx();
vertx.fileSystem().open("target/classes/les_miserables.txt", new OpenOptions(), result -> {
    if (result.succeeded()) {
        AsyncFile file = result.result();
        Buffer buff = Buffer.buffer(1000);
        for (int i = 0; i < 10; i++) {
            file.read(buff, i * 100, i * 100, 100, ar -> {
                if (ar.succeeded()) {
                    System.out.println("Read ok!");
                } else {
                    System.err.println("Failed to write: " + ar.cause());
                }
            });
        }
    } else {
        System.err.println("Cannot open file " + result.cause());
    }
});
```

#### 打开选项

打开 `AsyncFile` 时，您可以传递一个 [`OpenOptions`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html) 实例，这些选项描述了访问文件的行为。例如：您可使用 [`setRead`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setRead-boolean-)、[`setWrite`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setWrite-boolean-) 和 [`setPerm`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setPerms-java.lang.String-) 方法配置文件访问权限。

若打开的文件已经存在，则可以使用 [`setCreateNew`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setCreateNew-boolean-) 和 [`setTruncateExisting`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setTruncateExisting-boolean-) 配置对应行为。

您可以使用 [`setDeleteOnClose`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setDeleteOnClose-boolean-) 标记在关闭时或JVM停止时要删除的文件。

#### 将数据刷新到底层存储

在 `OpenOptions` 中，您可以使用 [`setDsync`](https://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setDsync-boolean-) 方法在每次写入时启用/禁用内容的自动同步。这种情况下，您可以使用 [`flush`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#flush--) 方法手动刷新OS缓存中的数据写入。

该方法也可使用一个处理器来调用，这个处理器在 `flush` 完成时被调用。

#### 将 AsyncFile 作为 ReadStream 和 WriteStream

`AsyncFile`实现了 `ReadStream` 和 `WriteStream` 接口。您可以使用泵将数据与其他读取和写入流进行数据*泵*送。例如，这会将内容复制到另外一个`AsyncFile`：

```java
Vertx vertx = Vertx.vertx();
final AsyncFile output = vertx.fileSystem().openBlocking("target/classes/plagiary.txt", new OpenOptions());

vertx.fileSystem().open("target/classes/les_miserables.txt", new OpenOptions(), result -> {
    if (result.succeeded()) {
        AsyncFile file = result.result();
        Pump.pump(file, output).start();
        file.endHandler((r) -> {
            System.out.println("Copy done");
        });
    } else {
        System.err.println("Cannot open file " + result.cause());
    }
});
```

您还可以使用泵将文件内容写入到HTTP 响应中，或者写入任意 `WriteStream`。

#### 从 Classpath 访问文件

当Vert.x找不到文件系统上的文件时，它尝试从类路径中解析该文件。请注意，类路径的资源路径不以 `/` 开头。

由于Java不提供对类路径资源的异步方法，所以当类路径资源第一次被访问时，该文件将复制到工作线程中的文件系统。当第二次访问相同资源时，访问的文件直接从（工作线程的）文件系统提供。即使类路径资源发生变化（例如开发系统中），也会提供原始内容。

您可以将系统属性`vertx.disableFileCaching`设置为`true`，禁用此（文件）缓存行为。文件缓存的路径默认为`.vertx`，它可以通过设置系统属性`vertx.cacheDirBase`进行自定义。

您还可以通过系统属性`vertx.disableFileCPResolving`设置为`true`来禁用整个类路径解析功能。

> 请注意：*当加载`io.vertx.core.impl.FileResolver`类时，这些系统属性将被评估一次，因此，在加载此类之前应该设置这些属性，或者在启动它时作为JVM系统属性来设置。*

#### 关闭 AsyncFile

您可调用 [`close`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#close--) 方法来关闭 `AsyncFile`。关闭是异步的，如果希望在关闭过后收到通知，您可指定一个处理器作为函数（`close`）参数传入。

## 数据报套接字（UDP）

在Vert.x中使用用户数据报协议（UDP）就是小菜一碟。

UDP是无连接的传输，这意味着您与远程客户端没有建立持续的连接。

所以，您发送和接收的数据包都要包含有远程的地址。

除此之外，UDP不像TCP的使用那样安全，这也就意味着不能保证发送的数据包一定会被对应的接收端（Endpoint）接收。

唯一的保证是，它既不会被完全接收到，也不会完全不被接收到，即只有部分会被接收到。

因为每一个数据包将会作为一个包发送，所以在通常情况下您不能发送大于网络接口的最大传输单元（MTU）的数据包。

但是要注意，即使数据包尺寸小于MTU，它仍然可能会发送失败。

它失败的尺寸取决于操作系统等（其他原因），所以按照经验法则就是尝试发送小数据包。

依照UDP的本质，它最适合一些允许丢弃数据包的应用（如监视应用程序）。

其优点是与TCP相比具有更少的开销，而且可以由`NetServer`和`NetClient`处理（参考前文）。

### 创建 DatagramSocket

要使用UDP，您首先要创建一个 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html) 实例，无论您是要仅仅发送数据或者收发数据，这都是一样的。

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
```

返回的 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html) 实例不会绑定到特定端口，如果您只想发送数据（如作为客户端）的话，这是没问题的，但更多详细的内容在下一节。

### 发送数据报包

如上所述，用户数据报协议（UDP）将数据分组发送给远程对等体，但是以不持续的方式来传送到它们。

这意味着每个数据包都可以发送到不同的远程对等体。

发送数据包很容易，如下所示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
Buffer buffer = Buffer.buffer("content");
// 发送Buffer
socket.send(buffer, 1234, "10.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
// 发送一个字符串
socket.send("A string used as content", 1234, "10.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
```

### 接收数据报包

若您想要接收数据包，则您需要调用 `listen(...)` 方法绑定 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)。

这样您就可以接收到被发送至 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html) 所监听的地址和端口的 [`DatagramPacket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html)。

除此之外，您还要设置一个`Handler`，每接收到一个 [`DatagramPacket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html) 时它都会被调用。

[`DatagramPacket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html) 有以下方法：

* [`sender`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html#sender--)：表示数据发送方的`InetSocketAddress`。
* [`data`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html#data--)：保存接收数据的`Buffer`。

当您需要监听一个特定地址和端口时，您可以像下边这样：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
  if (asyncResult.succeeded()) {
    socket.handler(packet -> {     
	  // 对包进行处理
   });
  } else {
    System.out.println("Listen failed" + asyncResult.cause());
  }
});
```

注意，即使 `AsyncResult` 成功，它只意味着它可能已经写入了网络堆栈，但不保证它已经到达或者将到达远端。

若您需要这样的保证，您可在TCP之上建立一些握手逻辑。

### 多播

#### 发送多播数据包

多播允许多个Socket接收相同的数据包，该目标可以通过加入到同一个可发送数据包的多播组来实现。

我们将在下一节中介绍如何加入多播组，从而接收数据包。

现在让我们专注于如何发送多播报文，发送多播报文与发送普通数据报报文没什么不同。

唯一的区别是您可以将多播组的地址传递给 `send` 方法发送出去。

如下所示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
Buffer buffer = Buffer.buffer("content");
// Send a Buffer to a multicast address
// 发送Buffer到多播地址
socket.send(buffer, 1234, "230.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
```

所有已经加入多播组 `230.0.0.1` 的Socket都将收到该报文。

#### 接收多播数据包

若要接收特定多播组的数据包，您需要通过调用 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html) 的`listen(...)`方法来绑定一个地址并且加入多播组，并加入多播组。

这样，您将能够接收到被发送到 [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html) 所监听的地址和端口的数据报，同时也可以接收被发送到该多播组的数据报。

除此之外，您还可设置一个处理器，它在每次接收到 `DatagramPacket` 时会被调用。

[`DatagramPacket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html) 有以下方法：

* `sender()`：表示数据报发送方的`InetSocketAddress`
* `data()`：保存接收数据的`Buffer`

因此，要监听指定的地址和端口、并且接收多播组`230.0.0.1`的数据报，您将执行如下操作：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
  if (asyncResult.succeeded()) {
    socket.handler(packet -> {    
	  // 对数据包进行处理
    });

	// 加入多播组
    socket.listenMulticastGroup("230.0.0.1", asyncResult2 -> {
        System.out.println("Listen succeeded? " + asyncResult2.succeeded());
    });
  } else {
    System.out.println("Listen failed" + asyncResult.cause());
  }
});
```

#### 取消订阅/离开多播组

有时候您想只在特定时间内接收多播组的数据包。

这种情况下，您可以先监听他们，之后再取消监听。

如下所示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
    if (asyncResult.succeeded()) {
      socket.handler(packet -> {
        // Do something with the packet
		// 处理数据报
     });

      // join the multicast group
	  // 加入多播组
      socket.listenMulticastGroup("230.0.0.1", asyncResult2 -> {
          if (asyncResult2.succeeded()) {
            // will now receive packets for group
			// 现在将接收组的数据包
            // do some work
			// 做一些工作
            socket.unlistenMulticastGroup("230.0.0.1", asyncResult3 -> {
              System.out.println("Unlisten succeeded? " + asyncResult3.succeeded());
            });
          } else {
            System.out.println("Listen failed" + asyncResult2.cause());
          }
      });
    } else {
      System.out.println("Listen failed" + asyncResult.cause());
    }
});
```

#### 屏蔽多播

除了取消监听一个多播地址以外，也可以做到屏蔽指定发送者地址的多播。

请注意这仅适用于某些操作系统和内核版本，所以请检查操作系统文档看是它是否支持。

这是专家级别的技巧。

要屏蔽来自特定地址的多播，您可以在 `DatagramSocket` 上调用 `blockMulticastGroup(...)`，如下所示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());

// Some code
// 一些代码

// This would block packets which are send from 10.0.0.2
// 这将阻止从10.0.0.2发送的数据包
socket.blockMulticastGroup("230.0.0.1", "10.0.0.2", asyncResult -> {
  System.out.println("block succeeded? " + asyncResult.succeeded());
});
```

#### DatagramSocket 属性

当创建[`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)时，您可以通过[`DatagramSocketOptions`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html)对象来设置多个属性以更改它的功能。这些（属性）列在这儿：

* [`setSendBufferSize`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setSendBufferSize-int-)以字节为单位设置发送缓冲区的大小。
* [`setReceiveBufferSize`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setReceiveBufferSize-int-)设置TCP接收缓冲区大小（以字节为单位）。
* [`setReuseAddress`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setReuseAddress-boolean-)若为`true`，则`TIME_WAIT`状态中的地址在关闭后可重用。
* [`setTrafficClass`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setTrafficClass-int-)
* [`setBroadcast`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setBroadcast-boolean-)设置或清除`SO_BROADCAST`套接字选项，设置此选项时，数据报（UDP）数据包可能会发送到本地接口的广播地址。
* [`setMulticastNetworkInterface`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setMulticastNetworkInterface-java.lang.String-)设置或清除`IP_MULTICAST_LOOP`套接字选项，设置此选项时，多播数据包也将在本地接口上接收。
* [setMulticastTimeToLive](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setMulticastTimeToLive-int-)设置`IP_MULTICAST_TTL`套接字选项。TTL表示“活动时间”，单这种情况下，它指定允许数据包经过的IP跳数，特别是用于多播流量。转发数据包的每个路由器或网管会递减TTL，如果路由器将TTL递减为0，则不会再转发。

#### DatagramSocket本地地址

若您在调用`listen(...)`之前已经绑定了[`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)，您可以通过调用[`localAddress`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html#localAddress--)来查找套接字的本地地址（即UDP Socket这边的地址，它将返回一个InetSocketAddress，否则返回null。

#### 关闭DatagramSocket

您可以通过调用[`close`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html#close-io.vertx.core.Handler-)方法来关闭Socket，它将关闭Socket并释放所有资源。


## DNS 客户端

通常情况下，您需要以异步方式来获取DNS信息。

但不幸的是，Java 虚拟机本身附带的API是不可能的，因此Vert.x提供了它自己的完全异步解析DNS的API。

若要获取 `DnsClient` 实例，您可以通过 `Vertx` 实例来创建一个。

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
```

你亦可通过`options`来创建`DnsClient`从而配置查询的过期时间。

```java
DnsClient client = vertx.createDnsClient(new DnsClientOptions()
  .setPort(53)
  .setHost("10.0.0.1")
  .setQueryTimeout(10000)
);
```

Creating the client with no arguments or omitting the server address will use the address of the server used internally for non blocking address resolution.

创建`DnsClient`的时候，不指定`options`参数或者不指定服务器地址的话，`DnsClient`则会使用服务器内部地址 来进行非阻塞的域名解析。

```java
DnsClient client1 = vertx.createDnsClient();

// Just the same but with a different query timeout
DnsClient client2 = vertx.createDnsClient(new DnsClientOptions().setQueryTimeout(10000));
```



### lookup

当尝试为一个指定名称元素获取A（ipv4）或 AAAA（ipv6）记录时，第一条被返回的（记录）将会被使用。它的操作方式和操作系统上使用 `nslookup` 类似。

要为 `vertx.io` 获取 A/AAAA 记录，您需要像下面那样做：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.lookup("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### lookup4

尝试查找给定名称的A（ipv4）记录。第一个返回的（记录）将会被使用，因此它的操作方式与操作系统上使用`nslookup`类似。

要查找 `vertx.io` 的A记录，您需要像下面那样做：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.lookup4("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### lookup6

尝试查找给定名称的 AAAA（ipv6）记录。第一个返回的（记录）将会被使用，因此它的操作方式与在操作系统上使用 `nslookup` 类似。

要查找 `vertx.io` 的 AAAA记录，您需要像下面那样做：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.lookup6("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveA

尝试解析给定名称的所有A（ipv4）记录，这与在unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有A记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveA("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<String> records = ar.result();
    for (String record : records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveAAAA

尝试解析给定名称的所有AAAA（ipv6）记录，这与在Unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有AAAA记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveAAAA("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<String> records = ar.result();
    for (String record : records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveCNAME

尝试解析给定名称的所有CNAME记录，这与在Unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有CNAME记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveCNAME("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<String> records = ar.result();
    for (String record : records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveMX

尝试解析给定名称的所有MX记录，MX记录用于定义哪个邮件服务器去接受`指定域`的电子邮件。

要查找您常用执行的`vertx.io`的所有MX记录：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveMX("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<MxRecord> records = ar.result();
    for (MxRecord record: records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

请注意，列表将包含按照它们优先级排序的[`MxRecord`](https://vertx.io/docs/apidocs/io/vertx/core/dns/MxRecord.html)，这意味着列表中优先级低的MX记录会第一个优先出现在列表中。

[`MxRecord`](https://vertx.io/docs/apidocs/io/vertx/core/dns/MxRecord.html)允许您通过下边提供的方法访问MX记录的优先级和名称：

```java
record.priority();
record.name();
```

### resolveTXT

尝试解析给定名称的所有TXT记录，TXT记录通常用于定义`域`的额外信息。

要解析`vertx.io`的所有TXT记录，您可以使用下边几行代码：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveTXT("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<String> records = ar.result();
    for (String record: records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveNS

尝试解析给定名称的所有`NS记录`，`NS记录`指定一个DNS服务器，这个服务器管理`指定域`的DNS信息。

要解析`vertx.io`的所有NS记录，您可以使用下边几行：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveNS("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<String> records = ar.result();
    for (String record: records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### resolveSRV

尝试解析给定名称的所有SRV记录，SRV记录用于定义服务端口和主机名等额外信息。一些协议需要这些额外信息。

要查找`vertx.io`的所有SRV记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolveSRV("vertx.io", ar -> {
  if (ar.succeeded()) {
    List<SrvRecord> records = ar.result();
    for (SrvRecord record: records) {
      System.out.println(record);
    }
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

请注意，列表将包含按照它们优先级排序的[`SrvRecord`](https://vertx.io/docs/apidocs/io/vertx/core/dns/SrvRecord.html)，这意味着优先级低的记录会第一个优先出现在列表中。

[`SrvRecord`](https://vertx.io/docs/apidocs/io/vertx/core/dns/SrvRecord.html)允许您访问SRV记录本身中包含的所有信息：

```java
record.priority();
record.name();
record.weight();
record.port();
record.protocol();
record.service();
record.target();
```

有关详细信息，请参阅API文档。

### resolvePTR

尝试解析给定名称的PTR记录，PTR记录将`ipaddress`映射到名称。

要解析IP地址`10.0.0.1`的PTR记录，您将使用`1.0.0.10.in-addr.arpa`的PTR概念。

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.resolvePTR("1.0.0.10.in-addr.arpa", ar -> {
  if (ar.succeeded()) {
    String record = ar.result();
    System.out.println(record);
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### reverseLookup

尝试对ipaddress进行反向查找，这与解析PTR记录类似。但是允许您传递`非有效PTR查询字符串的ipaddress`。

要做ipaddress 10.0.0.1的反向查找类似的事：

```java
DnsClient client = vertx.createDnsClient(53, "9.9.9.9");
client.reverseLookup("10.0.0.1", ar -> {
  if (ar.succeeded()) {
    String record = ar.result();
    System.out.println(record);
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

### 错误处理

如前边部分所述，`DnsClient` 允许您传递一个`Handler`，一旦查询完成将会传入一个`AsyncResult`给`Handler`并通知它。

在出现错误的情况下，通知中将包含一个`DnsException`，该异常会包含一个说明为何失败的[`DnsResponseCode`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html)。此`DnsResponseCode`可用于更详细检查原因。

可能的`DnsResponseCode`值是：

* [`NOERROR`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOERROR) 没有找到给定查询的记录
* [`FORMERROR`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#FORMERROR) 格式错误
* [`SERVFAIL`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#SERVFAIL) 服务器故障
* [`NXDOMAIN`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NXDOMAIN) 名称错误
* [`NOTIMPL`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOTIMPL) DNS 服务器没实现
* [`REFUSED`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#REFUSED) DNS 服务器拒绝查询
* [`YXDOMAIN`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#YXDOMAIN) 域名不应该存在
* [`YXRRESET`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#YXRRSET) 资源记录不应该存在
* [`NXRRSET`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NXRRSET) RRSET不存在
* [`NOTZONE`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOTZONE) 名称不在区域内
* [`BADVERS`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADVERS) 版本的扩展机制不好
* [`BADSIG`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADSIG) 非法签名
* [`BADKEY`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADKEY) 非法密钥
* [`BADTIME`](https://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADTIME) 错误时间戳

所有这些错误都由DNS服务器本身“生成”，您可以从 `DnsException` 中获取 `DnsResponseCode`，如：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.lookup("nonexisting.vert.xio", ar -> {
  if (ar.succeeded()) {
    String record = ar.result();
    System.out.println(record);
  } else {
    Throwable cause = ar.cause();
    if (cause instanceof DnsException) {
      DnsException exception = (DnsException) cause;
      DnsResponseCode code = exception.code();
      // ...
    } else {
      System.out.println("Failed to resolve entry" + ar.cause());
    }
  }
});
```

## 流

Vert.x有多个对象可以用于文件的读取和写入。

在 Vert.x 中，写调用是立即返回的，而写操作的实际是在内部队列中排队写入。

不难看出，若写入对象的速度比实际写入底层数据资源速度快，那么写入队列就会无限增长，最终导致内存耗尽。

为了解决这个问题，Vert.x API中的一些对象提供了简单的流程控制（回压 back-pressure）功能。

任何可控制的写入流对象都实现了 [`WriteStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html) 接口，相应的，任何可控制的读取流对象都实现了 [`ReadStream`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html) 接口。

让我们举个例子，我们要从`ReadStream`中读取数据，然后将数据写入`WriteStream`。

一个非常简单的例子是从`NetSocket`读取然后写回到同一个`NetSocket` —— 因为`NetSocket`既实现了`ReadStream`也实现了`WriteStream` 接口。请注意，这些操作适用于任何实现了`ReadStream`和 `WriteStream` 接口的对象，包括HTTP 请求、HTTP 响应、异步文件 I/O 和 WebSocket等。

一个最简单的方法是直接获取已经读取的数据，并立即将其写入`NetSocket`：

```java
NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    // Write the data straight back
	  // 直接写入数据
    sock.write(buffer);
  });
}).listen();
```

上面的例子有一个问题：如果从Socket读取数据的速度比写回Socket的速度快，那么它将在`NetSocket`的写队列中不断堆积，最终耗尽内存。这是有可能会发生的，例如，若Socket另一端的客户端读取速度不够快，无法快速地向连接的另一端回压。

由于 `NetSocket` 实现了 `WriteStream` 接口，我们可以在写入之前检查 `WriteStream` 是否已满：

```java
NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    if (!sock.writeQueueFull()) {
      sock.write(buffer);
    }
  });

}).listen();
```

这个例子不会耗尽内存，但如果写入队列已满，我们最终会丢失数据。我们真正想要做的是在写入队列已满时暂停读取 `NetSocket`：

```java
NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    sock.write(buffer);
    if (sock.writeQueueFull()) {
      sock.pause();
    }
  });
}).listen();
```

我们已经快达到我们的目标，但还没有完全实现。现在 `NetSocket` 在文件已满时会暂停，但是当写队列处理完成时，我们需要取消暂停：

```java
NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.handler(buffer -> {
    sock.write(buffer);
    if (sock.writeQueueFull()) {
      sock.pause();
      sock.drainHandler(done -> {
        sock.resume();
      });
    }
  });
}).listen();
```

在这里，我们的目标实现了。当写队列准备好接收更多的数据时，[`drainHandler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#drainHandler-io.vertx.core.Handler-)事件处理器将被调用，它会恢复`NetSocket`的状态，允许读取更多的数据。

在编写Vert.x 应用程序时，这样做是很常见的，因此我们提供了一个名为[`pipTo`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#pipeTo-io.vertx.core.streams.WriteStream-)方法，它为您完成所有这些艰苦的工作。您只需要把 `WriteStream`传给它 ，然后使用它：

```java
NetServer server = vertx.createNetServer(
  new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  sock.pipeTo(sock);
}).listen();
```

以上和下面更详细的例子完全一样，额外加上stream对于`失败`和`结束`的处理：最终 当`pipe`处于`success`或者`failure`这种结束状态时，`WriteStream`就会停止。

当读写操作结束时，你会收到提示：

```java
server.connectHandler(sock -> {

  // Pipe the socket providing an handler to be notified of the result
  // pipe和socket传输数据时，提供一个对于通知结果的handler
  sock.pipeTo(sock, ar -> {
    if (ar.succeeded()) {
      System.out.println("Pipe succeeded");
    } else {
      System.out.println("Pipe failed");
    }
  });
}).listen();
```

当你想要异步处理`源文件`时，你可以创建一个[Pipe](https://vertx.io/docs/apidocs/io/vertx/core/streams/Pipe.html)对象，这个对象会暂停`源文件流`，并在`源文件流`通过pipe传输到目标时恢复`源文件流`：

```java
server.connectHandler(sock -> {

  // Create a pipe to use asynchronously
  // 创建用于一步操作的pipe
  Pipe<Buffer> pipe = sock.pipe();

  // Open a destination file
  // 开启源文件流
  fs.open("/path/to/file", new OpenOptions(), ar -> {
    if (ar.succeeded()) {
      AsyncFile file = ar.result();

      // Pipe the socket to the file and close the file at the end
      // 用管道传输socket当中的信息到源文件流 并最终关闭源文件流
      pipe.to(file);
    } else {
      sock.close();
    }
  });
}).listen();
```

当你想取消传输操作，你需要关闭pipe：

```java
vertx.createHttpServer()
  .requestHandler(request -> {

    // Create a pipe that to use asynchronously
    // 创建异步操作管道
    Pipe<Buffer> pipe = request.pipe();

    // Open a destination file
    // 打开源文件流
    fs.open("/path/to/file", new OpenOptions(), ar -> {
      if (ar.succeeded()) {
        AsyncFile file = ar.result();

        // Pipe the socket to the file and close the file at the end
        // 用管道传输socket当中的信息到源文件流 并最终关闭源文件流
        pipe.to(file);
      } else {
        // Close the pipe and resume the request, the body buffers will be discarded
        // 关闭管道，恢复请求，body当中的缓冲数据被丢弃
        pipe.close();

        // Send an error response
        // 返回错误
        request.response().setStatusCode(500).end();
      }
    });
  }).listen(8080);
```

当`pipe`关闭，`steams handlers`会被重置，`ReadStream`恢复工作。

由以上看来，默认情况下，stream穿谁完毕之后，`源文件流`都会停止。你可以用`pipe`对象控制这些行为：

+ [endOnFailure](https://vertx.io/docs/apidocs/io/vertx/core/streams/Pipe.html#endOnFailure-boolean-) 控制失败时的操作
+ [endOnSuccess](https://vertx.io/docs/apidocs/io/vertx/core/streams/Pipe.html#endOnSuccess-boolean-) 控制stream结束时的操作
+ [endOnComplete](https://vertx.io/docs/apidocs/io/vertx/core/streams/Pipe.html#endOnComplete-boolean-) 控制所有情况下的操作

如下是一个例子

```java
src.pipe()
  .endOnSuccess(false)
  .to(dst, rs -> {
    // Append some text and close the file
    dst.end(Buffer.buffer("done"));
});
```



现在我们来看看 `ReadStream` 和 `WriteStream` 的方法。

### ReadStream

`ReadStream`（可读流） 接口的实现类包括：[`HttpClientResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html), [`DatagramSocket`](https://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html), [`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html), [`HttpServerFileUpload`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html), [`HttpServerRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html), [`MessageConsumer`](https://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html), [`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html), [`WebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html), [`TimeoutStream`](https://vertx.io/docs/apidocs/io/vertx/core/TimeoutStream.html), [`AsyncFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)。

函数：

* [`handler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#handler-io.vertx.core.Handler-)：设置一个处理器，它将从`ReadStream`读取对象
* [`pause`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#pause--)：暂停处理器，暂停时，处理器中将不会受到任何对象
* [`fetch`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#fetch-long-):  从stream中抓取指定数量的对象，这些对象中任意一个到达目的地之后，都会触发handler，fetch操作是累积的。
* [`resume`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#resume--)：恢复处理器，若任何对象到达目的地则handler将被触发
* [`exceptionHandler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#exceptionHandler-io.vertx.core.Handler-)：若ReadStream发生异常，将被调用
* [`endHandler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#endHandler-io.vertx.core.Handler-)：当流的数据读取完毕时将被调用。触发原因是读取到了`EOF`，可能分别来自如下：与`ReadStream`关联的文件、HTTP请求、或TCP Socket的连接被关闭



`ReadStream`有`flowing`和`fetch`两个模式：

- 最初 stream是`flowing` 模式
- 当 stream 处于`flowing`模式，stream中的元素被传输到`handler`
- 当stream处于`fetch`模式，只有指定数量的元素才被传输到`handler`

[pause](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#pause--),[resume](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#resume--) 和 [fetch](https://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#fetch-long-) 会改变`ReadStream`的模式

- `resume()` ：设置`ReadStream` 为 `flowing`模式
- `pause()` ：设置`ReadStream` 为 `fetch`模式 并设置demand值为0
- `fetch(long)` ：请求指定数量的stream元素并将该数量加到目前的demand值当中

### WriteStream

`WriteStream`（可写流）接口的实现类包括：[`HttpClientRequest`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html), [`HttpServerResponse`](https://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)，[`WebSocket`](https://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html), [`NetSocket`](https://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html), [`AsyncFile`](https://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)

函数：

* [`write`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#write-java.lang.Object-)：写入一个对象到 `WriteStream`，该方法将永远不会阻塞，内部是排队写入并且底层资源是异步写入。
* [`setWriteQueueMaxSize`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#setWriteQueueMaxSize-int-)：设置写入队列容量——`writeQueueFull`在队列写满时返回`true`。注意，当写队列已满时，调用写（操作）时 数据依然会被接收和排队。实际数量取决于流的实现，对于[`Buffer`](https://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)，`size`代表实际写入的字节数，而并非缓冲区的数量。
* [`writeQueueFull`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#writeQueueFull--)：若写队列被认为已满，则返回`true`。
* [`exceptionHandler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#exceptionHandler-io.vertx.core.Handler-)：若`WriteStream`发生异常，则被调用。
* [`drainHandler`](https://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#drainHandler-io.vertx.core.Handler-)：若`WriteStream`被认为不再满，则处理器将被调用。

## 记录解析器（Record Parser）

记录解析器（Record Parser）允许您轻松解析由字节序列或固定尺寸带分隔符的记录的协议。

它将输入缓冲区序列转换为已配置的缓冲区序列（固定大小或带分隔符的记录）。

例如，若您使用`\n`分割的简单ASCII文本协议，并输入如下：

```
buffer1:HELLO\nHOW ARE Y
buffer2:OU?\nI AM
buffer3: DOING OK
buffer4:\n
```

记录解析器将生成下结果：

```
buffer1:HELLO
buffer2:HOW ARE YOU?
buffer3:I AM DOING OK
```

我们来看看相关代码：

```java
final RecordParser parser = RecordParser.newDelimited("\n", h -> {
  System.out.println(h.toString());
});

parser.handle(Buffer.buffer("HELLO\nHOW ARE Y"));
parser.handle(Buffer.buffer("OU?\nI AM"));
parser.handle(Buffer.buffer("DOING OK"));
parser.handle(Buffer.buffer("\n"));
```

我们还可以生成固定尺寸的块，如下：

```java
RecordParser.newFixed(4, h -> {
  System.out.println(h.toString());
});
```

有关更多详细信息，请查看[`RecordParser`](https://vertx.io/docs/apidocs/io/vertx/core/parsetools/RecordParser.html)类。

## Json 解析器
你可以很容易的解析JSON格式的内容，但是这就要求一次性提供整个JSON内容，而如果你要提供一个非常大的JSON，JSON解析器恐怕就不是个方便的方式。

`non-blocking` JSON解析器 则是一个事件驱动的解析器, 它可以处理体量非常大的JSON。它将传输一系列`input buffer`到一系列JSON解析器事件。

```java
JsonParser parser = JsonParser.newParser();

// Set handlers for various events
// 设置不同事件的handler
parser.handler(event -> {
  switch (event.type()) {
    case START_OBJECT:
      // Start an objet
      break;
    case END_OBJECT:
      // End an objet
      break;
    case START_ARRAY:
      // Start an array
      break;
    case END_ARRAY:
      // End an array
      break;
    case VALUE:
      // Handle a value
      String field = event.fieldName();
      if (field != null) {
        // In an object
      } else {
        // In an array or top level
        if (event.isString()) {

        } else {
          // ...
        }
      }
      break;
  }
});
```
该解析器是非阻塞的，并且由`input buffer`驱动触发的事件

```java
JsonParser parser = JsonParser.newParser();

// start array event
// start object event
// "firstName":"Bob" event
parser.handle(Buffer.buffer("[{\"firstName\":\"Bob\","));

// "lastName":"Morane" event
// end object event
parser.handle(Buffer.buffer("\"lastName\":\"Morane\"},"));

// start object event
// "firstName":"Luke" event
// "lastName":"Lucky" event
// end object event
parser.handle(Buffer.buffer("{\"firstName\":\"Luke\",\"lastName\":\"Lucky\"}"));

// end array event
parser.handle(Buffer.buffer("]"));

// Always call end
parser.end();
```

事件驱动的解析操作提供了更多的控制，但代价就是要处理非常细粒度的事件（`event-mode`），这有时是不方便的。当你需要的时候，JSON解析器允许你将JSON做为值(`value-mode`)处理。

```java
JsonParser parser = JsonParser.newParser();

parser.objectValueMode();

parser.handler(event -> {
  switch (event.type()) {
    case START_ARRAY:
      // Start the array
      break;
    case END_ARRAY:
      // End the array
      break;
    case VALUE:
      // Handle each object
      break;
  }
});

parser.handle(Buffer.buffer("[{\"firstName\":\"Bob\"},\"lastName\":\"Morane\"),...]"));
parser.end();
```

`value-mode`可以在解析时启用或停用，并允许你在`event-mode`事件和 JSONObject的`value-mode`事件之间自由切换

```java
JsonParser parser = JsonParser.newParser();

parser.handler(event -> {
  // Start the object
  switch (event.type()) {
    case START_OBJECT:
      // 设置为 value-mode，自此开始，解析器则不会触发start-object事件
      parser.objectValueMode();
      break;
    case VALUE:
      // 处理每一个对象
      // 获得从对象中解析出来的字段
      String id = event.fieldName();
      System.out.println("User with id " + id + " : " + event.value());
      break;
    case END_OBJECT:
      // 设置为 event mode，所以解析器重新触发 start/end 事件
      parser.objectEventMode();
      break;
  }
});

parser.handle(Buffer.buffer("{\"39877483847\":{\"firstName\":\"Bob\"},\"lastName\":\"Morane\"),...}"));
parser.end();
```

你也可以对数组做同样的事情

```java
JsonParser parser = JsonParser.newParser();

parser.handler(event -> {
  // Start the object

  switch (event.type()) {
    case START_OBJECT:
      // 设置为value mode来处理每个元素，自此开始，解析器不会触发 start-array 事件
      parser.arrayValueMode();
      break;
    case VALUE:
      // 处理每一个数组
      // 获取对象中的字段
      System.out.println("Value : " + event.value());
      break;
    case END_OBJECT:
      // 设置为 event mode，从而解析器会重新触发 start/end 事件
      parser.arrayEventMode();
      break;
  }
});

parser.handle(Buffer.buffer("[0,1,2,3,4,...]"));
parser.end();
```

你也可以反解析到POJOs。

```java
parser.handler(event -> {
  // 获取每个对象
  // 获取对象中的字段
  String id = event.fieldName();
  User user = event.mapTo(User.class);
  System.out.println("User with id " + id + " : " + user.firstName + " " + user.lastName);
});
```

无论何时，解析器解析buffer失败之后，会抛出异常，除非你设置`exception handler`：


```java
JsonParser parser = JsonParser.newParser();

parser.exceptionHandler(err -> {
  // 捕捉所有的解析/反解析异常
});
```

解析器也可以解析JSON流：

- 连续的JSON流： `{"temperature":30}{"temperature":50}`
- 行分割的JSON流： `{"an":"object"}\r\n3\r\n"a string"\r\nnull`

更多细节，详见[JsonParser](https://vertx.io/docs/apidocs/io/vertx/core/parsetools/JsonParser.html)

## 线程安全

大多数Vert.x 对象可以从被不同的线程安全地访问，但在相同的上下文中访问它们时，性能才是最优的。

例如，若您部署了一个创建`NetServer`的Verticle，该`NetServer`在处理器中提供了`NetSocket` 实例，则最好始终从该Verticle的Event Loop中访问Socket 实例。

如您坚持使用标准的Vert.x Verticle部署模型，避免在 Verticle 之间分享对象，那这种情况您无需考虑。

## 运行阻塞式代码

在一个完美的世界中，不存在战争和饥饿，所有的API都将使用异步方式编写，兔兔和小羊羔将会在阳光明媚的绿色草地上手牵手地跳舞。

**但是……真实世界并非如此（您最近看新闻了吧？）**

事实是，很多，也非所有的库，特别是在JVM生态系统中有很多同步API，这些API中许多方法都是阻塞式的。一个很好的例子就是 JDBC API，它本质上是同步的，无论多么努力地去尝试，Vert.x都不能像魔法小精灵撒尘变法一样将它转换成异步API。

我们不会将所有的内容重写成异步方式，所以我们为您提供一种在 Vert.x 应用中安全调用"传统"阻塞API的方法。

如之前讨论，您不能在 Event Loop 中直接调用阻塞式操作，因为这样做会阻止 Event Loop 执行其他有用的任务。那您该怎么做？

可以通过调用 [`executeBlocking`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-) 方法来指定阻塞式代码的执行以及阻塞式代码执行后处理结果的异步回调。

```java
vertx.executeBlocking(future -> {
  // 调用一些需要耗费显著执行时间返回结果的阻塞式API
  String result = someAPI.blockingMethod("hello");
  future.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```

>Warning：
阻塞代码因为某些原因 要阻塞很长时间（即，不多余几秒钟）。要杜绝长时间的阻塞操作或者polling操作（即，一个线程陷入一个循环当以阻塞模式获取事件）。当阻塞时间超过了10秒，thread checker 会在console当中打印一条信息。长时间阻塞的操作需要交给额外的线程，这个线程由应用所管理，这个应用可以和verticles通过event-bus 或 [`runOnContext`](https://vertx.io/docs/apidocs/io/vertx/core/Context.html#runOnContext-io.vertx.core.Handler-) 交互


默认情况下，如果 `executeBlocking` 在同一个上下文环境中（如：同一个 Verticle 实例）被调用了多次，那么这些不同的 `executeBlocking` 代码块会 **顺序执行**（一个接一个）。

若您不需要关心您调用 [`executeBlocking`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-) 的顺序，可以将 `ordered` 参数的值设为 `false`。这样任何 `executeBlocking` 都会在 Worker Pool 中并行执行。

另外一种运行阻塞式代码的方法是使用 [Worker Verticle](https://vertx.io/docs/vertx-core/java/#worker_verticles)。

一个 Worker Verticle 始终会使用 Worker Pool 中的某个线程来执行。

默认情况下，阻塞式代码会在 Vert.x 的 Worker Pool 中执行，通过 [`setWorkerPoolSize`](https://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#setWorkerPoolSize-int-) 配置。

可以为不同的用途创建不同的Worker Pool：

```java
WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool");
executor.executeBlocking(promise -> {
  // 调用一些需要耗费显著执行时间返回结果的阻塞式API
  String result = someAPI.blockingMethod("hello");
  promise.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```

Worker Executor 在不需要的时候必须被关闭：

```java
executor.close();
```

当使用同一个名字创建了许多 worker 时，它们将共享同一个 pool。当所有的 worker executor 调用了 `close` 方法被关闭过后，对应的 worker pool 会被销毁。

如果 Worker Executor 在 Verticle 中创建，那么 Verticle 实例销毁的同时 Vert.x 将会自动关闭这个 Worker Executor。

Worker Executor 可以在创建的时候配置：

```java
int poolSize = 10;

// 2 minutes
long maxExecuteTime = 2;
TimeUnit maxExecuteTimeUnit = TimeUnit.MINUTES;

WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool", poolSize, maxExecuteTime, maxExecuteTimeUnit);
```

> 请注意：*这个配置信息在 worker pool 创建的时候设置。*
## Metrics SPI

默认情况下，Vert.x不会记录任何指标。相反，它为其他人提供了一个SPI，可以将其添加到类路径中。SPI是一项高级功能，允许实施者从Vert.x捕获事件以收集指标。有关详细信息，请参阅 [API 文档](https://vertx.io/docs/apidocs/io/vertx/core/spi/metrics/VertxMetrics.html)。

若使用[`setFactory`](https://vertx.io/docs/apidocs/io/vertx/core/metrics/MetricsOptions.html#setFactory-io.vertx.core.spi.VertxMetricsFactory-)嵌入了Vert.x实例，也可以用编程方式指定度量工厂。

## OSGi

Vert.x Core被打包成了 OSGi Bundle，因此可以在任何OSGi R4.2+环境中使用，如 `Apache Felix` 或 `Eclipse Equinox`，（这个）Bundle导出`io.vertx.core*`。

但是 Bundle 对 Jackson 和 Netty 有一些依赖，若部署Vert.x Core Bundle则需要：

* Jackson Annotation [2.6.0,3)
* Jackson Core [2.6.2,3)
* Jackson Databind [2.6.2,3)
* Netty Buffer [4.0.31,5)
* Netty Codec [4.0.31,5)
* Netty Codec/Socks [4.0.31,5)
* Netty Codec/Common [4.0.31,5)
* Netty Codec/Handler [4.0.31,5)
* Netty Codec/Transport [4.0.31,5)

下边是Apache Felix 5.2.0上的工作部署：

```
14|Active     |    1|Jackson-annotations (2.6.0)
15|Active     |    1|Jackson-core (2.6.2)
16|Active     |    1|jackson-databind (2.6.2)
18|Active     |    1|Netty/Buffer (4.0.31.Final)
19|Active     |    1|Netty/Codec (4.0.31.Final)
20|Active     |    1|Netty/Codec/HTTP (4.0.31.Final)
21|Active     |    1|Netty/Codec/Socks (4.0.31.Final)
22|Active     |    1|Netty/Common (4.0.31.Final)
23|Active     |    1|Netty/Handler (4.0.31.Final)
24|Active     |    1|Netty/Transport (4.0.31.Final)
25|Active     |    1|Netty/Transport/SCTP (4.0.31.Final)
26|Active     |    1|Vert.x Core (3.1.0)
```

在Equinox上，您可能需要使用下边的框架属性禁用ContextFilter：`eclipse.bundle.setTCCL=false`。

## vertx 命令行

vertx 命令行工具用于在终端中与 Vert.x 进行交互。主要用于运行 Vert.x Verticle。为此，您需要下载并安装Vert.x 发行版，并将安装目录中的`bin`添加到`PATH`环境变量中，还要确保您的`PATH`上有一个Java 8的JDK。

> *请注意：JDK需要支持Java代码的快速编译。*

### 运行 Verticle

您可以使用 `vertx run` 从命令行直接运行Vert.x 的 Verticle，以下是`run`命令的几个实例：

```
vertx run my-verticle.js                                 (1)
vertx run my-verticle.groovy                             (2)
vertx run my-verticle.rb                                 (3)

vertx run io.vertx.example.MyVerticle                    (4)
vertx run io.vertx.example.MVerticle -cp my-verticle.jar (5)

vertx run MyVerticle.java                                (6)
```

1. 部署一个JavaScript的Verticle
2. 部署一个Groovy的Verticle
3. 部署一个Ruby的Verticle
4. 部署一个已经编译好的Java的Verticle，类的根路径是当前目录
5. 部署一个已经打包成jar的Verticle，这个jar需要在类路径中
6. 编译Java源代码并进行部署

正如您在Java中可看到的，该Verticle的名称要么是Java 完全限定类名，也可以指定Java 源文件，Vert.x会为你编译它。

您可以用其他语言的前缀来指定Verticle的名称进行部署。例如：若Verticle是一个编译的Groovy 类，您可以使用语言前缀`groovy:`，因此Vert.x 知道它是一个Groovy 类而不是Java 类。

```
vertx run groovy:io.vertx.example.MyGroovyVerticle
```

`vertx run`命令可以使用几个可选参数，它们是：

* `-conf <config_file>`：提供了Verticle的一些配置，`config_file`是包含描述Verticle配置的JSON对象的文本文件的名称，该参数是可选的。
* `-cp <path>`：搜索Verticle和它使用的其他任何资源的路径，默认为`.`（当前目录）。若您的Verticle引用了其他脚本、类或其他资源（例如jar文件），请确保这些脚本、其他资源存在此路径上。该路径可以包含由以下内容分隔的多个路径条目：`:`（冒号）或`;`（分号）——这取决于操作系统。每个路径条目可以是包含脚本的目录的绝对路径或相对路径，也可以是`jar`或`zip`文件的绝对或相对文件名。一个示例路径可能是`-cp classes:lib/otherscripts:jars/myjar.jar:jars/otherjar.jar`。始终使用路径引用您的Verticle需要的任何资源，不要将它们放在系统类路径上，因为这会导致部署的Verticle之间的隔离问题。
* `-instances <instances>`：要实例化的Verticle实例的数目，每个Verticle实例都是严格单线程（运行）的，以便在可用的核上扩展应用程序，您可能需要部署多个实例。若省略，则部署单个实例。
* `-worker`：此选项可确定一个Verticle是否为Worker Verticle。
* `-cluster`：此选项确定Vert.x实例是否尝试与网络上的其他Vert.x实例形成集群，集群Vert.x实例允许Vert.x与其他节点形成分布式Event Bus。默认为false（非集群模式）。
* `-cluster-port`：若指定了`cluster`选项，则可以确定哪个端口将用于与其他Vert.x实例进行集群通信。默认为0——这意味着“选择一个空闲的随机端口”。除非您帧需要绑定特定端口，您通常不需要指定此参数。
* `-cluster-host`：若指定了`cluster`选项，则可以确定哪个主机地址将用于其他Vert.x实例进行集群通信。默认情况下，它将尝试从可用的接口中选一个。若您有多个接口而您想要使用指定的一个，就在这里指定。
* `-ha`：若指定，该Verticle将部署为（支持）高可用性（HA）。有关详细信息，请参阅相关章节。
* `-quorum`：该参数需要和`-ha`一起使用，它指定集群中所有HA部署ID处于活动状态的最小节点数，默认为0。
* `-hagroup`：该参数需要和`-ha`一起使用，它指定此节点将加入的HA组。集群中可以有多个HA组，节点只会故障转移到同一组中的其他节点。默认为`__DEFAULT__`。

您还可以使用下边方式设置系统属性：`-Dkey=value`。

下面有更多的例子：

使用默认设置运行JavaScript的Verticle：server.js：

```
vertx run server.js
```

运行指定类路径的预编译好的10个Java Verticle实例

```
vertx run com.acme.MyVerticle -cp "classes:lib/myjar.jar" -instances 10
```

通过源文件运行10个Java Verticle的实例

```
vertx run MyVerticle.java -instances 10
```

运行20个Ruby语言的Worker Verticle实例

```
vertx run order_worker.rb -instances 20 -worker
```

在同一台计算机上运行两个JavaScript Verticle，并让它们彼此在网络上的其他任何服务器上集群在一起：

```
vertx run handler.js -cluster
vertx run sender.js -cluster
```

运行一个Ruby Verticle并传入一些配置：

```
vertx run my_verticle.rb -conf my_verticle.conf
```

其中`my_verticle.conf`也许会包含以下配置：

```json
{
 "name": "foo",
 "num_widgets": 46
}
```

该配置可通过Core API在Verticle内部可用。

当使用Vert.x的高可用功能时，您可能需要创建一个Vert.x的 *裸* 实例。此实例在启动时未部署任何Verticle，但它若接收到若集群中的另一个节点死亡，则会在此节点运行之前挂掉的实例。如需要创建一个 *裸* 实例，执行以下命令：

```
vertx bare
```

根据您的集群配置，您可能需要添加`cluster-host`和`cluster-port`参数。

### 执行打包成 fat-jar 的Vert.x 应用

fat-jar 是一个包含了所有依赖项jar的可执行的jar，这意味着您不必在执行jar的机器上预先安装Vert.x。它像任何可执行的Java jar一样可直接执行：

```
java -jar my-application-fat.jar
```

对于这点，Vert.x 没什么特别的，您可以使用任何Java应用程序。

您可以创建自己的主类并在 MANIFEST 中指定，但建议您将代码编写成Verticle，并使用Vert.x中的[`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)类（`io.vertx.core.Launcher`）作为您的主类。这是在命令行中运行Vert.x使用的主类，因此允许您指定命令行参数，如 `-instances` 以便更轻松地扩展应用程序。

要将您的Verticle全部部署在这个`fat-jar`中时，您必须将下边信息写入MANIFEST：

* `Main-Class`设置为`io.vertx.core.Launcher`
* `Main-Verticle`指定要运行的Main Verticle（Java完全限定类名或脚本文件名）

您还可以提供您将传递给 `vertx run` 的常用命令行参数：

```
java -jar my-verticle-fat.jar -cluster -conf myconf.json
java -jar my-verticle-fat.jar -cluster -conf myconf.json -cp path/to/dir/conf/cluster_xml
```

> 注意：请参阅官方 Vert.x Examples 仓库中中的 Maven/Gradle 相应示例来了解如何将应用打包成 *fat-jar*。

通过 fat jar 运行应用时，默认会执行 `run` 命令。

### 显示Vert.x的版本

若想显示Vert.x的版本，只需执行：

```
vertx version
```

### 其他命令

除了`run`和`version`以外，`vertx`命令行和 `Launcher` 还提供了其他命令：

您可以使用下边命令创建一个`bare`实例：

```
vertx bare
# or
java -jar my-verticle-fat.jar bare
```

您还可以在后台启动应用程序：

```
java -jar my-verticle-fat.jar start -Dvertx-id=my-app-name
```

若`my-app-name`未设置，将生成一个随机的id，并在命令提示符中打印。您可以将`run`选项传递给`start`命令：

```
java -jar my-verticle-fat.jar start -Dvertx-id=my-app-name -cluster
```

一旦在后台启动，可以使用`stop`命令停止它：

```
java -jar my-verticle-fat.jar stop my-app-name
```

您还可以使用一下方式列出后台启动的Vert.x应用程序：

```
java -jar my-verticle-fat.jar list
```

`vertx`工具也可以使用`start`、`stop`和`list`命令，`start`命令支持几个选项：

* `vertx-id`：应用程序ID，若未设置，则使用随机UUID
* `java-opts`：Java虚拟机选项，若未设置，则使用`JAVA_OPTS`环境变量
* `redirect-output`：重定向生成的进程输出和错误流到父进程流

若选项值包含空白，请不要忘记在“”（双引号）之间包装值。

由于`start`命令产生一个新的进程，传递给JVM的java选项不会被传播，所以您必须使用`java-opts`来配置JVM（`-X`，`-D`...）。若您使用 `CLASSPATH` 环境变量，请确保路径下包含所有需要的jar（vertx-core、您的jar和所有依赖项）。

该命令集是可扩展的，请参考 [扩展 Vert.x 启动器](#扩展-vertx-启动器) 部分。

### 实时重部署

在开发时，可以方便在文件更改时实时重新部署应用程序。`vertx` 命令行工具和更普遍的 `Launcher` 类提供了这个功能。这里有些例子：

```
vertx run MyVerticle.groovy --redeploy="**/*.groovy" --launcher-class=io.vertx.core.Launcher
vertx run MyVerticle.groovy --redeploy="**/*.groovy,**/*.rb"  --launcher-class=io.vertx.core.Launcher
java io.vertx.core.Launcher run org.acme.MyVerticle --redeploy="**/*.class"  --launcher-class=io.vertx.core
.Launcher -cp ...
```

重新部署过程如下执行。首先，您的应用程序作为后台应用程序启动（使用`start`命令）。当发现文件更改时，该进程将停止并重新启动该应用。这样可避免泄露。

要启用实时重新部署，请将 `--redeploy` 选项传递给 `run` 命令。`--redeploy` 表示要监视的文件集，这个集合可使用 `Ant` 样式模式（使用 `**`，`*` 和 `?`），您也可以使用逗号（`,`）分隔它们来指定多个集合。模式相当于当前工作目录。

传递给 `run` 命令的参数最终会传递给应用程序，可使用 `--java-opts` 配置JVM虚拟机选项。例如，如果想传入一个 `conf` 参数或是系统属性，您可以使用 `--java-opts="-conf=my-conf.json -Dkey=value"`。

`--launcher-class` 选项确定应用程序的主类启动器。它通常是一个 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)，单您已使用了您自己的主类。

也可以在IDE中使用重部署功能：

* **Eclipse**：创建一个运行配置，使用`io.vertx.core.Launcher`类作为主类。在 *Program Arguments* 区域（参数选项卡中），写入`run your-verticle-fully-qualified-name --redeploy=**/*.java --launcher-class=io.vertx.core.Launcher`，您还可以添加其他参数。随着 Eclipse 在保存时会增量编译您的文件，重部署工作会顺利进行。
* **IntelliJ IDEA**：创建一个运行配置（应用）,将主类设置为`io.vertx.core.Launcher`。在程序参数中写：`run your-verticle-fully-qualified-name --redeploy=**/*.class --launcher-class=io.vertx.core.Launcher`。要触发重新部署，您需要显示构造项目或模块（Build -> Make project）。

要调试应用程序，请将运行配置创建为远程应用程序，并使用`--java-opts`配置调试器。每次重新部署后，请勿忘记重新插入(re-plug)调试器，因为它每次都会创建一个新进程。

您还可以在重新部署周期中挂接（hook）构建过程：

```
java -jar target/my-fat-jar.jar --redeploy="**/*.java" --on-redeploy="mvn package"
java -jar build/libs/my-fat-jar.jar --redeploy="src/**/*.java" --on-redeploy='./gradlew shadowJar'
```

"on-redeploy"选项指定在应用程序关闭后和重新启动之前调用的命令。因此，如果更新某些运行时工作，则可以钩住构建工具。例如，您可以启动`gulp`或`grunt`来更新您的资源。如果您的应用需要 `--java-opts`，不要忘记将它添加到命令参数里：

```
java -jar target/my-fat-jar.jar --redeploy="**/*.java" --on-redeploy="mvn package" --java-opts="-Dkey=val"
java -jar build/libs/my-fat-jar.jar --redeploy="src/**/*.java" --on-redeploy='./gradlew shadowJar' --java-opts="-Dkey=val"
```

重新部署功能还支持以下设置：

* `redeploy-scan-period`：文件系统检查周期（以毫秒为单位），默认为250ms
* `redeploy-grace-period`：在2次重新部署之间等待的时间（以毫秒为单位），默认为1000ms
* `redeploy-termination-period`：停止应用程序后等待的时间（在启动用户命令之前）。这个在Windows上非常有用，因为这个进程并没立即被杀死。时间以毫秒为单位，默认20ms

## 集群管理器

在 Vert.x 中，集群管理器可用于各种功能，包括：

* 对集群中 Vert.x 节点发现和分组
* 维护集群范围中的主题订阅者列表（所以我们可知道哪些节点对哪个Event Bus地址感兴趣）
* 分布式Map的支持
* 分布式锁
* 分布式计数器

集群管理器不处理Event Bus节点之间的传输，这是由 Vert.x 直接通过TCP连接完成。

Vert.x发行版中使用的默认集群管理器是使用的[Hazelcast](http://hazelcast.com/)集群管理器，但是它可以轻松被替换成实现了Vert.x集群管理器接口的不同实现，因为Vert.x集群管理器可替换的。

集群管理器必须实现[`ClusterManager`](https://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)接口，Vert.x在运行时使用Java的服务加载器（[Service Loader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)）功能查找集群管理器，以便在类路径中查找[`ClusterManager`](https://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)的实例。

若您在命令行中使用Vert.x并要使用集群，则应确保Vert.x安装的`lib`目录包含您的集群管理器jar。

若您在 Maven/Gradle 项目使用Vert.x，则只需将集群管理器jar作为项目依赖添加。

您也可以以编程的方式在嵌入Vert.x 时使用 [`setClusterManager`](https://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#setClusterManager-io.vertx.core.spi.cluster.ClusterManager-) 指定集群管理器。

## 日志记录

Vert.x使用内置的日志API进行记录日志，默认实现使用JDK（JUL）日志，不需要额外的依赖项。

### 配置JUL日志记录

一个JUL日志记录配置文件可以使用普通的JUL方式指定——通过提供一个名为`java.util.logging.config.file`的系统属性值为您的配置文件。更多关于此部分以及JUL配置文件结构的内容，请参阅JUL日志记录文档。

Vert.x还提供了一种更方便的方式指定配置文件，无需设置系统属性。您只需在您的类路径中提供名为`vertx-default-jul-logging.properties`的JUL配置文件（例如在您的fatjar中），Vert.x将使用该配置文件配置JUL。

### 使用另一个日志框架

如果您不希望Vert.x使用JUL记录日志，您可以为其配置另一个日志记录框架，例如Log4J或SLF4J。

为此，您应该设置一个名为`vertx.logger-delegate-factory-class-name`的系统属性，该属性的值是一个实现了 [`LogDelegateFactory`](https://vertx.io/docs/apidocs/io/vertx/core/spi/logging/LogDelegateFactory.html) 接口的Java 类名。我们为Log4J（版本1）、Log4J 2和SLF4J提供了预设的实现，类名为：`io.vertx.core.logging.Log4jLogDelegateFactory`，`io.vertx.core.logging.Log4j2LogDelegateFactory`和`io.vertx.core.logging.SLF4JLogDelegateFactory`。如您要使用这些实现，还应确保相关的Log4J或SLF4J的jar在您的类路径上。

请注意，提供的Log4J 1代理不支持参数化消息。Log4J 2的代理使用了像SLF4J代理这样的`{}`语法，JUL代理使用如`{x}`语法。

### 应用中记录日志

Vert.x本身只是一个库，您可以在自己的应用程序使用任何日志库的API来记录日志。

但是，若您愿意，也可以使用上述的Vert.x日志记录工具为应用程序记录日志。

为此，您需要使用[LoggerFactory](https://vertx.io/docs/apidocs/io/vertx/core/logging/LoggerFactory.html)获取一个[Logger](https://vertx.io/docs/apidocs/io/vertx/core/logging/Logger.html)对象以记录日志：

```
Logger logger = LoggerFactory.getLogger(className);

logger.info("something happened");
logger.error("oops!", exception);
```

> 注意，不同的日志实现会使用不同的占位符。这意味着，如果你使用了 Vert.x 的参数化的日志记录方法，当你切换日志的实现时，你可能需要修改你的代码。

### Netty日志记录

配置日志记录时，您也应该关心配置Netty日志记录。

Netty不依赖于外部日志配置（例如系统属性），而是根据Netty类可见的日志记录库来实现日志记录：

* 如`SLF4J`可见，则优先使用该库
* 否则若`Log4j`可见，再使用该库
* 否则退回使用`java.util.logging`

可通过设置`io.netty.util.internal.logging.InternalLoggerFactory`强制Netty使用某个特定实现。

```
// 强制使用Log4j日志记录
InternalLoggerFactory.setDefaultFactory(Log4JLoggerFactory.INSTANCE);
```

### 故障排除

#### SLF4J启动警告

若您在启动应用程序时看到以下信息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

这意味着您的类路径中有`SLF4J-API`却没绑定。`SLF4J`记录的消息将会丢失。您应该将绑定加入您的类路径。参考 [SLF4J user manual - Binding with a logging framework at deployment time](https://www.slf4j.org/manual.html#swapping) 选择绑定并配置。

请注意，Netty会寻找`SLF4-API`的jar，并在缺省情况下使用它。

#### 对等连接重置

若您的日志显示一堆：

```
io.vertx.core.net.impl.ConnectionBase
SEVERE: java.io.IOException: Connection reset by peer
```
这意味着客户端正在重置HTTP连接，而不是关闭它。此消息还可能表示您没有读取完整的有效负荷（连接在读取完全之前被切断）。

> 译者注：*通常情况下，这是正常的，无需担心，如果您打开浏览器，按快捷键不停滴刷新页面，就能看到该SEVERE日志。*

## 主机名解析

Vert.x 使用自带的网络地址解析器来执行主机名解析的工作（将主机名解析为IP地址），而没有使用JVM内置的阻塞式解析器。

把主机名解析成IP地址的操作将会使用到：

- 操作系统的 *hosts* 文件
- DNS查询服务器列表

默认情况下它使用系统环境中设定的DNS服务器地址列表，如果无法获取该列表，则会使用谷歌的公用DNS服务器地址 `8.8.8.8` 以及 `8.8.4.4` 。

也可以在创建 [`Vertx`
](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html) 实例的时候配置DNS服务器：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().
            addServer("192.168.0.1").
            addServer("192.168.0.2:40000"))
);
```

DNS服务器的默认端口为 53 ，当服务器使用不同的端口时，可以使用半角冒号作为分隔符来指定端口，例如： `192.168.0.2:40000` 。


> **请注意：**
> 
> 如果某些场景之下必须要使用JVM内置的解析器，此时可以通过在启动时设置系统属性 `-Dvertx.disableDnsResolver=true` 来激活JVM内置的解析器。

### 故障转移

 当一个服务器没有及时响应时，解析器会从列表中取出下一个服务器进行查询，该故障转移操作的次数限制可以通过 [`setMaxQueries
`](https://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setMaxQueries-int-) 来设置（默认设置是4次）。

### 服务器列表轮询

默认情况下，解析器总是使用服务器列表中的第一个服务器，剩下的服务器用于故障转移。

您可以将 [`setRotateServers`](https://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setRotateServers-boolean-) 设置为 `true`，此时解析器将会使用 round-robin 风格的轮询操作，将查询的负担分摊到列表中的每一个服务器上，从而避免所有的查询负担都落在列表中的第一个服务器上。

此时故障转移机制仍然有效，当某个服务器没有及时响应时，解析器会使用列表中的下一个服务器。

### 主机映射

操作系统自身的 *hosts* 文件用于查找主机名对应的IP地址。

除此之外也可以使用另外的 *hosts* 文件来代替操作系统自身的 *hosts* 文件：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().
            setHostsPath("/path/to/hosts"))
);
```

### DNS搜索域

默认情况下，解析器使用系统环境中设置的DNS搜索域。如果要使用显式指定的搜索域，可以使用以下方式：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().addSearchDomain("foo.com").addSearchDomain("bar.com"))
);
```

当使用搜索域列表时， “.” 符号数量的阈值一般为1，在Linux操作系统里该阈值由 `/etc/resolv.conf` 文件来指定，通过 [`setNdots`](https://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setNdots-int-) 可以人为指定该阈值的大小。

## 高可用与故障转移

Vert.x 可以支持 verticle 运行于高可用（HA）模式。这种模式之下，如果一个 vert.x 实例所运行的 verticle 突然宕掉，该 verticle 将会被迁移到其他的 vert.x 实例中（该 vert.x 实例必须处于同一个集群之中）。

### 自动故障转移

当运行 vert.x 时开启了高可用（HA）选项，此时如果某个 vert.x 实例中的某个 verticle 运行失败或者宕掉，该 verticle 将会被自动重新部署于集群中的另一个 vert.x 实例中。我们把这种机制称为 verticle故障转移。

只要在运行 vert.x 的命令行中追加 `-ha` 参数，就可以开启高可用模式：

```
vertx run my-verticle.js -ha
```

要让高可用机制起作用，您需要在集群中开启至少2个 Vert.x 实例，现在假设您已经在集群中运行了一个 Vert.x 实例，例如：

```
vertx run my-other-verticle.js -ha
```

此时如果运行 my-verticle.js 的 Vert.x 实例宕掉了（例如您可以使用 `kill -9` 命令强行杀掉这个进程来模拟此场景），运行 my-other-verticle.js 的 Vert.x 实例会自动地部署 my-verticle.js ，此时该 Vert.x 实例同时运行了这两个 verticle （my-other-verticle.js 和 my-verticle.js）。


> **请注意：**
> 
> 如果要使得这种迁移机制起作用，则必须保证第二个 vert.x 实例可以访问到该 verticle 对应的文件（在此场景中指的是 my-verticle.js）。
> 
> 
> **重要提示：**
> 
> 请注意，通过正常方式退出的 Vert.x 实例（例如使用 `CTRL-C` 组合键或者 `kill -SIGINT` 命令）不会触发故障转移操作。

您也可以启动若干个空白的 Vert.x 实例————指的是它们在启动时没有加载任何 verticle，此时，它们一样可以对集群中的其他节点起到故障转移的作用。启动一个空白的 Vert.x 实例很简单，只需要执行以下命令：

```
vertx run -ha
```

当使用 `-ha` 参数时， 可以不需要再追加 `-cluster` 参数，因为高可用模式是假定了您需要运行在集群模式之下的。

> **请注意：**
> 依据您的集群配置选项，您可能还是需要自定义集群管理器（默认使用 Hazelcast），以及追加集群主机（`cluster-host`）和集群端口（`cluster-port`）等参数。

### 高可用组

当 Vert.x 实例运行于高可用模式时，您还可以对其进行高可用分组，这里称之为*高可用组*。此处的*高可用组*指的是一个集群之中的节点的一种逻辑分组，被分配了高可用组的节点只会对同一个高可用组之下的其他节点执行故障转移操作。如果没有指定高可用组，系统会自动将节点分配到默认的`__DEFAULT__`高可用组。

在运行 verticle 时可以使用`-hagroup`参数指定高可用分组，例如：

```
vertx run my-verticle.js -ha -hagroup my-group
```

举个例子：

在第一个终端里运行：

```
vertx run my-verticle.js -ha -hagroup g1
```

在第二个终端里，我们以同一个高可用组运行另一个 verticle :

```
vertx run my-other-verticle.js -ha -hagroup g1
```

最后，在第三个终端里，我们以不同的高可用组再运行一个其他的 verticle ：

```
vertx run yet-another-verticle.js -ha -hagroup g2
```

如果我们杀掉第一个终端里的实例，这里面的 verticle 将会通过故障转移机制迁移到第二个终端里的实例中，而不是第三个终端里的实例中，因为第三个终端里的实例被分配了不同的高可用组。

如果杀掉第三个终端里的实例，则不会发生故障转移操作，因为此终端里的 vert.x 实例被分配了不同的高可用组。


### 处理网络分区 - Quora

高可用实现也支持 `quora` （一种多数派机制）。在分布式系统中， Quorum 是指一种投票机制，在这种投票机制之下，某个分布式事务只有获得不少于指定投票数量的票数，才允许执行某个操作。

启动 Vert.x 实例的时候，您可以将其设置成在进行高可用（HA）部署之前需要一个 `quorum` 。在这个语境之下， `quorum` 指的是集群中某个特定的分组内的节点数量的下限。典型的例如您将 `quorum` 的数值设置为 `1 + N/2` （现在以 Q 指代该数值，其中的 N 代表分组中的节点总数），那么如果集群中少于 Q 个节点的情况下，该高可用（HA）部署将被取消，待到节点数量达到这个 Q 数值的时候，会再次进行部署。这种机制可以防止出现网络分区（亦称“脑裂”）。

关于 `quora` 的更多信息请参考 [Quorum_(distributed_computing)](http://en.wikipedia.org/wiki/Quorum_(distributed_computing)) 。

要在运行 vert.x 实例的时候启用 `quorum` ，您只需要在命令行中指定 `-quorum` 参数，例如

在第一个终端中执行：

```
vertx run my-verticle.js -ha -quorum 3
```

此时 Vert.x 实例将会启动，但是并不会部署这个模块，因为现在只有1个节点，而不是3个。


在第二个终端中执行：

```
vertx run my-other-verticle.js -ha -quorum 3
```

此时 Vert.x 实例将会启动，但是并不会部署这个模块，因为现在只有2个节点，而不是3个。

在第三个终端中，您可以启动另一个 vert.x 实例：

```
vertx run yet-another-verticle.js -ha -quorum 3
```

哇！————我们有了3个节点，这正是 `quorum` 的数值。此时此刻这些模块将会被自动地部署到所有实例上。

如果我们关闭或者强行杀死其中一个节点，那么这些模块将会被自动卸载，因为节点数量已经不满足 `quorum` 数值条件。

Quora 也可以和高可用分组联合使用，此时 quora 仅在指定的分组中起作用。

## 本地传输

在BSD（OSX）和Linux操作系统中运行 Vert.x 的时候，如果条件允许，可以启用[本地传输](http://netty.io/wiki/native-transports.html)这种特性：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
  setPreferNativeTransport(true)
);

// 如果本地传输已启用，则返回 true 
boolean usingNative = vertx.isNativeTransportEnabled();
System.out.println("Running with native: " + usingNative);
```
> **请注意：**
> 如果倾向于启用本地传输而相关条件却不满足的时候（例如相关JAR包缺失），程序依然可以运行。如果您要求您的程序必须启用本地传输，您必须首先通过[`isNativeTransportEnabled`](https://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#isNativeTransportEnabled--)来确认是否启用了本地传输。



### Linux 下的本地传输

您需要在 `classpath` 中加入以下依赖：

```xml
<dependency>
 <groupId>io.netty</groupId>
 <artifactId>netty-transport-native-epoll</artifactId>
 <classifier>linux-x86_64</classifier>
 <!--<version>Should align with netty version that Vert.x uses</version>-->
</dependency>
```

Linux下的本地传输可以设置更多的网络选项：

- SO_REUSEPORT
- TCP_QUICKACK
- TCP_CORK
- TCP_FASTOPEN

```java
vertx.createHttpServer(new HttpServerOptions()
  .setTcpFastOpen(fastOpen)
  .setTcpCork(cork)
  .setTcpQuickAck(quickAck)
  .setReusePort(reusePort)
);
```

### BSD 下的本地传输

您需要在 `classpath` 中加入以下依赖：

```xml
<dependency>
 <groupId>io.netty</groupId>
 <artifactId>netty-transport-native-kqueue</artifactId>
 <classifier>osx-x86_64</classifier>
 <!--<version>必须和 Vert.x 所使用的 netty 的版本一致</version>-->
</dependency>
```

MacOS 中， `Sierra` 及以上的版本支持这种特性。

BSD 下的本地传输可以启用以下额外的网络选项：

- SO_REUSEPORT

```java
vertx.createHttpServer(new HttpServerOptions().setReusePort(reusePort));
```

### 域套接字

通过本地传输，网络服务可以使用域套接字：

```java
vertx.createNetServer().connectHandler(so -> {
  // 处理请求
}).listen(SocketAddress.domainSocketAddress("/var/tmp/myservice.sock"));
```

http服务示例：

```java
vertx.createHttpServer().requestHandler(req -> {
  // 处理请求
}).listen(SocketAddress.domainSocketAddress("/var/tmp/myservice.sock"), ar -> {
  if (ar.succeeded()) {
    // 绑定到 socket
  } else {
    ar.cause().printStackTrace();
  }
});
```

也适用于网络客户端：

```java
NetClient netClient = vertx.createNetClient();

// 仅在 Linux 和 BSD 中可以使用
SocketAddress addr = SocketAddress.domainSocketAddress("/var/tmp/myservice.sock");

// 连接到服务器
netClient.connect(addr, ar -> {
  if (ar.succeeded()) {
    // 连接成功
  } else {
    ar.cause().printStackTrace();
  }
});
```

http客户端示例：

```java
HttpClient httpClient = vertx.createHttpClient();

// 仅在 Linux 和 BSD 中可以使用
SocketAddress addr = SocketAddress.domainSocketAddress("/var/tmp/myservice.sock");

// 向服务器发送请求
httpClient.request(new RequestOptions()
  .setServer(addr)
  .setHost("localhost")
  .setPort(8080)
  .setURI("/"))
  .onSuccess(request -> {
    request.send().onComplete(response -> {
      // 处理响应信息
    });
  });
```

## 安全提示

Vert.x 是一套工具集，而不是一种强迫人们使用指定方式行事的框架，对于开发者而言，这赋予了你们强大的力量，但也使得你们必须负起不小的责任。

与任何一种工具集一样，写出不安全的程序是难以避免的，所以您在开发程序时需要时刻小心，特别是这个程序是暴露于毫无保护的公共场合（例如互联网）的情况下。

### Web 应用

如果要编写一个 web 应用程序，这里强烈建议您使用 Vert.x-Web 来实现资源服务和文件上传功能，而不是直接使用 Vert.x core 。

Vert.x-Web 会对请求路径进行规整化，这可以阻止那些不怀好意的人利用精心构建的特殊URL来访问web应用根目录之外的资源的企图。

在文件上传方面也是如此， Vert.x-Web 不会完全信赖客户所端提供的文件名，因为客户端有可能精心设置一个特殊的文件名，使得上传的文件被保存到磁盘上某个意料之外的位置上。 Vert.x-Web 可以保证上传的文件是被存放到磁盘上确切可知道位置的。

### 集群模式事件总线流量

在网络上使用集群模式的事件总线连接不同的 Vert.x 节点时，总线里的流量是未经加密的，因此，若您的 Vert.x 节点处于不可信任的网络之上，则应该避免使用这种方式向这样的 Vert.x 节点发送信息。



### 安全方面的标准最佳实践

任何服务都可能存在潜在的漏洞，无论是使用 Vert.x 还是任何其他工具包来进行编写，因此始终应该遵循安全最佳实践，特别是当您的服务面向公众时。

例如，您应该始终在DMZ（隔离区）中运行它们，并使用权限受限的用户账户，以确保服务被渗透以后只会遭受有限的破坏。

## Vert.x 命令行界面（CLI）API

Vert.x Core 提供了一套用于解析传递给程序的命令行参数的API。这套API也可以用与打印命令行相关参数、选项的详细帮助信息。即使这些功能远离Vert.x Core主题，该API已在 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 类中使用，因此您可以在 *fat-jar* 和 `vertx` 命令行工具中使用它们。此外，它支持多语言（可用于任何已支持的语言），并可在 Vert.x Shell 中使用。

Vert.x CLI 提供了一个编程模型用以描述命令行界面，这个编程模型也可以视为一种语法解析器，这个语法解析器支持不同类型的语法：

- POSIX 风格的选项参数 （例如： `tar -zxvf foo.tar.gz`）
- GNU 的长字符串风格的选项参数 （例如： `du --human-readable --max-depth=1`）
- Java 风格的属性参数 （例如： `java -Djava.awt.headless=true -Djava.net.useSystemProxies=true Foo`）
- 附带选项值的简短风格的选项参数 （例如： `gcc -O2 foo.c`）
- 包含单个连接符的长字符串风格的选项参数 (例如： `ant -projecthelp`）

使用这个命令行API只需要三个步骤：

1. 定义命令行接口
2. 解析用户输入的命令行
3. 进行查询/问答交互操作

### 定义阶段

每个命令行界面都必须定义所要使用的选项和参数集合。这些选项和参数也需命名。命令行API使用 [`Option`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html) 和 [`Argument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html) 类来描述选项和参数：

```java
CLI cli = CLI.create("copy")
    .setSummary("A command line interface to copy files.")
    .addOption(new Option()
        .setLongName("directory")
        .setShortName("R")
        .setDescription("enables directory support")
        .setFlag(true))
    .addArgument(new Argument()
        .setIndex(0)
        .setDescription("The source")
        .setArgName("source"))
    .addArgument(new Argument()
        .setIndex(1)
        .setDescription("The destination")
        .setArgName("target"));
```

正如您所见到的一样，您可以通过  [`CLI.create`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#create-java.lang.String-) 方法来创建一个新的 [`CLI`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html) 。此处传入的字符串参数就是这个CLI的名称。创建之后，可以给它设置摘要和描述。一般来说，摘要是指一行简短的文字说明，描述是指篇幅较长的详细说明。每个选项和参数可以使用[`addArgument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#addArgument-io.vertx.core.cli.Argument-)和[`addOption`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#addOption-io.vertx.core.cli.Option-)方法加入到`CLI`对象中。

### 选项列表

[`Option`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html)是指用户输入的命令行中出现的以 *键* 来标识的命令行参数。选项必须至少有一个长名称或一个短名称。通常情况下，长名称使用`--`前缀，短名称使用单个`-`前缀。这些名称都是大小写敏感的；但是，在查询/问答交互的环节中，如果输入的名称无法精确匹配，则会使用大小写不敏感的方式进行匹配。选项可以在用法说明的部分显示出相关的描述（见下文）。选项可以接收0个，1个或者若干个选项值。接收0个选项值的选项称作**标识（flag）**，标识必须使用[`setFlag`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setFlag-boolean-)来声明。缺省情况想，选项接收单个选项值，但是您也可以使用[`setMultiValued`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setMultiValued-boolean-)将其设置成接收多个选项值：

```java
CLI cli = CLI.create("some-name")
    .setSummary("A command line interface illustrating the options valuation.")
    .addOption(new Option()
        .setLongName("flag").setShortName("f").setFlag(true).setDescription("a flag"))
    .addOption(new Option()
        .setLongName("single").setShortName("s").setDescription("a single-valued option"))
    .addOption(new Option()
        .setLongName("multiple").setShortName("m").setMultiValued(true)
        .setDescription("a multi-valued option"));
```

选项可以标记必填。用户如果没有输入必填选项，则会在命令行解析的过程中抛出异常：

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("mandatory")
        .setRequired(true)
        .setDescription("a mandatory option"));
```

非必填选项可以拥有一个默认选项值，在用户没有输入对应的选项值时，则会启用这个默认选项值：

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("optional")
        .setDefaultValue("hello")
        .setDescription("an optional option with a default value"));
```

选项也可以通过[`setHidden`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setHidden-boolean-)方法设置成隐藏的。隐藏的选项不会在使用说明中显示出来，但是仍然可以起作用（提供给高级用户使用）。

如果选项值是一组固定的集合，您可以为其设置哪些输入内容的允许的：

An option can be hidden using the 
setHidden
 method. Hidden option are not listed in the usage, but can still be used in the user command line (for power-users).

If the option value is contrained to a fixed set, you can set the different acceptable choices:

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("color")
        .setDefaultValue("green")
        .addChoice("blue").addChoice("red").addChoice("green")
        .setDescription("a color"));
Options can also be instantiated from their JSON form.
```

### 参数

和选项不一样，参数不以 *键* 进行标识而是以其 *索引* 作为标识。例如，在 **`java com.acme.Foo`** 里， **`com.acme.Foo`** 就是一个参数。

参数没有名称，它们是以从 `0` 开始计数的索引为标识。第一个参数的索引为 `0`：

Unlike options, arguments do not have a key and are identified by their index. For example, in java com.acme.Foo, com.acme.Foo is an argument.

Arguments do not have a name, there are identified using a 0-based index. The first parameter has the index 0:

```java
CLI cli = CLI.create("some-name")
    .addArgument(new Argument()
        .setIndex(0)
        .setDescription("the first argument")
        .setArgName("arg1"))
    .addArgument(new Argument()
        .setIndex(1)
        .setDescription("the second argument")
        .setArgName("arg2"));
```

如果不设置参数的索引，则基于声明顺序自动计算索引值。

```java
CLI cli = CLI.create("some-name")
    // 索引将被设置成0
    .addArgument(new Argument()
        .setDescription("the first argument")
        .setArgName("arg1"))
    // 索引将被设置成1
    .addArgument(new Argument()
        .setDescription("the second argument")
        .setArgName("arg2"));
```

`argName` 是可选的，并且能在使用说明信息中使用。

和选项一样，[`Argument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html)也可以：

- 使用[`setHidden`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setHidden-boolean-)设置为隐藏的

- 使用[`setRequired`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setRequired-boolean-)设置为必填的

- 使用[`setDefaultValue`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setDefaultValue-java.lang.String-)设置默认参数值

- 使用[`setMultiValued`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setMultiValued-boolean-)来接收多个参数值————只有最后一个参数才允许设置成接收多个参数值。

参数也可以通过其对应格式的 JSON 数据来创建。

### 生成使用说明信息

当[`CLI`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)实例配置完成之后，您可以用它来生成使用说明信息：

```java
CLI cli = CLI.create("copy")
    .setSummary("A command line interface to copy files.")
    .addOption(new Option()
        .setLongName("directory")
        .setShortName("R")
        .setDescription("enables directory support")
        .setFlag(true))
    .addArgument(new Argument()
        .setIndex(0)
        .setDescription("The source")
        .setArgName("source"))
    .addArgument(new Argument()
        .setIndex(0)
        .setDescription("The destination")
        .setArgName("target"));

StringBuilder builder = new StringBuilder();
cli.usage(builder);
```

这可以生成诸如此类的使用说明信息：

```
Usage: copy [-R] source target

A command line interface to copy files.

 -R,--directory   enables directory support
```

如果需要调整这个使用说明信息，请参考 [`UsageMessageFormatter`](https://vertx.io/docs/apidocs/io/vertx/core/cli/UsageMessageFormatter.html) 类。

### 解析阶段

[`CLI`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html) 配置完成以后，您可以解析用户输入的命令行，并以此处理每个参数和选项：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
```

[`parse`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#parse-java.util.List-) 方法返回一个包含了这些值的 [`CommandLine`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html) 对象。默认情况下，它会对用户输入的命令行进行检查校验，并确认哪些必填选项和必填参数有无缺失，以及每个选项值的数量是否符合要求。您可以将 [`parse`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#parse-java.util.List-boolean-) 方法中的第二个参数传入 `false ` 值来禁用这项校验功能。这可以用来检查某个参数或选项是否存在，无论命令行输入是否合规。

您可以使用 [`isValid`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html#isValid--) 方法来检查 [`CommandLine`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html) 对象是否合规。

### 查询/问答交互阶段

命令行解析完成之后，您可以从 [`parse`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#parse-java.util.List-) 方法返回的 [`CommandLine`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html) 对象中获取到选项值和参数值：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
String opt = commandLine.getOptionValue("my-option");
boolean flag = commandLine.isFlagEnabled("my-flag");
String arg0 = commandLine.getArgumentValue(0);
```

其中一个选项可以标记为“帮助”。如果命令行启用了“帮助”选项，命令行的校验不会失败，而是有机会检查用户是否在寻求帮助：

```java
CLI cli = CLI.create("test")
    .addOption(
        new Option().setLongName("help").setShortName("h").setFlag(true).setHelp(true))
    .addOption(
        new Option().setLongName("mandatory").setRequired(true));

CommandLine line = cli.parse(Collections.singletonList("-h"));

// 此时不会视作解析失败，并且可以进行以下操作：
if (!line.isValid() && line.isAskingForHelp()) {
  StringBuilder builder = new StringBuilder();
  cli.usage(builder);
  stream.print(builder.toString());
}
```

### 类型化的选项和参数

上述的 [`Option`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html) 和 [`Argument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html) 类是无类型的，意味着只能从中获取到字符串类型的值。 [`TypedOption`](https://vertx.io/docs/apidocs/io/vertx/core/cli/TypedOption.html) 和 [`TypedArgument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/TypedArgument.html) 能让您对其赋予一个类型，这样（字符串类型的）原始值将被转换成对应的类型。

在 [`CLI`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html) 对象的定义中使用 [`TypedOption`](https://vertx.io/docs/apidocs/io/vertx/core/cli/TypedOption.html) 和 [`TypedArgument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/TypedArgument.html) 来取代 [`Option`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html) 和 [`Argument`](https://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html) ：

```java
CLI cli = CLI.create("copy")
    .setSummary("A command line interface to copy files.")
    .addOption(new TypedOption<Boolean>()
        .setType(Boolean.class)
        .setLongName("directory")
        .setShortName("R")
        .setDescription("enables directory support")
        .setFlag(true))
    .addArgument(new TypedArgument<File>()
        .setType(File.class)
        .setIndex(0)
        .setDescription("The source")
        .setArgName("source"))
    .addArgument(new TypedArgument<File>()
        .setType(File.class)
        .setIndex(0)
        .setDescription("The destination")
        .setArgName("target"));
```

这时您就可以通过如下方式获取转换后的值：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
boolean flag = commandLine.getOptionValue("R");
File source = commandLine.getArgumentValue("source");
File target = commandLine.getArgumentValue("target");
```

Vert.x CLI 可以转换具有如下特征的类：

- 拥有参数签名为一个 [`String`](https://vertx.io/docs/apidocs/java/lang/String.html) 类型的构造函数，例如[`File`](https://vertx.io/docs/apidocs/java/io/File.html) 或者 [`JsonObject`](https://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)
- 拥有一个名为 `from` 或者 `fromString` 的静态方法
- 拥有一个静态的 `valueOf` 方法，例如原始类型和枚举类型

此外您也可以实现自定义的 [`Converter`](https://vertx.io/docs/apidocs/io/vertx/core/cli/converters/Converter.html) 并在 `CLI` 对象使用它：

```java
CLI cli = CLI.create("some-name")
    .addOption(new TypedOption<Person>()
        .setType(Person.class)
        .setConverter(new PersonConverter())
        .setLongName("person"));
```

对于布尔类型而言，这些值将被视为 `true` ：`on`， `yes`， `1`， `true` 。

如果您的命令行选项存在 `enum` 类型，则会自动计算出一组选择范围。
If one of your option has an enum as type, it computes the set of choices automatically.

### 注解的使用

您也可以使用注解来定义 `CLI` 对象。可以通过在类和 `setter` 方法上使用注解来完成定义：

```java
@Name("some-name")
@Summary("some short summary.")
@Description("some long description")
public class AnnotatedCli {

  private boolean flag;
  private String name;
  private String arg;

  @Option(shortName = "f", flag = true)
  public void setFlag(boolean flag) {
    this.flag = flag;
  }

  @Option(longName = "name")
  public void setName(String name) {
    this.name = name;
  }

  @Argument(index = 0)
  public void setArg(String arg) {
   this.arg = arg;
  }
}
```

加上注解之后，您就可以使用以下方法来定义 [`CLI`](https://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html) 对象并将对应的值注入进去：

```java
CLI cli = CLI.create(AnnotatedCli.class);
CommandLine commandLine = cli.parse(userCommandLineArguments);
AnnotatedCli instance = new AnnotatedCli();
CLIConfigurator.inject(commandLine, instance);
```

## Vert.x 启动器（Launcher）

Vert.x [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 在 fat-jar 中作为主类，由 `vertx` 命令行程序调用。它可执行一组命令，如`run`、`bare` 和 `start`等

### 扩展 Vert.x 启动器（Launcher）

您可以通过实现自己的 [`Command`](https://vertx.io/docs/apidocs/io/vertx/core/spi/launcher/Command.html) 类来扩展命令集（仅限于Java）：

```java
@Name("my-command")
@Summary("A simple hello command.")
public class MyCommand extends DefaultCommand {

  private String name;

  @Option(longName = "name", required = true)
  public void setName(String n) {
    this.name = n;
  }

  @Override
  public void run() throws CLIException {
    System.out.println("Hello " + name);
  }
}
```

您还需要实现一个 [`CommandFactory`](https://vertx.io/docs/apidocs/io/vertx/core/spi/launcher/CommandFactory.html)：

```java
public class HelloCommandFactory extends DefaultCommandFactory<HelloCommand> {
  public HelloCommandFactory() {
   super(HelloCommand.class);
  }
}
```

然后创建 `src/main/resources/META-INF/services/io.vertx.core.spi.launcher.CommandFactory` 并且添加一行表示工厂类的完全限定名称：

```
io.vertx.core.launcher.example.HelloCommandFactory
```

构建包含命令的jar。确保包含了SPI文件（`META-INF/services/io.vertx.core.spi.launcher.CommandFactory`）。

然后，将包含该命令的jar放入fat-jar（或包含在其中）的类路径中，或放在Vert.x发行版的`lib`目录中，您将可以执行：

```java
vertx hello vert.x
java -jar my-fat-jar.jar hello vert.x
```
### 在 fat-jar 中使用启动器（Launcher）

要在 fat-jar 中使用 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 类，只需要将 *MANIFEST* 的 `Main-Class` 设置为 `io.vertx.core.Launcher`。另外，将 *MANIFEST* 中 `Main-Verticle` 条目设置为您的Main Verticle的名称。

默认情况下，它会执行 `run` 命令。但是，您可以通过设置 *MANIFEST* 的`Main-Command`条目来配置默认命令。若在没有命令的情况下启动 fat-jar使用默认命令。

### 启动器（Launcher）子类

您还可以创建 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 的子类来启动您的应用程序。这个类被设计成易于扩展的。

一个启动器 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 子类可以：

* 在 [`beforeStartingVertx`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html#beforeStartingVertx-io.vertx.core.VertxOptions-) 中自定义 Vert.x 配置
* 通过重写 [`afterStartingVertx`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html#afterStartingVertx-io.vertx.core.Vertx-) 来检索由“run”或“bare”命令创建的Vert.x实例
* 使用 [`getMainVerticle`](https://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#getMainVerticle--) 和 [`getDefaultCommand`](https://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#getDefaultCommand--) 方法配置默认的Verticle和命令
* 使用 [`register`](https://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#register-java.lang.Class-) 和 [`unregister`](https://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#unregister-java.lang.String-) 方法添加/删除命令

### 启动器（Launcher）和退出代码

当您使用 [`Launcher`](https://vertx.io/docs/apidocs/io/vertx/core/Launcher.html) 类作为主类时，它使用以下退出代码：

* `0` :进程顺利结束，或抛出未捕获的错误；
* `1` :用于通用错误；
* `11`:Vert.x无法初始化；
* `12`:生成的进程无法启动、发现或停止，该错误代码一般由`start`和`stop`命令使用；
* `14`:系统配置不符合系统要求（如找不到 `java` 命令）；
* `15`:主Verticle不能被部署；

## 配置 Vert.x 缓存

当 Vert.x 需要从类路径中读取文件（嵌入在 fat-jar 中，在类路径中jar文件或其他文件）时，它会把文件复制到缓存目录。背后原因很简单：从 jar 或从输入流读取文件是阻塞的。所以为了避免每次都付出损耗，Vert.x 会将文件复制到其缓存目录中，并随后读取该文件。也可以配置此行为。

首先，默认情况下，Vert.x 使用 `$CWD/.vertx` 作为缓存目录，它在此目录创建一个唯一的目录，以避免冲突。可以使用 `vertx.cacheDirBase` 系统属性配置该位置。如，若当前工作目录不可写（例如在不可变容器中），请使用以下命令启动应用程序：

```java
vertx run my.Verticle -Dvertx.cacheDirBase=/tmp/vertx-cache
# 或者
java -jar my-fat.jar vertx.cacheDirBase=/tmp/vertx-cache
```

> 重要提示： *该目录必须是可写的。*

当您编辑资源（如HTML、CSS或JavaScript）时，这种缓存机制可能令人讨厌，因为它仅仅提供文件的第一个版本（因此，如果您想重新加载页面，不会显示到您的编辑改变）。要避免此情况，请使用 `-Dvertx.disableFileCaching=true` 启动应用程序。使用此设置，Vert.x 仍然使用缓存，但会始终读取原文件然后刷新在缓存中的版本。因此，如果您编辑从类路径提供的文件并刷新浏览器，Vert.x 会从类路径读取它，将其复制到缓存目录并从中提供。不要在生产环境使用这个设置，它很有可能影响性能。

最后，您可以使用`-Dvertx.disableFileCPResolving=true`完全禁用高速缓存。这个设置有影响。Vert.x将无法从类路径中读取任何文件（仅从文件系统中读取）。使用此设置时要非常小心。

---

## 结语

---

> [原文档](https://vertx.io/docs/vertx-core/java/)更新于2017-03-15 15:54:14 CET
