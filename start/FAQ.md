# FAQ

此处会列出常见的关于Vert.x 各个组件的常见问题以及相应的注意事项和解决方案。

##### 问：Vert.x中各种Client该如何正确使用，用完是否需要关闭？

答：Vert.x中提供了各种预设客户端，例如HttpClient，JDBCClient，WebClient，KafkaClient，MongoDBClient等，一般情况下，建议将客户端与Verticle对象绑定，一个Verticle对象内保留一个特定客户端的引用，并在start方法中将其实例化，这样Vert.x会在执行deployVerticle的时候执行start方法，实例化并保存该对象，在Verticle生命周期内，不需要频繁创建和关闭同类型的客户端，建议在Verticle的生命周期内对于特定领域，只创建一个客户端并复用该客户端，例如：

```java
import io.vertx.core.AbstractVerticle;
import io.vertx.core.http.HttpClient;
import io.vertx.core.json.JsonObject;
import io.vertx.ext.jdbc.JDBCClient;
import io.vertx.ext.web.client.WebClient;

/**
 * Created by chengen on 21/04/2017.
 */
public class TestVerticle extends AbstractVerticle{
    //将客户端对象与Verticle对象绑定
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

##### 问：Vert.x中Future该如何正确使用，怎样规避回调地狱？

答：Future对象提供了一种异步结果的包装，用户可使用Future类中的setHandler方法来保存回调函数，然后在原先使用该回调函数的地方用completer方法予以填充，这样便可将回调函数从原参数中取出，以减少回调缩进，从而规避回调地狱，来看一个简单的例子：

```java
//未使用future时，回调函数嵌在send方法内部，以匿名函数的形式作为send的参数
vertx.eventBus().send("address","message", asyncResult->{
    System.out.println(asyncResult.result()); 
});
```

```java
Future<Message<String>> future = Future.future();
//将回调函数存入future中，从而实现代码的扁平化
future.setHandler(asyncResult -> {
    System.out.println(asyncResult.result());
});
//使用future之后，用completer方法填充参数
vertx.eventBus().send("address","message", future.completer());
```

一个复杂一点的例子：

```java
//以下程序先向address1发送一个message，然后等address1回复之后，将address1的回复消息发送给address2，最后将address2的回复打印到控制台上
vertx.eventBus().send("address1","message", asyncResult->{
    vertx.eventBus().send("address2", asyncResult.result(), asyncResult2->{
        System.out.println(asyncResult2.result());
    });
});
```

可以看到此时为了保证顺序结构，产生了两层缩进，回调金字塔开始形成，多次缩进之后便会出现所谓的回调地狱，这便是异步开发中为了保证顺序所可能会遇到的问题，那么我们可以通过以下方式解决代码过多缩进的问题：

```java
Future<Message<String>> future1 = Future.future();
Future<Message<String>> future2 = Future.future();

future1.setHandler(asyncResult -> {
   vertx.eventBus().send("address2", asyncResult.result(), future2.completer());
});

future2.setHandler(asyncResult-> {
    System.out.println(asyncResult.result());
});

vertx.eventBus().send("address1","message", future1.completer());
```

可以看到，使用了future之后，原先的两层缩进被抽取出来，变成了最多单层的缩进，从而使得代码可读性更强，更加美观。
