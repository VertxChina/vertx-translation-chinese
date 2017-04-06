# Vert.x Circuit Breaker

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 中英文对照表

- Circuit Breaker：熔断器
- metric：指标

## 组件介绍

Vert.x Circuit Breaker是 [熔断器模式](https://martinfowler.com/bliki/CircuitBreaker.html) 的Vert.x实现。

熔断器用来追踪故障次数，当失败次数达到阈值时触发熔断，并且可选择性提供失败回调。

熔断器支持以下的故障：

- 使用 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html) 时失败
- 运行时抛出异常
- 没有完成的 `Future`(超时)

熔断器要旨是保护 Vert.x 的 **非阻塞** 和 **异步** 的行为，以便受益于Vert.x 执行模型。

## 使用 Vert.x 熔断器

要使用 Vert.x 熔断器，只需要在依赖中增加以下代码片段：

- Maven (在 `pom.xml` 文件中):  

```xml
<dependency>
  <groupId>io.vertx</groupId>
  <artifactId>vertx-circuit-breaker</artifactId>
  <version>3.4.1</version>
</dependency>
```

- Gradle (在 `build.gradle` 文件中):

```groovy
compile 'io.vertx:vertx-circuit-breaker:3.4.1'
```

## 使用熔断器

为了使用熔断器我们需要以下的步骤：  

- 创建一个熔断器，并配置成你所需要的(超时，最大故障次数)
- 使用熔断器执行代码

以下是例子:

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions()
        .setMaxFailures(5) // 最大故障次数
        .setTimeout(2000) // 超时时间
        .setFallbackOnFailure(true) // 设置是否失败回调
        .setResetTimeout(10000) // 重置状态超时
);

breaker.execute(future -> {
  // 在熔断器中执行的代码
  // 这里的代码可以成功或者失败
  // 如果future在这里被标记为失败，熔断器将自增失败数
}).setHandler(ar -> {
  // 处理结果
});
```
执行块中接收 `Future` 作为参数，以表示操作和结果的成功或失败。 例如在下面的例子中，对应的结果就是REST调用的输出：

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
);

breaker.<String>execute(future -> {
  vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
    if (response.statusCode() != 200) {
      future.fail("HTTP error");
    } else {
      response
          .exceptionHandler(future::fail)
          .bodyHandler(buffer -> {
            future.complete(buffer.toString());
          });
    }
  });
}).setHandler(ar -> {
  // 处理结果
});
```

操作的结果以下面的方式提供：  

- 调用 `execute` 方式返回 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html)
- 调用 `executeAndReport` 时作为参数提供的 [`Future`](http://vertx.io/docs/apidocs/io/vertx/core/Future.html)

也可以提供一个失败时回调方法(fallback)：

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
);

breaker.executeWithFallback(
    future -> {
      vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
        if (response.statusCode() != 200) {
          future.fail("HTTP error");
        } else {
          response
              .exceptionHandler(future::fail)
              .bodyHandler(buffer -> {
                future.complete(buffer.toString());
              });
        }
      });
    }, v -> {
      // 当熔断器熔断时将调用此处代码
      return "Hello";
    })
    .setHandler(ar -> {
      // 处理结果
    });
```

熔断状态中都会调用失败回调（fallback），或者设置 [`isFallbackOnFailure`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreakerOptions.html#isFallbackOnFailure--)，其结果是失败回调函数的输出。失败回调函数将 [`Throwable`](http://vertx.io/docs/apidocs/java/lang/Throwable.html) 对象作为参数，并返回预期类型的​​对象。

失败回调可以直接设置在 [`CircuitBreaker`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreaker.html) 上：

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
).fallback(v -> {
  // 当熔断器熔断时将调用此处代码
  return "hello";
});

breaker.execute(
    future -> {
      vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
        if (response.statusCode() != 200) {
          future.fail("HTTP error");
        } else {
          response
              .exceptionHandler(future::fail)
              .bodyHandler(buffer -> {
                future.complete(buffer.toString());
              });
        }
      });
    });
```

可以指定熔断器在生效之前的尝试次数，使用 [`setMaxRetries`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreakerOptions.html#setMaxRetries-int-)。如果将其设置为高于0的值，则您的代码在最终失败之前进行尝试多次执行。如果代码在其中一个重试中成功，则处理程序将得到通知，并且跳过剩余的重试。此配置仅当熔断器未生效时工作。

## 回调

你能够配置熔断生效/关闭时回调。  

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx,
    new CircuitBreakerOptions().setMaxFailures(5).setTimeout(2000)
).openHandler(v -> {
  System.out.println("Circuit opened");
}).closeHandler(v -> {
  System.out.println("Circuit closed");
});

breaker.execute(
    future -> {
      vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
        if (response.statusCode() != 200) {
          future.fail("HTTP error");
        } else {
          // 处理结果
          future.complete();
        }
      });
    });
```

当熔断器决定尝试复位的时候（ half-open 状态），我们也可以注册 [`halfOpenHandler`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreaker.html#halfOpenHandler-io.vertx.core.Handler-) 的回调从而得到回调通知。

## Event Bus 通知

每次熔断器状态发生变化时，会在Event Bus上发布事件。事件发送的地址可以使用 [`setNotificationAddress`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreakerOptions.html#setNotificationAddress-java.lang.String-) 进行配置。如果将  `null` 传递给此方法，则通知将被禁用。默认情况下，使用的地址是 `vertx.circuit-breaker`。
每次事件信息包含以下：

- `state`: 熔断器的新状态 （`OPEN`，`CLOSED`，`HALF_OPEN`）
- `name`: 熔断器的名字
- `failures`: 故障的数量
- `node`: 节点的标志符（如果运行在单节点模式是 `local`）


## 半开启状态

当熔断器在熔断状态中，对其调用会立即失败，不会执行实际操作。经过适当的时间（[`setResetTimeout`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/CircuitBreakerOptions.html#setResetTimeout-long-) 设置），熔断器决定是否恢复状态，此时进入半开启状态（half-open state）。在这种状态下，允许下一次熔断器的调用实际调用如果成功，熔断器将复位并返回到关闭状态，回归正常的模式；但是如果这次调用失败，则熔断器返回到熔断状态，直到下次半开状态。

## 将熔断器指标推送到Hystrix Dashboard

[Netflix Hystrix](https://github.com/Netflix/Hystrix )带有一个仪表板（dashboard），用于显示熔断器的当前状态。 Vert.x 熔断器可以发布其指标（metric），以供Hystrix 仪表板使用。 Hystrix 仪表板需要一个发送指标的SSE流，此流由 [`HystrixMetricHandler`](http://vertx.io/docs/apidocs/io/vertx/circuitbreaker/HystrixMetricHandler.html) 这个 Vert.x Web Handler 提供：

```java
CircuitBreaker breaker = CircuitBreaker.create("my-circuit-breaker", vertx);
CircuitBreaker breaker2 = CircuitBreaker.create("my-second-circuit-breaker", vertx);

// 创建 Vert.x Web 路由
Router router = Router.router(vertx);
// 注册指标Handler
router.get("/hystrix-metrics").handler(HystrixMetricHandler.create(vertx));

// 创建HTTP服务器，并分配路由
vertx.createHttpServer()
  .requestHandler(router::accept)
  .listen(8080);
```

在Hystrix 仪表板中，配置流网址（stream url），如：[`http://localhost:8080/metrics`](http://localhost:8080/metrics)。仪表板将使用Vert.x熔断器的指标。

请注意，这些指标量是由 Vert.x Web Handler 使用 Event Bus 事件通知收集的。如果您不使用默认通知地址，则需要在创建时指定。

## 使用 Netflix Hystrix

[Hystrix](https://github.com/Netflix/Hystrix)提供了熔断器模式的实现。可以在Vert.x中使用Hystrix提供的熔断器或组合使用。本节介绍在Vert.x应用程序中使用Hystrix的技巧。

首先，您需要将Hystrix添加到你的依赖中。详细信息请参阅Hystrix页面。然后，您需要使用 `Command` 隔离“受保护的”调用。你可以这样执行它：

```java
HystrixCommand<String> someCommand = getSomeCommandInstance();
String result = someCommand.execute();
```


但是，命令执行是阻塞的，必须结合 `executeBlocking` 方法执行，或者在Worker Verticle中调用：

```java
HystrixCommand<String> someCommand = getSomeCommandInstance();
vertx.<String>executeBlocking(
  future -> future.complete(someCommand.execute()),
  ar -> {
    // 回到Event Loop线程中
    String result = ar.result();
  }
);
```


如果您使用Hystrix的异步支持，请注意，对应的回调函数不会在Vert.x线程中执行，并且必须在执行前保留对上下文的引用（使用 [ `getOrCreateContext` ](http://vertx.io/docs/apidocs/io/vertx/core/Vertx.html#getOrCreateContext--) )，并且在回调中，执行 `runOnContext` 函数将当前线程切换回Event Loop线程。如果不这样做的话，您将失去Vert.x并发模型的优势，并且必须自行管理线程同步和执行顺序：

```java
vertx.runOnContext(v -> {
  Context context = vertx.getOrCreateContext();
  HystrixCommand<String> command = getSomeCommandInstance();
  command.observe().subscribe(result -> {
    context.runOnContext(v2 -> {
      // 回到Vert.x Context下(Event Loop线程或Worker线程)
      String r = result;
    });
  });
});
```

---

> [原文档](http://vertx.io/docs/vertx-circuit-breaker/java/)最后更新于 2017-03-15 15:54:14 CET

[1]: http://vertx.io/docs/vertx-circuit-breaker/java/
[2]: https://github.com/vert-x3/vertx-circuit-breaker
[3]: https://github.com/vert-x3/vertx-examples/tree/master/circuit-breaker-examples
