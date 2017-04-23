# FAQ

此处会列出常见的关于Vert.x 各个组件的常见问题以及相应的注意事项和解决方案。

### 问：什么是显著执行时间？什么是异步？如何正确理解文档中说的不要阻塞Eventloop？

答：初次接触Vert.x的开发人员往往会在异步，显著执行时间，阻塞等概念理解上遇到一定的困难，在此一并做个解释和说明。

*注意：我们会尽量将原理讲得通俗易懂，但还是要求读者具备有线程，进程，内存，CPU，操作系统，数据库，算法等专业基础知识。*

计算机本质上是计算的机器，任何一个指令的执行，都需要一定的时间予以完成，这个时间可能长可能短，而这些指令的执行，都会分配给一个具体的线程，并在线程中执行完成。在Vert.x中，黄金原则是不阻塞Eventloop线程，而Eventloop顾名思义，是一个事件的循环，可以认为是一个在Vert.x生命周期内，不停轮询事件是否发生，并将发生的事件交给Handler予以处理的无限调度循环线程。那么为了不停地检查事件是否发生，该线程需要在短时间内完成一个调度循环。如果Eventloop在完成一个调度循环的时间过长，就有可能导致新发生的事件得不到及时的处理，进而导致单次事件响应时间过长，影响客户体验。

为了在短时间内完成调度循环，就需要用户正确估算出，哪些程序代码会相对长时间地占用Eventloop线程的执行时间，然后将该部分代码的执行交由**其它线程**去处理。值得注意的是，这里所说的其它线程，可能是内核线程，也就是操作系统的线程，也有可能是用户线程，用户线程中包括了其它应用程序的线程，也就是其它进程中的线程，或者是我们用户自定义的线程。

一个简单粗暴的判断标准：任何涉及到IO操作的代码，都可以认为是可能造成阻塞的代码，**纯粹内存操作的代码，只要执行时间没有明显超长（例如执行循环几万次的处理便可认为是执行时间超长），都可以认为是非阻塞代码**。

我们来看一个简单的例子：某个程序要求，当前线程发送一个请求给网络上另外一个服务器，然后获取到结果之后作出相应的处理。那么此时有一个明显的IO操作，就是发送网络请求并等待对方返回结果。因为网络的速度要远远慢于内存处理的速度，所以此时的操作便是非纯粹内存操作，就有可能造成线程的阻塞，那么此时应该将这个操作交给其它线程予以处理，在处理完成之前，释放当前线程，等处理完成之后，再由当前线程执行回调函数。Vert.x自带的网络客户端（NetClient，HttpClient等）已经帮您包装好了这部分逻辑代码，直接使用NetClient等客户端，Vert.x就会将发送请求并等待返回结果这部分代码交给另外一个线程予以执行，此时这另外一个线程是内核线程，这部分的异步处理由JVM以及操作系统完成，您不需要自己定义一个线程并执行。类似的，数据库的处理同样涉及IO操作，所以Vert.x自带的JDBC客户端（JDBCClient）会帮您完成这部分的封装，您只需要直接调用JDBCClient的各种API便可完成操作，此时有可能是其它进程中的线程，比如PostgresQL会使用进程池来建立连接池，但是该线程亦不需要您去创建，Vert.x帮您完成了这些操作。类似的，硬盘上的操作，比如文件系统的API，也有可能造成阻塞，所以Vert.x的文件系统API提供了非阻塞API，但是值得说明的是，硬盘上的操作，如果只是少量操作，执行时间上也不会明显超长，所以Vert.x同时提供了硬盘操作的阻塞和非阻塞两种API。最后我们来看一个纯粹内存操作同时又是阻塞的例子。

假设我们拿到两个数万个节点的链表（LinkedList<String>），要求删除两个链表的交集，那么在没有任何算法优化的前提下，该操作的时间复杂度是O(n^2)，又因为内存中该链表节点数庞大，多达数万个节点，所以如果在Eventloop中执行该操作，将有可能使得执行时间超长，此时需要将这部分代码交由其它线程予以执行，Vert.x提供了除了Eventloop线程池以外的线程池，名曰Worker线程池。此时就需要用户自行将该部分代码包装成Worker线程执行的代码，并交给Worker线程予以执行，执行完成之后再由Eventloop线程执行回调函数处理其结果。*注意：Vert.x中将代码交给Worker线程执行的方式有两种，一种是通过executeBlocking函数包装，另外一种是写入Worker Verticle中。*

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
   vertx.eventBus().send("address2", asyncResult.result().body(), future2.completer());
});

future2.setHandler(asyncResult-> {
    System.out.println(asyncResult.result().body());
});

vertx.eventBus().send("address1","message", future1.completer());
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
