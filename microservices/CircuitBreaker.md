# Vert.x 熔断器

Vert.x 熔断器是 Vert.x 熔断模式的实现。

熔断器用来追踪故障次数，当失败次数达到阀值时触发熔断。也可以选择回退。

支持以下的故障：

- 代码中使用 **Future** 部分发生失败
- 运行时抛出异常
- 没有完成的futures(超时)



熔断器要旨是保护 Vert.x 的非阻塞和异步的行为，使执行模型受益。

---

# 使用 vert.x 熔断器
使用 Vert.x 熔断器，只需要在依赖中增加以下代码片段：

- Maven (在 pom.xml 文件中):  

```
	<dependency>
	  <groupId>io.vertx</groupId>
	  <artifactId>vertx-circuit-breaker</artifactId>
	  <version>3.4.1</version>
	</dependency>
```

- Gradle (在 build.gradle 文件中):

```
	compile 'io.vertx:vertx-circuit-breaker:3.4.1'
```

---

# 使用熔断器

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
执行块中接收Future作为参数，以表示操作的成功或失败以。 例如使用REST调用作为示例：

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









