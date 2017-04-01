# Vert.x Rx
[源文档](http://vertx.io/docs/vertx-rx/java/#_vert_x_api_for_rxjava)

## 中英文对照表
* Rxified：Rx化
* flow：流
* read stream：可读流
* write stream：可写流
* observer：观察者
* subscriber：订阅者
* item：对象
* handler：（事件）处理器
* timer：定时器
* subscription(n.)：订阅

## Vert.x API for RxJava

[RxJava](https://github.com/ReactiveX/RxJava)是 JVM 上一个流行的

Vert.x 与 RxJava 的集成很自然，它使得我们无论什么时候，只要能使用流和异步结果，就能使用 observable 。

要使用 Vert.x 的 RxJava API，有两种方式：
* 通过原始的 Vert.x API 辅以[ RxHelper ](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html)类，这个辅助类提供了用于 Vert.x core API 和 RxJava API 之间互相转化的静态方法。
* 通过基于 Vert.x core API 增强的 Rx化的 Vert.x API。

### Read stream support
RxJava 中 `Observable` 的概念和 Vert.x 中 `ReadStream` 类是一对完美的匹配：都提供了一个对象流。

静态方法[ RxHelper.toObservable ](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toObservable-io.vertx.core.streams.ReadStream-)用于将 Vert.x 可读流转换为 `rx.Observable`：
```
FileSystem fileSystem = vertx.fileSystem();
fileSystem.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = RxHelper.toObservable(file);
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```

而 Rx化的 Vert.x API 在 [ReadStream](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/streams/ReadStream.html) 类上提供了[ toObservable ](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/streams/ReadStream.html#toObservable--)方法：
```
FileSystem fs = vertx.fileSystem();
fs.open("/data.txt", new OpenOptions(), result -> {
  AsyncFile file = result.result();
  Observable<Buffer> observable = file.toObservable();
  observable.forEach(data -> System.out.println("Read data: " + data.toString("UTF-8")));
});
```

这样的 observables 是所谓**热的** observables，即不管是否有订阅，它们都会产生通知^(1)^。

同样的，将一个 `Observable` 转变为 Vert.x `ReadStream` 也是可以的。

静态方法 [RxHelper.toReadStream](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toReadStream-rx.Observable-) 用于将 `rx.Observable` 转换为 Vert.x 可读流：
```
Observable<Buffer> observable = getObservable();
ReadStream<Buffer> readStream = RxHelper.toReadStream(observable);
Pump pump = Pump.pump(readStream, response);
pump.start();
```

### Handler support
[RxHelper](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html) 类可以创建 [ObservableHandler](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableHandler.html) 对象：这是一个 `Observable` 对象，它的 [toHandler](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableHandler.html#toHandler--) 方法会返回 `Handler<T>` 接口的实现：
```
ObservableHandler<Long> observable = RxHelper.observableHandler();
observable.subscribe(id -> {
  // Fired
});
vertx.setTimer(1000, observable.toHandler());
```

Rx化的 Vert.x API 未提供针对 handler 的 API。

### Async result support
以一个现有的 Vert.x `Handler<AsyncResult<T>>` 对象为基础，你可以创建一个 RxJava `Subscriber` ，然后将其注册在 `Observable` 或 `Single` 上：
```
observable.subscribe(RxHelper.toSubscriber(handler1));

// Subscribe to a Single
single.subscribe(RxHelper.toSubscriber(handler2));
```

在构建发生时，作为异步方法的最后一个参数的 Vert.x `Handler<AsyncResult<T>>` 可以被映射为单个元素的 observable：
* 当回调成功时，观察者的 `onNext` 方法将被调用，参数就是这个对象；且其后 `onComplete` 方法会立即被调用。
* 当回调失败时，观察者的 `onError` 方法将被调用。

方法 [RxHelper.observableFuture](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#observableFuture--) 创建了一个 [ObservableFuture](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableFuture.html) 对象，这这是一个 `Observable` 对象，它的 [toHandler](http://vertx.io/docs/apidocs/io/vertx/rx/java/ObservableFuture.html#toHandler--) 方法会返回 `Handler<AsyncResult<T>>` 接口的实现：

```
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

`ObservableFuture<Server>` 将会获得一个 `HttpServer` 对象；如果监听失败，订阅者将会得到通知。

方法 [RxHelper.toHandler](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#toHandler-rx.Observer-) 为观察者（`Observer`）和事件处理器（`Handler`）做了适配：
```
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

下面的代码也是可以的：
```
Action1<HttpServer> onNext = httpServer -> {};
Action1<Throwable> onError = httpServer -> {};
Action0 onComplete = () -> {};

Handler<AsyncResult<HttpServer>> handler1 = RxHelper.toFuture(onNext);
Handler<AsyncResult<HttpServer>> handler2 = RxHelper.toFuture(onNext, onError);
Handler<AsyncResult<HttpServer>> handler3 = RxHelper.toFuture(onNext, onError, onComplete);
```

Rx化的 Vert.x API 复制了类似的每一个方法，并冠以 `rx` 的前缀，它们都返回 RxJava 的 `Single` 对象：
```
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

这样的 Single 是**“冷的”**，对应的 API 方法将在注册时被调用。
> 注意：类似 `rx*` 的方法替换了以前版本中 `*Observable` 的方法，这样一个语义上的改变是为了与 RxJava 保持一致。

### Scheduler support
reactive 扩展有时候需要调度操作，例如 `Observable#timer` 方法将创建一个能周期性发射事件的定时器并返回之。缺省情况下，被调度的操作由 RxJava 管理，这意味着定时器的线程并不是 Vert.x 的线程，因此并不是在 Vert.x event loop 线程上执行的。

当 RxJava 的方法与需要调度器交互时，这是一个需要额外的 `rx.Scheduler` 参数的重载方法，[RxHelper.scheduler](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#scheduler-io.vertx.core.Vertx-) 方法返回的调度器能在这里派上用场。
```
Scheduler scheduler = RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.timer(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

对于阻塞型的被调度操作，可以通过 [RxHelper.blockingScheduler](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#blockingScheduler-io.vertx.core.Vertx-) 方法获得适用的调度器：
```
Scheduler scheduler = RxHelper.blockingScheduler(vertx);
Observable<Integer> obs = blockingObservable.observeOn(scheduler);
```

RxJava 也能被配置成使用 Vert.x 的调度器，得益于 [RxHelper.schedulerHook](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#schedulerHook-io.vertx.core.Vertx-) 创建的调度器钩子对象，对于 IO 操作这里使用了阻塞型的调度器：
```
RxJavaSchedulersHook hook = RxHelper.schedulerHook(vertx);
RxJavaHooks.setOnIOScheduler(f -> hook.getIOScheduler());
RxJavaHooks.setOnNewThreadScheduler(f -> hook.getNewThreadScheduler());
RxJavaHooks.setOnComputationScheduler(f -> hook.getComputationScheduler());
```

Rx化的 Vert.x API 在 [RxHelper](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html) 类中也提供了相似的方法：
```
Scheduler scheduler = io.vertx.rxjava.core.RxHelper.scheduler(vertx);
Observable<Long> timer = Observable.interval(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

```
RxJavaSchedulersHook hook = io.vertx.rxjava.core.RxHelper.schedulerHook(vertx);
RxJavaHooks.setOnIOScheduler(f -> hook.getIOScheduler());
RxJavaHooks.setOnNewThreadScheduler(f -> hook.getNewThreadScheduler());
RxJavaHooks.setOnComputationScheduler(f -> hook.getComputationScheduler());
```

基于一个命名的工作线程池（named worker pool）创建调度器也是可以的，如果你想为了调度阻塞操作复用特定的线程池，这将会很有帮助：
```
Scheduler scheduler = io.vertx.rxjava.core.RxHelper.scheduler(workerExecutor);
Observable<Long> timer = Observable.interval(100, 100, TimeUnit.MILLISECONDS, scheduler);
```

### Json unmarshalling
方法 [RxHelper.unmarshaller](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#unmarshaller-java.lang.Class-) 创建了一个 `rx.Observable.Operator` 对象，这个操作符的作用是将 json 格式的 `Observable<Buffer>` 转换为对象的 observable：
```
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
```
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

### Deploying a Verticle
Rx化的 API 不能部署一个已经存在的 Verticle 实例，辅助类的 [RxHelper.observableFuture](http://vertx.io/docs/apidocs/io/vertx/rx/java/RxHelper.html#observableFuture--) 方法为此提供了一个解决方案。

工作在 [RxHelper.deployVerticle](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#deployVerticle-io.vertx.rxjava.core.Vertx-io.vertx.core.Verticle-) 方法里完成，它部署一个 `Verticle ` 并返回部署 ID 的 `Observable<String>`。
```
Observable<String> deployment = RxHelper.deployVerticle(vertx, verticle);

deployment.subscribe(id -> {
  // 部署成功
}, err -> {
  // 部署失败
});
```

### HttpClient GET on subscription
对于通过订阅执行一个 HTTP GET 请求这样的需求，[RxHelper.get](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/RxHelper.html#get-io.vertx.rxjava.core.http.HttpClient-int-java.lang.String-java.lang.String-) 是很适合的：
```
Observable<HttpClientResponse> get = RxHelper.get(client, "http://the-server");

// 执行请求
get.subscribe(resp -> {
  // 获得响应
}, err -> {
  // 出错了
});
```

> 警告：执行 GET 请求时，这个 API 和 HttpClient 是不同的，当方法被调用时，它会返回一次性的 `Observable`。

## Rxified API

Rx化的 API 是 Vert.x API 的一个代码自动生成版本，就像 Vert.x 的 *JavaScript* 或 *Groovy* 版一样。这个 API 以 `io.vertx.rxjava` 为包名前缀，例如 `io.vertx.core.Vertx` 类被转化为 [Vertx](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html)类。

### Embedding Rxfified Vert.x
只需使用 [Vertx.vertx](http://vertx.io/docs/apidocs/io/vertx/rxjava/core/Vertx.html#vertx--) 方法：
```
Vertx vertx = io.vertx.rxjava.core.Vertx.vertx();
```

### As a Verticle



## 译者注
1. RxJava 里，`observable` 有“冷热”之分。区别在于冷的 `observable` 只有当订阅发生时，才会开始发射数据，而热的 `observable` 在创建完成后就会开始发射数据。更进一步的信息请查阅[ RxJava 关于 observable 的文档](http://reactivex.io/documentation/observable.html)。
