# Vert.x Core Manual - Draft

## 中英对照表

* Client：客户端
* Server：服务器
* Primitive：基本（描述类型）
* Writing：编写（有些地方译为开发）
* Fluent：流式的
* Reactor：反应堆
* Multi-Reactor：多反应堆
* Options：配置项，作为参数时候翻译成选项
* Context：上下文环境
* Undeploy：撤销（反部署，对应部署）
* Unregister：注销（反注册，对应注册）
* Destroyed：销毁
* Handler/Handle：处理器/处理，有些特定处理器未翻译，如Completion Handler等。
* Block：阻塞
* Out of Box：标准环境（开箱即用）
* Timer：计时器
* Eventloop Pool：事件轮询线程池，大部分地方未翻译
* Worker Pool：工作者线程池，大部分地方未翻译
* Sender：发送者
* Consumer：消费者
* Receiver/Recipient：接收者
* Entry：条目（一条key=value的键值对）
* Map：动词翻译成“映射”，名词为数据结构未翻译
* Logging：动词翻译成“记录”，名词翻译成日志器
* Trust Store：受信存储
* Frame：帧
* Event Bus：事件总线
* Buffer：缓冲区（一些地方使用的Vert.x类中的Buffer类则不翻译）
* Chunk：块（HTTP数据块，分块传输、分块模式中会用到）
* Pump：泵（平滑流式数据读入内存的机制，防止一次性将大量数据读入内存导致内存溢出）
* Header：请求/响应头
* Body：请求/响应体（有些地方翻译成请求/响应正文）
* Pipe：管道
* Round-Robin：轮询
* Flush：刷新
* Datagram：数据报
* Socket：套接字（有些地方未翻译，直接用的Socket）
* Multicast：多播（组播）
* High Availability：高可用性
* Fail-Over：故障转移

> 请注意  *Vert.x和Vertx的区别：文中所有Vert.x概念使用标准单词Vert.x，而Vertx通常表示Java中的类：_`io.vertx.core.Vertx`_。*

## 正文

Vert.x的核心Java API被我们称为**Vert.x Core**，[文档地址](https://github.com/eclipse/vert.x)。

Vert.x Core提供了下列功能

* 编写TCP客户端和服务器
* 编写支持WebSocket的HTTP客户端和服务器
* 事件总线
* 共享数据——本地的Map和分布式集群Map
* 周期性、延迟性动作
* 部署和撤销Verticle实例
* 数据报套接字
* DNS客户端
* 文件系统访问
* 高可用性
* 集群

Core中的功能相当底层，您在此不会找到诸如数据库访问、授权或高层Web应用的功能，您可以在**Vert.x ext** \[1\]（扩展包）中找到这些功能。

**Vert.x Core**小而轻，你可以只使用你需要的部分。它可整体嵌入现存应用中。我们并不会强迫您用特定的方式构造您的应用。

您亦可在其它Vert.x支持的语言中使用Core。很酷的是：我们并不强迫您在书写诸如JavaScript或Ruby时直接调用Java API，毕竟不同的语言有不同的代码风格，若强行让Ruby开发人员遵循Java的代码风格会很怪异，所以我们根据Java API自动生成了适应不同语言代码风格的API。

从现在开始文中我们使用core代表**Vert.x Core**。

如果您在使用Maven或Gradle \[2\]，将下列依赖项添加到您的项目描述（文件）中`dependencies`节点`section`来访问**Vert.x Core**的API：

* Maven（您的`pom.xml`中）

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-core</artifactId>
  <version>3.4.1</version>
</dependency>
```

* Gradle（您的`build.gradle`中）

```gradle
dependencies {
    compile 'io.vertx:vertx-core:3.4.1'
}
```

接下来讨论Vert.x Core中不同的概念和特性。

### 故事从Vert.x开始

> 请注意：*本文大部分内容专用于Java语言——若有需要可以切换到语言特定部分（手册中）。*

除非您拿到[Vert.x](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html)对象，否则在Vert.x领域中您做不了太多的事情。

它是Vert.x的控制中心，也是您做几乎一切事情（的基础），包括创建客户端和服务器、获取事件总线的引用、设置定时器等其他很多事情。

那么如何获取它的实例呢？

如果您用嵌入方式使用Vert.x，可通过以下代码创建实例：

```java
Vertx vertx = Vertx.vertx();
```

如您使用Verticles，在Verticle中会有一个内置的vertx对象，您可直接使用该内置对象，无需重新创建。

> 请注意：*大部分应用将只会需要一个Vert.x实例，但如果您有需要也可创建多个Vert.x实例，如：隔离的事件总线或不同组的客户端和服务器。*

#### 创建Vertx对象时指定配置项

如果缺省的配置不适合您，可在创建Vertx对象的同时指定配置项：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().setWorkerPoolSize(40));
```

[VertxOptions](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html)对象有很多配置，包括集群、高可用、池大小等。在Javadoc中描述了所有配置的细节。

#### 创建集群的Vert.x对象

如果您想创建一个集群的Vert.x（参考[event bus](#event_bus)章节了解更多事件总线集群细节），那么通常情况下您将需要使用另一种异步的方式来创建Vertx对象。

这是因为让不同的Vert.X实例组成一个集群需要一些时间（也许是几秒钟）。在这段时间内，我们不想去阻塞调用线程，所以我们将结果异步返回给您。

### 是流式的吗？

您也许注意到前边的例子里使用了一个流式的API。

一个流式的API表示将多个方法的调用链在一起。例如：

```java
request.response().putHeader("Content-Type", "text/plain").write("some text").end();
```

这是贯穿Vert.x的API中的一个通用模式，所以请适应这种代码风格。

这样的链式调用会让您的代码更为简洁。当然，如果您不喜欢Fluent的方式，我们不强制您用这种方式书写代码。如果您更倾向于用以下方式编码，您可以忽略它：

```java
HttpServerResponse response = request.response();
response.putHeader("Content-Type", "text/plain");
response.write("some text");
response.end();
```

### Don’t call us, we’ll call you

Vert.x的API大部分都是事件驱动。这意味着当您感兴趣的事情发生时，它会以事件的形式发送给您。

以下是一些事件的例子：

* 触发一个计时器
* 一个Socket收到了一些数据
* 从磁盘中读取了一些数据
* 发生了一个异常
* HTTP服务器收到了一个请求

您提供处理器给Vert.x API来处理事件。例如每隔一秒发送一个事件的计时器：

```java
vertx.setPeriodic(1000, id -> {
  // This handler will get called every second
  // 这个处理器将会每隔一秒被调用一次
  System.out.println("timer fired!");
});
```

又或者收到一个HTTP请求：

```java
server.requestHandler(request -> {
  // This handler will be called every time an HTTP request is received at the server
  // 服务器每次收到一个HTTP请求时这个处理器将被调用
  request.response().end("hello world!");
});
```

稍后当Vert.x有一个事件要传给您的处理器时，它会**异步地**调用这个处理器。

由此引入了一些Vert.x中的重要概念：

### 不要阻塞我

除了很少的特例（如以“Sync"结尾的某些文件系统操作），Vert.x中的所有API都不会阻塞调用线程。

如果可以立即提供结果，它将立即返回，否则您将在稍后提供一个处理器来接收事件。

因为没有一个Vert.x API会阻塞线程，这意味着您可以使用Vert.x用少量线程来处理大量并发。

当使用传统的阻塞式API做以下操作时，调用线程可能会被阻塞：

* 从Socket中读取数据
* 写数据到磁盘
* 发送消息给接收者并等待回复
* ...其他很多情况

在上述所有情况下，当您的线程在等待处理结果时它不做任何事——此时，这些线程并无实际用处。

这意味着如果您使用阻塞式API处理大量并发，您需要大量线程来防止应用程序逐步停止运转。

线程之间的切换以及线程本身（例如它们的栈）都会消耗内存。

这意味着，以现代应用程序所要求的并发级别，阻塞的方式将会难于扩展。

### 反应堆和多反应堆

我们前边提过Vert.x的API都是事件驱动的——当有事件时Vert.x将事件传给处理器。

在多数情况下，Vert.x使用被称为**Event Loop**的线程来调用您的处理器。

当Vert.x或应用程序块中没有任何阻塞时，这个**Event Loop**可以快速地运行，以便在事件到达时向不同的处理器成功传递事件。

当没有东西被阻塞住时，一个Event Loop可在短时间内传递大量的事件。例如，一个单独的**Event Loop**可以很快处理数千个HTTP请求。

我们称之为[反应堆模式](https://en.wikipedia.org/wiki/Reactor_pattern)。

您之前也许听说过它——例如Node.js实现了这种模式。

在一个标准反应堆实现中，有一个独立的Event Loop会轮询执行，当有事件到达时将这些事件传递给所有处理器。

一个线程的麻烦就是它在任何一个时间只能在一个单一的核上运行，如果您希望单线程反应堆应用（如您的Node.js应用）扩展到多核服务器上，则不得不启动并且管理许多不同的进程。

这一点Vert.x的工作有所不同，每个Vertx实例维护的不是一个Event Loop，而是多个。默认情况下，我们会根据机器上可用的核数量来设置Event Loop的数量，您亦可自行设置。

与Node.js不同，这意味着单个Vertx进程可以跨服务器扩展。

我们将这种模式称为多反应堆模式，区别于单线程反应堆模式。

_注意：即使一个Vertx实例维护了多个Event Loop，任何特定的处理器永远不会同时执行，大部分情况下（除开_[_Worker Verticle_](http://vertx.io/docs/vertx-core/java/#worker_verticles)之_外）它们总是在完全相同的Event Loop中被调用。_

### 黄金法则——不要阻塞Event Loop

尽管我们已经知道，Vert.x的API都是非阻塞式的并且不会阻塞Event Loop，但是这并不能帮您避免如果**您执意**在自己的处理器中阻塞Event Loop的情况发生。

如果这样做，该Event Loop在被阻塞时就不能做任何事情。如果您阻塞了Vertx实例中的所有Event Loop，那么您的应用就会完全停止！

所以不要这样做！**您被警告了哦**

这些阻塞做法包括：

* `Thead.sleep()`
* 等待一个锁
* 等待一个互斥量或监视器【Mutex & Monitor】（参考：Synchronized Section）
* 执行一个长时间数据库操作并等待其结果
* 执行一个很占可用时间的复杂运算
* 在循环语句中长时间逗留

如果上述任何一种情况阻止了Event Loop并占用了**显著执行时间**，那您应立即进入淘气楼梯反省，并等待进一步指示。

所以，什么是**显著执行时间**？

您要等多久？它取决于您的应用程序和所需的并发数量。

如果您有一个单独的Event Loop，而且您希望每秒处理10000个HTTP请求，很明显的是每一个请求处理时间不可以超过0.1毫秒，所以您不能阻塞任何过多（大于0.1毫秒）的时间。

**这个数学题并不难，将留给读者作为练习。**

如果您的应用程序没有响应，可能是一个迹象，表明您在某个地方阻塞了Event Loop。为了帮助您诊断类似问题，若Vert.x检测到Event Loop有一段时间没有响应，将会自动记录这种警告。若您在日志中看到类似警告，那么您需要检查您的代码。

```
Thread vertx-eventloop-thread-3 has been blocked for 20458 ms
```

Vert.x还将提供堆栈跟踪，以精确定位发生阻塞的位置。

如果想关闭这些警告或更改设置，您可以在创建Vertx对象之前在VertxOptions中完成此操作。

### 运行阻塞式代码

在一个完美的世界中，不存在战争和饥饿，所有的API都将使用异步方式编写，兔兔和小羊羔将会在阳光明媚的绿色草地上手牵手地跳舞。

**但是……真实世界并非如此（您最近看新闻了吧？）**

事实是，很多，也非所有的库，特别是在JVM生态系统中有很多同步API，这些API中许多方法都是阻塞式的。一个很好的例子就是JDBC API——它本质上是同步的，无论多么努力地去尝试，Vert.x都不能像魔法小精灵撒尘变法一样将它转换成异步API。

我们不会将所有的内容重写成异步方式，所以我们为您提供一种在Vert.x应用中安全调用"传统"阻塞API的方法。

如之前讨论，您不能在Event Loop中直接调用阻塞式操作，因为这样做会阻止Event Loop执行其他有用的任务。那您该怎么做？

可以通过调用[executeBlocking](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-)方法来指定阻塞式代码的执行以及阻塞式代码执行后处理结果的异步回调。

```java
vertx.executeBlocking(future -> {
  // Call some blocking API that takes a significant amount of time to return
  // 调用一些需要耗费显著执行时间返回结果的阻塞式API
  String result = someAPI.blockingMethod("hello");
  future.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```

默认情况下，如果executeBlocking在同一个上下文环境中（如：同一个Verticle实例）被调用了多次，那么这些不同的executeBlocking代码块会顺序执行（一个接一个）。

若您不需要关心您调用[executeBlocking](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-)的顺序，可以将`ordered`参数的值设为false。这样任何executeBlocking都会在一个Worker Pool \(1\)中并行执行。

另外一种运行阻塞式代码的方法是使用[Worker Verticle](http://vertx.io/docs/vertx-core/java/#worker_verticles)。

一个Worker Verticle始终会使用Worker Pool中的某个线程来执行。

默认的阻塞式代码会在Vert.x的Worker Pool中执行，通过[setWorkerPoolSize](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#setWorkerPoolSize-int-)配置。

出于不同的目的，可创建新的线程池：

```java
WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool");
executor.executeBlocking(future -> {
  // Call some blocking API that takes a significant amount of time to return
  // 调用一些需要耗费显著执行时间返回结果的阻塞式API
  String result = someAPI.blockingMethod("hello");
  future.complete(result);
}, res -> {
  System.out.println("The result is: " + res.result());
});
```

这个`worker executor` \(2\)在不需要的时候必需被关闭：

```java
executor.close();
```

当使用同一个名字创建了许多worker时，它们将共享同一个pool，所有的worker executor调用了`closed()`被关闭过后，这个worker pool会被销毁。

如果`executor`在Verticle中创建，那么Verticle实例销毁的同时Vert.x将会自动关闭这个`executor`。

worker executor可以在创建的时候配置：

```java
int poolSize = 10;

// 2 minutes
// 2分钟
long maxExecuteTime = 120000;

WorkerExecutor executor = vertx.createSharedWorkerExecutor("my-worker-pool", poolSize, maxExecuteTime);
```

_注意：这个配置信息在worker pool创建的时候设置。_

### 异步协调

多个操作调用结果的协调是由Vert.x中的[futures](http://vertx.io/docs/apidocs/io/vertx/core/Future.html)来完成。它支持并发合并（并行执行多个异步调用）和顺序合并（依次执行异步调用）。

#### 并发合并【Concurrent Composition】

**1.all**

[CompositeFuture.all](http://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#all-io.vertx.core.Future-io.vertx.core.Future-)方法接受多个future对象作为参数（最多6个），当所有的future都成功完成，该方法将返回一个_成功的_future；当任一个future执行失败，则返回一个_失败的_future：

```java
Future<HttpServer> httpServerFuture = Future.future();
httpServer.listen(httpServerFuture.completer());

Future<NetServer> netServerFuture = Future.future();
netServer.listen(netServerFuture.completer());

CompositeFuture.all(httpServerFuture, netServerFuture).setHandler(ar -> {
  if (ar.succeeded()) {
    // All servers started
    // 所有服务器启动完成
  } else {
    // At least one server failed
    // 有一个服务器启动失败
  }
});
```

所有被合并的future中的操作同时运行，当结果future返回时，附在返回future上的处理器（[Handler](http://vertx.io/docs/apidocs/io/vertx/core/Handler.html)）会被调用。当一个操作失败（传入一个future被标记成failure），则返回的future会被标记为失败。当所有的操作都成功时，返回的future将会成功完成。

您可以传入一个future的列表（默认为空）：

```java
CompositeFuture.all(Arrays.asList(future1, future2, future3));
```

**2.any**

不同于`all`的合并会等待所有的future成功执行（或任一失败），`any`的合并会等待第一个成功执行的future。[CompositeFuture.any](http://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#any-io.vertx.core.Future-io.vertx.core.Future-)接受多个future作为参数（最多6个），并将结果归并成一个future，future中的结果是_成功的_当任一future成功完成；是_失败的_当所有的future都执行失败：

```java
CompositeFuture.any(future1, future2).setHandler(ar -> {
  if (ar.succeeded()) {
    // At least one is succeeded
    // 至少一个成功
  } else {
    // All failed
    // 所有的都失败
  }
});
```

它也可使用future列表传参：

```java
CompositeFuture.any(Arrays.asList(f1, f2, f3));
```

**3.join**

`join`的合并会等待所有的future完成，无论成败。

[CompositeFuture.join](http://vertx.io/docs/apidocs/io/vertx/core/CompositeFuture.html#join-io.vertx.core.Future-io.vertx.core.Future-)接受多个future作为参数（最多6个），并将结果归并成一个future，future中的结果是_成功的_当全部future成功完成；是_失败的_当任一future执行失败。

```java
CompositeFuture.join(future1, future2, future3).setHandler(ar -> {
  if (ar.succeeded()) {
    // All succeeded
    // 所有都成功
  } else {
    // All completed and at least one failed
    // 至少一个失败
  }
});
```

它也可使用future列表传参：

```java
CompositeFuture.join(Arrays.asList(future1, future2, future3));
```

#### 顺序合并【Sequential Composition】

**1.compose**

和`all`以及`any`实现的并发合并不同，[compose](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#compose-io.vertx.core.Handler-io.vertx.core.Future-)方法作用于链式future（顺序合并）。

```java
FileSystem fs = vertx.fileSystem();
Future<Void> startFuture = Future.future();

Future<Void> fut1 = Future.future();
fs.createFile("/foo", fut1.completer());

fut1.compose(v -> {
  // When the file is created (fut1), execute this:
  // fut1中文件创建完成后执行
  Future<Void> fut2 = Future.future();
  fs.writeFile("/foo", Buffer.buffer(), fut2.completer());
  return fut2;
}).compose(v -> {
          // When the file is written (fut2), execute this:
          // fut2文件写入完成后执行
          fs.move("/foo", "/bar", startFuture.completer());
        },
        // mark startFuture it as failed if any step fails.
        // 如果任何一步失败，将startFuture标记成failed
        startFuture);
```

这里例子中，有三个操作被串起来了：

1. 一个文件被创建（`fut1`）
2. 一些东西被写入到文件（`fut2`）
3. 文件被移走（`startFuture`）

如果这三个步骤全部成功，则最终的future（`startFuture`）返回_成功_，其中任何一步失败，则最终future返回_失败_。

例子中使用了：

* [compose](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#compose-io.vertx.core.Handler-io.vertx.core.Future-)：当前future完成时，执行相关代码，并返回future。当返回的future完成时，合并完成。
* [compose](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#compose-io.vertx.core.Handler-io.vertx.core.Future-)：当前future完成时，执行相关代码，并完成下一个future的处理。

在第二个例子中，处理器（[Handler](http://vertx.io/docs/apidocs/io/vertx/core/Handler.html)）应该完成下一个（`next`）future过后来报告成功或者失败。

您可以使用[completer](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#completer--)方法来串起一个带操作结果的或失败的future，它可使您避免用传统方式编写代码：如果成功则完成future，否则就失败。

### Verticles

Vert.x提供了一个简单便捷的、可扩展的、类似Actor-Model的部署和并发模型机制，您可以用此模型机制来保管您自己的代码组件。

**这个模型是可选的，如果您不想这样做，Vert.x不会强迫您用这种方式创建您的应用程序。**

这个模型不能说是严格的Actor-Model的实现，但它确实有相似之处，特别是在并发、扩展和部署等方面。

要使用该模型，您需要将您的代码组织成**verticle**的集合。

Verticle是由Vert.x部署和运行的代码块，默认情况一个Vert.x实例维护了N（默认：N = 核 x 2）个Event Loop线程。Verticle实例可使用任意Vert.x支持的计算机语言编写，一个简单的应用程序也可以包含多种语言编写的Verticle。

您可以将Verticle想成[Actor Model](https://en.wikipedia.org/wiki/Actor_model)中的Actor。

一个应用程序通常是由同一时间运行在同一个Vert.x实例中的许多Verticle实例组合而成。不同的Verticle实例通过事件总线发送消息相互通信。

#### 编写Verticle

Verticle类必须实现[Verticle](http://vertx.io/docs/apidocs/io/vertx/core/Verticle.html)接口。

如果您喜欢可以直接实现该接口，但是通常直接从抽象类[AbstractVerticle](http://vertx.io/docs/apidocs/io/vertx/core/AbstractVerticle.html)继承更简单。

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

通常您需要像上边例子一样重写start方法。

当Vert.x部署一个Verticle时，它的start方法将被调用，这个方法执行完成后Verticle会是started（的状态）。

您同样可以重写stop方法，当Vert.x撤销一个Verticle时它会被调用，这个方法执行完成后Verticle会是stopped（的状态）。

#### Verticle异步启动和停止

有些时候您的Verticle启动会耗费一些时间，您想要在这个过程做一些事，并且您做的这些事并不想等到Verticle部署完成过后再发生。如：您想在start方法中部署其他的Verticle。

您不能在您的start方法中阻塞等待其他的Verticle部署完成，这样做会破坏[黄金法则](http://vertx.io/docs/vertx-core/java/#golden_rule)。

所以您要怎么做？

您可以实现异步版本的start方法来做这个事。这个版本的方法需要一个Future作参数，方法执行完时，Verticle实例并没有部署好（状态不是deployed）。

稍后，您完成了所有您需要做的事（如：启动其他Verticle），您可以调用Future的complete（或fails）方法来示意您完成了。

这儿有一个例子：

```java
public class MyVerticle extends AbstractVerticle {

  public void start(Future<Void> startFuture) {
    // Now deploy some other verticle:
    // 现在部署其他的一些verticle
    vertx.deployVerticle("com.foo.OtherVerticle", res -> {
      if (res.succeeded()) {
        startFuture.complete();
      } else {
        startFuture.fail(res.cause());
      }
    });
  }
}
```

同样的，这儿也有一个异步版本的stop方法，如果您想做一些耗时的Verticle清理工作，您可以使用它。

```java
public class MyVerticle extends AbstractVerticle {

  public void start() {
    // Do something
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

_注意：您不需要在一个Verticle的stop方法中手工去撤销启动时部署的子Verticle，当父Verticle在撤销时Vert.x会自动撤销任何子Verticle。_

#### Verticle类型

这儿有三种不同类型的Verticle：

* **Stardand Verticles**：这是常用的一类Verticle——它们会被一个Event Loop线程执行，稍后的章节我们会讨论更多。
* **Worker Verticles**：这类Verticle会使用Worker Pool中的线程，一个实例绝对不会被多个线程同时执行。
* **Multi-threaded worker verticles**：这类Verticle也会使用Worker Pool中的线程，一个实例可以由多个线程同时执行。

#### Standard Verticles

当Standard Verticle被创建时，它会被分派给一个Event Loop线程，并在这个Event Loop中执行它的start方法。当您在一个Event Loop调用了Core API中处理器上任意其他方法时，Vert.x将保证这些处理器在调用时在相同的Event Loop中执行。

这意味着我们可以保证您的Verticle实例中所有的代码都是在相同Event Loop中执行（只要您不创建自己的线程并调用它！）

同样意味着您可以将您的应用中的所有代码用单线程方式编写，让Vert.x去担心线程和扩展问题。（您）不再担心同步和不稳定，当多线程应用开发使用传统方式手工回滚时，这种避免其他竞争条件和死锁（的方式）会更流行。

#### Worker Verticles

一个Worker Verticle和Standard Verticle很像，但它并不是由一个Event Loop来执行，而是由Vert.x中的Worker Pool中的线程执行。

Worker Verticle被设计来调用阻塞式代码，但它不会阻塞任何Event Loop。

如果您不想使用Worker Verticle来运行阻塞式代码，您还可以在一个Event Loop中直接使用[内联阻塞式代码](http://vertx.io/docs/vertx-core/java/#blocking_code)（前文中executeBlocking）。

若您想要将Verticle部署成一个Worker Verticle，用[setWorker](http://vertx.io/docs/apidocs/io/vertx/core/DeploymentOptions.html#setWorker-boolean-)来做：

```java
DeploymentOptions options = new DeploymentOptions().setWorker(true);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

Worker Verticle实例绝对不会在Vert.x中被多个线程同时执行，但它可以在不同时间由不同线程执行。

**Multi-threaded worker verticles**

一个multi-threaded worker verticle近似于普通的Worker Verticle，但是它可以由不同的线程同时执行。

*警告：Multi-threaded worker verticle是一个高级功能，大部分应用程序不会需要它。由于在这些Verticle中的并发性，您必须非常小心和使用Java技术中多线程编程方式保持状态一致性。*

#### 编程方式部署Verticle

您可以指定一个Verticle名称或传入您已经创建好的Verticle实例，使用任意一个[deployVerticle](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#deployVerticle-io.vertx.core.Verticle-)方法来部署Verticle。

*注意：部署Verticle仅限Java语言*

```java
Verticle myVerticle = new MyVerticle();
vertx.deployVerticle(myVerticle);
```

您同样可以指定Verticle的名称来部署它。

这个Verticle的名称会用于查找实例化Verticle的特定[VerticleFactory](http://vertx.io/docs/apidocs/io/vertx/core/spi/VerticleFactory.html)。

不同的Verticle Factory会对实例化不同语言的Verticle以及其他原因如加载服务、运行时从Maven中获取Verticle等——合法可用。

这允许您部署用任何Vert.x支持的语言编写的Verticle实例。

这儿有一个部署不同类型Verticle的例子：

```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle");

// Deploy a JavaScript verticle
// 部署JavaScript的Verticle
vertx.deployVerticle("verticles/myverticle.js");

// Deploy a Ruby verticle verticle
// 部署Ruby的Verticle
vertx.deployVerticle("verticles/my_verticle.rb");
```

#### Verticle名称到Factory的映射规则

若使用名称部署Verticle，这个名称用于选择一个将要实例化Verticle的Factory。

Verticle名称可以有一个前缀——使用字符串紧跟着一个冒号，它用于查找存在的Factory，参考例子。

```
js:foo.js // 使用JavaScript的Factory
groovy:com.mycompany.SomeGroovyCompiledVerticle // 用Groovy的Factory
service:com.mycompany:myorderservice // 用Service的Factory
```

如果不指定前缀，Vert.x将根据提供名字后缀来查找对应Factory，如：

```
foo.js // 将使用JavaScript的Factory
SomeScript.groovy // 将使用Groovy的Factory
```

若前缀后缀都没指定，Vert.x将假定这个名字是一个Java限定类全名（FQCN）然后尝试实例化它。

#### 如何定位Verticle Factory？

大部分Verticle Factory会在Vert.x启动时注册并且从类路径（CLASSPATH）中加载。

您同样可以使用编程的方式去注册或注销Verticle Factory：[registerVerticleFactory](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#registerVerticleFactory-io.vertx.core.spi.VerticleFactory-)和[unregisterVerticleFactory](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#unregisterVerticleFactory-io.vertx.core.spi.VerticleFactory-)

#### 等待部署完成

Verticle的部署是异步方式，可能在deploy方法调用返回后一段时间才会完成部署。

如果您想要在部署完成时发出通知则可以指定一个Completion Handler。

```java
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", res -> {
  if (res.succeeded()) {
    System.out.println("Deployment id is: " + res.result());
  } else {
    System.out.println("Deployment failed!");
  }
});
```

如果部署成功，这个Completion Handler的结果（result）中将会传入一个包含了部署id的字符串。

这个部署ID可以在之后您想要撤销它时使用。

#### 撤销Verticle

部署（好的Verticle）可以使用[undeploy](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#undeploy-java.lang.String-)被撤销。

撤销也是异步方式，因此若您想要在撤销完成过后收到通知则可以指定另一个Completion Handler。

```java
vertx.undeploy(deploymentID, res -> {
  if (res.succeeded()) {
    System.out.println("Undeployed ok");
  } else {
    System.out.println("Undeploy failed!");
  }
});
```

#### 设置Verticle实例数

当使用名称部署一个Verticle时，您可以指定您想要部署的Verticle实例的数量。

```java
DeploymentOptions options = new DeploymentOptions().setInstances(16);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

若跨多核简化水平扩展时这个功能很有用。如，您的计算机上有一个包含Web服务器的Verticle需要部署以及跨多核，因此您需要部署多个实例来利用所有的（服务器）核。

#### 传入配置给Verticle

一个JSON格式的配置可在部署时传给Verticle

```java
JsonObject config = new JsonObject().put("name", "tim").put("directory", "/blah");
DeploymentOptions options = new DeploymentOptions().setConfig(config);
vertx.deployVerticle("com.mycompany.MyOrderProcessorVerticle", options);
```

这个配置之后通过[Context](http://vertx.io/docs/apidocs/io/vertx/core/Context.html)对象或直接使用[config](http://vertx.io/docs/apidocs/io/vertx/core/AbstractVerticle.html#config--)方法访问【Available】。

这个配置会返回一个JSON对象，因此您可以用下边代码读取之中数据：

```java
System.out.println("Configuration: " + config().getString("name"));
```

#### 访问Verticle环境变量

环境变量和系统属直接通过Java API访问：

```java
System.getProperty("prop");
System.getenv("HOME");
```

#### Verticle隔离组【Isolation Groups】

默认情况，Vert.x有一个水平类路径【flat classpath】，当Vert.x部署Verticle时它会调用当前类加载器来做——它不会创建一个新的（类加载器），大多数情况下，这是最简单、最清晰和最干净的事。

但是在某些情况下，您可能需要部署一个Verticle，它包含的类要与应用程序中其他类隔离开来。

可能是这种情况，例如：您想要在一个Vert.x实例中部署两个同名不同类的Verticle，或者在同一个jar库文件中您有两个不同版本的Verticle。

当使用隔离组时，您需要用[setIsolatedClassed](http://vertx.io/docs/apidocs/io/vertx/core/DeploymentOptions.html#setIsolatedClasses-java.util.List-)方法，并提供一个您想隔离的类名列表给它——（列表）项可以是一个Java限定类全名如`com.mycompany.myproject.engine.MyClass`，也可以是包含通配符的可匹配某个包或子包的任何类。例如`com.mycompany.myproject.*`将会匹配所有`com.mycompany.myproject`包或任意子包中的任意类名。

请注意仅仅只有匹配的类会被隔离——其他任意类会被当前类加载器加载。

若您想要加载的类和资源不存在于主类路径，您可使用[setExtraClasspath](http://vertx.io/docs/apidocs/io/vertx/core/DeploymentOptions.html#setExtraClasspath-java.util.List-)将额外的类路径添加到这里。

*警告：谨慎使用此功能，类加载器可能是一个蠕虫病毒，除其他事情外它会使调试变得困难。*

以下是使用隔离组隔离Verticle的部署例子：

```java
DeploymentOptions options = new DeploymentOptions().setIsolationGroup("mygroup");
options.setIsolatedClasses(Arrays.asList("com.mycompany.myverticle.*",
                   "com.mycompany.somepkg.SomeClass", "org.somelibrary.*"));
vertx.deployVerticle("com.mycompany.myverticle.VerticleClass", options);
```

#### 高可用性【High Availability】

Verticle可以启用高可用方式（HA）部署，在上下文环境中，当其中一个部署在Vert.x实例中的Verticle突然死掉，这个Verticle可以在集群环境中的另一个Vert.x实例中重新部署。

若要启用高可用方式运行一个Verticle，仅需要追加`-ha`参数：

```
	vertx run my-verticle.js -ha
```

当启用高可用方式时，不需要追加`-cluster`参数。

关于高可用的功能和配置可参考[高可用和故障转移](http://vertx.io/docs/vertx-core/java/#_high_availability_and_fail_over)

#### 从命令行运行Verticle

您可以在Maven或Gradle项目中以正常方式添加Vert.x Core到项目依赖节点，从哪里直接使用Vert.x。

但是，您也可以从命令行直接运行Vert.x的Verticle。

为此，您需要下载并安装Vert.x的发行版【distribution】，并且将安装的`bin`目录添加到您的`PATH`环境变量中，还要确保您的`PATH`中设置了Java 8的JDK环境。

*注意：JDK需要支持Java代码的快速编译【Fly compilcation】。*

现在您可以使用`vertx run`命令运行Verticle了，这儿是一些例子：

```
# Run a JavaScript verticle
# 运行JavaScript的Verticle
vertx run my_verticle.js

# Run a Ruby verticle
# 运行Ruby的Verticle
vertx run a_n_other_verticle.rb

# Run a Groovy script verticle, clustered
# 使用集群模式运行Groovy的Verticle
vertx run FooVerticle.groovy -cluster
```

您甚至可以不必编译Java源代码，直接运行它：

```
vertx run SomeJavaSourceFile.java
```

Vert.x将在运行它之前对Java源代码文件执行快速编译，这对于快速原型制作很有用并非常适用于演示——没有必要首先设置一个Maven或Gradle来构建！

有关在命令行执行`vertx`可用的各种选项完整信息，可以直接在命令行键入`vertx`（查看）。

#### 导致Vert.x退出

所有Vert.x实例维护的线程不是守护线程，所以它们将会阻止JVM退出。

如果您正在嵌入Vert.x并且完成了它，您可以调用[close](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#close--)方法关闭它。

这将关闭所有内部线程池并关闭其他资源，而且将允许JVM退出。

#### Context对象

当Vert.x传递一个事件给处理器或者调用Verticle的`start`或`stop`方法时，它的执行会关联一个`Context`对象。通常一个context又是一个event-loop context并且绑定到一个特定的event-loop线程，所以执行该context总是在同一个event loop线程中；同样Worker Vertcle以及运行的内联阻塞式代码中，一个worker context将会绑定其执行关联到一个特定的worker pool的线程中。

您可用[getOrCreateContext](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#getOrCreateContext--)方法获取context：

```java
Context context = vertx.getOrCreateContext();
```

若已经有一个context和当前线程关联，那么它直接重用这个context对象，如果没有则它会创建一个新的。您可以测试获取的context的类型：

```java
Context context = vertx.getOrCreateContext();
if (context.isEventLoopContext()) {
  System.out.println("Context attached to Event Loop");
} else if (context.isWorkerContext()) {
  System.out.println("Context attached to Worker Thread");
} else if (context.isMultiThreadedWorkerContext()) {
  System.out.println("Context attached to Worker Thread - multi threaded worker");
} else if (! Context.isOnVertxThread()) {
  System.out.println("Context not attached to a thread managed by vert.x");
}
```

当您获取了这个context对象，您就可以在context中异步执行代码了。换句话说，您提交一个任务（和之后的任务）最终将会在同一个context中运行：

```java
vertx.getOrCreateContext().runOnContext( (v) -> {
  System.out.println("This will be executed asynchronously in the same context");
});
```

若同一个context中运行了多个处理器，它们也许想要共享数据，这个context对象提供了在context中存储和读取数据（实现）共享的方法。例如：它允许您将数据传递到[runOnContext](http://vertx.io/docs/apidocs/io/vertx/core/Context.html#runOnContext-io.vertx.core.Handler-)运行的某些操作中。

```java
final Context context = vertx.getOrCreateContext();
context.put("data", "hello");
context.runOnContext((v) -> {
  String hello = context.get("data");
});
```

这个context对象还可以让您使用[config](http://vertx.io/docs/apidocs/io/vertx/core/Context.html#config--)方法访问Verticle的配置信息，查看传入配置给Verticle章节了解更多配置信息。

#### 执行周期性/延迟性操作

Vert.x中，想要延迟之后执行或定期执行操作很常见。

在Standard Verticle中您不能让线程休眠以引入延迟，因为它会阻塞Event Loop线程。

取而代之是使用Vert.x定时器，定时器可以是一次性【One-shot】或定期的【periodic】，我们将讨论二者。

**一次性计时器【One-shot】**

一个一次性计时器会在一定延迟后调用一个Event Handler，以毫秒为单位。

您可用[setTimer](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setTimer-long-io.vertx.core.Handler-)方法传递延迟和一个处理器来设置计时器的触发。

```java
long timerID = vertx.setTimer(1000, id -> {
  System.out.println("And one second later this is printed");
});

System.out.println("First this is printed");
```

返回值是一个唯一的计时器id，该id可用于之后取消该计时器，这个计时器id会传入给处理器。

**周期性计时器【Periodic】**

您同样可以使用[setPeriodic](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setPeriodic-long-io.vertx.core.Handler-)设置一个周期性触发的计时器。

这儿将有一个等于该时间的初始化延迟。

setPeriodic的返回值也是一个唯一的计时器id，若之后该计时器需要取消则使用该id。

传给这个计时器中处理器的参数也是这个唯一的计时器id。

请记住这个计时器将会定期触发，如果您的定期（任务）执行【treatment】将会花费大量时间，您的计时器事件能持续执行或最坏的情况：重叠。

这种情况，您应考虑使用[setTimer](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#setTimer-long-io.vertx.core.Handler-)，一旦任务执行完成您可设置下一个计时器。

```java
long timerID = vertx.setPeriodic(1000, id -> {
  System.out.println("And every second this is printed");
});

System.out.println("First this is printed");
```

**取消计时器**

指定一个计时器id并调用[cancelTimer](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#cancelTimer-long-)来取消一个周期性计时器。如：

```java
vertx.cancelTimer(timerID);
```

**Verticle中自动清除**

如果您在Verticle中创建了计时器，当这个Verticle被撤销时这个计时器会被自动关闭。

#### Verticle工作线程池

Verticle使用Vert.x中的工作线程池【Worker Pool】来执行阻塞式行为——如：[executeBlocking](http://vertx.io/docs/apidocs/io/vertx/core/Context.html#executeBlocking-io.vertx.core.Handler-boolean-io.vertx.core.Handler-)或Worker Verticle。

还可以在部署配置项中指定不同的工作线程池：

```java
vertx.deployVerticle("the-verticle", new DeploymentOptions().setWorkerPoolName("the-specific-pool"));
```

### Event Bus

Event Bus是Vert.x中的神经系统【Nervous System】。

每一个Vert.x实例都有一个单独的Event Bus实例，可通过方法[eventBus](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#eventBus--)来获得。

Event Bus您的应用中的不同部分相互通信，无论它们使用哪一种语言实现，以及它们是否在同一个Vert.x实例中，或在不同的Vert.x实例中。

它甚至可以桥接允许在浏览器中运行的客户端JavaScript在相同的Event Bus上相互通信。

Event Bus可形成跨越多个服务器节点和多个浏览器的分布式对等消息系统。

Event Bus支持发布/订阅、点对点、请求/响应的消息通信（模式）。

Event Bus的API很简单，它主要涉及注册处理器、撤销处理器和发送和发布消息。

首先看些理论：

#### 理论

**寻址【Addressing】**

消息在Event Bus上会发送到一个地址【Address】。

同任何花哨的寻址方式相比Vert.x（寻址）并不麻烦，Vert.x中的地址是一个简单的字符串，任意字符串都合法。然后，使用某种模式仍然是明智之举，如：使用据点来划分名空间。

一些合法的地址形如：europe.news.feed1、acme.games.pacman, sausages和X。

**处理器【Handlers】**

消息在处理器中被接收，您可以在一个地址上注册一个处理器。

同一个地址可以注册许多不同的处理器，一个处理器也可以在许多不同的地址上注册。

**发布/订阅消息【Publish/Subscribe Messaging】**

Event Bus支持发布消息功能。

消息将被发布到一个地址中，发布意味着将详细传递给所有注册在该地址上的处理器。

这和发布/订阅模式很类似。

**点对点和请求-响应消息**

Event Bus也支持点对点消息。

消息将被发送到一个地址中，Vert.x将会把消息分发【route】到某个注册在该地址上的处理器。

若这个地址上有不止一个注册过的处理器，它将使用不严格的轮询算法选择其中一个。

通过点对点的消息传递，可在消息发送的时候指定可选的Reply Handler。

当接收者收到消息并且已经被处理时，它可以选择性决定回复该消息，若选择回复则Reply Handler将会被调用。

当发送者收到回复消息时，它也可以回复，这个过程可以不断重复，它允许在两个不同的Verticle之间设置一个对话框。

这就是一种称为**请求-响应**模式的消息模式。

**尽力服务传输【Best-effort delivery】**

Vert.x会尽它最大努力去传递消息，并且保证消息不会出现丢失，这种称为**尽力服务**传输。

但是，在Event Bus中全部或部分（位置）发生故障，则可能会丢失消息。

若您的应用关心丢失的消息，您应该编写具有幂等性的处理器，并且您的发送者可以在恢复后重试。

**消息类型【Types of messages】**

标准Vert.x允许任何基本/简单类型、String或[buffers](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)作为消息发送。

不过在Vert.x中通常以[JSON](http://json.org/)格式作为发送消息的惯例和常用实践。

对于Vert.x支持的所有语言，JSON非常容易创建、读取和解析，因此它已经成为了Vert.x中的通用语【lingua franca】。

但是若您不想用它，并不强制您使用JSON。

Event Bus非常灵活并且支持在Event Bus中发送任意对象，您可以针对您想要发送的对象自定义一个[codec](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageCodec.html)来实现。

#### Event Bus API

让我们跳到API

**获取Event Bus**

您可以使用下边代码获取Event Bus的一个引用：

```java
EventBus eb = vertx.eventBus();
```

在一个Vert.x实例中它是单例的。

**注册处理器**

最简单的注册处理器的方式是使用[consumer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#consumer-java.lang.String-io.vertx.core.Handler-)，这儿有个例子：

```java
EventBus eb = vertx.eventBus();

eb.consumer("news.uk.sport", message -> {
  System.out.println("I have received a message: " + message.body());
});
```

当一个消息达到您的处理器，它将被调用，传递[消息](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html)。

consumer()调用过后的返回对象是一个[MessageConsumer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html)的实例。

该对象随后可用于撤销处理器、或将处理器用作流。

或者、您可以不设置处理器而使用[consumer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#consumer-java.lang.String-io.vertx.core.Handler-)返回一个MessageConsumer，之后再来设置处理器。如：

```java
EventBus eb = vertx.eventBus();

MessageConsumer<String> consumer = eb.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
});
```

在集群的Event Bus上注册处理器时，注册会花费一些时间才能到达集群中的所有节点。

若您希望在完成后收到通知，您可以在MessageConsumer对象上注册一个[completion handler](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html#completionHandler-io.vertx.core.Handler-)。

```java
consumer.completionHandler(res -> {
  if (res.succeeded()) {
    System.out.println("The handler registration has reached all nodes");
  } else {
    System.out.println("Registration failed!");
  }
});
```

**撤销处理器**

要撤销处理器，请调用[unregister](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html#unregister--)。

若您位于一个集群的Event Bus中，撤销处理器同样会花费一些时间跨节点传播，若您想在完成后收到通知：

```java
consumer.unregister(res -> {
  if (res.succeeded()) {
    System.out.println("The handler un-registration has reached all nodes");
  } else {
    System.out.println("Un-registration failed!");
  }
});
```

**发布消息**

发布消息很简单，仅用[publish](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#publish-java.lang.String-java.lang.Object-)指定一个地址去发布

```java
eventBus.publish("news.uk.sport", "Yay! Someone kicked a ball");
```

这个消息将会传递给所有在地址news.uk.sport上注册过的处理器。

**发送消息**

发送消息将会把结果传递给在该地址注册的所有处理器中的一个来接收，这是点对点模式，处理器的选择使用不严格的轮询算法。

您可以使用[send](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#send-java.lang.String-java.lang.Object-)来发送消息：

```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball");
```

**设置消息头**

在Event Bus上发送的消息可包含头信息。

这可通过在发送或发布时提供的[DeliveryOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html)来指定。

```java
DeliveryOptions options = new DeliveryOptions();
options.addHeader("some-header", "some-value");
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball", options);
```

**消息顺序**

Vert.x将按照特定发送者发送消息的顺序来传递消息给特定处理器。

**消息对象**

您在Message Handler中接收到的一个对象就是一个[Message](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html)。

消息的[body](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#body--)对应于发送或发布的对象。

消息的头信息在[headers](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#headers--)中可用。

**确认消息/发送回复**

当使用[send](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#send-java.lang.String-java.lang.Object-)(发送消息）时Event Bus会尝试将详细传递到在Event Bus上注册过的[MessageConsumer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html)中。

在某些情况下，发送者知道消费者何时收到消息并“处理”消息是有用的。

要确认消息已被处理，消费者可以通过调用[reply](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#reply-java.lang.Object-)来回复消息。

当这种情况发生，它会将消息回发给发送者并且在回复中调用Reply Handler。

一个例子将使之更清楚：

接收者【Receiver】：

```java
MessageConsumer<String> consumer = eventBus.consumer("news.uk.sport");
consumer.handler(message -> {
  System.out.println("I have received a message: " + message.body());
  message.reply("how interesting!");
});
```

发送者【Sender】：

```java
eventBus.send("news.uk.sport", "Yay! Someone kicked a ball across a patch of grass", ar -> {
  if (ar.succeeded()) {
    System.out.println("Received reply: " + ar.result().body());
  }
});
```

这个回复可有用信息在消息体中。

什么是“处理”？实际上它是应用程序定义的，完全取决于消费者如何执行，不是Vert.x中的Event Bus本身知道或关心的东西，

一些例子：

* 一个简单的实现了一个返回当天时间服务的消息消费者会去确认回复的消息体中包含了当天时间信息。
* 一个实现了持久化队列的消息消费者，也许会用`true`确认消息已经成功持久化到存储中，或`false`表示（持久化）失败。
* 一个处理订单的消息消费者也许会用`true`确认这个订单已经成功处理，因此它可以从数据库中删除。

**带超时的发送**

当发送带有Reply Handler的消息时，可以在[DeliveryOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html)中指定一个超时时间。

如果恢复在这个时间只能没有收到，则Reply Handler将被调用失败。默认超时是30秒。

**发送故障**

消息发送可能会因为其他原因故障，包括：

* 没有和发送消息对应的处理器可用
* 接收者调用了[fail](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/Message.html#fail-int-java.lang.String-)显示声明故障

在所有情况Reply Handler将会针对特定的故障被调用。

**消息编解码器【Message Codecs】**

若您定义并注册了一个[消息编解码](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageCodec.html)器，您可以在Event Bus中发送任意您想要的对象。

消息编解码器有一个名称，并且在发送或发布消息时在[DeliveryOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html)中指定。

```java
eventBus.registerCodec(myCodec);

DeliveryOptions options = new DeliveryOptions().setCodecName(myCodec.name());

eventBus.send("orders", new MyPOJO(), options);
```

若您总是希望将特定类型对象用于相同的编解码器，那么您可以为其注册默认编解码器，这样您就不需要在[DeliveryOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/DeliveryOptions.html)中指定特定编解码器。

```java
eventBus.registerDefaultCodec(MyPOJO.class, myCodec);

eventBus.send("orders", new MyPOJO());
```

您可以使用[unregisterCodec](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBus.html#unregisterCodec-java.lang.String-)撤销一个消息编解码器。

消息编解码器并不一定使用同一类型进行编码和解码。例如您可以编写一个编解码器允许MyPOJO类被编码发送，但是当消息发送给处理器后解码成MyOtherPOJO类。

**集群Event Bus**

Event Bus不仅仅存在于单个Vert.x实例中，通过您在网络上将不同的Vert.x实例集群在一起，它可以形成一个单一的、分布式的Event Bus。

**编程化集群**

若您用编程的方式创建Vert.x实例，则可以通过将Vert.x实例配置成集群来获取集群的Event Bus。

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

您应该确保您有一个[ClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)实现类在您的类路径中，如默认的`HazelcastClusterManager`。

**通过命令行集群**

您可以用下边命令运行一个Vert.x集群

```
vertx run my-verticle.js -cluster
```

**Verticle中的自动清理**

若您在Verticle中注册了Event Bus的处理器，这些处理器在Verticle被撤销的时候会自动被注销。

### 配置Event Bus

Event Bus可以配置，当Event Bus运行在集群模式这是特别有用的。在引擎【hood】之下，Event Bus使用TCP连接发送和接收消息，因此[EventBusOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBusOptions.html)可以让您配置TCP连接的所有方面。由于Event Bus作为了服务器和客户端，这些配置近似于[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)和[NetServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)。

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

上边代码段描述了如何在Event Bus中使用SSL连接替换纯的TCP连接。

*警告：要在集群模式下强制执行安全性，您必须将集群管理器【Cluster】配置成加密的或强制安全性（的方式）。参考集群管理器的文档获取更多细节。*

Event Bus的配置需要在所有集群节点中保持一致性。

[EventBusOptions](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/EventBusOptions.html)还允许您指定Event Bus是否集群、主机信息和端口，您可使用[setClustered](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#setClustered-boolean-)、[getClusterHost](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#getClusterHost--)和[getClusterPort](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#getClusterPort--)这样做。

在容器中使用时，您也可以配置公共主机和端口号：

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

### JSON

和其他一些语言不同，Java没有对JSON的原生支持【first class support】，因此我们提供了两个类，以便在Vert.x应用中处理JSON更容易。

#### JSON对象

[JsonObject](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)类用来描述JSON对象。

一个JSON对象基本上只是一个Map结构，它具有字符串的键，值可以是任意一种JSON支持的类型（string, number, boolean）。

JSON对象也支持null值。

**创建JSON对象**

可以使用默认构造函数创建空的JSON对象。

您可以从JSON格式的字符串创建一个JSON对象：

```java
String jsonString = "{\"foo\":\"bar\"}";
JsonObject object = new JsonObject(jsonString);
```

您可以从一个Map创建一个JSON对象：

```java
Map<String, Object> map = new HashMap<>();
map.put("foo", "bar");
map.put("xyz", 3);
JsonObject object = new JsonObject(map);
```

**将条目放入JSON对象**

使用[put](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html#put-java.lang.String-java.lang.Enum-)方法可以将值放入到JSON对象。

因为Vert.x支持Fluent的API，所以这个方法调用可以是链式化的。

```java
JsonObject object = new JsonObject();
object.put("foo", "bar").put("num", 123).put("mybool", true);
```

**从JSON对象获取值**

您可使用`getXXX`方法从JSON对象中获取值。例如：

```java
String val = jsonObject.getString("some-key");
int intVal = jsonObject.getInteger("some-other-key");
```

**JSON对象和Java对象间的映射**

您可以从Java对象的字段创建一个JSON对象，如下所示：

(官缺）

也可以实例化一个Java对象并从JSON对象填充其字段值。如下所示：

```java
request.bodyHandler(buff -> {
  JsonObject jsonObject = buff.toJsonObject();
  User javaObject = jsonObject.mapTo(User.class);
});
```

请注意上述映射方向都使用了Jackson的`ObjectMapper#convertValue()`来执行映射，字段和构造函数可见性的影响、有关跨对象引用的序列化完和反序列化等可参考Jackson的文档获取更多信息。

然而在最简单的情况下，所有Java类中字段都是public（或者有public的getter/setter）时，并且有一个public的默认构造函数（或不定义构造函数），`mapFrom`和`mapTo`都应该成功。

只要对象图是非循环的，引用对象通过to/from执行过滤性的序列化和反序列化时同样会作用于嵌套JSON对象。

**将JSON对象编码成String**

您可使用[encode](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html#encode--)将一个对象编码成字符串格式。

#### JSON数组

[JsonArray](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html)类用来描述JSON数组。

一个JSON数组是一系列值的有序集（string、number、boolean）。

JSON数组同样可以包含null值。

**创建JSON数组**

可以使用默认构造函数创建空的JSON数组。

您可以从JSON格式的字符串创建一个JSON数组：

```java
String jsonString = "[\"foo\",\"bar\"]";
JsonArray array = new JsonArray(jsonString);
```

**将条目添加JSON数组**

您可以使用[add](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html#add-java.lang.Enum-)方法添加条目到JSON数组中

```java
JsonArray array = new JsonArray();
array.add("foo").add(123).add(false);
```

**从JSON数组中获取值**

您可使用`getXXX`方法从JSON数组中获取值。例如：

```java
String val = array.getString(0);
Integer intVal = array.getInteger(1);
Boolean boolVal = array.getBoolean(2);
```

**将JSON数组编码成String**

您可使用[encode](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonArray.html#encode--)将一个数组编码成字符串格式。

### Buffers

在Vert.x内部大部分数据使用Buffers的格式【Shuffled】

一个Buffer是可以读取或写入的0个或多个字节序列，并且根据需要可以自动扩容、将任意字节写入Buffer。您也许可以将Buffer想成智能字节数组。

#### 创建Buffer

可以使用一个静态方法[Buffer.buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#buffer--)来创建Buffer。

Buffer可以从字符串或字节数组初始化，或者创建空的Buffer。

这儿有一些创建Buffer的例子：

创建一个空的Buffer：

```java
Buffer buff = Buffer.buffer();
```

从字符串创建一个Buffer，这个Buffer中的字符串必须可用UTF-8编码：

```java
Buffer buff = Buffer.buffer("some string");
```
从字符串创建一个BUffer，这个字符串可以用指定的编码方式编码，例如：

```java
Buffer buff = Buffer.buffer("some string", "UTF-16");
```

从字节数组byte[]创建Buffer：

```java
byte[] bytes = new byte[] {1, 3, 5};
Buffer buff = Buffer.buffer(bytes);
```

创建一个带有初始化尺寸的Buffer。若您知道您的Buffer会写入一定量的数据，您可以创建Buffer并指定它的尺寸。这使得这个Buffer初始化时分配了更多的内存，比数据写入时重新调整尺寸效率更高。

注意以这种方式创建的Buffer是空的，它也不会创建一个填满了0的Buffer。

```java
Buffer buff = Buffer.buffer(10000);
```

#### 写入Buffer

写入Buffer的方式有两种：追加和随机访问。任何一种情况下Buffer始终进行自动扩容，所以不可能在Buffer中遇到`IndexOutOfBoundsException`。

1. **追加到Buffer**

您可以使用`appendXXX`方法追加数据到Buffer，它存在各种不同数据类型的方法。

因为`appendXXX`方法的返回值就是Buffer自身，所以它可以链式化【Fluent】:

```java
Buffer buff = Buffer.buffer();

buff.appendInt(123).appendString("hello\n");

socket.write(buff);
```

2. **随机访问写Buffer**

您还可以指定一个索引值，通过`setXXX`方法写入数据到Buffer，它也存在各种不同数据类型的方法。所有的set方法都会将索引值作为第一个参数——这表示Buffer中开始写入数据的位置。

Buffer始终根据需要进行自动扩容：

```java
Buffer buff = Buffer.buffer();

buff.setInt(1000, 123);
buff.setString(0, "hello");
```

#### 从Buffer中读取

可使用`getXXX`方法从Buffer中读取数据，它存在各种不同数据类型的方法，这些方法的第一个参数是从哪里获取数据的索引（获取位置）。

```java
Buffer buff = Buffer.buffer();
for (int i = 0; i < buff.length(); i += 4) {
  System.out.println("int value at " + i + " is " + buff.getInt(i));
}
```

#### 使用无符号数据

可使用`getUnsignedXXX`，`appendUnsignedXXX`和`setUnsignedXXX`方法将无符号正数从Buffer中读取或追加/设置到Buffer。这对实现优化网络协议和最小化带宽消耗而实现的编解码器是很有用的。

下边例子中，值200倍设置到一个仅占用一个字节的固定位置：

```java
Buffer buff = Buffer.buffer(128);
int pos = 15;
buff.setUnsignedByte(pos, (short) 200);
System.out.println(buff.getUnsignedByte(pos));
```
控制台中显示'200'。

#### Buffer长度

可使用[length](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#length--)获取Buffer长度，Buffer的长度值是Buffer中包含字节（数据）的最大索引 + 1。

#### 拷贝Buffer

可使用[copy](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#copy--)创建一个Buffer的副本。

#### 分片Buffer

一个分片Buffer是基于原始Buffer的一个新的Buffer，如，它不会拷贝底层数据。使用[slice](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html#slice--)创建一个分片Buffer。

#### Buffer重用

将Buffer写入到一个Socket或其他类似位置后，Buffer就不可被重用了。

### 编写TCP服务器和客户端

Vert.x允许您很容易编写非阻塞的TCP客户端和服务器。

#### 创建一个TCP服务器

使用所有默认配置项，最简单地创建一个TCP服务器如下：

```java
NetServer server = vertx.createNetServer();
```

#### 配置一个TCP服务器

若您不想使用默认配置，可以在创建时通过传入一个[NetServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)实例来配置服务器：

```java
NetServerOptions options = new NetServerOptions().setPort(4321);
NetServer server = vertx.createNetServer(options);
```

#### 启动服务监听【Server Listening】

要告诉服务器监听传入的请求，您可以使用其中一个[listen](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#listen--)方式。

在配置项中告诉服务器监听指定的主机和端口：

```java
NetServer server = vertx.createNetServer();
server.listen();
```

或在调用`listen`时指定主机和端口号，这样就忽略了配置项（中的主机和端口）：

```java
NetServer server = vertx.createNetServer();
server.listen(1234, "localhost");
```

默认主机名是`0.0.0.0`，它表示：监听所有可用地址；默认端口号是`0`，这也是一个特殊值，它告诉服务器本地没有使用的端口中随机选择一个并且使用它。

实际的绑定也是异步的，因此服务器也许并没有在调用listen返回时监听，而是在一段时间过后（才监听）。

若您希望在服务器实际监听时收到通知，您可以向listen提供一个处理器。例如：

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

#### 监听随机端口

若设置监听端口为`0`，服务器将随机寻找一个没有使用的端口来监听。

若您想知道服务器实际监听的端口，可以调用[actualPort](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#actualPort--)方法：

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

#### 收到传入连接【Incoming Connection】的通知

若您想要在连接创建完时收到通知，则需要设置一个[connectHandler](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#connectHandler-io.vertx.core.Handler-)：

```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  // Handle the connection in here
  // 在这里处理连接
});
```

当连接成功时，将会传入一个[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)实例给处理器，并调用它。

这是一个与实际连接类似的socket接口，它允许您读取和写入数据、以及执行各种其他操作如关闭socket。

#### 从Socket读取数据

您可以在Socket中调用[handler](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#handler-io.vertx.core.Handler-)设置处理器从Socket中读取数据。

每次当Socket接收到数据时，会传入一个[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)实例给处理器，并调用它。

```java
NetServer server = vertx.createNetServer();
server.connectHandler(socket -> {
  socket.handler(buffer -> {
    System.out.println("I received some bytes: " + buffer.length());
  });
});
```

#### 写数据到Socket

您可使用[write](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#write-io.vertx.core.buffer.Buffer-)方法写入数据到Socket：

```java
Buffer buffer = Buffer.buffer().appendFloat(12.34f).appendInt(123);
socket.write(buffer);

// Write a string in UTF-8 encoding
// 以UTF-8的编码方式写入一个字符串
socket.write("some data");

// Write a string using the specified encoding
// 以指定的编码方式写入一个字符串
socket.write("some data", "UTF-16");
```

写入操作是异步的，直到调用write方法返回过后一段时间它才发生。

#### 关闭处理器【Close Handler】

若您想要在Socket关闭时收到通知，可（在Socket上）设置一个[closeHandler](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#closeHandler-io.vertx.core.Handler-)：

```java
socket.closeHandler(v -> {
  System.out.println("The socket has been closed");
});
```

#### 处理异常

当Socket发生异常时，您可以设置一个[exceptionHandler](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#exceptionHandler-io.vertx.core.Handler-)来接收（捕捉）任何异常信息。

#### Event Bus写处理器

每个Socket会自动在Event Bus中注册一个处理器，当这个处理器中收到任意Buffer（数据）时，它将（数据）写入到自身。

这使您可以将数据写入到处于完全不同Verticle实例的Socket中，甚至通过将Buffer发送到不同Vert.x实例中处理器绑定的地址上。

处理器地址由[writeHandlerID](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#writeHandlerID--)给出。

#### 本地和远程地址

一个[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)的本地地址可通过[localAddress](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#localAddress--)获取。

一个[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)的远程地址（即连接的另一端的地址）可通过[remoteAddress](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#remoteAddress--)获取。

#### 从类路径中发送文件或资源

文件和类路径资源可以直接使用[sendFile](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#sendFile-java.lang.String-)写入Socket，这是发送文件很有效的方式，若操作系统支持它可以被操作系统内核直接处理。

请阅读[从类路径提供文件](http://vertx.io/docs/vertx-core/java/#classpath)章节了解类路径的限制或禁用它。

#### Streaming Socket

[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)的实例也是[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)和[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)实例，因此它们可以用于将数据抽取到其他读取和写入流中。

有关更多信息，请参阅[流和泵](http://vertx.io/docs/vertx-core/java/#streams)的章节。

#### 升级连接到SSL/TLS

一个非SSL/TLS连接可以通过[upgradeToSsl](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html#upgradeToSsl-io.vertx.core.Handler-)方法升级到SSL/TLS连接。

必须为服务器或客户端配置SSL/TLS才能正常工作，有关详情，请参阅[SSL/TLS](http://vertx.io/docs/vertx-core/java/#ssl)章节。

#### 关闭TCP服务器

调用[close](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServer.html#close--)方法关闭服务器，关闭服务器将关闭所有打开的连接并释放所有服务器资源。

关闭实际上是异步的，直到方法调用返回过后一段时间才会实际关闭，若您想在实际关闭完成时收到通知，那么您可以传递一个处理器。

当关闭完全完成后，这个处理器将被调用：

```java
server.close(res -> {
  if (res.succeeded()) {
    System.out.println("Server is now closed");
  } else {
    System.out.println("close failed");
  }
});
```

#### Verticle中自动清理

若您在Verticle内创建了TCP服务器和客户端，这些服务器和客户端将会在Verticle撤销时自动关闭。

#### 扩展 - 共享TCP服务器

任何TCP服务器中的处理器总是在相同的Event Loop线程上执行。

这意味着如果您在多核的服务器上运行，并且只部署了一个实例，那么您的服务器上最多只能使用一个核。

为了利用更多的服务器核，您将需要部署更多的服务器实例。

您可以在代码中以编程方式实例化更多（Server的）实例：

```java
for (int i = 0; i < 10; i++) {
  NetServer server = vertx.createNetServer();
  server.connectHandler(socket -> {
    socket.handler(buffer -> {
      // Just echo back the data
	  // 仅仅回传数据
      socket.write(buffer);
    });
  });
  server.listen(1234, "localhost");
}
```

或者，如果您使用的是Verticle，您可以通过在命令行上使用`-instances`选项来简单部署更多的服务器实例：

```
vertx run com.mycompany.MyVerticle -instances 10
```

或者使用编程方式部署您的Verticle时：

```java
DeploymentOptions options = new DeploymentOptions().setInstances(10);
vertx.deployVerticle("com.mycompany.MyVerticle", options);
```

一旦您这样做，您将发现echo服务器在功能上与之前相同，但是服务器上的所有核都可以被利用，并且可以处理更多的工作。

在这一点上，您可能会问自己：如何让多台服务器在同一主机和端口上侦听？当您尝试部署一个以上的实例时，您是否会遇到端口冲突？

*Vert.x在这里有一点魔法。*

当您在与现有服务器相同的主机和端口上部署另一个服务器实例时，实际上它并不会尝试创建在同一主机/端口上侦听的新服务器实例。

相反，它内部仅仅维护一个服务器实例，并且当传入连接到达时，它以循环方式将其分发给任何连接处理器。

因此，Vert.x TCP服务器可以扩展可用核，而每个实例保持单线程。

#### 创建TCP客户端

使用所有默认选项创建TCP客户端的最简单方法如下：

```java
NetClient client = vertx.createNetClient();
```

#### 配置一个TCP客户端

如果您不想使用默认值，则可以在创建实例时传入[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)给客户端：

```java
NetClientOptions options = new NetClientOptions().setConnectTimeout(10000);
NetClient client = vertx.createNetClient(options);
```

#### 创建连接

您可以使用[connect](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClient.html#connect-int-java.lang.String-io.vertx.core.Handler-)创建到服务器的连接，请指定服务器的端口和主机，以及在连接成功时使用包含[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)的结果调用的处理器，若连接失败，则会发生故障。

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

#### 配置重连

可以将客户端配置为在无法连接的情况下自动重试连接到服务器，这是使用[setReconnectInterval](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#setReconnectInterval-long-)和[setReconnectAttempts](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#setReconnectAttempts-int-)配置的。

*注意：目前如果连接失败，Vert.x将不尝试重新连接，重新连接尝试和间隔（参数）仅适用于创建初始连接。*

```java
NetClientOptions options = new NetClientOptions().
  setReconnectAttempts(10).
  setReconnectInterval(500);

NetClient client = vertx.createNetClient(options);
```

默认情况下，多个连接尝试被禁用了。

#### 记录网络活动

为了调试，网络活动可以被记录：

```java
NetServerOptions options = new NetServerOptions().setLogActivity(true);

NetServer server = vertx.createNetServer(options);
```

对于客户端：

```java
NetClientOptions options = new NetClientOptions().setLogActivity(true);

NetClient client = vertx.createNetClient(options);
```

Netty使用`DEBUG`级别和`io.netty.handler.logging.LoggingHandler`类记录网络活动，使用网络活动记录时，需要注意以下几点：

* 记录不是由Vert.x的日志器【logging】执行而是由Netty执行
* 这个功能不能用于生产环境

您应该阅读[Netty日志器](http://vertx.io/docs/vertx-core/java/#netty-logging)章节

#### 配置服务器和客户端使用SSL/TLS

TCP客户端和服务器通过配置而使用传输层安全性【[Transport Layer Security](http://en.wikipedia.org/wiki/Transport_Layer_Security)】——早期版本的TLS被称为SSL。

无论是否使用SSL/TLS，服务器和客户端的API都是相同的，并且可以传入配置用于创建服务器或客户端的[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)或[NetServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)实例。

##### 在服务器上启用SSL/TLS

SSL/TLS使用[ssl](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html#setSsl-boolean-)来启用。

默认是禁用的。

##### 指定服务器的密钥/证书

SSL/TLS服务器通常向客户端提供证书，以便验证服务器的身份。

可以通过以下几种方式为服务器配置证书/密钥：

第一种方法是指定包含证书和私钥的Java密钥库位置。

可以使用JDK附带的[keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html)实用程序来管理Java密钥受信存储。

还应提供密钥存储的密码：

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setKeyStoreOptions(
  new JksOptions().
    setPath("/path/to/your/server-keystore.jks").
    setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
```

或者，您可以自己读取密钥库到一个Buffer，并将它直接提供：

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

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)），通常与`.pfx`或`.p12`扩展名也可以与JKS密钥存储相似的方式加载：

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setPfxKeyCertOptions(
  new PfxOptions().
    setPath("/path/to/your/server-keystore.pfx").
    setPassword("password-of-your-keystore")
);
NetServer server = vertx.createNetServer(options);
```

它也支持Buffer的配置：

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

另外一种单独提供服务器私钥和证书的方法是使用`.pem`文件。

```java
NetServerOptions options = new NetServerOptions().setSsl(true).setPemKeyCertOptions(
  new PemKeyCertOptions().
    setKeyPath("/path/to/your/server-key.pem").
    setCertPath("/path/to/your/server-cert.pem")
);
NetServer server = vertx.createNetServer(options);
```

它也支持Buffer的配置：

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

##### 指定服务器信任

SSL/TLS服务器可以使用证书颁发机构来验证客户端的身份。

证书颁发机构可通过多种方式为服务器配置：

可使用JDK随附的[keytool](http://docs.oracle.com/javase/6/docs/technotes/tools/solaris/keytool.html)实用程序来管理Java受信存储。

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

或者您可以自己读取受信存储到Buffer，并将它直接提供：

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

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)），通常与`.pfx`或`.p12`扩展名也可以与JKS受信存储相似的方式加载：

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

它也支持Buffer的配置：

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

另一种提供服务器证书颁发机构的方法是使用一个列表.pem文件。

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

它也支持Buffer的配置：

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

##### 客户端启用SSL/TLS

网络客户端也可以轻松地配置为SSL，使用SSL时，它们和标准套接字的使用具有完全相同的API。

若要启用NetClient上的SSL，可调用函数`setSSL(true)`。

##### 受信客户端配置

若客户端使[trustAll](http://vertx.io/docs/apidocs/io/vertx/core/net/ClientOptionsBase.html#setTrustAll-boolean-)设置为true，则客户端将信任所有服务端证书。连接仍然会被加密，但这种模式很容易受到“中间人”的攻击。即您无法确定您正连接到谁，请谨慎使用。默认值为false。

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustAll(true);
NetClient client = vertx.createNetClient(options);
```

若客户端没设置[trustAll](http://vertx.io/docs/apidocs/io/vertx/core/net/ClientOptionsBase.html#setTrustAll-boolean-)，则必须配置客户端受信存储，并且受信客户端应该包含服务器的证书。

默认情况下，客户端禁用主机验证。要启用主机验证，请将算法设置为在客户端上使用（目前仅支持HTTPS和LDAPS）：

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setHostnameVerificationAlgorithm("HTTPS");
NetClient client = vertx.createNetClient(options);
```

和服务器配置相同，也可通过以下几种方式配置受信客户端：

第一种方法是指定包含证书颁发机构的Java受信库的位置。

它只是一个标准的Java密钥存储，与服务器端的密钥存储相同。通过在[jsk options](http://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html)上使用[path](http://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html#setPath-java.lang.String-)设置客户端受信存储位置。如果服务器在连接期间提供不在客户端受信存储中的证书，则尝试连接将不会成功。

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

它也支持Buffer的配置：

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

通常使用`.pfx`或`.p12`扩展名的PKCS＃12格式（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)）的证书颁发机构也可以与JKS受信存储相似的方式加载：

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

它也支持Buffer的配置：

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

另一种提供服务器证书颁发机构的方法是使用一个列表.pem文件。

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setPemTrustOptions(
    new PemTrustOptions().
      addCertPath("/path/to/your/ca-cert.pem")
  );
NetClient client = vertx.createNetClient(options);
```

它也支持Buffer的配置：

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

##### 指定客户端的密钥/证书

如果服务器需要客户端认证，那么当连接时，客户端必须向服务器显示自己的证书。 可通过以下几种方式配置客户端：

第一种方法是指定包含密钥和证书的Java密钥库的位置，它只是一个常规的Java密钥存储。 使用[jks options](http://vertx.io/docs/apidocs/io/vertx/core/net/JksOptions.html)上的功能路径设置客户端密钥库位置。

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setKeyStoreOptions(
  new JksOptions().
    setPath("/path/to/your/client-keystore.jks").
    setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
```

它也支持Buffer的配置：

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

PKCS＃12格式的密钥/证书（[http://en.wikipedia.org/wiki/PKCS_12](http://en.wikipedia.org/wiki/PKCS_12)），通常与`.pfx`或`.p12`扩展名也可以与JKS密钥存储相似的方式加载：

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setPfxKeyCertOptions(
  new PfxOptions().
    setPath("/path/to/your/client-keystore.pfx").
    setPassword("password-of-your-keystore")
);
NetClient client = vertx.createNetClient(options);
```

它也支持Buffer的配置：

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

另一种单独提供服务器私钥和证书的方法是使用.pem文件。

```java
NetClientOptions options = new NetClientOptions().setSsl(true).setPemKeyCertOptions(
  new PemKeyCertOptions().
    setKeyPath("/path/to/your/client-key.pem").
    setCertPath("/path/to/your/client-cert.pem")
);
NetClient client = vertx.createNetClient(options);
```

它也支持Buffer的配置：

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

请记住pem的配置和私钥是不加密的。

##### 用于测试和开发目的的自签名证书

*警告：不要在生产设置中使用，注意，这里生成的密钥非常不安全。*

通常情况下，无论是单位/集成测试还是运行应用程序的开发版本都需要自签名证书。

[SelfSignedCertificate](http://vertx.io/docs/apidocs/io/vertx/core/net/SelfSignedCertificate.html)可用于提供自签名PEM证书，并可以传递[KeyCertOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/KeyCertOptions.html)和[TrustOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/TrustOptions.html)配置（给它）：

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

注意：自签名证书也适用于其他TCP协议，如HTTPS：

```java
SelfSignedCertificate certificate = SelfSignedCertificate.create();

vertx.createHttpServer(new HttpServerOptions()
  .setSsl(true)
  .setKeyCertOptions(certificate.keyCertOptions())
  .setTrustOptions(certificate.trustOptions()))
  .requestHandler(req -> req.response().end("Hello!"))
  .listen(8080);
```

##### 待撤销证书颁发机构

可以将信任配置为对不应再受信任的待撤销证书使用证书吊销列表（CRL），[crlPath](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html#addCrlPath-java.lang.String-)配置CRL列表以使用：

```java
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(trustOptions).
  addCrlPath("/path/to/your/crl.pem");
NetClient client = vertx.createNetClient(options);
```

它也支持Buffer的配置：

```java
Buffer myCrlAsABuffer = vertx.fileSystem().readFileBlocking("/path/to/your/crl.pem");
NetClientOptions options = new NetClientOptions().
  setSsl(true).
  setTrustStoreOptions(trustOptions).
  addCrlValue(myCrlAsABuffer);
NetClient client = vertx.createNetClient(options);
```

##### 配置密码套件【Cipher Suite】

默认情况下，TLS配置将使用运行Vert.x的JVM密码套件，该密码套件可以配置一套启用的密码：

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

密码套件可在[NetServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)或[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)配置项中指定。

##### 配置TLS协议版本

默认情况下，TLS配置将使用以下协议版本：SSLv2Hello、TLSv1、TLSv1.1和TLSv1.2。 协议版本可以通过显式添加启用协议进行配置：

```java
NetServerOptions options = new NetServerOptions().
  setSsl(true).
  setKeyStoreOptions(keyStoreOptions).
  addEnabledSecureTransportProtocol("TLSv1.1").
  addEnabledSecureTransportProtocol("TLSv1.2");
NetServer server = vertx.createNetServer(options);
```

协议版本可在[NetServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetServerOptions.html)或[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)配置项中指定。

##### SSL引擎

引擎实现可以配置为使用OpenSSL而不是JDK实现（来支持SSL）。 OpenSSL提供比JDK引擎更好的性能和CPU使用率、以及JDK版本独立性。

引擎选项的使用是：

* [getSslEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#getSslEngineOptions--)选项设置时
* 否则[JdkSSLEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/JdkSSLEngineOptions.html)

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

##### 应用层协议协商【ALPN】

ALPN【Application-Layer Protocol Negotiation】是应用层协议协商的TLS扩展，它被HTTP/2使用：在TLS握手期时，客户端给出其接受的应用协议列表，并且服务器使用（自身）支持的协议响应。

标准的Java 8不支持ALPN，所以ALPN应该通过其他方式启用：

* OpenSSL支持
* Jetty-ALPN支持

引擎选项可使用:

* [getSslEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#getSslEngineOptions--)选项设置时
* JDK中ALPN可用时使用[JdkSSLEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/JdkSSLEngineOptions.html)
* OpenSSL中ALPN可用时使用[OpenSSLEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/OpenSSLEngineOptions.html)
* 否则失败

**OpenSSL ALPN支持**

OpenSSL提供了原生【Native】的ALPN支持。

OpenSSL需要配置[setOpenSslEngineOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/TCPSSLOptions.html#setOpenSslEngineOptions-io.vertx.core.net.OpenSSLEngineOptions-)并在类路径上使用[netty-tcnative](http://netty.io/wiki/forked-tomcat-native.html)的jar库。依赖于tcnative的实现它需要OpenSSL安装在您的操作系统中。

**Jetty-ALPN支持**

Jetty-ALPN是一个小型的jar，它覆盖了几种Java 8发行版用以支持ALPN。

JVM必须将`alpn-boot-${version}.jar`放在它的bootclasspath中启动：

```
-Xbootclasspath/p:/path/to/alpn-boot${version}.jar
```

其中${version}取决于JVM的版本，如*OpenJDK 1.8.0u74*中的*8.1.7.v20160121*，这个完整列表可以在[Jetty-ALPN](http://www.eclipse.org/jetty/documentation/current/alpn-chapter.html)页面上找到。

主要缺点就是版本取决于JVM。

为了解决这个问题，可以使用[Jetty ALPN agent](https://github.com/jetty-project/jetty-alpn-agent)。agent是一个JVM代理，它会为运行它的JVM选择正确的ALPN版本：

```
-javaagent:/path/to/alpn/agent
```

#### 客户端连接使用代理

[NetClient](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClient.html)支持HTTP/1.x *CONNECT*、*SOCKS4a*或*SOCKS5*代理。

代理可以在[NetClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/NetClientOptions.html)内设置[ProxyOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/ProxyOptions.html)来配置代理类型、主机名、端口、可选的用户名和密码。

以下是一个例子：

```java
NetClientOptions options = new NetClientOptions()
  .setProxyOptions(new ProxyOptions().setType(ProxyType.SOCKS5)
    .setHost("localhost").setPort(1080)
    .setUsername("username").setPassword("secret"));
NetClient client = vertx.createNetClient(options);
```

DNS解析会一直在代理服务器上执行，为了实现SOCKS4客户端的功能，需要先在本地解析DNS地址。

### 编写HTTP服务器和客户端

Vert.x允许您轻松编写非阻塞HTTP客户端和服务器。

Vert.x支持HTTP/1.0、HTTP/1.1和HTTP/2协议。

用于HTTP的基本API对HTTP/1.x和HTTP/2是相同的，特定的API功能也可用于处理HTTP/2协议。

#### 创建一个HTTP服务器

使用所有默认选项创建HTTP服务器的最简单方法如下：

```java
HttpServer server = vertx.createHttpServer();
```

#### 配置一个HTTP服务器

若您不想用默认值，可以在创建服务器时传递一个[HttpServerOptions](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html)实例给它：

```java
HttpServerOptions options = new HttpServerOptions().setMaxWebsocketFrameSize(1000000);

HttpServer server = vertx.createHttpServer(options);
```

#### 配置一个HTTP/2的服务器

Vert.x支持TLS `h2`和TCP `h2c`之上的HTTP/2协议。

* `h2`用于通过应用层协议协商（ALPN）协商的TLS时识别HTTP/2协议
* `h2c`在TCP上以明文形式使用时识别HTTP/2协议，这样的连接是使用HTTP/1.1升级请求或直接建立的

要处理h2请求，TLS必须调用[setUseAlpn](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setUseAlpn-boolean-)启用：

```java
HttpServerOptions options = new HttpServerOptions()
    .setUseAlpn(true)
    .setSsl(true)
    .setKeyStoreOptions(new JksOptions().setPath("/path/to/my/keystore"));

HttpServer server = vertx.createHttpServer(options);
```

ALPN是一个TLS的扩展，它在客户端和服务器开始交换数据之前协商协议。

不支持ALPN的客户端仍然可以执行经典的SSL握手。

通常情况，ALPN会对`h2`协议达成一致，尽管服务器或客户端决定了仍然使用HTTP/1.1协议。

要处理`h2c`请求，TLS必须被禁用，服务器将升级到HTTP/2以满足任何希望升级到HTTP/2的HTTP/1.1请求。它还将接受以`PRI*HTTP/2.0\r\nSM\r\n`开始的`h2c`直接连接。

*警告：大多数浏览器不支持h2c，所以在服务网站时，您应该使用h2而不是h2c。*

当服务器接受HTTP/2连接时，它会向客户端发送其初始设置。定义客户端如何使用连接，服务器的默认初始设置为：

* [getMaxConcurrentStreams](http://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html#getMaxConcurrentStreams--)：按照HTTP/2 RFC建议推荐值为100
* 其他默认的HTTP/2的设置

*注意：Worker Verticles和HTTP/2不兼容*

#### 记录网络服务器活动

为了进行调试，可记录网络活动。

```java
HttpServerOptions options = new HttpServerOptions().setLogActivity(true);

HttpServer server = vertx.createHttpServer(options);
```

有关详细说明，请参阅有[记录网络活动](http://vertx.io/docs/vertx-core/java/#logging_network_activity)章节。

#### 开始服务器监听

要告诉服务器监听传入的请求，您可以使用其中一个[listen](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#listen--)方式。

在配置项中告诉服务器监听指定的主机和端口：

```java
HttpServer server = vertx.createHttpServer();
server.listen();
```

或在调用`listen`时指定主机和端口号，这样就忽略了配置项（中的主机和端口）：

```java
HttpServer server = vertx.createHttpServer();
server.listen(8080, "myhost.com");
```

默认主机名是`0.0.0.0`，它表示：监听所有可用地址；默认端口号是`80`。

实际的绑定也是异步的，因此服务器也许并没有在调用listen返回时监听，而是在一段时间过后（才监听）。

若您希望在服务器实际监听时收到通知，您可以向listen提供一个处理器。例如：

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

#### 收到传入请求【Incoming requests】的通知

若您需要在收到请求时收到通知，则需要设置一个[requestHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#requestHandler-io.vertx.core.Handler-)

```java
HttpServer server = vertx.createHttpServer();
server.requestHandler(request -> {
  // Handle the request in here
  // 在这里处理请求
});
```

#### 处理请求

当请求到达时，它传入一个称为[HttpServerRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html)的实例并调用请求处理器，此对象表示服务端HTTP请求。

当请求的头信息被完全读取时调用该处理器。

如果请求包含请求体，那么该请求体将在请求处理器被调用后的某个时间到达服务器。

服务请求对象允许您检索[uri](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uri--)，[path](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#path--)，[params](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--)和[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--)等其他事。

每一个服务请求对象和一个服务响应对象绑定，您可以用[response](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#response--)获取一个[HttpServerResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)对象的引用。

这是服务器处理请求并回复“hello world”的简单示例。

```java
vertx.createHttpServer().requestHandler(request -> {
  request.response().end("Hello world");
}).listen(8080);
```

**请求版本【version】**

在请求中指定的HTTP版本可通过[version](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#version--)获取。

**请求方法【method】**

使用[method](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#method--)读取请求中的HTTP方法（即GET、POST、PUT、DELETE、HEAD、OPTIONS等）。

**请求URI【uri】**

使用[uri](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uri--)读取请求中的URI路径。

请注意，这是在HTTP请求中传递的实际URI，它总是一个相对的URI。

这个URI是在[Section 5.1.2 of the HTTP specification - Request-URI](http://www.w3.org/Protocols/rfc2616/rfc2616-sec5.html)中定义的。

**请求路径【path】**

使用[path](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#path--)读取URI中的路径部分。

例如，请求的URI为：

```
a/b/c/page.html?param1=abc&param2=xyz
```

路径部分应该是：

```
/a/b/c/page.html
```

**请求查询【query】**

使用[query](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#query--)读取URI中的查询部分。

例如，请求的URI为：

```
a/b/c/page.html?param1=abc&param2=xyz
```

查询部分应该是：

```
param1=abc&param2=xyz
```

**请求头【headers】**

使用[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--)获取HTTP请求中的请求头信息。

这个方法返回一个[MultiMap](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)的实例——它像一个普通的Map或Hash、单它还允许同一个键支持多个值——因为HTTP允许同一个键支持多个请求头的值。

它还具有不区分大小写的键，这意味着您可以执行以下操作：

```java
MultiMap headers = request.headers();

// Get the User-Agent:
// 读取User-Agent
System.out.println("User agent is " + headers.get("user-agent"));

// You can also do this and get the same result:
// 这样做可以得到和上边相同的结果
System.out.println("User agent is " + headers.get("User-Agent"));
```

**请求主机【host】**

使用[host](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#host--)返回HTTP请求中的主机名。

对于HTTP/1.x请求返回请求头中的`host`值，对于HTTP/1请求则返回伪头中的`:authority`的值。

**请求参数【Parameters】**

使用[params](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#params--)返回HTTP请求中的参数信息。

像[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#headers--)一样它也会返回一个[MultiMap](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)，因为可以有多个具有相同名称的参数。

请求参数在请求URI的path部分之后，例如URI是：

```
/page.html?param1=abc&param2=xyz
```

那么参数将包含以下内容：

```
param1: 'abc'
param2: 'xyz'
```

请注意，这些请求参数是从请求的URI中解析读取的，若您已经将表单属性作为在`multi-part/form-data`请求正文中提交的HTML表单的一部分发送，那么它们将不会显示在此处的参数中。

**远程地址**

可以使用[remoteAddress](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#remoteAddress--)读取请求发送者的地址。

**绝对URI**

HTTP请求中传递的URI通常是相对的，若您想要读取请求中和相对URI对应的绝对URI，可调用[absoluteURI](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#absoluteURI--)

**结束处理器【End Handler】**

当整个请求（包括任何正文）已经被完全读取时，请求中的[endHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#endHandler-io.vertx.core.Handler-)会被调用。

**请求体中读取数据**

HTTP请求通常包含我们需要读取的主体。如前所述，当请求头部达到时，请求处理器会被调用，因此请求对象在此时没有请求体。

这是因为请求体可能非常大（如文件上传），并且我们不会在内容发送给您之前将其全部缓冲存储在内存中，这可能会导致服务器耗尽可用内存。

要接收请求体，您可在请求中调用[handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#handler-io.vertx.core.Handler-)设置一个处理器，每次请求体的一小块（数据）收到时，该处理器都会被调用。以下是一个例子：

```java
request.handler(buffer -> {
  System.out.println("I have received a chunk of the body of length " + buffer.length());
});
```

传递给处理器的对象是一个[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)，当数据从网络到达时，处理器可以多次被调用，这取决于请求体的大小。

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

这是一个常见的情况，Vert.x为您提供了一个[bodyHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#bodyHandler-io.vertx.core.Handler-)来执行此操作，当所有请求体被收到时，bodyHandler处理器程序会被调用一次：

```java
request.bodyHandler(totalBuffer -> {
  System.out.println("Full body received, length = " + totalBuffer.length());
});
```

**Pumping请求**

请求对象是一个[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)，因此您可以将请求体读取到任何[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)实例中。

有关详细的说明，请参阅[流和泵](http://vertx.io/docs/vertx-core/java/#streams)的章节。

**处理HTML表单**

您可使用内容类型为`application/x-www-form-urlencoded`或`multipart/form-data`提交HTML表单。

对于使用URL编码过的表单，表单属性会被URL编码，如同普通查询参数一样。

对于multi-part表单它会在请求体中被编码，而且在整个请求体被完全读取之前它是不可用的。

multi-part表单还可以包含文件上传。

若您想要读取multi-part表单的属性，您应该告诉Vert.x您会在读取任何正文之前调用[setExpectMultipart(true)](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#setExpectMultipart-boolean-)，然后在整个请求体都被阅读后，您可以使用[formAttributes](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#formAttributes--)来读取实际的表单属性。

```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.endHandler(v -> {
    // The body has now been fully read, so retrieve the form attributes
    // 请求体被完全读取，所以直接读取表单属性
    MultiMap formAttributes = request.formAttributes();
  });
});
```

**处理表单文件上传**

Vert.x可以在处理编码过的multi-part请求体中处理文件上传。

要接收文件，您可以告诉Vert.x系统使用multi-part表单，并根据请求设置[uploadHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#uploadHandler-io.vertx.core.Handler-)。

当服务器每次接收到上传请求时，该处理器将被调用一次。

传递给处理器的对象是一个[HttpServerFileUpload](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html)实例。

```java
server.requestHandler(request -> {
  request.setExpectMultipart(true);
  request.uploadHandler(upload -> {
    System.out.println("Got a file upload " + upload.name());
  });
});
```

文件上传可能很大，我们不会在单个缓冲区中提供（包含）整个上传（数据），因为这样会导致内存耗尽，相反上传数据是以块【Chunk】的形式被接收的：

```java
request.uploadHandler(upload -> {
  upload.handler(chunk -> {
    System.out.println("Received a chunk of the upload of length " + chunk.length());
  });
});
```

上传对象是一个[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)，因此您可以将请求体读取到任何[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)实例中。有关详细的说明，请参阅[流和泵](http://vertx.io/docs/vertx-core/java/#streams)的章节。

若您只是想将文件上传到服务器的某个磁盘，可以使用[streamToFileSystem](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html#streamToFileSystem-java.lang.String-)：

```java
request.uploadHandler(upload -> {
  upload.streamToFileSystem("myuploads_directory/" + upload.filename());
});
```

*警告：确保您检查了生产系统的文件名，以避免恶意客户将文件上传到文件系统中的任意位置。有关详细信息，参阅[安全说明](http://vertx.io/docs/vertx-core/java/#_security_notes)。*

**处理压缩体**

Vert.x可以处理在客户端通过deflate或gzip算法压缩过的请求体信息。

若要启用解压缩功能则您要在创建服务器时调用[setDecompressionSupported](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setDecompressionSupported-boolean-)设置配置项。

默认请求下解压缩是被禁用的。

**接收自定义HTTP/2帧**

HTTP/2是用于HTTP请求/响应模型的包含各种帧的一种帧协议，该协议允许发送和接收其他类型的帧。

若要接收自定义帧，您可以在请求中使用[customFrameHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#customFrameHandler-io.vertx.core.Handler-)，每次当自定义的帧数据到达时，这个处理器会被调用。这而是一个例子：

```java
request.customFrameHandler(frame -> {

  System.out.println("Received a frame type=" + frame.type() +
      " payload" + frame.payload().toString());
});
```

HTTP/2帧不受流程控制——当接收到自定义帧时，不论请求（内容）是暂停或不存在，帧处理器都将立即被调用。

**非标准的HTTP方法**

[OTHER](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpMethod.html#OTHER)的HTTP方法可用于非标准方法，在这种情况下，[rawMethod](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#rawMethod--)返回客户端发送的（实际）HTTP方法。

#### 发回响应

服务器响应对象是一个[HttpServerResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)的一个实例，它可以从request对应的[response](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#response--)中读取。

您可以使用响应对象回写一个响应到HTTP客户端。

**设置状态代码和消息**

响应的默认HTTP状态代码为200，表示OK。

可使用[setStatusCode](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#setStatusCode-int-)设置不同状态代码。

您还可用[setStatusMessage](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#setStatusMessage-java.lang.String-)指定自定义状态消息。

若您不指定状态消息，默认将使用和状态代码对应的消息。

*注意：对于HTTP/2中的状态不会在响应中描述——因为协议不会将消息发送回客户端。*

**编写HTTP响应**

想要将数据写入HTTP响应，您可使用任意一个[write](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#write-io.vertx.core.buffer.Buffer-)方法。

它们可以在响应结束之前（ended)被多次调用，它们也可以通过以下几种方式调用：

对用单个缓冲区：

```java
HttpServerResponse response = request.response();
response.write(buffer);
```

写入字符串，这种请求字符串将使用UTF-8进行编码，并将结果写入到报文【wire】中。

```java
HttpServerResponse response = request.response();
response.write("hello world!");
```

写入带编码方式的字符串，这种情况字符串将使用指定的编码方式编码，并将结果写入到报文【wire】中。

```java
HttpServerResponse response = request.response();
response.write("hello world!", "UTF-16");
```

响应写入是异步的，并且在写操作队列（完成）之后会立即返回。

若您只需要将单个字符串或Buffer写入到HTTP响应，则可将它直接写入并使用[end](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#end-java.lang.String-)方法结束这次响应。

第一次将结果写入响应您需要设置响应头，因此，若您不使用HTTP分块，那么必须在写入响应之前设置Content-Length头，否则会太迟。若您使用HTTP分块则不需要担心这点。

**结束HTTP响应**

一旦您完成了HTTP响应，可调用[end](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#end-java.lang.String-)结束它。

这可以通过几种方式完成：

没有参数，直接结束响应：

```java
HttpServerResponse response = request.response();
response.write("hello world!");
response.end();
```

您也可以和调用write方法一样传string或buffer给end方法。这种情况，它和先调用带string/buffer参数的write方法，之后调用无参end方法一样，例如：

```java
HttpServerResponse response = request.response();
response.end("hello world!");
```

**关闭底层连接**

您可以调用[close](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#close--)方法关闭底层TCP连接。

当响应结束时，Vert.x将自动关闭非keep-alive的连接。

默认情况下，Vert.x不会自动关闭keep-alive的连接，若您想要在一段空闲时间之后让Vert.x自动关闭keep-alive的连接，则使用[setIdleTimeout](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setIdleTimeout-int-)配置。

HTTP/2连接在关闭响应之前会发送GOAWAY帧。

**设置响应头**

HTTP响应头可直接添加到响应中，通常直接操作[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#headers--)：

```java
HttpServerResponse response = request.response();
MultiMap headers = response.headers();
headers.set("content-type", "text/html");
headers.set("other-header", "wibble");
```

或您可使用[putHeader](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#putHeader-java.lang.String-java.lang.String-)

```java
HttpServerResponse response = request.response();
response.putHeader("content-type", "text/html").putHeader("other-header", "wibble");
```

响应头必须在写入响应正文消息之前进行设置。

**分块HTTP响应和尾**

Vert.x支持分块传输编码[HTTP Chunked Transfer Encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding)。

这允许HTTP响应体以块的形式写入，通常在响应体预先不知道尺寸、需要将很大响应正文以流式传输到客户端时使用。

您使用分块模式的HTTP响应，如下所示：

```java
HttpServerResponse response = request.response();
response.setChunked(true);
```

默认是不分块的，当处于分块模式，每次调用任意一个[write](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#write-io.vertx.core.buffer.Buffer-)方法将导致新的HTTP块被写出。

在分块模式下，您还可以将响应的HTTP响应尾写入响应，这种方式实际上是在写入响应的最后一块。

*注意：分块响应在HTTP/2流中无效。*

若要添加尾到响应，则可直接调用[trailers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#trailers--)。

```java
HttpServerResponse response = request.response();
response.setChunked(true);
MultiMap trailers = response.trailers();
trailers.set("X-wibble", "woobble").set("X-quux", "flooble");
```

或者调用[putTrailer](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#putTrailer-java.lang.String-java.lang.String-)：

```java
HttpServerResponse response = request.response();
response.setChunked(true);
response.putTrailer("X-wibble", "woobble").putTrailer("X-quux", "flooble");
```

**直接从磁盘或类路径读文件**

若您正在写一个Web服务器，一种从磁盘中（读取并）提供文件的方法是将文件作为[AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)对象打开并其泵送到HTTP响应中。

或您可以使用[readFile](http://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html#readFile-java.lang.String-io.vertx.core.Handler-)一次性加载它，并直接将其写入响应。

或者，Vert.x提供了一种方法，允许您在一个操作中将文件从磁盘或文件系统中（读取并）提供给HTTP响应。若底层操作系统支持，这会导致操作系统不通过用户空间复制而直接将（文件内容中）字节数据从文件传输到Socket。

这是使用[sendFile](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#sendFile-java.lang.String-)完成的，对于大文件处理通常更有效，而这个方法对于小文件可能很慢。

这儿是一个非常简单的Web服务器，它使用sendFile从文件系统中（读取并）提供文件：

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

发送文件是异步的，可能在调用返回一段时间后才能完成。如果要在文件写入时收到通知，可以在sendFile（官缺：中设置一个处理器）。

请阅读[从类路径提供文件](http://vertx.io/docs/vertx-core/java/#classpath)章节了解类路径的限制或禁用它。

*注意：若在HTTPS协议中使用sendFile，它将会通过用户空间进行复制，因为若内核将数据直接从磁盘复制到Socket，则不会给我们任何加密的机会。*

*警告：若您要直接使用Vert.x编写Web服务器，请注意，您想提供文件和类路径之外访问的位置——用户是无法直接利用路径访问的，更安全的做法是使用Vert.x Web替代。*

当需要提供文件的一部分，从给定的字节开始，您可以像下边这样做：

```java
vertx.createHttpServer().requestHandler(request -> {
  long offset = 0;
  try {
    offset = Long.parseLong(request.getParam("start"));
  } catch (NumberFormatException e) {
    // error handling...
    // 异常信息
  }

  long end = Long.MAX_VALUE;
  try {
    end = Long.parseLong(request.getParam("end"));
  } catch (NumberFormatException e) {
    // error handling...
    // 异常信息
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
    // error handling...
  }

  request.response().sendFile("web/mybigfile.txt", offset);
}).listen(8080);
```

**Pumping响应**

服务器响应也是一个[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)实例，因此您可以从任何[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)泵送数据，如[AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)、[NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html)、[WebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html)或[HttpServerRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html)。

这儿有一个例子，它回应了任何PUT方法的响应中的请求体，它为请求体使用了Pump，所以即使HTTP请求体很大并填满了内存，任何一个时候它依旧会工作：

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

**写HTTP/2帧**

HTTP/2是用于HTTP请求/响应模型的包含各种帧的一种帧协议，该协议允许发送和接收其他类型的帧。

要发送这样的帧，您可以在响应中使用[writeCustomFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#writeCustomFrame-int-int-io.vertx.core.buffer.Buffer-)，以下是一个例子：

```java
int frameType = 40;
int frameStatus = 10;
Buffer payload = Buffer.buffer("some data");

// Sending a frame to the client
// 向客户端发送一帧
response.writeCustomFrame(frameType, frameStatus, payload);
```

这些帧被立即发送，并且不受流程控制的影响——当这样的帧被发送到那里时，可以在其他的DATA帧之前完成。

**流重置**

HTTP/1.x不允许请求或响应流执行清除重置，如当客户端上传的资源已经存在于服务器上，服务器依然需要接受整个响应。

HTTP/2在请求/响应期间随时支持流重置：

```java
request.response().reset();
```

默认的`NO_ERROR(0)`错误代码会发送，您也可以发送另外一个错误代码：

```java
request.response().reset(8);
```

HTTP/2规范中定义了可用的[错误代码](http://httpwg.org/specs/rfc7540.html#ErrorCodes)列表：

若使用了[request handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#exceptionHandler-io.vertx.core.Handler-)和[response handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#exceptionHandler-io.vertx.core.Handler-)两个处理器过后，在流重置完成时您将会收到通知。

```java
request.response().exceptionHandler(err -> {
  if (err instanceof StreamResetException) {
    StreamResetException reset = (StreamResetException) err;
    System.out.println("Stream reset " + reset.getCode());
  }
});
```

**服务器推送**

服务器推送是HTTP/2支持的一个新功能，可以为单个客户端请求并行发送多个响应。

当服务器处理请求时，它可以向客户端推送请求/响应：

```java
HttpServerResponse response = request.response();

// Push main.js to the client
// 推送main.js到客户端
response.push(HttpMethod.GET, "/main.js", ar -> {

  if (ar.succeeded()) {

    // The server is ready to push the response
    // 服务器准备推送响应
    HttpServerResponse pushedResponse = ar.result();

    // Send main.js response
    // 发送（推送）main.js响应
    pushedResponse.
        putHeader("content-type", "application/json").
        end("alert(\"Push response hello\")");
  } else {
    System.out.println("Could not push client resource " + ar.cause());
  }
});

// Send the requested resource
// 发送请求的资源内容
response.sendFile("<html><head><script src=\"/main.js\"></script></head><body></body></html>");
```

当服务器准备推送响应时，推送响应处理器【push response handler】会被调用，并会发送响应。

推送响应处理器客户能会收到故障，如：客户端可能取消推送，因为它已经在缓存中包含了main.js，并不在需要它。

必须在启动响应结束之前（调end）调用[push](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#push-io.vertx.core.http.HttpMethod-java.lang.String-java.lang.String-io.vertx.core.Handler-)方法，但是在推送响应过后依然可以写响应。


#### HTTP压缩

标准环境的Vert.x是支持HTTP压缩的。

这意味着在响应发送回客户端之前，您可以将响应体自动压缩。

若客户端不支持HTTP压缩，则它可以发回没有压缩过的请求。

这允许它同时处理支持HTTP压缩的客户端和不支持的客户端。

要启用压缩，可以使用[setCompressionSupported](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionSupported-boolean-)进行配置。

默认情况下，未启用压缩。

当启用HTTP压缩时，服务器将检查客户端请求头中是否包含了Accept-Encoding并支持常用的deflate和gzip压缩算法、Vert.x两者都支持。

若找到这样的请求头，服务器将使用所支持的压缩算法之一自动压缩响应正文并发送回客户端。

注意：压缩可以减少网络流量，单是CPU密集度会更高。

为了解决后边一个问题，Vert.x也允许您调整原始的gzip/deflate压缩算法的“压缩级别【Compression Level】”参数

压缩级别允许根据所得数据的压缩比和压缩/解压的计算成本来配置gzip/deflate算法。

压缩级别是从“1”到“9”的整数值，其中“1”表示更低的压缩比但是最快的算法，“9”表示可用的最大压缩比但比较慢的算法。

使用高于1-2的压缩级别通常允许仅仅保存一些字节大小——它的增益不是线性的，并取决于要压缩的特定数据——但它可以满足服务器所要求的CPU周期的不可控的成本（注意现在Vert.x不支持任何缓存形式的响应数据，如静态文件，因此压缩是在每个请求体生成时进行的）,它可生成压缩过的响应数据、并对接收的响应解码（膨胀）——和客户端使用的方式一致，（这种）操作随着压缩级别的增长会变得更加倾向于CPU密集型。

默认情况下——如果通过[setCompressionSupported](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionSupported-boolean-)启用压缩——Vert.x使用“6”作为压缩级别，但是该参数可使用[setCompressionLevel](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setCompressionLevel-int-)来更改。

#### 创建一个HTTP客户端

您可通过下边方式创建一个具有默认配置的[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)实例：

```java
HttpClient client = vertx.createHttpClient();
```

若您想要配置客户端选项，可按下边方式创建：

```java
HttpClientOptions options = new HttpClientOptions().setKeepAlive(false);
HttpClient client = vertx.createHttpClient(options);
```

Vert.x支持基于TLS h2和TCP h2c的HTTP/2协议。

默认情况下，HTTP客户端会执行HTTP/1.1请求，若要执行HTTP/2请求，则调用[setProtocolVersion](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setProtocolVersion-io.vertx.core.http.HttpVersion-)必须设置成[HTTP_2](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpVersion.html#HTTP_2)。

对于h2请求，必须使用应用层协议协商【Application-Layer Protocol Negotiation】启用TLS：

```java
HttpClientOptions options = new HttpClientOptions().
    setProtocolVersion(HttpVersion.HTTP_2).
    setSsl(true).
    setUseAlpn(true).
    setTrustAll(true);

HttpClient client = vertx.createHttpClient(options);
```

对于h2c请求，TLS必须禁用，客户端将执行HTTP/1.1请求并尝试升级到HTTP/2：

```java
HttpClientOptions options = new HttpClientOptions().setProtocolVersion(HttpVersion.HTTP_2);

HttpClient client = vertx.createHttpClient(options);
```

h2c连接也可以直接建立，如连接可以使用前文提到的方式创建，当[setHttp2ClearTextUpgrade](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2ClearTextUpgrade-boolean-)选项设置为false时：建立连接后，客户端将发送HTTP/2连接前缀，并期望从服务器接收相同的连接偏好。

HTTP服务器可能不支持HTTP/2，当响应到达时，可以使用[version](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#version--)检查响应实际HTTP版本。

当客户端连接到HTTP/2服务器时，它将向服务器发送其[初始设置](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#getInitialSettings--)。设置了定义服务器如何使用连接、客户端的默认初始设置是由HTTP/2 RFC定义的。

#### 记录网络客户端活动

为了进行调试，可以记录网络活动：

```java
HttpClientOptions options = new HttpClientOptions().setLogActivity(true);
HttpClient client = vertx.createHttpClient(options);
```

有关详细说明，请参阅有[记录网络活动](http://vertx.io/docs/vertx-core/java/#logging_network_activity)章节。

#### 发出请求

HTTP客户端是很灵活的，您可以通过各种方式发出请求。

通常您希望使用HTTP客户端向同一个主机/端口发送很多请求。为避免每次发送请求时重复设主机/端口，您可以为客户端配置默认主机/端口：

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

// Specify both port and host name
// 指定端口和主机名
client.getNow(8080, "myserver.mycompany.com", "/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// This time use the default port 80 but specify the host name
// 这次使用默认端口80和指定的主机名
client.getNow("foo.othercompany.com", "/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

用客户端发出请求的所有不同方式都支持这两种指定主机/端口的方法。

**无请求体简单请求**

通常，您想发出没有请求体的HTTP请求，这种情况通常如HTTP GET、OPTIONS和HEAD请求。

使用Vert.x客户端执行这种请求最简单的方式是使用加了前缀的`Now`方法，如[getNow](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#getNow-io.vertx.core.http.RequestOptions-io.vertx.core.Handler-)。

这些方法会创建HTTP请求，并在单个方法调用中发送它，而且允许您提供一个处理器，当HTTP响应发送回来时调用该处理器。

```java
HttpClient client = vertx.createHttpClient();

// Send a GET request
// 发送GET请求
client.getNow("/some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// Send a GET request
// 发送HEAD请求
client.headNow("/other-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

**写通用请求**

在有些时候您在运行时【run-time】之前不知道发送请求的HTTP方法，对于该用例，我们提供通用请求方法[request](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#request-io.vertx.core.http.HttpMethod-io.vertx.core.http.RequestOptions-)，允许您在运行时指定HTTP方法：

```java
HttpClient client = vertx.createHttpClient();

client.request(HttpMethod.GET, "some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end();

client.request(HttpMethod.POST, "foo-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).end("some-data");
```

**写请求体**

有时您想要写入一个包含了请求体的请求，或者也许您想要在发送请求之前写入（数据到）请求头。

为此，您可以调用其中一个指定的请求方法如[post](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#post-io.vertx.core.http.RequestOptions-)或一个其他通用请求方法如[request](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#request-io.vertx.core.http.HttpMethod-io.vertx.core.http.RequestOptions-)。

这些方法都不会立即发送请求，而是返回一个[HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html)实例，它可以用来写数据到请求体和请求头。

这儿有一些写包含m的请求体的POST请求例子：

```java
HttpClient client = vertx.createHttpClient();

HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});

// Now do stuff with the request
// 现在准备请求的一些东西
request.putHeader("content-length", "1000");
request.putHeader("content-type", "text/plain");
request.write(body);

// Make sure the request is ended when you're done with it
// 确保请求完成后结束
request.end();

// Or fluently:
// 或使用Fluent的API
client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-length", "1000").putHeader("content-type", "text/plain").write(body).end();

// Or event more simply:
// 或事情更简单
client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).putHeader("content-type", "text/plain").end(body);
```

可使用存在的写方法写入UTF-8编码的字符串、指定编码的字符串、或写Buffer：

```java
// 写UTF-8编码的字符串
request.write("some data");

// Write string encoded in specific encoding
// 写指定编码的字符串
request.write("some other data", "UTF-16");

// Write a buffer
// 写Buffer
Buffer buffer = Buffer.buffer();
buffer.appendInt(123).appendLong(245l);
request.write(buffer);
```

若您仅需要写单个字符串或Buffer到HTTP请求中，您可以在写入后调用end函数结束单次调用的请求。

```java
request.end("some simple data");

// Write buffer and end the request (send it) in a single call
// 在单次调用中写Buffer并结束请求（直接发送）
Buffer buffer = Buffer.buffer().appendDouble(12.34d).appendLong(432l);
request.end(buffer);
```

当您写入请求时，第一次调用write方法将先将请求头写入到（请求）报文中。

实际写入操作是异步的，它在调用返回一段时间后才发生。

带请求体的非分块HTTP请求需要提供Content-Length头。

因此，若您不使用HTTP分块，则必须在写入请求之前设置Content-Length头，否则会太迟。

若您在调用其中一个end方法处理string或buffer，在写入请求体之前，Vert.x将自动计算并设置Content-Length。

若您在使用HTTP分块模式，则不需要Content-Length头，因此您不必先计算大小。


#### 写请求头

您可以直接使用multi-map结构的[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#headers--)来设置请求头：

```java
MultiMap headers = request.headers();
headers.set("content-type", "application/json").set("other-header", "foo");
```

这个headers是一个[MultiMap](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)的实例，它提供了添加、设置、删除条目的操作，HTTP头允许一个特定的键包含多个值。

您也可以使用[putHeader](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#putHeader-java.lang.String-java.lang.String-)编写头文件：

```java
request.putHeader("content-type", "application/json").putHeader("other-header", "foo");
```

若您想写入（数据）请求头，则您必须在写入任何请求体之前这样做来设置请求头。

**非标准的HTTP方法**

[OTHER](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpMethod.html#OTHER)的HTTP方法用于非标准HTTP方法，当使用此方法时，必须使用[setRawMethod](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setRawMethod-java.lang.String-)来设置需要发送到服务器的raw方法。

**结束HTTP请求**

一旦完成了HTTP请求，您必须调用其中一个[end](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#end-java.lang.String-)操作来结束该请求。

结束一个请求时，若请求头尚未被写入，会导致它们被写入，并且请求被标记成完成的。

请求可以通过多种方式结束。无参简单结束请求的方式如：

```java
request.end();
```

或可以在调用end时提供string或buffer，这个和先调用带string/buffer参数的write方法之后再调用无参end一样。

```java
request.end("some-data");

// End it with a buffer
// 使用buffer结束
Buffer buffer = Buffer.buffer().appendFloat(12.3f).appendInt(321);
request.end(buffer);
```

**分块HTTP请求**

Vert.x支持[HTTP Chunked Transfer Encoding](http://en.wikipedia.org/wiki/Chunked_transfer_encoding)的请求。

这允许使用块方式写入HTTP请求体，这个在请求体比较大需要流式发送到服务器，或预先不知道大小时很常用。

您可使用[setChuncked](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setChunked-boolean-)将HTTP请求转成分块模式。

在分块模式下，每次调用write方法将导致新的块被写入到报文，这种模式中，无需先设置请求头中的Content-Length。

```java
request.setChunked(true);

// Write some chunks
// 写一些块
for (int i = 0; i < 10; i++) {
  request.write("this-is-chunk-" + i);
}

request.end();
```

**请求超时**

您可使用[setTimeout](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setTimeout-long-)设置一个特定HTTP请求的超时时间。

若请求在超时期限内未返回任何数据，则异常将会被传给异常处理器【exception handler】（若提供），并且请求将会被关闭。

**处理异常**

您可以通过在[HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html)实例上设置异常处理器来处理（捕捉）和请求对应的异常：

```java
HttpClientRequest request = client.post("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
request.exceptionHandler(e -> {
  System.out.println("Received exception: " + e.getMessage());
  e.printStackTrace();
});
```

这种处理器不处理需要在[HttpClientResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html)中应处理的非2xx响应：

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

*重要：XXXNow方法不接收异常处理器（做参数）*

**客户端请求中指定处理器**

不像在调用中提供响应处理器来创建客户端请求对象，相反您可以当请求创建时不提供处理器、稍后在请求对象中调用[handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#handler-io.vertx.core.Handler-)来设置。如：

```java
HttpClientRequest request = client.post("some-uri");
request.handler(response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

**使用流式请求**

[HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html)实例也是一个[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)，这意味着您可以从任何[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)实例读取数据。

例如，您可以将磁盘上的文件直接泵送【Pump】到HTTP请求体中，如下所示：

```java
request.setChunked(true);
Pump pump = Pump.pump(file, request);
file.endHandler(v -> request.end());
pump.start();
```

**写HTTP/2帧**

HTTP/2是用于HTTP请求/响应模型的具有各种帧的一个帧协议，该协议允许发送和接收其他类型的帧。

要发送这样的帧，您可以使用[write](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#write-io.vertx.core.buffer.Buffer-)写入请求，以下是一个例子：

```java
int frameType = 40;
int frameStatus = 10;
Buffer payload = Buffer.buffer("some data");

// Sending a frame to the server
// 发送一帧到服务器
request.writeCustomFrame(frameType, frameStatus, payload);
```

**流重置**

HTTP/1.x不允许请求或响应流进行重置，如当客户端上传了服务器上存在的资源时，服务器依然要接收整个响应。

HTTP/2在请求/响应期间随时支持流重置：

```java
request.reset();
```

默认情况，发送`NO_ERROR(0)`错误代码，可发送另一个代码：

```java
request.reset(8);
```

HTTP/2规范定义了可使用的[错误代码](http://httpwg.org/specs/rfc7540.html#ErrorCodes)列表。

若使用了[request handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#exceptionHandler-io.vertx.core.Handler-)和[response handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#exceptionHandler-io.vertx.core.Handler-)两个处理器过后，在流重置完成时您将会收到通知。

```java
request.exceptionHandler(err -> {
  if (err instanceof StreamResetException) {
    StreamResetException reset = (StreamResetException) err;
    System.out.println("Stream reset " + reset.getCode());
  }
});
```

#### 处理HTTP响应

您可以在请求方法中指定处理器或通过[HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html)对象直接设置处理器来接收到[HttpClientResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html)的实例。

您可以调用[statusCode](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#statusCode--)和[statusMessage](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#statusMessage--)从响应中查询响应的状态代码和状态消息。

```java
client.getNow("some-uri", response -> {
  // the status code - e.g. 200 or 404
  // 状态代码：200、404
  System.out.println("Status code is " + response.statusCode());

  // the status message e.g. "OK" or "Not Found".
  // 状态消息：OK、Not Found
  System.out.println("Status message is " + response.statusMessage());
});
```

**使用流式响应**

[HttpClientResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html)实例也是一个[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)实例，这意味着您可以泵送数据到任何[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)实例。

**响应头和尾**

HTTP响应可包含头信息，使用[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#headers--)来读取响应头。

（该方法）返回的对象是[MultiMap](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)，因为HTTP响应头中单个键也可关联多个值。

```java
String contentType = response.headers().get("content-type");
String contentLength = response.headers().get("content-lengh");
```

分块HTTP响应还可以包含响应尾——这实际上是在发送响应体的最后一个（数据）块。

您可使用[trailers](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#trailers--)方法读取响应尾，尾数据也是一个[MultiMap](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html)。

**读取请求体**

当从报文【Wire】中读取到响应头时，响应处理器就会被调用。

如果响应有响应体，它可能会在响应头被读取后的某个时间以分片的方式到达。 在调用响应处理器之前，我们不要等待所有的响应体到达，因为它可能非常大而要等待很长时间、又或者会花费大量内存。

当响应体的某部分（数据）到达时，[handler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#handler-io.vertx.core.Handler-)将会被调用，而且传入的[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)中包含了响应体的这一分片（部分）内容，

```java
client.getNow("some-uri", response -> {

  response.handler(buffer -> {
    System.out.println("Received a part of the response body: " + buffer);
  });
});
```

若您知道响应体不是很大，并想在处理之前在所有内存中聚合，那么您可以自己聚合：

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

或者当响应已被完全读取时，您可以使用bodyHandler以便读取整个响应体：

```java
client.getNow("some-uri", response -> {

  response.bodyHandler(totalBuffer -> {
    // Now all the body has been read
    // 现在所有的响应体都读取了
    System.out.println("Total response body length is " + totalBuffer.length());
  });
});
```

**结束响应处理器**

当整个响应体被完全读取或者无响应体的响应头被完全读取时，响应的[endHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#endHandler-io.vertx.core.Handler-)就会被调用。

**从响应中读取Cookie**

您可用[cookies](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html#cookies--)方法从响应中获取cookies列表。

或者您可以在响应中自己解析Set-Cookie头。

**30x重定向处理器**

客户端可配置成遵循HTTP重定向：当客户端接收到301、302、303或307状态代码时，它遵循由Location响应头提供的重定向，响应处理器将传递重定向响应以替代原始响应。

这有个例子：

```java
client.get("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).setFollowRedirects(true).end();
```

重定向策略如下：

* 当接收到301、302或303状态代码时，使用GET方法执行重定向
* 当接收到307状态代码时，使用相同的HTTP方法和缓存的请求体执行重定向

> 警告： *随后的重定向会缓存请求体*

默认情况最大的重定向数为`16`，您可使用[setMaxRedirects](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxRedirects-int-)设置。

```java
HttpClient client = vertx.createHttpClient(
    new HttpClientOptions()
        .setMaxRedirects(32));

client.get("some-uri", response -> {
  System.out.println("Received response with status code " + response.statusCode());
}).setFollowRedirects(true).end();
```

没有放之四海而皆准的策略，缺省的重定向策略可能不能满足您的需要。

默认重定向策略可使用自定义实现更改：

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
这个策略将会处理接收到的原始[HttpClientResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html)，并返回`null`或`Future<HttpClientRequest>`。

* 当返回的是null时，处理原始响应
* 当返回的是Future时，请求将在它成功完成后发送
* 当返回的是Future时，请求失败时调用设置的异常处理器

返回的请求必须是未发送的，这样原始请求处理器才会被发送而且客户端之后才能发送请求。大多数原始请求设置将会传播（拷贝）到新请求中：

* 请求头，除非您已经设置了一些头（包括[setHost](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#setHost-java.lang.String-)）
* 请求体，除非返回的请求使用了GET方法
* 响应处理器
* 请求异常处理器
* 请求超时

**100-持续处理【Continue Handling】**

根据[HTTP 1.1规范](http://www.w3.org/Protocols/rfc2616/rfc2616-sec8.html)，一个客户端可以设置请求头`Expect: 100-Continue`，并且在发送剩余请求体之前发送请求头。

然后服务器可以通过回复临时响应状态`Status: 100 (Continue)`来告诉客户端可以发送请求的剩余部分。

这里的想法是允许服务器在发送大量数据之前授权、接收/拒绝请求，若请求不能被接收，则发送大量数据信息会浪费带宽，并将服务器绑定在读取刚刚丢失的无用数据中。

Vert.x允许您在客户端请求对象中设置一个[continueHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#continueHandler-io.vertx.core.Handler-)。

如果服务器发回一个状态`Status: 100 (Continue)`则表示（客户端）可以发送请求的剩余部分。

这和[sendHead](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#sendHead--)结合起来发送请求的头信息。

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

在服务端，一个Vert.x的HTTP服务器可配置成接收到`Expect: 100-Continue`头时自动发回100 Continue临时响应信息。

这个可通过调用[setHandle100ContinueAutomatically](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setHandle100ContinueAutomatically-boolean-)来设置。

若您想要决定是否手动发送持续响应，则此属性可设置成false（默认值），那么您可以检查头信息并且调用[writeContinue](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html#writeContinue--)以使（告诉）客户端持续发送请求体：

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

您也可以通过直接发送故障状态代码来拒绝该请求：这种情况下，请求体应该被忽略或连接应该被关闭（100-Continue是一个性能提示，并不是逻辑协议约束）：

```java
httpServer.requestHandler(request -> {
  if (request.getHeader("Expect").equalsIgnoreCase("100-Continue")) {

    //
    boolean rejectAndClose = true;
    if (rejectAndClose) {

      // Reject with a failure code and close the connection
      // this is probably best with persistent connection
      // 使用失败代码拒绝，并关闭这个连接，这可能是持久连接最好的
      request.response()
          .setStatusCode(405)
          .putHeader("Connection", "close")
          .end();
    } else {

      // Reject with a failure code and ignore the body
      // this may be appropriate if the body is small
      // 使用失败代码拒绝，忽略请求体，若体很小，这是适用的
      request.response()
          .setStatusCode(405)
          .end();
    }
  }
});
```

**客户端端推送**

服务器推送是一个HTTP/2的新功能，它可以为单个客户端并行发送多个响应。

可以在接受服务器推送的请求/响应的请求上设置推送处理器：

```java
HttpClientRequest request = client.get("/index.html", response -> {
  // Process index.html response
  // 处理index.html响应
});

// Set a push handler to be aware of any resource pushed by the server
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
    // 处理它
  }
});
```

若没有设置任何处理器时，任何被推送的流将被客户端自动重置流（错误代码`8`）。

**接收自定义HTTP/2帧**

HTTP/2是用于HTTP请求/响应模型的具有各种帧的一个帧协议，该协议允许发送和接收其他类型的帧。

要接收自定义帧，您可以在请求中使用customFrameHandler，每次自定义帧到达时就会调用它。以下是一个例子：

```java
response.customFrameHandler(frame -> {

  System.out.println("Received a frame type=" + frame.type() +
      " payload" + frame.payload().toString());
});
```

#### 客户端启用压缩

标准的HTTP客户端支持HTTP压缩。

这意味着客户端可以让远程服务器知道它支持压缩，并且能处理压缩过的响应体（数据）。

HTTP服务器可以自由地使用自己支持的压缩算法之一进行压缩，也可以在不压缩的情况下将响应体发回。所以这仅仅是一个HTTP服务器的提示，将来可能被忽略。

要告诉服务器当前客户端支持哪种压缩，则它（请求头）将包含一个`Accept-Encoding`头，其值为可支持的压缩算法，（该值可）支持多种压缩算法。这种情况Vert.x将添加以下头：

```java
Accept-Encoding: gzip, deflate
```

服务器将从其中（算法）选择一个，您可以通过服务器发回的响应中响应头`Content-Encoding`来检测服务器是否适应这个正文。

若响应体通过gzip压缩，它将包含例如下边的头：

```java
Content-Encoding: gzip
```

创建客户端时可使用[setTryUseCompression](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setTryUseCompression-boolean-)设置配置项启用压缩。

默认情况压缩被禁用。

#### HTTP/1.x Pooling和Keep alive

HTTP的Keep alive允许HTTP连接用于多个请求，当您向同一台服务器发送多个请求时，可以更加有效使用连接。

对于HTTP/1.x版本，HTTP客户端支持连接池，它允许您重用请求之间的连接。

为了连接池（能）工作，配置客户端时，keep alive必须通过[setKeepAlive](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setKeepAlive-boolean-)设置成true，默认值为true。

当keep alive启用时，Vert.x将为每一个发送的HTTP/1.0请求添加一个`Connection: Keep-Alive`头。当keep alive禁用时，Vert.x将为每一个HTTP/1.1请求添加一个`Connection: Close`头——表示在响应完成后连接将被关闭。

可使用[setMaxPoolSize](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxPoolSize-int-)为每个服务器配置连接池的最大连接数。

当启用连接池创建请求时，若存在少于已经为服务器创建的最大连接数，Vert.x将创建一个新连接，否则直接将请求添加到队列中。

Keep Alive的连接将不会被客户端自动关闭，要关闭它们您可以关闭客户端实例。

或者，您可使用[setIdleTimeout](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setIdleTimeout-int-)设置空闲时间——在设置的时间内然后没使用的连接将被关闭。请注意空闲超时值以秒为单位而不是毫秒。

#### HTTP/1.1 Pipe-lining

客户端还支持连接上的请求管道。

管道意味着在返回一个响应之前，在同一个连接上发送另一个请求，管道不适合所有请求。

若要启用管道，必须调用[setPipelining](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setPipelining-boolean-)方法，默认管道是禁止的。

当启用管道时，请求可以不等待以前的响应返回而写入到连接。

单个连接的管道请求限制数由[setPipeliningLimite](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setPipeliningLimit-int-)设置，此选项定义了发送到服务器的等待响应的最大请求数。这个限制可以确保和同一个服务器的连接分发到客户端的公平性。

#### HTTP/2 多路复用【Multiplexing】

HTTP/2提倡使用服务器的单一连接，默认情况下，HTTP客户端针对每个服务器都使用单一连接，同样服务器上的所有流都会复用到对应连接中。

当客户端需要使用连接池并使用超过一个连接时，则可使用[setHttp2MaxPoolSize](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2MaxPoolSize-int-)。

当您希望限制每个连接的多路复用流数量而使用连接池而不是单个连接时，可使用[setHttp2MultiplexingLimit](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setHttp2MultiplexingLimit-int-)。

```java
HttpClientOptions clientOptions = new HttpClientOptions().
    setHttp2MultiplexingLimit(10).
    setHttp2MaxPoolSize(3);

// Uses up to 3 connections and up to 10 streams per connection
// 每个连接最多可用三个连接，可连接10个流
HttpClient client = vertx.createHttpClient(clientOptions);
```

连接的复用限制是在客户端上设置限制单个连接的流数量，如果服务器使用[SETTINGS_MAX_CONCURRENT_STREAMS](http://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html#setMaxConcurrentStreams-long-)设置了下限，则有效值可以更低。

HTTP/2连接不会被客户端自动关闭，若要关闭它们，可以调用[close](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#close--)来关闭客户端实例。

或者，您可以使用[setIdleTimeout](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setIdleTimeout-int-)设置空闲时间——这个时间内没有使用的任何连接将被关闭，注意，空闲时间以秒为单位，不是毫秒。

#### HTTP连接

[HttpConnection](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html)提供了处理HTTP连接事件、生命周期、设置的API。

HTTP/2实现了完整[HttpConnection](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html)的API。

HTTP/1.x实现了部分[HttpConnection](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html)的API：仅关闭操作，实现了关闭处理器和异常处理器。该协议并不提供其他操作的语义。

**服务器连接**

[connection](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#connection--)方法会返回服务器上的请求连接：

```java
HttpConnection connection = request.connection();
```

可以在服务器上设置连接处理器【connection handler】，任意连接传入时可得到通知：

```java
HttpServer server = vertx.createHttpServer(http2Options);

server.connectionHandler(connection -> {
  System.out.println("A client connected");
});
```

**客户端连接**

[connection](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html#connection--)方法会返回客户端上的连接请求：

```java
HttpConnection connection = request.connection();
```

可以在请求上设置连接处理器在连接发生时通知：

```java
request.connectionHandler(connection -> {
  System.out.println("Connected to the server");
});
```

**连接设置**

HTTP/2的配置由[Http2Settings](http://vertx.io/docs/apidocs/io/vertx/core/http/Http2Settings.html)数据对象来配置。

每个Endpoint都必须遵守连接另一端的发送设置。

当建立连接时，客户端和服务器交换（各自的）初始配置，初始设置由客户端上的[setInitialSettings](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setInitialSettings-io.vertx.core.http.Http2Settings-)和服务器上的[setInitialSettings](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerOptions.html#setInitialSettings-io.vertx.core.http.Http2Settings-)配置。

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

相反，在收到新的远程设置时会通知[remoteSettingsHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#remoteSettingsHandler-io.vertx.core.Handler-)：

```java
connection.remoteSettingsHandler(settings -> {
  System.out.println("Received new settings");
});
```

*注意：这仅适用于HTTP/2协议。*

**连接ping**

HTTP/2连接ping对于确定连接往返时间或检查连接有效性很有用：[ping](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#ping-io.vertx.core.buffer.Buffer-io.vertx.core.Handler-)发送`PING`帧到远程Endpoint：

```java
Buffer data = Buffer.buffer();
for (byte i = 0;i < 8;i++) {
  data.appendByte(i);
}
connection.ping(data, pong -> {
  System.out.println("Remote side replied");
});
```

当接收到`PING`帧时，Vert.x将自动发送确认，可设置处理器当收到ping帧时发送通知调用处理器：

```java
connection.pingHandler(ping -> {
  System.out.println("Got pinged by remote side");
});
```

处理器只是接到通知，确认被发送，这个功能旨在基于HTTP/2协议之上实现。

*注意：这仅适用于HTTP/2协议。*

**连接关闭/离开**

调用[shutdown](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdown--)将发送`GOAWAY`帧到远程的连接，要求其停止创建流：客户端将停止执行新请求，并且服务器将停止推送响应。发送`GOAWAY`帧后，连接将等待一段时间（默认为30秒），直到所有当前流关闭和连接关闭：

```java
connection.shutdown();
```

[shutdownHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdownHandler-io.vertx.core.Handler-)通知何时关闭流，连接尚未关闭。

有可能只需发送`GOAWAY`帧，和关闭主要的区别在于它将只是告诉远程连接停止创建新流，并没有计划关闭连接：

```java
connection.goAway(0);
```

相反，也可以在收到`GOAWAY`时收到通知：

```java
connection.goAwayHandler(goAway -> {
  System.out.println("Received a go away frame");
});
```

当所有当前流已经关闭并且可关闭连接时，[shutdownHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#shutdownHandler-io.vertx.core.Handler-)将被调用：

```java
connection.goAway(0);
connection.shutdownHandler(v -> {

  // All streams are closed, close the connection
  // 所有流被关闭，连接也关闭
  connection.close();
});
```

当接收到`GOAWAY`时也适用。

*注意：这仅适用于HTTP/2协议。*

**连接关闭**

连接[close](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#close--)方法可关闭连接：

* 它会关闭HTTP/1.x的Socket
* HTTP/2中不延迟关闭，`GOAWAY`将会在连接关闭之前被发送

连接关闭时[closeHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpConnection.html#closeHandler-io.vertx.core.Handler-)将发出通知。

#### HttpClient使用

HttpClient可以在一个Verticle中使用或者嵌入使用。

在Verticle中使用时，Verticle应该使用自己的客户端实例。

一般来说，不应该在不同的Vert.x上下文环境之间共享客户端，因为它可能导致意外行为。

例如：保持活动连接将在打开连接的请求上下文环境调用客户端处理器，后续请求将使用相同上下文环境。

当这种情况发生时，Vert.x会检测到并记录下边警告：

```
Reusing a connection with a different context: an HttpClient is probably shared between different Verticles
```

HttpClient可以嵌套在非Vert.x线程中，如单元测试或纯Java的`main`：客户端处理器将被不同的Vert.x线程和上下文调用，这样的上下文会根据需要创建。对于生产环境，不推荐这样使用。

#### Server sharing

当几个HTTP服务器在同一个端口上监听时，vert.x使用轮询策略来管理请求处理。

我们用一个Verticle来创建一个HTTP服务器如：

**io.vertx.examples.http.sharing.HttpServerVerticle**

```java
vertx.createHttpServer().requestHandler(request -> {
  request.response().end("Hello from server " + this);
}).listen(8080);
```

这个服务正在监听`8080`端口，所以，当这个Verticle被实例化多次，如：vertx运行：

```
vertx run io.vertx.examples.http.sharing.HttpServerVerticle -instances 2
```

将会发生什么？如果两个Verticle都绑定到同一个端口，您将收到一个Socket异常；幸运的是，vert.x正在为您处理这种情况。在与现有服务器相同的主机和端口上部署另一个服务器时，实际上并不会尝试创建在同一主机/端口上监听的新服务器，它只绑定一次到Socket，当接收到请求时，会按照轮询策略调用服务器处理器。

我们现在想象一个客户端如：

```java
vertx.setPeriodic(100, (l) -> {
  vertx.createHttpClient().getNow(8080, "localhost", "/", resp -> {
    resp.bodyHandler(body -> {
      System.out.println(body.toString("ISO-8859-1"));
    });
  });
});
```

Vert.x将请求顺序委托给其中一个服务器：

```
Hello from i.v.e.h.s.HttpServerVerticle@1
Hello from i.v.e.h.s.HttpServerVerticle@2
Hello from i.v.e.h.s.HttpServerVerticle@1
Hello from i.v.e.h.s.HttpServerVerticle@2
...
```

因此，服务器可直接扩展可用的核，而每个Vert.x中的Verticle实例仍然严格使用单线程，您不需要像编写负载均衡器【Load-Balancers】那样使用任何特殊技巧去编写，以便在多核机器上扩展服务器。

#### Vert.x中使用HTTPS

Vert.x的HTTP服务器和客户端可以配置成和网络服务器完全相同的方式使用HTTPS。

有关详细信息，请参阅配置网络服务器以使用[SSL](http://vertx.io/docs/vertx-core/java/#ssl)。

SSL可以通过每个请求的[RequestOptions](http://vertx.io/docs/apidocs/io/vertx/core/http/RequestOptions.html)来启用/禁用，或在指定模式时调用[requestAbs](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#requestAbs-io.vertx.core.http.HttpMethod-java.lang.String-)：

```java
client.getNow(new RequestOptions()
    .setHost("localhost")
    .setPort(8080)
    .setURI("/")
    .setSsl(true), response -> {
  System.out.println("Received response with status code " + response.statusCode());
});
```

[setSsl](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setSsl-boolean-)设置将用作客户端默认配置。

[setSsl](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setSsl-boolean-)将覆盖默认客户端设置：

* 即使客户端配置成使用SSL/TLS，该值设置成`false`将禁用SSL/TLS。
* 即使客户端配置成不适用SSL/TLS，该值设置成`true`将启用SSL/TLS，实际的客户端SSL/TLS（如受信、密钥/证书、密码、ALPN、……）将被重用。

同样：[requestAbs](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#requestAbs-io.vertx.core.http.HttpMethod-java.lang.String-)同样也会（在调用时）覆盖默认客户端设置。

#### WebSocket

[WebSocket](http://en.wikipedia.org/wiki/WebSocket)是一种Web技术，可以在HTTP服务器和HTTP客户端（通常是浏览器）之间实现全双工Socket连接。

Vert.x在客户端和服务器端都支持WebSocket。

**服务器端WebSocket**

在服务器端处理WebSocket有两种方法。

1).*WebSocket处理器*

第一种方法涉及在服务器实例上提供一个[websocketHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServer.html#websocketHandler-io.vertx.core.Handler-)。

当对服务器创建WebSocket连接时，将传入一个[ServerWebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html)的一个实例给该处理器并调用它。

```java
server.websocketHandler(websocket -> {
  System.out.println("Connected!");
});
```

您可以调用[reject](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#reject--)来拒绝一个WebSocket。

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

2).*更新到WebSocket*

处理WebSocket的第二种方法是处理从客户端发送的HTTP升级请求，调用服务器请求对象的[upgrade](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html#upgrade--)方法。

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

3).*服务器WebSocket*

[ServerWebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html)实例能够让您读取在WebSocket握手中的HTTP请求的[headers](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#headers--)，[path](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#path--)，[query](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#query--)和[URI](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html#uri--)。

**客户端WebSocket**

Vert.x的[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)支持WebSocket。

您可以调用[websocket](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html#websocket-io.vertx.core.http.RequestOptions-io.vertx.core.Handler-)操作之一的创建WebSocket连接到服务器，并提供处理器。

当连接建立时，处理器将被调用并且使用[WebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html)实例：

```java
client.websocket("/some-uri", websocket -> {
  System.out.println("Connected!");
});
```

**写入消息到WebSocket**

若您想将一个WebSocket消息写入WebSocket，可使用[writeBinaryMessage](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeBinaryMessage-io.vertx.core.buffer.Buffer-)或[writeTextMessage](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeTextMessage-java.lang.String-)来执行该操作：

```java
Buffer buffer = Buffer.buffer().appendInt(123).appendFloat(1.23f);
websocket.writeBinaryMessage(buffer);

// Write a simple text message
// 写一个简单文本消息
String message = "hello";
websocket.writeTextMessage(message);
```

若WebSocket消息大于使用[setMaxWebsocketFrameSize](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html#setMaxWebsocketFrameSize-int-)设置的WebSocket的帧的最大值，则Vert.x在将其发送到报文之前将其拆分为多个WebSocket帧。

**写入帧到WebSocket**

WebSocket消息可以由多个帧组成，在这种情况下，第一帧是二进制或文本帧（text|binary），后边跟着零个或多个连续帧。

消息中的最后一帧标记成final。

要发送多个帧组成的消息，请使用[WebSocketFrame.binaryFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#binaryFrame-io.vertx.core.buffer.Buffer-boolean-)，[WebSocketFrame.textFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#textFrame-java.lang.String-boolean-)或[WebSocketFrame.continuationFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html#continuationFrame-io.vertx.core.buffer.Buffer-boolean-)创建帧，并使用[writeFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFrame-io.vertx.core.http.WebSocketFrame-)将其写入WebSocket。

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

许多情况下，您只需要发送一个包含了单个最终帧的WebSocket消息，因此我们提供了一些使用[writeFinalBinaryFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFinalBinaryFrame-io.vertx.core.buffer.Buffer-)和[writeFinalTextFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#writeFinalTextFrame-java.lang.String-)的快捷方法。

下边是示例：

```java
websocket.writeFinalTextFrame("Geronimo!");

// Send a websocket messages consisting of a single final binary frame:
// 发送由单个最终二进制帧组成的websocket消息：
Buffer buff = Buffer.buffer().appendInt(12).appendString("foo");

websocket.writeFinalBinaryFrame(buff);
```

**从WebSocket读取帧**

要从WebSocket读取帧，您可以使用[frameHandler](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html#frameHandler-io.vertx.core.Handler-)。

当帧到达时，会传入一个[WebSocketFrame](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketFrame.html)实例给帧处理器，并调用它，例如：

```java
websocket.frameHandler(frame -> {
  System.out.println("Received a frame of size!");
});
```

**关闭WebSocket**

完成之后，请使用[close](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocketBase.html#close--)方法关闭Socket连接。

**流式WebSocket**

[WebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html)实例也是[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)和[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)，因此可以和泵一起使用。

当使用WebSocket作为写入流或读取流时，它只能用于不分割多个帧的二进制帧一起使用的WebSocket连接。

#### 使用HTTP/HTTPS连接代理

HTTP客户端支持通过HTTP代理（如Squid）或*SOCKS4a*或*SOCKS5*代理访问HTTP/HTTPS的URL。CONNECT协议使用HTTP/1.x但可以连接到HTTP/1.x和HTTP/2服务器。

连接到h2c（未加密HTTP/2服务器）可能不受HTTP代理支持，因为它们仅支持HTTP/1.1。

可以通过[HttpClientOptions](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientOptions.html)中的[ProxyOptions](http://vertx.io/docs/apidocs/io/vertx/core/net/ProxyOptions.html)对象配置来配置代理（包括代理类型、主机名、端口和可选用户名和密码）。

以下是使用HTTP代理的例子：

```java
HttpClientOptions options = new HttpClientOptions()
    .setProxyOptions(new ProxyOptions().setType(ProxyType.HTTP)
        .setHost("localhost").setPort(3128)
        .setUsername("username").setPassword("secret"));
HttpClient client = vertx.createHttpClient(options);
```

当客户端连接到HTTP URL时，它连接到代理服务器，并在HTTP请求中提供完整URL（"GET http://www.somehost.com/path/file.html HTTP/1.1"）。

当客户端连接到HTTPS URL时，它要求代理使用CONNECT方法创建到远程主机的通道【tunnel】。

对于SOCKS5代理：

```java
HttpClientOptions options = new HttpClientOptions()
    .setProxyOptions(new ProxyOptions().setType(ProxyType.SOCKS5)
        .setHost("localhost").setPort(1080)
        .setUsername("username").setPassword("secret"));
HttpClient client = vertx.createHttpClient(options);
```

DNS解析会一直在代理服务器上执行，为了实现SOCKS4客户端的功能，需要先在本地解析DNS地址。

**自动清理**

如果您正在从Verticle内部创建HTTP服务器和客户端，则在撤销该Verticle时，这些服务器和客户端将自动关闭。

### 使用Vert.x共享数据

共享数据包含的功能允许您可以安全地在应用程序的不同部分之间、同一Vert.x实例中的不同应用程序之间或集群中的不同Vert.x实例之间安全地共享数据。

共享数据包括本地共享Map、分布式、集群范围Map、异步集群范围锁和异步集群范围计数器【local shared maps, distributed, cluster-wide maps, asynchronous cluster-wide locks, asynchronous cluster-wide counters.】。

*重要：分布式数据结构的行为取决于您使用的集群管理器，网络分区面临的备份（复制）和行为由集群管理器和它的配置来定义。请参阅集群管理器文档以及底层框架手册。*

#### 本地共享Map【Local shared map】

本地共享Map【[Local Shared Map](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/LocalMap.html)】允许您在同一个Vert.x实例中的不同Event Loop（如不同的Verticle中）之间安全共享数据。

本地共享Map仅允许将某些数据类型作为键值和值，这些类型必须是不可变的，或可以像[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)那样复制某些其他类型。在后一种情况中，键/值将被复制，然后再放到Map中。

这样，我们可以确保在Vert.x应用程序不同线程之间——无法共享访问可变状态【*Shared Access to Mutable State*】，因此您不必担心通过同步访问来保护该状态。

以下是使用共享本地Map的示例：

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

#### 集群范围异步Map【Cluster-wide asynchronous map】

集群范围异步Map允许从集群的任何节点将数据放到Map中，并从任何其他节点读取。

这使得它们对于托管Vert.x Web应用程序的服务器场【farm】中的会话状态存储非常有用。

您可以使用[getClusterWideMap](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getClusterWideMap-java.lang.String-io.vertx.core.Handler-)获取[AsyncMap](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html)的实例。

获取Map是异步的，返回结果可以传给您指定的处理器中，以下是一个例子：

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

**将数据放入Map**

您可以使用[put](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html#put-java.lang.Object-java.lang.Object-io.vertx.core.Handler-)将数据放入Map。

实际上put方法是异步的，一旦完成它会通知处理器：

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

**从Map读取数据**

您可以使用[get](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html#get-java.lang.Object-io.vertx.core.Handler-)从Map读取数据。

实际的get也是异步的，一段时间过后它会通知处理器并传入结果。

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

**其他Map操作**

您还可以从异步Map中删除条目、清除Map、读取它的尺寸。

有关更多信息，请参阅[API文档](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/AsyncMap.html)。

#### 集群范围锁

集群范围锁【[Cluster wide locks](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html)】允许您在集群中获取独占锁——当您想要在任何时间只在集群一个节点上执行某些操作或访问资源时，这很有用。

集群范围锁具有异步API，它和大多数等待锁释放的阻塞调用线程的API锁不相同。

可使用[getLock](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getLock-java.lang.String-io.vertx.core.Handler-)获取锁。

它不会阻塞，但当锁可用时，将[Lock](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html)的实例传入处理器并调用它，表示您现在拥有该锁。

若您拥有的锁没有其他调用者，集群上的任何地方都可以获得该锁。

当您有用锁完成后，您可以调用[release](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/Lock.html#release--)来释放它，所以另一个调用者可获得它。

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

您可以为锁设置一个超时，若在超时时间期间无法获取锁，处理器将被调用失败：

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

#### 集群范围计时器

集群范围计时器【Cluster-wide counters】在应用程序不同节点之间维护原子计数器通常很有用。

您可以用[Counter](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/Counter.html)来做到这一点。

您可以通过[getCounter](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/SharedData.html#getCounter-java.lang.String-io.vertx.core.Handler-)获取一个实例：

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

有更多信息，请参阅[API文档](http://vertx.io/docs/apidocs/io/vertx/core/shareddata/Counter.html)。

### Vert.x访问文件系统

Vert.x中的[FileSystem](http://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html)对象提供了许多操作文件系统的方法。

每个Vert.x实例有一个文件系统对象，您可以使用[fileSystem](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#fileSystem--)获取它。

（每个操作）都提供了阻塞和非阻塞版本，非阻塞版本采用处理器，当操作完成或发生错误时调用该处理器。

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

阻塞的版本名为`xxxBlocking`，它要么返回结果或直接抛出异常。很多情况下，根据操作系统和文件系统，一些潜在的阻塞操作可以快速返回，这就是我们为什么提供它。但是强烈建议您在Event Loop中使用它之前测试使用它们究竟需要耗费多长时间，以避免使用黄金法则。

以下是使用阻塞API的拷贝示例：

```java
FileSystem fs = vertx.fileSystem();

// Copy file from foo.txt to bar.txt synchronously
// 同步拷贝从foo.txt到bar.txt
fs.copyBlocking("foo.txt", "bar.txt");
```

许多操作存在copy、move、truncate、chmod和许多其他文件操作，我们不会在这里列出所有内容，请参考[API文档](http://vertx.io/docs/apidocs/io/vertx/core/file/FileSystem.html)获取完整列表。

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

#### 异步文件

Vert.x提供了异步文件抽象（对象），允许您操作文件系统上的文件。

您可以像下边代码打开一个[AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)：

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

AsyncFile实现了ReadStream和WriteStream，因此您可以将文件和其他流对象配合泵工作，如网络Socket、HTTP请求和响应、WebSocket。

它们还允许您直接读写。

**随机访问写**

要使用AsyncFile进行随机访问写，请使用[write](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#write-io.vertx.core.buffer.Buffer-long-io.vertx.core.Handler-)方法。

这个方法的参数有：

* buffer：要写入（数据）的（目标）Buffer
* position：一个整数指定在文件中写入Buffer的位置，若位置大于或等于文件大小，文件将被扩展【放大】以适应偏移量
* handler：结果处理器

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

**随机访问读**

要使用AsyncFile进行随机访问读，请使用[read](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#read-io.vertx.core.buffer.Buffer-int-long-int-io.vertx.core.Handler-)方法。

该方法的参数有：

* buffer：读取数据的Buffer
* offset：读取数据将被放到Buffer中的偏移量
* position：从文件中读取数据的位置
* length：要读取的数据的字节数
* handler：结果处理器

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

**打开选项**

打开AsyncFile时，您可以传递一个[OpenOptions](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html)实例，这些选项描述了访问文件的行为。例如：您可使用[setRead](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setRead-boolean-)，[setWrite](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setWrite-boolean-)和[setPerm](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setPerms-java.lang.String-)方法配置文件访问权限。

若打开的文件已经存在，则可以使用[setCreateNew](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setCreateNew-boolean-)和[setTruncateExisting](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setTruncateExisting-boolean-)配置对应行为。

您可以使用[setDeleteOnClose](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setDeleteOnClose-boolean-)标记在关闭时或JVM停止时要删除的文件。

**将数据刷新到底层存储**

在OpenOptions中，您可以使用[setDsync](http://vertx.io/docs/apidocs/io/vertx/core/file/OpenOptions.html#setDsync-boolean-)在每次写入时启用/禁用内容的自动同步。这种情况下，您可以使用[flush](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#flush--)方法手动刷新OS缓存中的数据写入。

该方法也可使用一个处理器来调用，这个处理器在flush完成时被调用。

**将AsyncFile作为ReadStream和WriteStream**

AsyncFile实现了ReadStream和WriteStream。您可以使用泵将数据与其他读取和写入流进行数据泵送。例如，这会将内容复制到另外一个AsyncFile：

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

您还可以使用泵将文件内容写入到HTTP响应中，或者写入任意WriteStream。

**从类路径访问文件**

当Vert.x找不到文件系统上的文件时，它尝试从类路径中解析该文件。请注意，类路径的资源路径不以`/`开头。

由于Java不提供对类路径资源的异步方法，所以当类路径资源第一次被访问时，该文件将复制到工作线程中的文件系统。当第二次访问相同资源时，访问的文件直接从（工作线程的）文件系统提供。即使类路径资源发生变化（例如开发系统中），也会提供原始内容。

您可以将系统属性`vertx.disableFileCaching`设置为true，禁用此（文件）缓存行为。文件缓存的路径默认为`.vertx`，它可以通过设置系统属性`vertx.cacheDirBase`进行自定义。

您还可以通过系统属性`vertx.disableFileCPResolving`设置为true来禁用整个类路径解析功能。

*注意：当加载`io.vertx.core.impl.FileResolver`类时，这些系统属性将被检查【Evaluate】一次，因此，在加载此类之前应该设置这些属性，或者在启动它时作为JVM系统属性来设置。*

**关闭AsyncFile**

您可调用[close](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html#close--)来关闭一个AsyncFile。关闭是异步的，如果希望在关闭过后收到通知，您可指定一个处理器作为函数（close）参数传入。

### 数据报套接字（UDP）

Vert.x中使用用户数据报协议（UDP）是一块蛋糕。

UDP是无连接的传输，基本上意味着您没有与远程对等体的持续连接。

相反您可以发送和接收包，并且每个包包含远程地址。

UDP不像TCP的使用那样安全，这意味着不能保证发送的数据包会被对应的Endpoint接收。

唯一的保证是，它将会完全接收或者不完全接收。

您通常也不能发送大于网络接口MTU大小的数据，这是因为每一个数据包将会作为一个包发送。

但是要注意，即使数据包尺寸小于MTU，它仍然可能会失败。

它失败的尺寸取决于操作系统等（其他），所以经验法则是尝试发送小数据包。

由于UDP的本质，最适合一些允许丢弃数据包的应用（如监视应用程序）。

其优点是与TCP相比具有更少的开销，可以由NetServer和NetClient处理（参考前文）。

#### 创建一个DatagramSocket

要使用UDP，您首先要创建一个[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)，若您只是想要发送数据、发送、接收，这并不重要。

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
```

返回的[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)实例不会绑定到特定端口，若您只想发送数据（如客户端）这不是一个问题，但更多的在下一节。

#### 发送数据报包

如前所述，用户数据报协议（UDP）将数据分组发送给远程对等体，但不以持续的方式连接到它们。

这意味着每个数据包都可以发送到另一个远程对等体。

发送数据包很容易，如下所示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
Buffer buffer = Buffer.buffer("content");
// Send a Buffer
// 发送Buffer
socket.send(buffer, 1234, "10.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
// Send a String
// 发送一个字符串
socket.send("A string used as content", 1234, "10.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
```

#### 接收数据报包

若您想要接收数据包，则您需要调用`listen(...)`绑定一个[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)。

这样您就可以接收[DatagramPacket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html)的数据，发送到[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)监听的地址和端口。

除此之外，您还要设置一个Handler，将在接收每个[DatagramPacket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html)时被调用。

[DatagramPacket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html)有以下方法：

* [sender](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html#sender--)：表示数据发送方的InetSocketAddress。
* [data](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html#data--)：保存接收数据的Buffer。

所以要监听一个特定地址和端口，您可以像下边这样：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
  if (asyncResult.succeeded()) {
    socket.handler(packet -> {
      // Do something with the packet
	  // 使用包做些事
    });
  } else {
    System.out.println("Listen failed" + asyncResult.cause());
  }
});
```

注意，即使AsyncResult成功，它只意味着它可能写入了网络堆栈，但不保证它已经到达或者将到达远程对等体。

若您需要这样的保证，您可在TCP之上建立一些握手逻辑。

#### 多播【Multicast】

**发送多播数据包**

多播允许多个Socket接收相同的数据包，该任务可以通过加入可发送数据包的相同多播组完成。

我们将在下一节中介绍如何加入多播组，从而接收数据包。

现在让我们专注于如何发送这些（数据），发送多播报文与发送普通数据报报文没什么不同。

唯一的区别是您可以将多播组的地址传递给send方法发送出去。

这里显示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
Buffer buffer = Buffer.buffer("content");
// Send a Buffer to a multicast address
// 发送Buffer到多播地址
socket.send(buffer, 1234, "230.0.0.1", asyncResult -> {
  System.out.println("Send succeeded? " + asyncResult.succeeded());
});
```

加入多播组`230.0.0.1`的所有Socket都将收到该报文。

**接收多播数据包**

若要接收特定多播组的数据包，则需要通过在其上调用`listen(...)`来绑定一个[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)，并加入多播组。

这样，您将能够接收发送到[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)监听的地址和端口的DatagramPacket，也可以接受发送到多播组的数据报。

除此之外，您可设置一个处理器，每次接收到DatagramPacket时会被调用。

[DatagramPacket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramPacket.html)有以下方法：

* sender()：表示数据报发送方的InetSocketAddress
* data()：保存接收数据的Buffer

因此，要监听指定的地址和端口、并且接收多播组`230.0.0.1`的数据报，您将执行如下操作：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
  if (asyncResult.succeeded()) {
    socket.handler(packet -> {
      // Do something with the packet
	  // 用数据报做些事
    });

    // join the multicast group
	// 加入多播组
    socket.listenMulticastGroup("230.0.0.1", asyncResult2 -> {
        System.out.println("Listen succeeded? " + asyncResult2.succeeded());
    });
  } else {
    System.out.println("Listen failed" + asyncResult.cause());
  }
});
```

**取消/离开多播组**

有时候您想在有限时间内为多播组接收数据包。

这种情况下，您可以先监听他们，之后unlisten。

这里显示：

```java
DatagramSocket socket = vertx.createDatagramSocket(new DatagramSocketOptions());
socket.listen(1234, "0.0.0.0", asyncResult -> {
    if (asyncResult.succeeded()) {
      socket.handler(packet -> {
        // Do something with the packet
		// 用数据报做些事
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

**阻塞多播**

除了unlisten一个多播地址以外，也有可能阻塞指定发送者地址的多播。

注意：这仅适用于某些操作系统和内核版本，所以请检查操作系统文档看是它是否支持（该功能）。

这是一个专家功能。

要阻塞来自特定地址的多播，您可以在DatagramSocket上调用`blockMulticastGroup(...)`，如下所示：

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

**DatagramSocket属性**

当创建[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)时，您可以通过[DatagramSocketOptions](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html)对象设置多个属性来更改它的行为。这些（属性）列在这儿：

* [setSendBufferSize](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setSendBufferSize-int-)以字节为单位设置发送缓冲区的大小。
* [setReceiveBufferSize](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setReceiveBufferSize-int-)设置TCP接收缓冲区大小（以字节为单位）。
* [setReuseAddress](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setReuseAddress-boolean-)若为true，则`TIME_WAIT`状态中的地址在关闭后可重用。
* [setTrafficClass](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setTrafficClass-int-)
* [setBroadcast](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setBroadcast-boolean-)设置或清除`SO_BROADCAST`套接字选项，设置此选项时，数据报（UDP）数据包可能会发送到本地接口的广播地址。
* [setMulticastNetworkInterface](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setMulticastNetworkInterface-java.lang.String-)设置或清除`IP_MULTICAST_LOOP`套接字选项，设置此选项时，多播数据包也将在本地接口上接收。
* [setMulticastTimeToLive](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocketOptions.html#setMulticastTimeToLive-int-)设置`IP_MULTICAST_TTL`套接字选项。TTL表示“活动时间【Time To Live】”，单这种情况下，它指定允许数据包经过的IP跳数【Hops】，特别是用于多播流量。转发数据包的每个路由器或网管会递减TTL，如果路由器将TTL递减为0，则不会再转发。

**DatagramSocket本地地址**

您可以通过调用[localAddress](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html#localAddress--)来查找套接字的本地地址（即UDP Socket这边的地址）。若您之前调用`listen(...)`绑定了[DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html)则将返回一个InetSocketAddress，否则返回null。

**关闭DatagramSocket**

您可以通过调用[close](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html#close-io.vertx.core.Handler-)方法来关闭Socket，它将关闭Socket并释放所有资源。


### DNS客户端

通常情况下，您将需要以异步方式来获取DNS信息。

不幸的是，Java虚拟机本身附带的API是不可能的，因此Vert.x提供了它自己的完全异步解析DNS的API。

若要获取DnsClient实例，您将通过Vertx实例创建一个新的。

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
```

请注意，您可以传入InetSocketAddress参数的变量，以指定更多的DNS服务器来尝试查询解析DNS。它将按照此处指定的相同顺序查询DNS服务器，若头一个在第一次使用时产生了错误下一个将（在产生错误的地方）使用。

#### lookup

尝试查找给定名称的A（ipv4）或AAAA（ipv6）记录。第一个返回的（记录）将会被使用，因此它的操作方式和操作系统上使用`nslookup`类似。

要查找`vertx.io`的A/AAAA记录，您通常会使用它：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.lookup("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

#### lookup4

尝试查找给定名称的A（ipv4）记录。第一个返回的（记录）将会被使用，因此它的操作方式与操作系统上使用`nslookup`类似。

要查找`vertx.io`的A记录，您通常会使用它：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.lookup4("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

#### lookup6

尝试查找给定名称的AAAA（ipv6）记录。第一返回的（记录）将会被使用，因此它的操作方式与在操作系统上使用`nslookup`类似。

要查找`vertx.io`的AAAA记录，您通常会使用它：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.lookup6("vertx.io", ar -> {
  if (ar.succeeded()) {
    System.out.println(ar.result());
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

#### resolveA

尝试解析给定名称的所有A（ipv4）记录，这与在unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有A记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

#### resolveAAAA

尝试解析给定名称的所有AAAA（ipv6）记录，这与在unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有AAAA记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

#### resolveCNAME

尝试解析给定名称的所有CNAME记录，这与在unix操作系统上使用`dig`类似。

要查找`vertx.io`的所有CNAME记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

#### resolveMX

尝试解析给定名称的所有MX记录，MX记录用于定义哪个邮件服务器接受给定域的电子邮件。

要查找您常用执行的`vertx.io`的所有MX记录：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

请注意，列表将包含按照它们优先级排序的[MxRecord](http://vertx.io/docs/apidocs/io/vertx/core/dns/MxRecord.html)，这意味着列表中优先级低的MX记录会第一个优先出现在列表中。

[MxRecord](http://vertx.io/docs/apidocs/io/vertx/core/dns/MxRecord.html)允许您通过下边提供的方法访问MX记录的优先级和名称：

```java
record.priority();
record.name();
```

#### resolveTXT

尝试解析给定名称的所有TXT记录，TXT记录通常用于定义域的额外信息。

要解析`vertx.io`的所有TXT记录，您可以使用下边几行：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

#### resolveNS

尝试解析给定名称的所有NS记录，NS记录指定哪个DNS服务器托管给定域的DNS信息。

要解析`vertx.io`的所有NS记录，您可以使用下边几行：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

#### resolveSRV

尝试解析给定名称的所有SRV记录，SRV记录用于定义服务端口和主机名等额外信息。一些协议需要这个额外信息。

要查找`vertx.io`的所有SRV记录，您通常会执行以下操作：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
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

请注意，列表将包含按照它们优先级排序的[SrvRecord](http://vertx.io/docs/apidocs/io/vertx/core/dns/SrvRecord.html)，这意味着优先级低的记录会第一个优先出现在列表中。

[SrvRecord](http://vertx.io/docs/apidocs/io/vertx/core/dns/SrvRecord.html)允许您访问SRV记录本身中包含的所有信息：

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

#### resolvePTR

尝试解析给定名称的PTR记录，PTR记录将`ipaddress`映射到名称。

要解析IP地址`10.0.0.1`的PTR记录，您将使用`1.0.0.10.in-addr.arpa`的PTR概念。

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.resolvePTR("1.0.0.10.in-addr.arpa", ar -> {
  if (ar.succeeded()) {
    String record = ar.result();
    System.out.println(record);
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

#### reverseLookup

尝试对ipaddress进行反向查找，这与解析PTR记录类似，但是允许您只传递ipaddress，而不是有效的PTR查询字符串。

要做ipaddress 10.0.0.1的反向查找类似的事：

```java
DnsClient client = vertx.createDnsClient(53, "10.0.0.1");
client.reverseLookup("10.0.0.1", ar -> {
  if (ar.succeeded()) {
    String record = ar.result();
    System.out.println(record);
  } else {
    System.out.println("Failed to resolve entry" + ar.cause());
  }
});
```

#### 错误处理

如前边部分所述，DnsClient允许您传递一个Handler，一旦查询完成将会传入一个AsyncResult给处理器并通知它。

在出现错误的情况下，通知中将包含一个DnsException，该异常会打开一个[DnsResponseCode](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html)用于分辨为何失败。此DnsResponseCode可用于更详细检查原因。

可能的DnsResponseCode值是：

* [NOERROR](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOERROR)没有找到给定查询的记录
* [FORMERROR](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#FORMERROR)格式错误
* [SERVFAIL](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#SERVFAIL)服务器故障
* [NXDOMAIN](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NXDOMAIN)名称错误
* [NOTIMPL](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOTIMPL)DNS服务器没实现
* [REFUSED](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#REFUSED)DNS服务器拒绝查询
* [YXDOMAIN](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#YXDOMAIN)域名不应该存在
* [YXRESET](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#YXRRSET)资源记录不应该存在
* [NXRRSET](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NXRRSET)RRSET不存在
* [NOTZONE](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#NOTZONE)名称不在区域内
* [BADVERS](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADVERS)版本的扩展机制不好
* [BADSIG](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADSIG)非法签名
* [BADKEY](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADKEY)非法密钥
* [BADTIME](http://vertx.io/docs/apidocs/io/vertx/core/dns/DnsResponseCode.html#BADTIME)错误时间戳

所有这些错误都由DNS服务器本身“生成”，您可以从DnsException中获取DnsResponseCode，如：

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

### 流【Stream】

Vert.x有几个对象可以让项【items】被读取和写入。

在以前的版本中，`streams.adoc`软件包正在独占操作[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)对象。从现在开始，流不再与Buffer耦合，它们可以和任意类型的对象一起工作。

在Vert.x中，写调用立即返回，并且内部排队写入。

不难看出，若写入对象的速度比实际写入底层数据资源速度快，那么写入队列就会无限增长，最终导致内存耗尽。

为了解决这个问题，Vert.x API中的一些对象提供了简单的流程控制（回压）功能。

任何流程可感知的写入流对象都实现了[WriteStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)，而任何流程可感知的读取流对象都实现了[ReadStream](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html)。

让我们举个例子，我们要从ReadStream中读取数据，然后将数据写入WriteStream。

一个非常简单的例子是从NetSocket读取然后写回到同一个NetSocket——因为NetSocket实现了ReadStream和WriteStream。请注意，这适用于任何符合ReadStream和WriteStream的对象，包括HTTP请求、HTTP响应、异步文件I/O、WebSocket等。

这样做的一个原生的方法是直接获取已经读取的数据，并立即将其写入NetSocket：

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

上面的例子有一个问题：如果数据从Socket读取的速度比可以写回Socket的速度快，那么它将在NetSocket的写队列中累计，最终用完RAM。这可能会发生，例如，若Socket另一端的客户端读取速度不够快，则有效地将连接回压。

由于NetSocket实现了WriteStream，我们可以在写入之前检查WriteStream是否已满：

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

这个例子不会用完RAM，单如果写入队列已满，我们最终会丢失数据。我们真正想要做的是在写入队列已满时暂停NetSocket：

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

我们几乎在那儿，单不完全相同。NetSocket现在在文件已满时暂停，但是当写队列处理挤压时，我们依然要取消暂停：

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

在那里我们有它。当写队列准备好接收更多的数据时，[drainHandler](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#drainHandler-io.vertx.core.Handler-)事件处理器将被调用，它会恢复NetSocket，允许读取更多的数据。

在编写Vert.x应用程序时，这样做是很常见的，因此我们提供了一个名为[Pump](http://vertx.io/docs/apidocs/io/vertx/core/streams/Pump.html)的帮助类，它为您完成所有这些艰苦的工作。您只需要给ReadStream追加上WriteStream，然后启动它：

```java
NetServer server = vertx.createNetServer(
    new NetServerOptions().setPort(1234).setHost("localhost")
);
server.connectHandler(sock -> {
  Pump.pump(sock, sock).start();
}).listen();
```

这和更加详细的例子完全一样。

现在我们来看看ReadStream和WriteStream的方法。

#### ReadStream

ReadStream的实现类包括：[HttpClientResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html), [DatagramSocket](http://vertx.io/docs/apidocs/io/vertx/core/datagram/DatagramSocket.html), [HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html), [HttpServerFileUpload](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerFileUpload.html), [HttpServerRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html), [MessageConsumer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageConsumer.html), [NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html), [WebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html), [TimeoutStream](http://vertx.io/docs/apidocs/io/vertx/core/TimeoutStream.html), [AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)。

函数：

* [handler](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#handler-io.vertx.core.Handler-)：设置一个处理器，它将从ReadStream读取项【items】
* [pause](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#pause--)：暂停处理器，暂停时，处理器中将不会受到任何项【items】
* [resume](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#resume--)：恢复处理器，若任何项到达则处理器将被调用
* [exceptionHandler](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#exceptionHandler-io.vertx.core.Handler-)若ReadStream发生异常，将被调用
* [endHandler](http://vertx.io/docs/apidocs/io/vertx/core/streams/ReadStream.html#endHandler-io.vertx.core.Handler-)：当流到达时将被调用。这有可能是到达了描述文件的EOF、达到HTTP请求的请求结束、或TCP Socket的连接被关闭

#### WriteStream

WriteStream的实现类包括：[HttpClientRequest](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientRequest.html), [HttpServerResponse](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerResponse.html)，[WebSocket](http://vertx.io/docs/apidocs/io/vertx/core/http/WebSocket.html), [NetSocket](http://vertx.io/docs/apidocs/io/vertx/core/net/NetSocket.html), [AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html), [MessageProducer](http://vertx.io/docs/apidocs/io/vertx/core/eventbus/MessageProducer.html)

函数：

* [write](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#write-java.lang.Object-)：写入一个对象到WriteStream，该方法将永远不会阻塞，内部是排队写入并且底层资源是异步写入。
* [setWriteQueueMaxSize](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#setWriteQueueMaxSize-int-)：设置写入队列被认为是*full*的对象的数量——方法writeQueueFull返回true。注意，当写队列被认为已满时，若写（操作）被调用则数据依然会被接收和排队。实际数量取决于流的实现，对于[Buffer](http://vertx.io/docs/apidocs/io/vertx/core/buffer/Buffer.html)，尺寸代表实际写入的字节数，而并非缓冲区的数量。
* [writeQueueFull](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#writeQueueFull--)：若写队列被认为已满，则返回true。
* [exceptionHandler](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#exceptionHandler-io.vertx.core.Handler-)：若WriteStream发生异常，则被调用。
* [drainHandler](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#drainHandler-io.vertx.core.Handler-)：若WriteStream被认为不再满，则处理器将被调用。

#### Pump

泵【Pump】的实例有以下几种方法：

* [start](http://vertx.io/docs/apidocs/io/vertx/core/streams/Pump.html#start--)：启动泵。
* [stop](http://vertx.io/docs/apidocs/io/vertx/core/streams/Pump.html#stop--)：停止泵，当泵启动时它要处于停止模式。
* [setWriteQueueMaxSize](http://vertx.io/docs/apidocs/io/vertx/core/streams/Pump.html#setWriteQueueMaxSize-int-)：与WriteStream上的[setWriteQueueMaxSize](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html#setWriteQueueMaxSize-int-)相同。

一个泵可以启动和停止多次。

当泵首次创建时，它不会启动，您需要调用start()方法来启动它。

### 记录解析器【Record Parser】

记录解析器允许您轻松解析由字节序列或固定尺寸带分隔符的记录的协议。

它将输入缓冲区序列转换为已配置的缓冲区序列（固定大小或带分隔符的记录）。

例如，若您使用`\n`分割的简单ASCII文本协议，并输入如下：

```
buffer1:HELLO\nHOW ARE Y
buffer2:OU?\nI AM
buffer3: DOING OK
buffer4:\n
```

记录解析器将产生

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

有关更多详细信息，请查看[RecordParser](http://vertx.io/docs/apidocs/io/vertx/core/parsetools/RecordParser.html)类。

### 线程安全

大多数Vert.x对象可以从不同的线程安全地访问，但是，当从相同的上下文访问它们时，性能会被优化。

例如，若您部署了一个创建NetServer的Verticle，该NetServer在处理器中提供了NetSocket实例，则最好始终从该Verticle的Event Loop中访问Socket实例。

若您坚持使用Vert.x中的Standard Verticle部署模型，可以避免在Verticle之间共享对象，那么您没有必要考虑这样的情况。

### Metrics SPI

默认情况下，Vert.x不会记录任何指标。相反，它为其他人提供了一个SPI，可以将其添加到类路径中。SPI是一项高级功能，允许实施者从Vert.x捕获事件以收集指标。有关详细信息，请参阅[API文档](http://vertx.io/docs/apidocs/io/vertx/core/spi/metrics/VertxMetrics.html)。

若使用[setFactory](http://vertx.io/docs/apidocs/io/vertx/core/metrics/MetricsOptions.html#setFactory-io.vertx.core.spi.VertxMetricsFactory-)嵌入了Vert.x实例，也可以用编程方式指定度量工厂。

### OSGi

Vert.x Core被打包成了OSGi Bundle，因此可以在任何OSGi R4.2+环境中使用，如`Apache Felix`或`Eclipse Equinox`，（这个）Bundle导出`io.vertx.core*`。

但是Bundle对Jackson和Netty有一些依赖，若部署Vert.x Core Bundle则需要：

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

### 'vertx`命令行

vertx命令用于和命令行中的Vert.x进行交互，主要用于运行Vert.x Verticle。为此，您需要下载并安装Vert.x发行版，并将安装目录中的`bin`添加到`PATH`环境变量中，还要确保您的`PATH`上有一个Java 8的JDK。

*注意：JDK需要支持Java代码的快速编译。*

#### 运行Verticle

您可以使用`vertx run`从命令行直接运行Vert.x的Verticle，以下是`run`命令的几个实例：

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

正如您在Java中可看到的，该Verticle的名称要么是Java完全限定类名，也可以指定Java源文件，Vert.x会为你编译它。

您可以用其他语言的前缀来指定Verticle的名称进行部署。例如：若Verticle是一个编译的Groovy类，您可以使用语言前缀`groovy:`，因此Vert.x知道它是一个Groovy类而不是Java类。

```
vertx run groovy:io.vertx.example.MyGroovyVerticle
```

`vertx run`命令可以使用几个可选参数，它们是：

* `-conf <config_file>`：提供了Verticle的一些配置，`config_file`是包含描述Verticle配置的JSON对象的文本文件的名称，该参数是可选的。
* `-cp <path>`：搜索Verticle和它使用的其他任何资源的路径，默认为`.`（当前目录）。若您的Verticle引用了其他脚本、类或其他资源（例如jar文件），请确保这些脚本、其他资源存在此路径上。该路径可以包含由以下内容分隔的多个路径条目：`:`（冒号）或`;`（分号）——这取决于操作系统。每个路径条目可以是包含脚本的目录的绝对路径或相对路径，也可以是`jar`或`zip`文件的绝对或相对文件名。一个示例路径可能是`-cp classes:lib/otherscripts:jars/myjar.jar:jars/otherjar.jar`。始终使用路径引用您的Verticle需要的任何资源，不要将它们放在系统类路径【System Classpath】上，因为这会导致部署的Verticle之间的隔离问题。
* `-instances <instances>`：要实例化的Verticle实例的数目，每个Verticle实例都是严格单线程（运行）的，以便在可用的核上扩展应用程序，您可能需要部署多个实例。若省略，则部署单个实例。
* `-worker`：此选项可确定一个Verticle是否为Worker Verticle。
* `-cluster`：此选项确定Vert.x实例是否尝试与网络上的其他Vert.x实例形成集群，集群Vert.x实例允许Vert.x与其他节点形成分布式Event Bus。默认为false（非集群模式）。
* `-cluster-port`：若指定了`cluster`选项，则可以确定哪个端口将用于与其他Vert.x实例进行集群通信。默认为0——这意味着“选择一个空闲的随机端口”。除非您帧需要绑定特定端口，您通常不需要指定此参数。
* `-cluster-host`：若指定了`cluster`选项，则可以确定哪个主机地址将用于其他Vert.x实例进行集群通信。默认情况下，它将尝试从可用的接口中选一个。若您有多个接口而您想要使用指定的一个，就在这里指定。
* `-ha`：若指定，该Verticle将部署为（支持）高可用性（HA）。有关详细信息，请参阅相关章节。
* `-quorum`：该参数需要和`-ha`一起使用，它指定集群中所有HA部署ID处于活动状态的最小节点数，默认为0。
* `-hagroup`：该参数需要和`-ha`一起使用，它指定此节点将加入的HA组。集群中可以有多个HA组，节点只会故障转移到同一组中的其他节点。默认为`__DEFAULT__`。

您还可以使用下边方式设置系统属性：`-Dkey=value`。

这里有更多的例子：

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

当使用Vert.x的高可用功能时，您可能需要创建一个Vert.x的裸实例。此实例在启动时未部署任何Verticle，但它若接收到若集群中的另一个节点死亡，则会创建一个新的裸实例，并且启动：

```
vertx bare
```

根据您的集群配置，您可能需要添加`cluster-host`和`cluster-port`参数。

#### 执行打包成fat-jar的Vert.x应用

一个fat-jar是一个包含了所有依赖项jar的可执行的jar，这意味着您不必在执行jar的机器上预先安装Vert.x。它像任何可执行的Java jar一样可直接执行：

```
java -jar my-application-fat.jar
```

对于这点，Vert.x没什么特别的，您可以使用任何Java应用程序。

您可以创建自己的主类并在MANIFEST中指定，单建议您将代码编写成Verticle，并使用Vert.x中的[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)类（`io.vertx.core.Launcher`）作为您的主类。这是在命令行中运行Vert.x使用的主类，因此允许您指定命令行参数，如`-instances`以便更轻松地扩展应用程序。

要将您的Verticle全部部署在这个`fatjar`中时，您必须将下边信息写入MANIFEST：

* `Main-Class`设置为`io.vertx.core.Launcher`
* `Main-Verticle`指定主要Verticle（Java完全限定类名或脚本文件名）

您还可以提供您将传递给`vertx run`的常用命令行参数：

```
java -jar my-verticle-fat.jar -cluster -conf myconf.json
java -jar my-verticle-fat.jar -cluster -conf myconf.json -cp path/to/dir/conf/cluster_xml
```

*注意：请参阅示例存Repository中的Maven/Gradle最简单的Maven/Gradle的Verticle示例了解如何构造fatjar的应用。*

一个fat jar默认会执行`run`命令。

#### 显示Vert.x的版本

若想显示Vert.x的版本，则：

```
vertx version
```

#### 其他命令

除了`run`和`version`以外，`vertx`命令行和`Launcher`还提供了其他命令：

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

由于`start`命令产生一个新的进程，传递给JVM的java选项不会被传播，所以您必须使用`java-opts`来配置JVM（`-X`，`-D`...）。若您使用`CLASSPATH`环境变量，请确保路径下包含所有需要的jar（vertx-core、您的jar和所有依赖项）。

该命令集是可扩展的，请参考[Extending the vert.x Launcher](http://vertx.io/docs/vertx-core/java/#_extending_the_vert_x_launcher)部分。

#### 实时重部署【Live Redeploy】

在开发时，可以方便在文件更改时实时重新部署应用程序。`vertx`命令行工具和更普通`Launcher`类提供了这个功能，这里有些例子：

```
vertx run MyVerticle.groovy --redeploy="**/*.groovy" --launcher-class=io.vertx.core.Launcher
vertx run MyVerticle.groovy --redeploy="**/*.groovy,**/*.rb"  --launcher-class=io.vertx.core.Launcher
java io.vertx.core.Launcher run org.acme.MyVerticle --redeploy="**/*.class"  --launcher-class=io.vertx.core
.Launcher -cp ...
```

重新部署过程如下执行。首先，您的应用程序作为后台应用程序启动（使用`start`命令）。当发现文件更改时，该进程将停止并重新启动该应用、这样可避免泄露。

要启用实时重新部署，请将`--redeploy`选项传递给`run`命令。`--redeploy`表示要监视的文件集，这个集合可使用`Ant`样式模式（使用**，*和?），您也可以使用逗号（,）分隔它们来指定多个集合。模式相当于当前工作目录。

传递给`run`命令的参数最终会传递给应用程序，可使用`--java-opts`配置JVM虚拟机选项。

`--launcher-class`选项确定应用程序的主类启动器。它通常是一个[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)，单您已使用了您自己的主类。

也可以在IDE中使用重部署功能：

* **Eclipse**：创建一个运行配置，使用`io.vertx.core.Launcher`类作为主类。在*Program Arguments*区域（参数选项卡中），写入`run your-verticle-fully-qualified-name --redeploy=**/*.java --launcher-class=io.vertx.core.Launcher`，您还可以添加其他参数。随着Eclipse在保存时会增量编译您的文件，重部署工作会顺利进行。
* **IntelliJ**：创建一个运行配置（应用）,将Main类设置为`io.vertx.core.Launcher`。在程序参数中写：`run your-verticle-fully-qualified-name --redeploy=**/*.class --launcher-class=io.vertx.core.Launcher`。要触发重新部署，您需要显示构造项目或模块（Build -> Make project）。

要调试应用程序，请将运行配置创建为远程应用程序，并使用`--java-opts`配置调试器。每次重新部署后，请勿忘记重新插入【`re-plug`】调试器，因为它每次都会创建一个新进程。

您还可以在重新部署周期中挂接构建过程：

```
java -jar target/my-fat-jar.jar --redeploy="**/*.java" --on-redeploy="mvn package"
java -jar build/libs/my-fat-jar.jar --redeploy="src/**/*.java" --on-redeploy='./gradlew shadowJar'
```

"on-redeploy"选项指定在应用程序关闭后和重新启动之前调用的命令。因此，如果更新某些运行时工作，则可以钩住构建工具。例如，您可以启动`gulp`或`grunt`来更新您的资源。

重新部署功能还支持以下设置：

* `redeploy-scan-period`：文件系统检查周期（以毫秒为单位），默认为250ms
* `redeploy-grace-period`：在2次重新部署之间等待的时间（以毫秒为单位），默认为1000ms
* `redeploy-termination-period`：停止应用程序后等待的时间（在启动用户命令之前）。这个在Windows上非常有用，因为这个进程并没立即被杀死。时间以毫秒为单位，默认20ms

### 集群管理器

在Vert.x中，集群管理器可用于各种功能，包括：

* 对集群中Vert.x节点发现和分组
* 维护集群范围中的主题订阅者列表（所以我们可知道哪些节点对哪个Event Bus地址感兴趣）
* 分布式Map的支持
* 分布式锁
* 分布式计数器

集群管理器不处理Event Bus节点之间的传输，这是由Vert.x直接通过TCP连接完成。

Vert.x发行版中使用的默认集群管理器是使用的[Hazelcast](http://hazelcast.com/)集群管理器，但是它可以轻松被替换成实现了Vert.x集群管理器接口的不同实现，因为Vert.x集群管理器可插拔的。

集群管理器必须实现[ClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)接口，Vert.x在运行时使用Java的服务加载器【[Service Loader](https://docs.oracle.com/javase/8/docs/api/java/util/ServiceLoader.html)】功能查找集群管理器，以便在类路径中查找[ClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/spi/cluster/ClusterManager.html)的实例。

若您在命令行中使用Vert.x并要使用集群，则应确保Vert.x安装的`lib`目录包含您的集群管理器jar。

若您在Maven/Gradle项目使用Vert.x，则只需将集群管理器jar作为项目依赖添加。

您也可以以编程的方式在嵌入Vert.x时使用[setClusterManager](http://vertx.io/docs/apidocs/io/vertx/core/VertxOptions.html#setClusterManager-io.vertx.core.spi.cluster.ClusterManager-)指定集群管理器。

### 日志记录

Vert.x使用内置的日志记录API进行日志记录，默认实现使用JDK（JUL）日志记录，因此不需要额外的依赖项。

#### 配置JUL日志记录

一个JUL日志记录配置文件可以使用普通的JUL方式指定——通过提供一个名为`java.util.logging.config.file`的系统属性、值为您的配置文件（位置）。有关此更多信息和JUL配置文件的结构，请参阅JUL日志记录文档。

Vert.x还提供了一种更方便的方式指定配置文件，而无需设置系统属性。您只需在您的类路径中提供名为`vertx-default-jul-logging.properties`的JUL配置文件（如，您的fatjar），Vert.x将使用该配置文件配置JUL。

#### 使用另一个日志框架

如果您不希望Vert.x使用JUL自己的日志记录，您可以为其配置另一个日志记录框架，例如Log4J或SLF4J。

为此，您应该设置一个名为`vertx.logger-delegate-factory-class-name`的系统属性，该名称是一个实现了[LogDelegateFactory](http://vertx.io/docs/apidocs/io/vertx/core/spi/logging/LogDelegateFactory.html)接口的Java类名称。我们为Log4J（版本1）、Log4J（版本2）和SLF4J提供了预构建的实现，类名（分别）为：`io.vertx.core.logging.Log4jLogDelegateFactory`，`io.vertx.core.logging.Log4j2LogDelegateFactory`和`io.vertx.core.logging.SLF4JLogDelegateFactory`。如果要使用这些实现，您还应确保相关的Log4J或SLF4J的jar位于您的类路径上。

请注意，提供的Log4J（版本1）代理不支持参数化消息。Log4J（版本2）的代理使用了像SLF4J代理这样的`{}`语法，JUL代理使用如`{x}`语法。

#### 应用中记录日志

Vert.x本身只是一个库，您可以在自己的应用程序使用任何日志库的API来记录日志。

但是，若您愿意，可以使用上述的Vert.x日志记录工具为应用程序提供日志记录。

为此，您可以使用[LoggerFactory](http://vertx.io/docs/apidocs/io/vertx/core/logging/LoggerFactory.html)获取一个[Logger](http://vertx.io/docs/apidocs/io/vertx/core/logging/Logger.html)实例用来记录日志：

```
Logger logger = LoggerFactory.getLogger(className);

logger.info("something happened");
logger.error("oops!", exception);
```

#### Netty日志记录

配置日志记录时，您还应该关心（如何）配置Netty日志记录。

Netty不依赖于外部日志配置（例如系统属性），而是根据Netty类可见的日志记录库来实现日志记录配置：

* 如果可见，先使用`SLF4J`库
* 否则若可见，再使用`Log4j`库
* 否则退回使用`java.util.logging`

通过直接在`io.netty.util.internal.logging.InternalLoggerFactory`上设置Netty的内部记录器实现，强制执行日志记录器实现。

```
// Force logging to Log4j
// 强制使用Log4j日志记录
InternalLoggerFactory.setDefaultFactory(Log4JLoggerFactory.INSTANCE);
```

#### 排除故障

**SLF4J启动警告**

若您在启动应用程序时看到以下信息：

```
SLF4J: Failed to load class "org.slf4j.impl.StaticLoggerBinder".
SLF4J: Defaulting to no-operation (NOP) logger implementation
SLF4J: See http://www.slf4j.org/codes.html#StaticLoggerBinder for further details.
```

这意味着您的类路径中有`SLF4J-API`却没有实际的绑定。使用`SLF4J`记录的消息将被删除，则您应该添加绑定到您的类路径。检查[https://www.slf4j.org/manual.html#swapping](https://www.slf4j.org/manual.html#swapping)选择绑定并进行配置。

请注意，Netty会寻找`SLF4-API`的jar，并在默认情况下使用它。

**对等连接重置**

若您的日志显示一堆：

```
io.vertx.core.net.impl.ConnectionBase
SEVERE: java.io.IOException: Connection reset by peer
```
这意味着客户端正在重置HTTP连接，而不是关闭它。此消息还表示您可能没有使用完整有效的内容（连接在您访问之前被切断）。

### 主机名解析

Vert.x使用地址解析器将主机名解析为IP地址，而不是JVM内置的阻塞解析器。

主机名使用一下方式解析为IP地址：

* 操作系统的hosts文件
* DNS查询服务器列表

默认情况下，它将使用环境中系统DNS服务器地址的列表，若该列表无法检索，将使用Google的公共DNS服务器"8.8.8.8"和"8.8.4.4"。

创建[Vertx](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html)实例时也可配置DNS服务器：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().
            addServer("192.168.0.1").
            addServer("192.168.0.2:40000"))
);
```

DNS服务器的默认端口为`53`，当服务器使用不同的端口时，可以使用冒号分隔符设置该端口：`192.168.0.2:40000`。

*注意：有时可能需要使用JVM内置解析器，JVM系统属性`-Dvertx.disableDnsResolver=true`激活该行为*

#### 故障转移【Failover】

当服务器没有及时回复时，尝试从列表中选择下一个解析器，搜索（数量）的限制由[setMaxQueries](http://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setMaxQueries-int-)设置（默认值是4个查询）

若解析器在[getQueryTimeout](http://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#getQueryTimeout--)毫秒内没有收到正确答案（默认为5秒），DNS查询被视为失败。

#### 服务器列表轮询

默认情况下，DNS服务器选择使用第一个，其余的服务器用于故障转移。

您可以配置[setRotateServers](http://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setRotateServers-boolean-)为true，让解析器使用轮询选择。它会在服务器之间传播查询负载并避免所有的查找都找到列表中的第一个服务器。

故障转移仍然适用，并将使用列表中的下一个服务器。

#### 主机映射

操作系统的hosts文件用于对ipaddress执行主机名查找。

可替换主机文件：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().
            setHostsPath("/path/to/hosts"))
);
```

#### 搜索域名

默认情况下，解析器将使用环境中的系统DNS搜索域，或者，可提供明确的显示搜索域列表：

```java
Vertx vertx = Vertx.vertx(new VertxOptions().
    setAddressResolverOptions(
        new AddressResolverOptions().addSearchDomain("foo.com").addSearchDomain("bar.com"))
);
```

当使用搜索域列表时，点数的阀值为1或从Linux上的`/etc/resolv.conf`加载，也可使用[setNdots](http://vertx.io/docs/apidocs/io/vertx/core/dns/AddressResolverOptions.html#setNdots-int-)配置特定值。

### 高可用/故障转移

Vert.x允许您运行支持高可用（HA——High Availability）的Verticle，这种情况下，当运行Verticle的Vert.x实例突然死亡时，该Veritlce将迁移到另一个Vert.x实例。这个Vert.x实例必须在同一个集群中。

#### 自动故障转移

当Vert.x启用HA运行时，若一个运行了Verticle的Vert.x实例失败或死亡，则Verticle将自动重新部署到集群中的另一个Vert.x实例中。我们称这个为Verticle故障转移。

若要启用HA运行Vert.x，需要添加`-ha`参数到命令行：

```
vertx run my-verticle.js -ha
```

现在开启了HA环境，在集群中需要多添加一个Vert.x实例，所以假设您已经有另一个已经启动的Vert.x实例，例如：

```
vertx run my-other-verticle.js -ha
```

如果运行了`my-verticle.js`的Vert.x实例现在死了（您可以通过执行`kill -9`杀死进程来测试），运行`my-other-verticle.js`的Vert.x实例将自动重新部署`my-verticle.js`，现在这个Vert.x实例正在运行两个Verticle。

*注意：只有当第二个Vert.x实例可访问verticle文件（这里是my-verticle.js）时，迁移才是可能的。*
*重要：请注意，干净地关闭Vert.x实例不会导致故障转移发生：**CTRL-C**或**kill -SIGNINT***

您也可以启动裸的Vert.x实例——即最初不运行任何Verticle的实例，它们也将为集群中的节点进行故障转移。要启动一个裸实例，您只需做：

```
vertx run -ha
```

当使用`-ha`开关时，您不需要提供`-cluster`开关，因为若要使用HA就假定是集群。

*注意：根据您的集群配置，可能需要自定义集群管理器配置（默认为Hazelcast）和/或添加集群主机cluster-host和集群端口cluster-port参数。*

#### HA组

当使用Vert.x运行实例时，还可以选择指定的HA组。HA组表示集群中的逻辑节点组。只有具有相同HA组的节点能执行故障转移。若不指定HA组，则使用默认组`__DEFAULT__`。

要指定一个HA组，您可以在运行该Verticle时使用`-hagroup`开关。

```
vertx run my-verticle.js -ha -hagroup my-group
```

我们来看一个例子：

在第一个终端运行：

```
vertx run my-verticle.js -ha -hagroup g1
```

在第二个终端中，让我们使用相同组运行另一个Verticle：

```
vertx run my-other-verticle.js -ha -hagroup g1
```

最后，在第三个终端中，使用不同组启动另一个Verticle：

```
vertx run yet-another-verticle.js -ha -hagroup g2
```

如果终端1中的实例被杀掉，则它将故障转移到终端2中的实例，而不是具有不同组的终端3中的实例。

若终端3中的实例被杀掉，因为这个组中没有其他Vert.x实例，则它不会故障转移。

#### 处理网络分区——Quora

高可用HA实现支持Quora，Quorum是分布式事务必须获得的最小票数才能被允许在分布式系统中执行操作的一个参数。

在启动Vert.x实例时，您可以指示它在部署任何HA部署之前需要一个`quorum`。该上下文环境中，一个quorum是集群中特定组的最小节点数。通常您选择quorum大小为`Q = 1 + N / 2`，其中N是组中节点数。若集群中的Q节点少于HA节点，HA部署将被撤销。如果/当quorum重新获取时，他们将重新部署。通过这样做您可以防止网络分区，a.k.a. *split brain*

这里有更多关于quora的[信息](http://en.wikipedia.org/wiki/Quorum_(distributed_computing)。

若要使用quorum运行Vert.x实例，您可以在命令行中指定`-quorum`，例如：

在第一个终端：

```
vertx run my-verticle.js -ha -quorum 3
```
此时，Vert.x实例将启动但不部署模块（尚未）因为目前集群中只有1个节点，而不是3个。

在第二个终端：

```
vertx run my-other-verticle.js -ha -quorum 3
```
此时，Vert.x实例将启动不部署模块（还），因为集群中只有两个节点，而不是3个。

在第三个控制台，您可以启动另一个Vert.x的实例：

```
vertx run yet-another-verticle.js -ha -quorum 3
```

妙极！——我们有三个节点，这是quorum设置的值，此时，模块将自动部署在所有实例上。

若我们现在关闭或杀死其中一个节点，那么这些模块将在其他节点上自动撤销，因为不再满足quorum（法定人数）。

Quora也可以与HA组合使用，在这种情况下，每个特定组会解决Quora。

### 安全注意事项

Vert.x是一个工具包，而不是一个舆论框架来强迫您以某种方式做事情。这会给开发人员巨大的力量，同时也承担了很大责任。

与任何工具包一样，可以编写不安全的应用程序，因此在开发应用时应特别注意（如通过互联网）。

#### Web应用

如果编写Web应用程序，强烈建议您直接使用Vert.x Web而不是Vert.x Core来提供资源和处理文件上传。

Vert.x Web对请求中的路径进行了规范，以防止恶意客户端伪造URL来访问Web根目录外部的资源。

类似地，对于文件上传Vert.x Web提供上传到磁盘上已知位置的功能，并且不依赖上传中客户端提供的文件名，可以将其上传到磁盘的其他位置。

Vert.x Core本身不提供这样的检查，因此作为一个开发人员自己实现它将取决于您。

#### 集群Event Bus流量

当在网络上的不同Vert.x节点之间创建集群Event Bus时，流量将通过未加密报文【Wire】发送，因此若您有要发送的机密数据，而您的Vert.x节点不在受信的网络上，则不要使用。

#### 标准安全最佳实践

任何服务都可能存在潜在的漏洞，无论是使用Vert.x还是任何其他工具包，因此始终遵循安全最佳实践，特别是当您的服务面向公众。

如，您应该始终在DMZ中运行它们，并使用具有受限权限的用户账户，以限制服务受到损害的程度。

### Vert.x命令行接口API

Vert.x Core提供了一个用于解析传递给程序命令行参数的API。

它还可以打印详细说明可用于命令行工具的选项帮助消息。即使这些功能远离Vert.x Core主题，该API也可在[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)类中使用，也可在fat-jar和`vertx`命令行工具中使用。另外，它是polyglot（可用于任何支持的语言），并在Vert.x Shell中使用。

Vert.x CLI提供了一个描述命令行界面的模型，也是一个解析器，这个解析器可支持不同的语法：

* 类似POSIX选项（即`tar -zxvf foo.tar.gz`）
* 类似GNU选项（即`du --human-readable --max-depth=1`）
* 类似Java属性（即`java -Djava.awt.headless=true -Djava.net.useSystemProxies=true Foo`）
* 具有附加值的短选项（即`gcc -O2 foo.c`）
* 单个连字符的长选项（即`ant -projecthelp`）

使用CLI API的三个步骤如下：

1. 命令行接口的定义
2. 解析用户命令行
3. 查询/询问

#### 定义阶段

每个命令行界面必须定义将要使用的选项和参数集合。它也需要一个名字。CLI API使用[Option](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html)和[Argument](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html)类来描述选项和参数：

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

您可以看到，您可以使用[CLI.create](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#create-java.lang.String-)创建一个新的[CLI](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)。传递的字符串是CLI的名称。创建后，您可以设置摘要和描述，摘要的目的是简短（一行），而描述可以包含更多细节。每个选项和参数也使用[addArgument](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#addArgument-io.vertx.core.cli.Argument-)和[addOption](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#addOption-io.vertx.core.cli.Option-)方法添加到CLI对象上。

**选项**

[Option](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html)是由用户命令行中存在的键【key】标识的命令行参数。选项至少必须有一个长名或一个短命，长名称通常使用`--`前缀，而短名称与单个`-`一起使用。选项可以获取用法中显示的描述（见下文）。选项可以接受0、1或几个值。接受0值的选项是一个标志，必须使用[setFlag](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setFlag-boolean-)声明。默认情况下，选项会接受一个值，但是您可以使用[setMultiValued](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setMultiValued-boolean-)配置该选项接收多个值：

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

选项可以标记为必填项，在用户命令行中未设置必填选项在解析阶段会引发异常：

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("mandatory")
        .setRequired(true)
        .setDescription("a mandatory option"));
```

非必填选项可以具有默认值，如果用户没有在命令行中设置该选项，即将使用该值：

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("optional")
        .setDefaultValue("hello")
        .setDescription("an optional option with a default value"));
```

可以使用[setHidden](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html#setHidden-boolean-)方法隐藏选项，隐藏选项不在用法中列出，但仍可在用户命令行中使用（对于高级用户）。

如果选项值与固定值相违背，则可以设置不同的可接受选项：

```java
CLI cli = CLI.create("some-name")
    .addOption(new Option()
        .setLongName("color")
        .setDefaultValue("green")
        .addChoice("blue").addChoice("red").addChoice("green")
        .setDescription("a color"));
```

也可以从JSON表单中实例化选项。

**参数**
和选项不同，参数不具有键【key】标识并由其索引标识。例如，在`java com.acme.Foo`中，`com.acme.Foo`是一个参数。

参数没有名称，使用基于0的索引进行标识，第一个参数的索引为0：

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

如果不设置参数索引，则基于声明顺序会自动计算。

```java
CLI cli = CLI.create("some-name")
    // will have the index 0
	// 索引为0
    .addArgument(new Argument()
        .setDescription("the first argument")
        .setArgName("arg1"))
    // will have the index 1
	// 索引为1
    .addArgument(new Argument()
        .setDescription("the second argument")
        .setArgName("arg2"));
```

`argName`是可选的，并在消息中使用。

相比选项，[Argument](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html)可以：

* 使用[setHidden](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setHidden-boolean-)隐藏
* 使用[setRequired](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setRequired-boolean-)设置必填
* 使用[setDefaultValue](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setDefaultValue-java.lang.String-)设置默认值
* 使用[setMultiValued](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html#setMultiValued-boolean-)设置接收多个值——只有最后一个参数可以是多值的。

参数也可以从JSON表单中实例化。

**生成Usage**

一旦您的[CLI](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)实例配置好后，您可以生成Usage信息：

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

上边生成的Usage信息如下：

```
Usage: copy [-R] source target

A command line interface to copy files.

  -R,--directory   enables directory support
```

若需要调整使用消息（的格式），请检查[UsageMessageFormatter](http://vertx.io/docs/apidocs/io/vertx/core/cli/UsageMessageFormatter.html)类

#### 解析阶段【1.Parsing】

一旦您的[CLI](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)实例配置好后，您可以解析用户命令行来评估【Evaluate】每个选项和参数：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
```

[parse](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#parse-java.util.List-)解析方法返回包含值的[CommandLine](http://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html)对象。默认情况下，它验证用户命令行，并检查每个必填选项和参数的设置以及每个选项接收的值的数量。您可以通过传递false作为[parse](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html#parse-java.util.List-)的第二个参数来禁用验证。如果要检查参数或选项，即使解析的命令行无效，这也是有用的。

您可以使用[isValid](http://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html#isValid--)来检查[CommandLine](http://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html)是否有效。

#### 查询/询问阶段【2.Query/Interrogation】

解析后，您可以从解析方法返回的[CommandLine](http://vertx.io/docs/apidocs/io/vertx/core/cli/CommandLine.html)对象中读取选项和参数的值：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
String opt = commandLine.getOptionValue("my-option");
boolean flag = commandLine.isFlagEnabled("my-flag");
String arg0 = commandLine.getArgumentValue(0);
```

您的一个选项可以被标记为“帮助”，如果用户命令行启用“帮助”选项，验证将不会失败，但是可以让您有机会检查用户是否需要帮助：

```java
CLI cli = CLI.create("test")
    .addOption(
        new Option().setLongName("help").setShortName("h").setFlag(true).setHelp(true))
    .addOption(
        new Option().setLongName("mandatory").setRequired(true));

CommandLine line = cli.parse(Collections.singletonList("-h"));

// The parsing does not fail and let you do:
// 解析不会失败，您可以做：
if (!line.isValid() && line.isAskingForHelp()) {
  StringBuilder builder = new StringBuilder();
  cli.usage(builder);
  stream.print(builder.toString());
}
```

#### 有类型选项和参数

描述[Option](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html)和[Argument](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html)类是无类型的，这意味着仅读取String值。

[TypedOption](http://vertx.io/docs/apidocs/io/vertx/core/cli/TypedOption.html)和[TypedArgument](http://vertx.io/docs/apidocs/io/vertx/core/cli/TypedArgument.html)可以指定一个类型，因此（String）原始值将转换为指定的类型。

在[CLI](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)定义中使用[TypedOption](http://vertx.io/docs/apidocs/io/vertx/core/cli/TypedOption.html)和[TypedArgument](http://vertx.io/docs/apidocs/io/vertx/core/cli/TypedArgument.html)，而不是[Option](http://vertx.io/docs/apidocs/io/vertx/core/cli/Option.html)和[Argument](http://vertx.io/docs/apidocs/io/vertx/core/cli/Argument.html)。

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

然后，您可以按下边方式获取转换的值：

```java
CommandLine commandLine = cli.parse(userCommandLineArguments);
boolean flag = commandLine.getOptionValue("R");
File source = commandLine.getArgumentValue("source");
File target = commandLine.getArgumentValue("target");
```

Vert.x CLI可以转换的类：

* 具有单个[String](http://vertx.io/docs/apidocs/java/lang/String.html)参数的构造函数，例如[File](http://vertx.io/docs/apidocs/java/io/File.html)或[JsonObject](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)
* 使用静态的`from`或`fromString`方法
* 使用静态`valueOf`方法，如基础类型和枚举

此外，您可以实现自己的转换器并指定CLI使用此转换器：

```java
CLI cli = CLI.create("some-name")
    .addOption(new TypedOption<Person>()
        .setType(Person.class)
        .setConverter(new PersonConverter())
        .setLongName("person"));
```

对于布尔值，布尔值将被计算为`true`:`on`，`yes`，`1`，`true`。

若您的一个选项是`enum`类型，则（系统）会自动计算一组选项。

#### 使用Annotation

您还可以使用Annotation定义CLI，定义使用类和setter方法上的Annotation来完成：

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

一旦被注解，您可以定义[CLI](http://vertx.io/docs/apidocs/io/vertx/core/cli/CLI.html)并使用以下命令注入值：

```java
CLI cli = CLI.create(AnnotatedCli.class);
CommandLine commandLine = cli.parse(userCommandLineArguments);
AnnotatedCli instance = new AnnotatedCli();
CLIConfigurator.inject(commandLine, instance);
```

### Vert.x启动器

Vert.x [Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)在fat-jar中作为主类，由`vertx`命令行实用程序调用，它可执行一组命令，如`run`, `bare`, `start`...

#### 扩展Vert.x启动器

您可以通过实现自己的[Command](http://vertx.io/docs/apidocs/io/vertx/core/spi/launcher/Command.html)来扩展命令集（仅限于Java）：

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

您还需要实现一个[CommandFactory](http://vertx.io/docs/apidocs/io/vertx/core/spi/launcher/CommandFactory.html)：

```java
public class HelloCommandFactory extends DefaultCommandFactory<HelloCommand> {
  public HelloCommandFactory() {
   super(HelloCommand.class);
  }
}
```

然后创建`src/main/resources/META-INF/services/io.vertx.core.spi.launcher.CommandFactory`并且添加一行表示工厂类的完全限定名称：

```
io.vertx.core.launcher.example.HelloCommandFactory
```

构建包含命令的jar，确保包含了SPI文件（`META-INF/services/io.vertx.core.spi.launcher.CommandFactory`）。

然后，将包含该命令的jar放入fat-jar（或包含在其中）的类路径中，或放在Vert.x发行版的`lib`目录中，您将可以执行：

```java
vertx hello vert.x
java -jar my-fat-jar.jar hello vert.x
```
#### 使用fat-jar中的启动器【Launcher】

要在fat-jar中使用[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)类，只需要将*MANIFEST*的`Main-Class`设置为`io.vertx.core.Launcher`。另外，将*MANIFEST*中`Main-Verticle`条目设置为您的主Verticle的名称。

默认情况下，它执行了`run`命令。但是，您可以通过设置*MANIFEST*的`Main-Command`条目来配置默认命令。若在没有命令的情况下启动fat-jar，则使用默认命令。

#### 子分类启动器

您还可以创建[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)来启动子应用程序的子类，这个类被设计成易于扩展的。

一个[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)子类可以：

* 在[beforeStartingVertx](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html#beforeStartingVertx-io.vertx.core.VertxOptions-)中自定义Vert.x配置
* 通过覆盖[afterStartingVertx](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html#afterStartingVertx-io.vertx.core.Vertx-)来读取由“run”或“bare”命令创建的Vert.x实例
* 使用[getMainVerticle](http://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#getMainVerticle--)和[getDefaultCommand](http://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#getDefaultCommand--)配置默认的Verticle和命令
* 使用[register](http://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#register-java.lang.Class-)和[unregister](http://vertx.io/docs/apidocs/io/vertx/core/impl/launcher/VertxCommandLauncher.html#unregister-java.lang.String-)添加/删除命令

#### 启动器和退出代码

当您使用[Launcher](http://vertx.io/docs/apidocs/io/vertx/core/Launcher.html)类作为主类时，它使用以下退出代码：

* 若进程顺利结束，或抛出未捕获的错误：`0`
* 用于通用错误：`1`
* 若Vert.x无法初始化：`11`
* 若生成的进程无法启动、发现或停止：`12`，该错误代码一般由`start`和`stop`命令使用
* 若系统配置不符合系统要求（如java命令找不到）：`14`
* 若主Verticle不能被部署：`15`

### 配置Vert.x缓存

当Vert.x需要从类路径中读取文件（嵌入在fat-jar中，类路径中jar文件或其他文件）时，将其复制到缓存目录。背后原因很简单：从jar或从输入流读取文件是阻塞的，所以为了避免每次都付出代价，Vert.x会将文件复制到其缓存目录中，并随后读取该文件。这个行为也可配置。

首先，默认情况下，Vert.x使用`$CWD/.vertx`作为缓存目录，它在此之间创建一个唯一的目录，以避免冲突。可以使用`vertx.cacheDirBase`系统属性配置该位置。如，若当前工作目录不可写（例如在不可变容器上下文环境中），请使用以下命令启动应用程序：

```java
vertx run my.Verticle -Dvertx.cacheDirBase=/tmp/vertx-cache
# 或者
java -jar my-fat.jar vertx.cacheDirBase=/tmp/vertx-cache
```

*重要：该目录必须是可写的。*

当您编辑资源（如HTML、CSS或JavaScript）时，这种缓存机制可能令人讨厌，因为它仅仅提供文件的第一个版本（因此，若您想重新加载页面，则不会看到您的编辑改变）。要避免此行为，请使用`-Dvertx.disableFileCaching=true`启动应用程序。使用此设置，Vert.x仍然使用缓存，单始终使用原始源刷新存储在缓存中的版本。因此，如果您编辑从类路径提供的文件并刷新浏览器，Vert.x会从类路径读取它，将其复制到缓存目录并从中提供。不要在生产环境使用这个设置，它很有可能影响性能。

最后，您可以使用`-Dvertx.disableFileCPResolving=true`完全禁用高速缓存，这个设置不是没有后果的。Vert.x将无法从类路径中读取任何文件（仅从文件系统）。使用此设置时要非常小心。

## 引用

1. Vert.x的扩展包是Vert.x的子项目集合，类似[Web](http://vertx.io/docs/#web)、[Web Client](http://vertx.io/docs/#web-client)、[Data Access](http://vertx.io/docs/#data_access)等。
2. 两种常用的项目构建工具。

## 注释

## 结语

