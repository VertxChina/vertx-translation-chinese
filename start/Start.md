# Start

## Hello World

欢迎来到Vert.x的世界，相信您在接触Vert.x的同时，迫不及待想动手试一试，如您在学习计算机其它知识一样，总是从Hello World开始，下面我们将引导您制作一个最基本简单的Hello World例子，但在此之前，我们需要您具备有以下基础知识：

1. Java基础知识，您不需要了解Java EE或者是Java ME的知识，但是需要您对Java有所了解，在此文档中，我们不会介绍任何关于Java SE又称Core Java的知识点。请注意：Vert.x 3以上版本需要Java 8以上版本方能运行；

2. Maven相关知识，您需要知道什么Maven是做什么用的，以及如何使用Maven；

3. 互联网的基础知识，知道什么是网络协议，尤其是TCP，HTTP协议。

本文将会建立一个基本的HTTP服务器，并监听8080端口，对于任何发往该服务器以及端口的请求，服务器会返回一个Hello World字符串。

首先新建一个Maven项目，一个基本的Maven项目目录结构如下所示：

```
├── pom.xml
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       └── java
```

随后在`pom.xml`中加入相关的依赖和插件，如下所示：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.example</groupId>
    <artifactId>vertx-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <vertx.version>3.4.1</vertx.version>
        <main.class>io.example.Main</main.class>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-core</artifactId>
            <version>${vertx.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>${main.class}</Main-Class>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                            <artifactSet />
                            <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-prod.jar</outputFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

跟其它Maven项目一样，我们首先定义了项目的GroupId，ArtifactId以及版本号，随后我们定义了两个属性，分别是：`vertx.version`，也就是Vert.x的版本号，此处我们使用最新的Vert.x版本，也就是3.4.1；以及`main.class`，也就是我们要使用的包含有main函数的主类。之后我们引入了两个Maven插件，分别是`maven-compiler-plugin`和`maven-shade-plugin`，前者用来将.java的源文件编译成.class的字节码文件，后者可将编译后的.class字节码文件打包成可执行的jar文件，俗称`fat-jar`。

然后我们在`src/main/java/io/example`目录下新建两个java文件，分别是`Main.java`和`MyFirstVerticle.java`，代码如下：

Main.java

```java
package io.example;

import io.vertx.core.Vertx;

/**
 * Created by chengen on 26/04/2017.
 */
public class Main {
    public static void main(String[] args){
        Vertx vertx = Vertx.vertx();

        vertx.deployVerticle(MyFirstVerticle.class.getName());
    }
}
```

MyFirstVerticle.java

```java
package io.example;

import io.vertx.core.AbstractVerticle;

/**
 * Created by chengen on 26/04/2017.
 */
public class MyFirstVerticle extends AbstractVerticle {
    public void start() {
        vertx.createHttpServer().requestHandler(req -> {
            req.response()
                    .putHeader("content-type", "text/plain")
                    .end("Hello World!");
        }).listen(8080);
    }
}
```

然后用Maven的`mvn package`命令打包，随后在src的同级目录下会出现target目录，进入之后，会出现`vert-example-1.0-SNAPSHOT.jar`和`vert-example-1.0-SNAPSHOT-prod.jar`两个jar文件，后者是可执行文件，在有图形界面的操作系统中，您可双击执行，或者用以下命令：`java -jar vert-example-1.0-SNAPSHOT-prod.jar`执行。

随后打开浏览器，在浏览器的地址栏中输入：http://localhost:8080/ 便可看到熟悉的Hello World!啦。

## 启动器

我们也可以使用`Launcher`来替代`Main`类，这也是官方推荐的方式，在`pom.xml`中加入`main.verticle`属性，并将该属性值设置为`maven-shade-plugin`插件的`manifestEntries`的`Main-Verticle`对应的值，最后修改`main.class`为`io.vertx.core.Launcher`，修改后的`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.example</groupId>
    <artifactId>vertx-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <vertx.version>3.4.1</vertx.version>
        <main.class>io.vertx.core.Launcher</main.class>
        <main.verticle>io.example.MainVerticle</main.verticle>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-core</artifactId>
            <version>${vertx.version}</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>${main.class}</Main-Class>
                                        <Main-Verticle>${main.verticle}</Main-Verticle>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                            <artifactSet />
                            <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-prod.jar</outputFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

然后在`src/main/java/io/example`目录下新增`MainVerticle.java`文件，代码如下：

```java
package io.example;

import io.vertx.core.AbstractVerticle;

/**
 * Created by chengen on 26/04/2017.
 */
public class MainVerticle extends AbstractVerticle {
    public void start() {
        vertx.deployVerticle(MyFirstVerticle.class.getName());
    }
}
```

然后重新打包后执行，便可再次看到Hello World!。

> 请注意：*重新打包之前，您可能需要清除之前编译后留下的文件，用mvn clean package命令打包。*

## 测试

下面我们将会介绍测试部分，首先引入两个新的测试依赖，修改后的`pom.xml`如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>io.example</groupId>
    <artifactId>vertx-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <vertx.version>3.4.1</vertx.version>
        <main.class>io.vertx.core.Launcher</main.class>
        <main.verticle>io.example.MainVerticle</main.verticle>
    </properties>

    <dependencies>
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-core</artifactId>
            <version>${vertx.version}</version>
        </dependency>
        
        <dependency>
            <groupId>io.vertx</groupId>
            <artifactId>vertx-unit</artifactId>
            <version>${vertx.version}</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <artifactId>maven-compiler-plugin</artifactId>
                <version>3.3</version>
                <configuration>
                    <source>1.8</source>
                    <target>1.8</target>
                </configuration>
            </plugin>

            <plugin>
                <groupId>org.apache.maven.plugins</groupId>
                <artifactId>maven-shade-plugin</artifactId>
                <version>2.4.2</version>
                <executions>
                    <execution>
                        <phase>package</phase>
                        <goals>
                            <goal>shade</goal>
                        </goals>
                        <configuration>
                            <transformers>
                                <transformer implementation="org.apache.maven.plugins.shade.resource.ManifestResourceTransformer">
                                    <manifestEntries>
                                        <Main-Class>${main.class}</Main-Class>
                                        <Main-Verticle>${main.verticle}</Main-Verticle>
                                    </manifestEntries>
                                </transformer>
                            </transformers>
                            <artifactSet />
                            <outputFile>${project.build.directory}/${project.artifactId}-${project.version}-prod.jar</outputFile>
                        </configuration>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>
</project>
```

随后在`src/test/java/io/example`目录下新增`MyFirstVerticleTest.java`文件：

```java
package io.example;

import io.vertx.core.Vertx;
import io.vertx.ext.unit.Async;
import io.vertx.ext.unit.TestContext;
import io.vertx.ext.unit.junit.VertxUnitRunner;
import org.junit.After;
import org.junit.Before;
import org.junit.Test;
import org.junit.runner.RunWith;

/**
 * Created by chengen on 26/04/2017.
 */
@RunWith(VertxUnitRunner.class)
public class MyFirstVerticleTest {

    private Vertx vertx;

    @Before
    public void setUp(TestContext context) {
        vertx = Vertx.vertx();
        vertx.deployVerticle(MyFirstVerticle.class.getName(), context.asyncAssertSuccess());
    }

    @After
    public void tearDown(TestContext context) {
        vertx.close(context.asyncAssertSuccess());
    }

    @Test
    public void testApplication(TestContext context) {
        final Async async = context.async();

        vertx.createHttpClient().getNow(8080, "localhost", "/", response -> {
            response.handler(body -> {
                context.assertTrue(body.toString().contains("Hello"));
                async.complete();
            });
        });
    }
}
```

执行该测试案例便可得到期望的结果，理解测试代码并不难，留给读者作为练习。

至此，大功告成，欢迎来到Vert.x的世界。
