# Vert.x Web Client

- [原文档][1]
- [组件源码][2]
- [组件示例][3]

## 中英对照表

* Pump：泵（平滑流式数据读入内存的机制，防止一次性将大量数据读入内存导致内存溢出）
* Response Codec：响应编解码器（编码及解码工具）
* Body Codec：响应体编解码器

## 组件介绍

Vert.x Web Client（Web客户端）是一个异步的 HTTP 和 HTTP/2 客户端。

Web Client使得发送 HTTP 请求以及从 Web 服务器接收 HTTP 响应变得更加便捷，同时提供了额外的高级功能，例如：

- JSON体的编码和解码

- 请求和响应泵

- 请求参数的处理

- 统一的错误处理

- 提交表单

制作Web Client的目的并非为了替换Vert.x Core中的 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)，而是基于该客户端，扩展并保留其便利的设置和特性，例如请求连接池（Pooling），HTTP/2的支持，流水线／管线的支持等。当您需要对 HTTP 请求和响应做细微粒度控制时，您应当使用 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

另外Web Client并未提供 WebSocket API，此时您应当使用 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

## 使用Web Client

如需使用Vert.x Web Client，请先加入以下依赖：

- Maven（在 `pom.xml`文件中）：

  ```xml
  <dependency>
    <groupId>io.vertx</groupId>
    <artifactId>vertx-web-client</artifactId>
    <version>3.4.1</version>
  </dependency>
  ```

- Gradle（在`build.gradle`文件中）：

  ```gradle
  dependencies {
    compile 'io.vertx:vertx-web-client:3.4.1'
  }
  ```

## 对Vert.x Core HTTP Client的回顾

Vert.x Web Client使用Vert.x Core的API，如您对此还不熟悉，请先熟悉 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 的一些基本概念。

## 创建Web Client

您可使用缺省设置创建一个 [`WebClient`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClient.html)：

```java
WebClient client = WebClient.create(vertx);
```

您亦可使用配置选项来创建客户端：

```java
WebClientOptions options = new WebClientOptions()
  .setUserAgent("My-App/1.2.3");
options.setKeepAlive(false);
WebClient client = WebClient.create(vertx, options);
```

Web Client配置选项继承自 `HttpClient` 配置选项，使用时可根据实际情况选择。

如已在程序中创建 `HttpClient`，可用以下方式复用：

```java
WebClient client = WebClient.wrap(httpClient);
```

## 发送请求

### 无请求体的简单请求

一般情况下，HTTP GET，OPTIONS以及HEAD请求没有请求体，可用以下方式发送无请求体的HTTP Requests（HTTP请求）：

```java
WebClient client = WebClient.create(vertx);

// 发送GET请求
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .send(ar -> {
    if (ar.succeeded()) {
      // 获取响应
      HttpResponse<Buffer> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });

//发送HEAD请求
client
  .head(8080, "myserver.mycompany.com", "/some-uri")
  .send(ar -> {
    if (ar.succeeded()) {
      // 获取响应
      HttpResponse<Buffer> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

您可用以下链式方式向请求URI添加查询参数

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .addQueryParam("param", "param_value")
  .send(ar -> {});
```

在请求URI中的参数将会被预填充

```java
HttpRequest<Buffer> request = client.get(8080, "myserver.mycompany.com", "/some-uri?param1=param1_value&param2=param2_value");

// 添加param3（参数3）
request.addQueryParam("param3", "param3_value");

// 覆盖param2（参数2）
request.setQueryParam("param2", "another_param2_value");
```

设置请求URI将会自动清除已有的查询参数

```java
HttpRequest<Buffer> request = client.get(8080, "myserver.mycompany.com", "/some-uri");

// 添加param1（参数1）
request.addQueryParam("param1", "param1_value");

// 覆盖param1（参数1）同时新增param2（参数2）
request.uri("/some-uri?param1=param1_value&param2=param2_value");
```

### 填充请求体

如需要发送请求体，可使用相同的API并在最后加上 `sendXXX` 方法发送相应的请求体。

例如用 [`sendBuffer`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendBuffer-io.vertx.core.buffer.Buffer-io.vertx.core.Handler-) 方法发送一个缓冲体：

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendBuffer(buffer, ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

有时候我们并不希望将所有数据一次性全部读入内存，因为文件太大或希望同时处理多个请求，希望每个请求仅使用最小的内存。出于此目的，Web Client可用 [`sendStream`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendStream-io.vertx.core.streams.ReadStream-io.vertx.core.Handler-) 方法发送流式数据 `ReadStream<Buffer>`（例如 [`AsyncFile`](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html) 便是一个 `ReadStream<Buffer>`）：

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendStream(stream, resp -> {});
```

Web Client会为您设置好传输泵以平滑传输流。如果流长度未知则使用分块传输（chunked transfer）。

如已知流的大小，可在HTTP协议头中设置 `content-length` 属性

```java
fs.open("content.txt", new OpenOptions(), fileRes -> {
  if (fileRes.succeeded()) {
    ReadStream<Buffer> fileStream = fileRes.result();

    String fileLen = "1024";

    // 用POST方法发送文件
    client
      .post(8080, "myserver.mycompany.com", "/some-uri")
      .putHeader("content-length", fileLen)
      .sendStream(fileStream, ar -> {
        if (ar.succeeded()) {
          // Ok
        }
      });
  }
});
```

此时POST方法不会使用分块传输。

#### JSON体

有时您需要在请求体中使用JSON格式，可使用 [`sendJsonObject`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendJsonObject-io.vertx.core.json.JsonObject-io.vertx.core.Handler-) 方法发送 [`JsonObject`](http://vertx.io/docs/apidocs/io/vertx/core/json/JsonObject.html)：

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendJsonObject(new JsonObject()
    .put("firstName", "Dale")
    .put("lastName", "Cooper"), ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

在Java，Groovy以及Kotlin语言中，您亦可使用 [`sendJson`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendJson-java.lang.Object-io.vertx.core.Handler-) 方法发送POJO（Plain Old Java Object），该方法会自动调用 [`Json.encode`](http://vertx.io/docs/apidocs/io/vertx/core/json/Json.html#encode-java.lang.Object-) 方法将 POJO 映射为 JSON：

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendJson(new User("Dale", "Cooper"), ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

> 请注意： *[`Json.encode`](http://vertx.io/docs/apidocs/io/vertx/core/json/Json.html#encode-java.lang.Object-) 方法使用Jackson的 mapper将 POJO 映射成 JSON。*

#### 表单提交

您可使用 [`sendForm`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendForm-io.vertx.core.MultiMap-io.vertx.core.Handler-) 方法发送HTTP表单。

```java
MultiMap form = MultiMap.caseInsensitiveMultiMap();
form.set("firstName", "Dale");
form.set("lastName", "Cooper");

// 用URL编码方式提交表单
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendForm(form, ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

缺省情况下，提交表单的请求头中的 `content-type` 属性值为 `application/x-www-form-urlencoded`，您亦可将其设置为 `multipart/form-data`：

```java
MultiMap form = MultiMap.caseInsensitiveMultiMap();
form.set("firstName", "Dale");
form.set("lastName", "Cooper");

// 用分块方式编码提交表单
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .putHeader("content-type", "multipart/form-data")
  .sendForm(form, ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

> 请注意：*当前版本并不支持分块文件编码（multipart files，即文件上传），该功能可能在将来版本中予以支持。*

### 填充请求头

您可使用以下方式填充请求头：

```java
HttpRequest<Buffer> request = client.get(8080, "myserver.mycompany.com", "/some-uri");
MultiMap headers = request.headers();
headers.set("content-type", "application/json");
headers.set("other-header", "foo");
```

此处 Headers 是一个 [`MultiMap`](http://vertx.io/docs/apidocs/io/vertx/core/MultiMap.html) 对象，提供了增加、设置以及删除头属性操作的入口。HTTP头的某些特定属性允许设置多个值。

您亦可通过 `putHeader` 方法写入头属性：

```java
HttpRequest<Buffer> request = client.get(8080, "myserver.mycompany.com", "/some-uri");
request.putHeader("content-type", "application/json");
request.putHeader("other-header", "foo");
```

### 重用请求

[`send`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#send-io.vertx.core.Handler-) 方法可被重复多次调用，这使得配置以及重用 [`HttpRequest`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html) 对象变得更加便捷：

```java
HttpRequest<Buffer> get = client.get(8080, "myserver.mycompany.com", "/some-uri");
get.send(ar -> {
  if (ar.succeeded()) {
    // Ok
  }
});

// 再次发送同样的请求
get.send(ar -> {
  if (ar.succeeded()) {
    // Ok
  }
});

```

当您需要更改请求时，可用 [`copy`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#copy--) 方法复制一份请求的拷贝：

```java
HttpRequest<Buffer> get = client.get(8080, "myserver.mycompany.com", "/some-uri");
get.send(ar -> {
  if (ar.succeeded()) {
    // Ok
  }
});

// 获取同样的请求
get.copy()
  .putHeader("an-header", "with-some-value")
  .send(ar -> {
  if (ar.succeeded()) {
    // Ok
  }
```

### 超时

您可通过 [`timeout`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#timeout-long-) 方法设置超时时间。

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .timeout(5000)
  .send(ar -> {
    if (ar.succeeded()) {
      // Ok
    } else {
      // 此处可填入超时处理部分代码
    }
  });
```

若请求在设定时间内没返回任何数据，则一个超时异常将会传递给响应处理代码。

## 处理HTTP响应

Web Client请求发送之后，返回的结果将会被包装在异步结果 [`HttpResponse`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpResponse.html) 中。

当响应被成功接收到之后，相应的回调函数将会被触发。

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .send(ar -> {
    if (ar.succeeded()) {

      HttpResponse<Buffer> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

> 警告：*缺省状况下，响应会被完全缓冲读入内存，请用 [`BodyCodec.pipe`](http://vertx.io/docs/apidocs/io/vertx/ext/web/codec/BodyCodec.html#pipe-io.vertx.core.streams.WriteStream-) 方法将响应写入流。*

### 响应编解码器

缺省状况下，响应以缓冲形式提供，并不提供任何形式的解码。

可用 [`BodyCodec`](http://vertx.io/docs/apidocs/io/vertx/ext/web/codec/BodyCodec.html) 将响应定制成以下类型：

- 普通字符串
- JSON对象
- 将JSON映射成POJO
- [`WriteStream`](http://vertx.io/docs/apidocs/io/vertx/core/streams/WriteStream.html)

响应体编解码器对二进制数据流解码，以节省您在响应处理中的代码。

使用 [`BodyCodec.jsonObject`](http://vertx.io/docs/apidocs/io/vertx/ext/web/codec/BodyCodec.html#jsonObject--) 将结果解码为JSON对象：

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .as(BodyCodec.jsonObject())
  .send(ar -> {
    if (ar.succeeded()) {
      HttpResponse<JsonObject> response = ar.result();

      JsonObject body = response.body();

      System.out.println("Received response with status code" + response.statusCode() + " with body " + body);
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

在Java，Groovy以及Kotlin语言中，JSON对象可被解码映射成POJO：

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .as(BodyCodec.json(User.class))
  .send(ar -> {
    if (ar.succeeded()) {
      HttpResponse<User> response = ar.result();

      User user = response.body();

      System.out.println("Received response with status code" + response.statusCode() + " with body " +
        user.getFirstName() + " " + user.getLastName());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

当响应结果较大时，请使用 [`BodyCodec.pipe`](http://vertx.io/docs/apidocs/io/vertx/ext/web/codec/BodyCodec.html#pipe-io.vertx.core.streams.WriteStream-) 方法。响应体编解码器将响应结果压入 `WriteStream` 并在最后发出成功或失败的信号。

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .as(BodyCodec.pipe(writeStream))
  .send(ar -> {
    if (ar.succeeded()) {

      HttpResponse<Void> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

最后，如您对响应结果不感兴趣，可用 [`BodyCodec.none`](http://vertx.io/docs/apidocs/io/vertx/ext/web/codec/BodyCodec.html#none--) 废弃响应体。

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .as(BodyCodec.none())
  .send(ar -> {
    if (ar.succeeded()) {

      HttpResponse<Void> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

若无法预知响应内容类型，您依旧可以在获取结果之后，用 `bodyAsXXX()` 方法将其转换成特定的类型

```java
client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .send(ar -> {
    if (ar.succeeded()) {

      HttpResponse<Buffer> response = ar.result();

      // 将结果解码为Json对象
      JsonObject body = response.bodyAsJsonObject();

      System.out.println("Received response with status code" + response.statusCode() + " with body " + body);
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

> 警告：*这种方式仅对响应结果为缓冲体有效。*

### 处理30x重定向

缺省状况下，客户端将会依照30x状态码自动重定向，您可使用 [`WebClientOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClientOptions.html) 予以配置：

```java
WebClient client = WebClient.create(vertx, new WebClientOptions().setFollowRedirects(false));
```

客户端将会执行最多达`16`次重定向，该参数亦可在 [`WebClientOptions`](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClientOptions.html) 配置：

```java
WebClient client = WebClient.create(vertx, new WebClientOptions().setMaxRedirects(5));
```

## 使用HTTPS

Vert.x Web Client可用与 [`HttpClient`](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html) 相同方式配置HTTPS协议。

您可对每个请求单独设置：

```java
client
  .get(443, "myserver.mycompany.com", "/some-uri")
  .ssl(true)
  .send(ar -> {
    if (ar.succeeded()) {
      // 获取响应
      HttpResponse<Buffer> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

或使用绝对路径：

```java
client
  .getAbs("https://myserver.mycompany.com:4043/some-uri")
  .send(ar -> {
    if (ar.succeeded()) {
      // 获取响应
      HttpResponse<Buffer> response = ar.result();

      System.out.println("Received response with status code" + response.statusCode());
    } else {
      System.out.println("Something went wrong " + ar.cause().getMessage());
    }
  });
```

## RxJava API

RxJava的 [`HttpRequest`](http://vertx.io/docs/apidocs/io/vertx/rxjava/ext/web/client/HttpRequest.html) 提供了原版API的响应式版本，[`rxSend`](http://vertx.io/docs/apidocs/io/vertx/rxjava/ext/web/client/HttpRequest.html#rxSend--) 方法返回一个可被订阅的 `Single<HttpResponse<Buffer>>`，故单个 `Single` 可被多次订阅。

```java
Single<HttpResponse<Buffer>> single = client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .rxSend();

// 发送一次请求，并处理其响应，rx通常通过订阅触发各种响应
single.subscribe(response -> {
  System.out.println("Received 1st response with status code" + response.statusCode());
}, error -> {
  System.out.println("Something went wrong " + error.getMessage());
});

// 再次发送请求
single.subscribe(response -> {
  System.out.println("Received 2nd response with status code" + response.statusCode());
}, error -> {
  System.out.println("Something went wrong " + error.getMessage());
});
```

获取到的 `Single` 可与其它RxJava API自然组合成链式处理

```java
Single<String> url = client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .rxSend()
  .map(HttpResponse::bodyAsString);

// 用flatMap将返回值内的链接作为参数传入lambda，在lambda中将其设置成发送请求，并返回Single，在下一步订阅中予以触发
url
  .flatMap(u -> client.getAbs(u).rxSend())
  .subscribe(response -> {
    System.out.println("Received response with status code" + response.statusCode());
  }, error -> {
    System.out.println("Something went wrong " + error.getMessage());
  });
```

之前的例子可写成

```java
Single<HttpResponse<JsonObject>> single = client
  .get(8080, "myserver.mycompany.com", "/some-uri")
  .putHeader("some-header", "header-value")
  .addQueryParam("some-param", "param value")
  .as(BodyCodec.jsonObject())
  .rxSend();
single.subscribe(resp -> {
  System.out.println(resp.statusCode());
  System.out.println(resp.body());
});
```

当发送请求体为 `Observable<Buffer>` 时，应使用 [`sendStream`](http://vertx.io/docs/apidocs/io/vertx/rxjava/ext/web/client/HttpRequest.html#sendStream-rx.Observable-io.vertx.core.Handler-)：

```java
Observable<Buffer> body = getPayload();

Single<HttpResponse<Buffer>> single = client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .rxSendStream(body);
single.subscribe(resp -> {
  System.out.println(resp.statusCode());
  System.out.println(resp.body());
});
```

当订阅时，`body` 将会被订阅，其内容将会被用于请求中。

---

> [原文](http://vertx.io/docs/vertx-web-client/java/)上次编辑于2017-03-15 15:54:14 欧洲中部时间

[1]: http://vertx.io/docs/vertx-web-client/java/
[2]: https://github.com/vert-x3/vertx-web/tree/master/vertx-web-client
[3]: https://github.com/vert-x3/vertx-examples/tree/master/web-client-examples
