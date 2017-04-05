# Vert.x Rx

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 中英文对照表

* observable sequences：可观察序列
* Rxified：Rx化
* operator：操作符
* lift：变换
* flow：流
* read stream：可读流
* write stream：可写流
* observer：观察者
* subscriber：订阅者
* item：对象
* handler：（事件）处理器
* timer：定时器
* subscription(n.)：订阅
* unmarshall：重组

## Vert.x API for RxJava

[RxJava](https://github.com/ReactiveX/RxJava) 是 JVM 上一个流行的库，用于组合异步的、使用可观察序列的、基于事件的程序。

Vert.x 与 RxJava 集成起来很自然：它使得无论什么时候，只要我们能使用流和异步结果，就能使用 Observable。

要使用 Vert.x 的 RxJava API，有两种方式：

* 通过原始的 Vert.x API 辅以[ `RxHelper` ](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html)类，这个辅助类提供了用于 Vert.x Core API 和 RxJava API 之间互相转化的静态方法。
* 通过基于 Vert.x Core API 增强的Rx化的 Vert.x API。

### 可读流支持

RxJava 中 `Observable` 的概念和 Vert.x 中 `ReadStream` 类是一对完美的匹配：都提供了一个对象流。

静态方法[ `RxHelper.toObservable` ](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toObservable-io.vertx.core.streams.ReadStream-)用于将 Vert.x 可读流转换为 `rx.Observable`：

```java
FileSystem fileSystem = vertx.fileSystem();
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = RxHelper.toObservable(file);
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```

而 Rx化的 Vert.x API 在 [`ReadStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/streams/ReadStream.html) 类上提供了[ `toObservable` ](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/streams/ReadStream.html#toObservable--)方法：

```java
FileSystem fs = vertx.fileSystem();
fs.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = file.toObservable();
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```

这样的 `Observable` 是所谓 **hot** observable，即不管是否有订阅，它们都会产生通知。

> 译者注：在 RxJava 里，`Observable` 有“冷热”(cold/hot)之分。区别在于 cold observable 只有当订阅发生时，才会开始发射数据，而 hot observable 在创建完成后就会开始发射数据。更进一步的信息请查阅[ RxJava 关于 Observable 的文档](http://reactivex.io/documentation/observable.html)。

同样的，将一个 `Observable` 转变为 Vert.x `ReadStream` 也是可以的。

静态方法 [`RxHelper.toReadStream`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toReadStream-rx.Observable-) 用于将 `rx.Observable` 转换为 Vert.x 可读流：

```java
Observable<Buffer> observable = getObservable();
ReadStream<Buffer> readStream = RxHelper.toReadStream(observable);
Pump pump = Pump.pump(readStream, response);
pump.start();
```

### 处理器支持

[`RxHelper`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html) 类可以创建 [`ObservableHandler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableHandler.html) 对象。这是一个 `Observable` 对象，它的 [`toHandler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableHandler.html#toHandler--) 方法会返回 `Handler<T>` 接口的实现：

```java
ObservableHandler<Long> observable = RxHelper.observableHandler();
observable.subscribe(id -> {
  // Fired
});
vertx.setTimer(1000, observable.toHandler());
```

Rx化的 Vert.x API 未提供针对 `Handler` 的 API。

### 异步结果支持

以一个现有的 Vert.x `Handler<AsyncResult<T>>` 对象为基础，你可以创建一个 RxJava `Subscriber`，然后将其注册在 `Observable` 或 `Single` 上：

```java
observable.subscribe(RxHelper.toSubscriber(handler1));

// Subscribe to a Single
single.subscribe(RxHelper.toSubscriber(handler2));
```

在构造(construct)发生时，作为异步方法的最后一个参数的 Vert.x `Handler<AsyncResult<T>>` 可以被映射为单个元素的 `Observable`：

* 当回调成功时，观察者的 `onNext` 方法将被调用，参数就是这个对象；且其后 `onComplete` 方法会立即被调用。
* 当回调失败时，观察者的 `onError` 方法将被调用。

[`RxHelper.observableFuture`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#observableFuture--) 方法可以创建一个 [`ObservableFuture`](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableFuture.html) 对象。这这是一个 `Observable` 对象，它的 [`toHandler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableFuture.html#toHandler--) 方法会返回 `Handler<AsyncResult<T>>` 接口的实现：

```java
ObservableFuture<HttpServer> observable = RxHelper.observableFuture();
observable.subscribe(
    server -> {
      // Server is listening
    },
    failure -> {
      // Server could not start
    }
);
vertx.createHttpServer(new HttpServerOptions().
    setPort(1234).
    setHost("localhost")
).listen(observable.toHandler());
```

我们可以从 `ObservableFuture<Server>` 中获取单个 `HttpServer` 对象。如果端口监听失败，订阅者将会得到通知。

[`RxHelper.toHandler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toHandler-rx.Observer-) 方法为观察者（`Observer`）和事件处理器（`Handler`）做了适配：

```java
Observer<HttpServer> observer = new Observer<HttpServer>() {
  @Override
  public void onNext(HttpServer o) {
  }
  @Override
  public void onError(Throwable e) {
  }
  @Override
  public void onCompleted() {
  }
};
Handler<AsyncResult<HttpServer>> handler = RxHelper.toFuture(observer);
```

下面的代码也是可以的（直接基于`Action`）：

```java
Action1<HttpServer> onNext = httpServer -> {};
Action1<Throwable> onError = httpServer -> {};
Action0 onComplete = () -> {};

Handler<AsyncResult<HttpServer>> handler1 = RxHelper.toFuture(onNext);
Handler<AsyncResult<HttpServer>> handler2 = RxHelper.toFuture(onNext, onError);
Handler<AsyncResult<HttpServer>> handler3 = RxHelper.toFuture(onNext, onError, onComplete);
```

Rx化的 Vert.x API 复制了类似的每一个方法，并冠以 `rx` 的前缀，它们都返回 RxJava 的 `Single` 对象：

```java
Single<HttpServer> single = vertx
  .createHttpServer()
  .rxListen(1234, "localhost");

// 订阅绑定端口的事件
single.
    subscribe(
        server -> {
          // Server is listening
        },
        failure -> {
          // Server could not start
        }
    );
```

这样的 Single 是 **“冷的”(cold)**，对应的 API 方法将在注册时被调用。

> 注意：类似 `rx*` 的方法替换了以前版本中 `*Observable` 的方法，这样一个语义上的改变是为了与 RxJava 保持一致。

### 调度器支持

有时候 Reactive 扩展库需要执行一些可调度的操作，例如 `Observable#timer` 方法将创建一个能周期性发射事件的定时器并返回之。缺省情况下，这些可调度的操作由 RxJava 管理，这意味着定时器线程并非 Vert.x 线程，因此（这些操作）并不是在 Vert.x Event Loop 线程上执行的。

在 RxJava 中，有些操作通常会有接受一个 `rx.Scheduler` 参数的重载方法用于设定 `Scheduler`。`RxHelper` 类提供了一个 [`RxHelper.scheduler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#scheduler-io.vertx.core.Vertx-) 方法，其返回的调度器可供 RxJava 的这些方法使用。比如：

```java
Scheduler scheduler = RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.timer(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

对于阻塞型的可调度操作（blocking scheduled actions），我们可以通过 [`RxHelper.blockingScheduler`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#blockingScheduler-io.vertx.core.Vertx-) 方法获得适用的调度器：

```java
Scheduler scheduler = RxHelper.blockingScheduler(vertx);
Observable<Integer> obs = blockingObservable.observeOn(scheduler);
```

RxJava 也能被配置成使用 Vert.x 的调度器，这得益于 [`RxHelper.schedulerHook`](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#schedulerHook-io.vertx.core.Vertx-) 方法创建的调度器钩子对象。对于 IO 操作这里使用了阻塞型的调度器：

```java
RxJavaSchedulersHook hook = RxHelper.schedulerHook(vertx);
RxJavaHooks.setOnIOScheduler(f -> hook.getIOScheduler());
RxJavaHooks.setOnNewThreadScheduler(f -> hook.getNewThreadScheduler());
RxJavaHooks.setOnComputationScheduler(f -> hook.getComputationScheduler());
```

Rx化的 Vert.x API 在 [`RxHelper`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html) 类中也提供了相似的方法：

```java
Scheduler scheduler = io.vertx.rxjava.core.RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.interval(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

```java
RxJavaSchedulersHook hook = io.vertx.rxjava.core.RxHelper.schedulerHook(vertx);
RxJavaHooks.setOnIOScheduler(f -> hook.getIOScheduler());
RxJavaHooks.setOnNewThreadScheduler(f -> hook.getNewThreadScheduler());
RxJavaHooks.setOnComputationScheduler(f -> hook.getComputationScheduler());
```

基于一个命名的工作线程池（named worker pool）创建调度器也是可以的，如果你想为了调度阻塞操作复用特定的线程池，这将会很有帮助：

```java
Scheduler scheduler = io.vertx.rxjava.core.RxHelper.scheduler(workerExecutor);
Observable<Long> timer = Observable.interval(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

### JSON解码

[RxHelper.unmarshaller](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#unmarshaller-java.lang.Class-) 方法创建了一个 `rx.Observable.Operator` 对象，这个操作符的作用是将 `Observable<Buffer>` 变换为对象的 Observable：

```java
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = RxHelper.toObservable(file);
  observable.lift(RxHelper.unmarshaller(MyPojo.class)).subscribe(
      mypojo -> {
        // 处理对象
      }
  );
});
```

Rx化的辅助类也能做同样的事：

```java
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = file.toObservable();
  observable.lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class)).subscribe(
      mypojo -> {
        // 处理对象
      }
  );
});
```

### 部署Verticle

Rx化的 API 不能部署一个已经存在的 Verticle 实例。[RxHelper.observableFuture](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#observableFuture--) 方法为此提供了一个解决方案。

所有工作都在 [RxHelper.deployVerticle](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#deployVerticle-io.vertx.rxjava.core.Vertx-io.vertx.core.Verticle-) 方法里自动完成，它会部署一个 `Verticle ` 并返回包含部署 ID 的 `Observable<String>`。

```java
Observable<String> deployment = RxHelper.deployVerticle(vertx, verticle);

deployment.subscribe(id -> {
  // 部署成功
}, err -> {
  // 部署失败
});
```

### HttpClient GET on subscription

对于通过订阅执行一个 HTTP GET 请求这样的需求，利用 [RxHelper.get](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#get-io.vertx.rxjava.core.http.HttpClient-int-java.lang.String-java.lang.String-) 方法是很适合的：

```java
Observable<HttpClientResponse> get = RxHelper.get(client, "http://the-server");

// 执行请求
get.subscribe(resp -> {
  // 获得响应
}, err -> {
  // 出错了
});
```

> 警告：执行 GET 请求时，这个 API 和 `HttpClient` 是不同的。当方法被调用时，它会返回一次性的 `Observable`。

## Rxified API

Rx化的 API 是 Vert.x API 的一个代码自动生成版本，就像 Vert.x 的 *JavaScript* 或 *Groovy* 版本一样。这个 API 以 `io.vertx.rxjava` 为包名前缀，例如 `io.vertx.core.Vertx` 类被转化为 [·io.vertx.rxjava.core.Vertx·](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html)类。

### Embedding Rxfified Vert.x

只需使用 [Vertx.vertx](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html#vertx--) 方法：

```java
Vertx vertx = io.vertx.rxjava.core.Vertx.vertx();
```

### 作为Verticle

通过继承 [`AbstractVerticle`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/AbstractVerticle.html) 类，它会做一些包装（你将获得一个 RxJava Verticle）：

```java
class MyVerticle extends io.vertx.rxjava.core.AbstractVerticle {
  public void start() {
    // Use Rxified Vertx here
  }
}
```

部署一个 RxJava Verticle 不需要特别的部署器，使用 Java 部署器即可。

### API 样例

让我们通过研究一些样例来了解相关 API 吧。

#### Event Bus 消息流

很自然地，[`MessageConsumer`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/eventbus/MessageConsumer.html) 类提供了相关的 `Observable<Message<T>>`：

```java
EventBus eb = vertx.eventBus();
MessageConsumer<String> consumer = eb.<String>consumer("the-address");
Observable<Message<String>> observable = consumer.toObservable();
Subscription sub = observable.subscribe(msg -> {
  // 获得消息
});

// 10秒后注销
vertx.setTimer(10000, id -> {
  sub.unsubscribe();
});
```

[MessageConsumer](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/eventbus/MessageConsumer.html) 类提供了 [Message](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/eventbus/Message.html) 的流，如果需要，还可以通过 [body](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/eventbus/Message.html#body--) 方法获得消息体组成的新流：

```java
EventBus eb = vertx.eventBus();
MessageConsumer<String> consumer = eb.<String>consumer("the-address");
Observable<String> observable = consumer.bodyStream().toObservable();
```

RxJava 的 map/reduce 组合风格在这里是相当有用的：

```java
Observable<Double> observable = vertx.eventBus().
    <Double>consumer("heat-sensor").
    bodyStream().
    toObservable();

observable.
    buffer(1, TimeUnit.SECONDS).
    map(samples -> samples.
        stream().
        collect(Collectors.averagingDouble(d -> d))).
    subscribe(heat -> {
      vertx.eventBus().send("news-feed", "Current heat is " + heat);
    });
```

#### 定时器

定时器任务可以通过 [`timerStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html#timerStream-long-) 方法来创建：

```java
vertx.timerStream(1000).
    toObservable().
    subscribe(
        id -> {
          System.out.println("Callback after 1 second");
        }
    );
```

周期性的任务可以通过 [`periodicStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html#periodicStream-long-) 方法来创建：

```java
vertx.periodicStream(1000).
    toObservable().
    subscribe(
        id -> {
          System.out.println("Callback every second");
        }
    );
```

通过注销操作可以取消对 Observable 的订阅：

```
vertx.periodicStream(1000).
    toObservable().
    subscribe(new Subscriber<Long>() {
      public void onNext(Long aLong) {
        // 回调
        unsubscribe();
      }
      public void onError(Throwable e) {}
      public void onCompleted() {}
    });
```

#### HTTP客户端请求

[`toObservable`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/HttpClientRequest.html#toObservable--) 方法提供了带有 [`HttpClientResponse`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClientResponse.html) 对象的一次性回调，请求错误同样会在 Observable 中反映处理。

```java
HttpClient client = vertx.createHttpClient(new HttpClientOptions());
HttpClientRequest request = client.request(HttpMethod.GET, 8080, "localhost", "/the_uri");
request.toObservable().subscribe(
    response -> {
      // 处理响应
    },
    error -> {
      // 无法连接
    }
);
request.end();
```

通过 [`toObservable`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/HttpClientResponse.html#toObservable--) 方法可以将响应当成 `Observable<Buffer>` 来处理：

```java
request.toObservable().
    subscribe(
        response -> {
          Observable<Buffer> observable = response.toObservable();
          observable.forEach(
              buffer -> {
                // 处理 buffer
              }
          );
        }
    );
```

`flatMap ` 操作也能获得同样的流：

```java
request.toObservable().
    flatMap(HttpClientResponse::toObservable).
    forEach(
        buffer -> {
          // Process buffer
        }
    );
```

通过静态方法 [`RxHelper.unmarshaller`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#unmarshaller-java.lang.Class-) ，我们也能将 `Observable<Buffer>` 重组为对象。这个方法创建了一个 `Rx.Observable.Operator`（Rx 操作符）供重组操作使用：

```java
request.toObservable().
    flatMap(HttpClientResponse::toObservable).
    lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class)).
    forEach(
        pojo -> {
          // Process pojo
        }
    );
```

#### HTTP服务端请求

[requestStream](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/HttpServer.html#requestStream--) 方法对到达的每个请求都提供了回调：

```java
Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  // 处理请求
});
```

[`HttpServerRequest`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpServerRequest.html) 可以被适配为 `Observable<Buffer>`：

```java
Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  Observable<Buffer> observable = request.toObservable();
});
```

[`RxHelper.unmarshaller`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#unmarshaller-java.lang.Class-) 方法可以用来解析 JSON 格式的请求并将其映射为对象：

```java
Observable<HttpServerRequest> requestObservable = server.requestStream().toObservable();
requestObservable.subscribe(request -> {
  Observable<MyPojo> observable = request.
      toObservable().
      lift(io.vertx.rxjava.core.RxHelper.unmarshaller(MyPojo.class));
});
```

#### WebSocket客户端
当 WebSocket 连接上或失败时，[`websocketStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/HttpClient.html#websocketStream-io.vertx.core.http.RequestOptions-) 方法对此提供了一次性的回调：

```java
HttpClient client = vertx.createHttpClient(new HttpClientOptions());
client.websocketStream(8080, "localhost", "/the_uri").toObservable().subscribe(
    ws -> {
      // Use the websocket
    },
    error -> {
      // Could not connect
    }
);
```

[WebSocket](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/WebSocket.html) 对象可以轻松地转换为 `Observable<Buffer>`：

```java
socketObservable.subscribe(
    socket -> {
      Observable<Buffer> dataObs = socket.toObservable();
      dataObs.subscribe(buffer -> {
        System.out.println("Got message " + buffer.toString("UTF-8"));
      });
    }
);
```

#### WebSocket服务端

每当有新连接到达时，[`websocketStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/http/HttpServer.html#websocketStream--) 方法都会提供一次回调：

```java
Observable<ServerWebSocket> socketObservable = server.websocketStream().toObservable();
socketObservable.subscribe(
    socket -> System.out.println("Web socket connect"),
    failure -> System.out.println("Should never be called"),
    () -> {
      System.out.println("Subscription ended or server closed");
    }
);
```

[`ServerWebSocket`](http://vertx.io/docs/apidocs/io/vertx/core/http/ServerWebSocket.html) 对象可以轻松地转换为 `Observable<Buffer>`：

```java
socketObservable.subscribe(
    socket -> {
      Observable<Buffer> dataObs = socket.toObservable();
      dataObs.subscribe(buffer -> {
        System.out.println("Got message " + buffer.toString("UTF-8"));
      });
    }
);
```

---

> [原文档](http://vertx.io/docs/vertx-rx/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-rx/java/
[2]: https://github.com/vert-x3/vertx-rx
[3]: https://github.com/vert-x3/vertx-examples/tree/master/rx-examples
