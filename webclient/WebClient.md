# Vert.x Web

Vert.x Web Client（Web客户端）是一个异步的HTTP和HTTP2客户端。

Web客户端使得发送HTTP请求以及从Web服务器接收HTTP响应变得更加便捷，同时提供了额外的高级功能，例如：

- Json体的编码和解码

- 请求和响应泵

- 请求参数的处理

- 统一的错误处理

- 提交表单

Web客户端的出现并未完全替代Vert.x Core中的[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)，而是基于该客户端，继承并扩展了它的设置以及便利的特性，例如请求连接池（Pooling），HTTP/2的支持，流水线／管线的支持等……当您需要对HTTP请求和响应做细微粒度控制时，您应当使用[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

另外Web客户端并未提供WebSocket API，此时您应当使用[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)。

# 使用Web客户端

如需使用Vert.x Web客户端，请先加入以下依赖：

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



# 对Vert.x Core HTTP客户端的回顾

Vert.x Web客户端使用Vert.x Core的API，如您对此还不熟悉，请先熟悉[HttpClient](http://vertx.io/docs/apidocs/io/vertx/core/http/HttpClient.html)的一些基本概念。

# 创建Web客户端

您可使用缺省设置创建一个[WebClient](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/WebClient.html)：

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

Web Client配置选项继承自Http Client配置选项，使用时可根据实际情况选择。

如已在程序中创建Http Client，可用以下方式复用：

```java
WebClient client = WebClient.wrap(httpClient);
```

# 发送请求

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

您可用以下链式方式添加查询参数

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

如需要发送请求体，可使用相同的API并在最后加上`sendXXX`方法发送相应的请求体。

例如用[sendBuffer](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendBuffer-io.vertx.core.buffer.Buffer-io.vertx.core.Handler-)方法发送一个缓冲体

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendBuffer(buffer, ar -> {
    if (ar.succeeded()) {
      // Ok
    }
  });
```

有时候我们并不希望将所有内容全部读入内存，因为文件太大或您希望同时处理多个请求，希望每个请求仅使用最小的内存。出于此目的，Web客户端可用[sendStream](http://vertx.io/docs/apidocs/io/vertx/ext/web/client/HttpRequest.html#sendStream-io.vertx.core.streams.ReadStream-io.vertx.core.Handler-)方法发送流式数据`ReadStream<Buffer>`（例如[AsyncFile](http://vertx.io/docs/apidocs/io/vertx/core/file/AsyncFile.html)便是一个`ReadStream<Buffer>`）

```java
client
  .post(8080, "myserver.mycompany.com", "/some-uri")
  .sendStream(stream, resp -> {});
```

Web客户端会为您设置好传输泵以平滑传输流。此时会使用分块传输因为流长度未知。

如已知流的大小，可设置HTTP协议头中的`content-length`属性

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

### Json体

