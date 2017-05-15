# FAQ

此处会列出常见的关于Vert.x 各个组件的常见问题以及相应的注意事项和解决方案。

### 问：怎样正确理解Vert.x机制？

答：Vert.x其实就是建立了一个Verticle内部的线程安全机制，让用户可以排除多线程并发冲突的干扰，专注于业务逻辑上的实现，用了Vert.x，您就不用操心多线程和并发的问题了。Verticle内部代码，除非声明Verticle是Worker Verticle，否则Verticle内部环境全部都是线程安全的，不会出现多个线程同时访问同一个Verticle内部代码的情况。

> 请注意：*一般情况下，用了Vert.x的Verticle之后，原则上synchronized，Lock，volatile，static对象，java.util.concurrent, HashTable, Vector, Thread, Runnable, Callable, Executor, Task, ExecutorService等这些并发和线程相关的东西就不再需要使用了，可以由Verticle全面接管，如果您不得不在Vert.x代码中使用上诉内容，则多少暗示着您的设计或者使用Vert.x的姿势出现了问题，建议再斟酌商榷一下。*

### 问：Verticle对象和处理器（Handler）是什么关系？Vert.x如何保证Verticle内部线程安全？

答：Verticle对象往往包含有一个或者多个处理器（Handler），在Java代码中，后者经常是以Lambda也就是匿名函数的形式出现，比如：

```java
vertx.createHttpServer().requestHandler(req->{
	//blablabla            
}).listen(8080);
```

这里的Lambda/匿名函数/req->{//blablabla}就是一个处理器（Handler），在随后的例子中，我们用1stHandler以及2ndHandler来指代具体的匿名函数，让代码更加清晰明了，放在Verticle中类似：

```java
public class MyVerticle extends AbstractVerticle {
    public void start() throws Exception {
        vertx.createHttpServer().requestHandler(1stHandler).listen(8080);
	vertx.createHttpServer().requestHandler(2ndHandler).listen(8081);
    }
}
```

Java中，Lambda本身也是一个对象，是一个@FunctionalInterface的对象，Verticle对象中包含了一个或者多个处理器（Handler）对象，比如上述例子中MyVerticle中就包含有两个handler。在Vert.x中，完成Verticle的部署之后，真正调用处理逻辑的入口往往是处理器（Handler），Vert.x保证同一个普通Verticle（也就是EventLoop Verticle，非Worker Verticle）内部的所有处理器（Handler）都只会由同一个EventLoop线程调用，由此保证Verticle内部的线程安全。所以我们可以放心地在Verticle内部声明各种线程不安全的属性变量，并在handler中分享他们，比如：

```java
public class MyVerticle extends AbstractVerticle {
    int i = 0;//属性变量
    
    public void start() throws Exception {
        vertx.createHttpServer().requestHandler(req->{
            i++;
	    req.response().end();//要关闭请求，否则连接很快会被占满
        }).listen(8080);

        vertx.createHttpServer().requestHandler(req->{
            System.out.println(i);
	    req.response().end(“”+i);
        }).listen(8081);
    }
}
```

访问 http://localhost:8080 就会使计数器加1，访问 http://localhost:8081 将会看到具体的计数。同理，也可以将i替换成HashMap等线程不安全对象，不需要使用ConcurrentHashMap或HashTable，可在Verticle内部安全使用。

### 问：什么是显著执行时间？什么是异步？如何正确理解文档中说的不要阻塞Eventloop？

答：初次接触Vert.x的开发人员往往会在异步，显著执行时间，阻塞等概念理解上遇到一定的困难，在此一并做个解释和说明。

*注意：我们会尽量将原理讲得通俗易懂，但还是要求读者具备有线程，进程，内存，CPU，操作系统，数据库，算法等专业基础知识。*

计算机本质上是计算的机器，任何一个指令的执行，都需要一定的时间予以完成，这个时间可能长可能短，而这些指令的执行，都会分配给一个具体的线程，并在线程中执行完成。在Vert.x中，黄金原则是不阻塞Eventloop线程，而Eventloop顾名思义，是一个事件的循环，可以认为是一个在Vert.x生命周期内，不停轮询事件是否发生，并将发生的事件交给Handler予以处理的无限调度循环线程。那么为了不停地检查事件是否发生，该线程需要在短时间内完成一个调度循环。如果Eventloop在完成一个调度循环的时间过长，就有可能导致新发生的事件得不到及时的处理，进而导致单次事件响应时间过长，影响客户体验。

为了在短时间内完成调度循环，就需要用户正确估算出，哪些程序代码会相对长时间地占用Eventloop线程的执行时间，然后将该部分代码的执行交由**其它线程**去处理。值得注意的是，这里所说的其它线程，可能是内核线程，也就是操作系统的线程，也有可能是用户线程，用户线程中包括了其它应用程序的线程，也就是其它进程中的线程，或者是我们用户自定义的线程。

一个简单粗暴的判断标准：任何涉及到IO操作的代码，都可以认为是可能造成阻塞的代码，**纯粹内存操作的代码，只要执行时间没有明显超长（例如执行循环几万次的处理便可认为是执行时间超长），都可以认为是非阻塞代码**。

我们来看一个简单的例子：某个程序要求，当前线程发送一个请求给网络上另外一个服务器，然后获取到结果之后作出相应的处理。那么此时有一个明显的IO操作，就是发送网络请求并等待对方返回结果。因为网络的速度要远远慢于内存处理的速度，所以此时的操作便是非纯粹内存操作，就有可能造成线程的阻塞，那么此时应该将这个操作交给其它线程予以处理，在处理完成之前，释放当前线程，等处理完成之后，再由当前线程执行回调函数。Vert.x自带的网络客户端（NetClient，HttpClient等）已经帮您包装好了这部分逻辑代码，直接使用NetClient等客户端，Vert.x就会将发送请求并等待返回结果这部分代码交给另外一个线程予以执行，此时这另外一个线程是内核线程，这部分的异步处理由JVM以及操作系统完成，您不需要自己定义一个线程并执行。类似的，数据库的处理同样涉及IO操作，所以Vert.x自带的JDBC客户端（JDBCClient）会帮您完成这部分的封装，您只需要直接调用JDBCClient的各种API便可完成操作，此时有可能是其它进程中的线程，比如PostgresQL会使用进程池来建立连接池，但是该线程亦不需要您去创建，Vert.x帮您完成了这些操作。类似的，硬盘上的操作，比如文件系统的API，也有可能造成阻塞，所以Vert.x的文件系统API提供了非阻塞API，但是值得说明的是，硬盘上的操作，如果只是少量操作，执行时间上也不会明显超长，所以Vert.x同时提供了硬盘操作的阻塞和非阻塞两种API。最后我们来看一个纯粹内存操作同时又是阻塞的例子。

假设我们拿到两个数万个节点的链表（LinkedList<String>），要求删除两个链表的交集，那么在没有任何算法优化的前提下，该操作的时间复杂度是O(n^2)，又因为内存中该链表节点数庞大，多达数万个节点，所以如果在Eventloop中执行该操作，将有可能使得执行时间超长，此时需要将这部分代码交由其它线程予以执行，Vert.x提供了除了Eventloop线程池以外的线程池，名曰Worker线程池。此时就需要用户自行将该部分代码包装成Worker线程执行的代码，并交给Worker线程予以执行，执行完成之后再由Eventloop线程执行回调函数处理其结果。*注意：Vert.x中将代码交给Worker线程执行的方式有两种，一种是通过executeBlocking函数包装，另外一种是写入Worker Verticle中。*

### 问：为什么Verticle之间传递的消息要求是immutable（不可变）的？

答：因为immutable（不可变的）的东西线程安全，可以被多个线程安全地并发访问，线程在使用的时候拷贝一份也不会有并发问题，Java里面字符串（String）对象是immutable（不可变）的，所以缺省情况下事件总线（eventbus）上传递的消息是字符串，vert.x也实现了事件总线上消息类型的编解码器，除了字符串以外，还支持少量的其它类型，比如原始数据类型及其包装类，比如字节流（byte[]），比如Json对象（JsonObject, JsonArray）还有Buffer，这些对象在传递过程中是不可变的。

Vert.x线程模型保证Verticle内部代码线程安全，同时要求在Verticle之间传递的消息是不可变的，通过此方法保证Verticle之间传递的消息也是线程安全的，从而进一步保证Vert.x内部整体是线程安全的，从而将开发人员从繁琐的，容易错的各种多线程并发问题中解脱出来。

### 问：我刚拿到一个第三方类库（lib/jars），怎么判断这个类库中的方法是异步还是同步的？有没有简单粗暴的方法可以一眼看出来？

答：严格说来，要认真看代码文档，比如JavaDoc，来判断方法是异步还是同步的，如果文档中没有明确说明，**则默认是同步的**，异步API往往出现在IO相关的方法中，所谓IO一般认为是涉及到硬盘（比如在硬盘上存取文件），网络（通过网络发送一个请求并获取相应结果），其它进程的操作（操作数据库）等，一般纯粹的，本进程内的内存操作并不被认为是涉及IO操作，这个时候将方法或API制作成异步的并无实际意义，所以一般非IO相关操作都被认为是同步的，同时并不会占用显著执行时间，所以不会阻塞线程。

> 请注意：*Vert.x中绝大多数涉及IO操作除非有明确说明，例如以Sync后缀命名的方法，否则均是异步的，另外在Vert.x中，EventBus的相关操作也被认为是涉及IO操作。*

有一个粗暴简单的判断同步还是异步的方法，就看给出的API或者方法中，是否有回调函数，如果有回调函数，且这个回调函数的参数是AsyncResult，则可以判断该API或方法是异步的。

### 问：Vert.x中各种Client该如何正确使用，用完是否需要关闭？

答：Vert.x中提供了各种预设客户端，例如HttpClient，JDBCClient，WebClient，MongoClient等，一般情况下，建议将客户端与Verticle对象绑定，一个Verticle对象内保留一个特定客户端的引用，并在start方法中将其实例化，这样Vert.x会在执行deployVerticle的时候执行start方法，实例化并保存该对象，在Verticle生命周期内，不需要频繁创建和关闭同类型的客户端，建议在Verticle的生命周期内对于特定领域，只创建一个客户端并复用该客户端，例如：

```java
import io.vertx.core.AbstractVerticle;
import io.vertx.core.http.HttpClient;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.jdbc.JDBCClient;
import io.vertx.ext.web.client.WebClient;

/**
 * Created by chengen on 21/04/2017.
 */
public class TheVerticle extends AbstractVerticle{
    //将客户端对象与Verticle对象绑定，这里选取了三种不同的客户端作为示范
    HttpClient httpClient;
    WebClient webClient;
    JDBCClient jdbcClient;

    public void start(){
      	//创建客户端
        httpClient = vertx.createHttpClient();

        webClient = WebClient.wrap(httpClient);

        JsonObject config = new JsonObject()
                .put("jdbcUrl", "...")
                .put("maximumPoolSize", 30)
                .put("username", "db user name")
                .put("password", "***")
                .put("provider_class", "...");

        jdbcClient = JDBCClient.createShared(vertx, config);
      
      	//using clients 
    }
}
```

每次使用客户端完之后，**无需**调用client.close();方法关闭客户端，频繁创建销毁客户端会在一定程度上消耗系统资源，降低性能，同时增加开发人员的负担，Vert.x提供客户端的目的就在于复用连接以减少资源消耗，提升性能，同时简化代码，减轻开发人员的负担。如您关闭客户端，在下一次使用该客户端的时候，需要重新创建客户端。

### 问：Vert.x中Future该如何正确使用，怎样规避回调地狱？

答：Future对象提供了一种异步结果的包装，用户可使用Future类中的setHandler方法来保存回调函数，然后在原先使用该回调函数的地方用completer方法予以填充，这样便可将回调函数从原参数中取出，以减少回调缩进，从而规避回调地狱，来看一个简单的例子：

```java
//未使用future时，回调函数嵌在send方法内部，以匿名函数的形式作为send的参数
vertx.eventBus().send("address","message", asyncResult->{
    System.out.println(asyncResult.result().body()); 
});
```

以上是未使用future时的代码，以下是使用future改造后的代码：

```java
Future<Message<String>> future = Future.future();
//将回调函数存入future中，从而实现代码的扁平化
future.setHandler(asyncResult -> {
    System.out.println(asyncResult.result().body());
});
//使用future之后，用completer方法填充参数
vertx.eventBus().send("address","message", future.completer());
```

一个复杂一点的例子：

```java
//以下程序先向address1发送一个message，然后等address1回复之后，将address1的回复消息发送给address2，最后将address2的回复打印到控制台上
vertx.eventBus().send("address1","message", asyncResult->{
    vertx.eventBus().send("address2", asyncResult.result().body(), asyncResult2->{
        System.out.println(asyncResult2.result().body());
    });
});
```

可以看到此时为了保证顺序结构，产生了两层缩进，回调金字塔开始形成，多次缩进之后便会出现所谓的回调地狱，这便是异步开发中为了保证顺序所可能会遇到的问题，那么我们可以通过以下方式解决代码过多缩进的问题：

```java
Future<Message<String>> future1 = Future.future();
Future<Message<String>> future2 = Future.future();

future1.setHandler(asyncResult -> {
   vertx.eventBus().send("address2", asyncResult.result().body(), future2);
});

future2.setHandler(asyncResult-> {
    System.out.println(asyncResult.result().body());
});

vertx.eventBus().send("address1","message", future1);
```

可以看到，使用了future之后，原先的两层缩进被抽取出来，变成了最多单层的缩进，从而使得代码可读性更强，更加美观。

值得注意的是，示范代码可能会抛出NullPointerException，因为当操作失败时，asyncResult.result()方法返回值为null，此时调用.body()方法会抛出空指针异常，在生产环境中正确写法应该是：

```
if(asyncResult.succeeded()){
    System.out.println(asyncResult.result().body());
}else if(asyncResult.failed()){
    System.out.println(asyncResult.cause());
}else{
    System.out.println("Why am I here?");
}
```

示范代码中为了解释future的用法简写了代码。

#### 使用 Future 来包装异步代码块 (3.4.0+)

早期版本的 Future，针对每一个异步的过程，需要在代码中声明中间变量 future 来对应异步回调，例如上文例子中的：

```java
Future<Message<String>> future1 = Future.future();
Future<Message<String>> future2 = Future.future();
```

在最新的版本中(3.4.0+)，可以通过 [Future.future](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#future-io.vertx.core.Handler-) 方法以函数式的方式直接用 Future 来包装一个异步的代码块，例如：

```java
Future.<Message<String>>future((future) ->
  vertx.eventBus().send("address1", "someMessage", future)
);
```

该方法的输入参数是一个 `Function`，该 `Function` 会以一个新的 `Future` 实例为参数被调用。由于 `Future` 自身实现了 `Handler<AsyncResult>`，因此你可以将它直接作为回调的 `Handler` 传入到异步方法里。该方法的返回值是提供给异步调用使用的 `Future` 实例。由此可以避免为嵌套的多个异步操作定义不同的 Future 变量，使代码更为简洁。

以下两种写法是等效的：

```java
vertx.eventBus().send("address","message", future.completer());
//或
vertx.eventBus().send("address","message", future);
```

复杂一点的例子：
```java
future.compose(message ->
  Future.<Message<String>>future(f ->
    vertx.eventBus().send("address", message.body(), f)
  )
);
//以上和以下两种写法是等效的
future.compose(message ->{
  Future<Message<String>> f = Future.future();
  vertx.eventBus().send("address",message.body(),f.completer());
  return f;
});
```

#### 使用组合来实现链式调用 (3.4.0+)

Future 接口提供了 `compose` 方法来链式地组合多个异步操作。在介绍这个方法的用途时，我们先考虑一个传统的同步操作和异常处理的方式。**假设**我们有一个同步的方法 `send` 会抛出一些异常（注意，以下代码只是同步代码的示例，和 vert.x 无关）：

```java
try {
	String msg1 = send('address1', 'message');
	String msg2 = send('address2', msg1);
	String msg3 = send('address3', msg2);
	//deal with result
} catch (Exception e) {
	//deal with exception
}
```

在这个例子里，如果任意一个 `send` 方法抛出异常，则会立即跳转到 catch 的代码块中。

下面我们将这个 `send` 方法通过 `compose` 替换为 vert.x 的异步版本，如下：

```java
Future.<Message<String>>future(f ->
  vertx.eventBus().send("address1", "message", f)
).compose((msg) ->
  Future.<Message<String>>future(f ->
    vertx.eventBus().send("address2", msg.body(), f)
  )
).compose((msg) ->
  Future.<Message<String>>future(f ->
    vertx.eventBus().send("address3", msg.body(), f)
  )
).setHandler((res) -> {
  if (res.failed()) {
    //deal with exception
    return;
  }
  //deal with the result
});
```

每一个 `compose` 方法需要传入一个 `Function`，`Function` 的输入是前一个异步过程的返回值（此处的返回值不是 AsyncResult，而是具体的返回内容），需要返回一个新的需要链接的 `Future`。 该 `Function`方法当且仅当前一个异步流程执行成功时才会被调用。

上述的例子描述了这样一个流程：

1. 首先通过 eventbus 发送消息 `message` 到 `address1`
2. 如果第一步成功，则发送第一步的消息的返回值到 `address2`
3. 如果第二步成功，则发送第二部的消息的返回值到 `address3`
4. **如果以上任何一步失败**，则不会继续执行下一个异步流程，直接执行最终的 Handler ，并且 `res.successed()` 为 `false`，可以通过 `res.cause()` 来获得异常对象
5. 如果以上三步全都成功，则同样执行 Handler，`res.successed()` 为 `true`，可以通过 `res.result()` 获取最后一步的结果。

通过 `compose` 方法来组织代码最大的价值在于可以 **让异步代码的执行顺序和代码的编写顺序看起来一致**，并在任何一步抛出异常时直接退出到最后一个 handler 来处理，**不需要针对每一个异步操作都编写异常处理的逻辑**。这对于编写复杂的异步流程时是非常有用的。

[Future.compose](http://vertx.io/docs/apidocs/io/vertx/core/Future.html#compose-java.util.function.Function-) 这个方法的行为现在非常接近于 JDK1.8 提供的 [CompletableFuture.thenCompose()](https://docs.oracle.com/javase/8/docs/api/java/util/concurrent/CompletableFuture.html#thenCompose-java.util.function.Function-)，也很接近于 EcmaScript6 的 [Promise API](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise) 的的接口约定，其实都是关于 Promise 模式的应用。关于更多 Promise 模式的信息还可以参考这里 [https://en.wikipedia.org/wiki/Futures_and_promises](https://en.wikipedia.org/wiki/Futures_and_promises)

### 问：Vert.x Web中如何将根路径对应到某个特定的html文件？

答：用reroute和staticHandler：

```java
router.route("/").handler(ctx->ctx.reroute("/static/index.html"));
```
### 问：我之前有过Spring，Akka，Node.js或Go的经验，请问Vert.x的概念有我熟悉的吗？

答：严格说来，不同框架和语言之间的概念无法一一对应，但如果我们不那么严格地去深究细节，Vert.x定义的概念可以从其它框架以及语言中找到一些痕迹，以下是Vert.x中定义概念跟其它框架和语言定义概念的比较，同一行中的概念可被认为是相似的：

|Vert.x|Akka|Spring|EJB|Node.js|Go|
|:---:|:---:|:---:|:---:|:---:|:---:|
|Standard Verticle|-|-|-|Reactor|-|
|Worker Verticle|-|-|Stateless Session Bean|-|-|
|Multiple Threaded<br>Worker Verticle|-|Bean(Singleton)|-|-|-|
|Handler|Actor|-|-|-|Goroutine|
