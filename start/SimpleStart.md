
# SimpleStart

## Hello World

Stay Hungry. Stay Foolish. 如果您是Java新手，对于命令行感到发怵的话，我们特意为您准备了可视化傻瓜化入门教程，以降低Vert.x的入门门槛。但是再怎么傻瓜，也还是需要您具备有基本的Java语法知识，我们并不会在此介绍任何关于Java的语法知识，尤其是Java 1.8版本的基础知识，因为Vert.x 3以上版本要求Java 8以上版本方可运行，且大量应用了1.8加入的新语法糖，比如匿名函数Lambda。

本文将会建立一个基本的HTTP服务器，并监听8080端口，对于任何发往该服务器以及端口的请求，服务器会返回一个Hello World字符串。与Start部分不同的是：

1. SimpleStart将会使用IDE，也就是集成开发环境来简化开发流程；

2. SimpleStart将会使用Gradle，而非Maven来简化相关设置。

首先去JetBrain官方网站：www.jetbrains.com 免费下载集成开发环境（IDE）IntelliJ IDEA，有两个版本，社区版（Community）和终极版（Ultimate），后者需要付费使用，相比起前者而言，多了很多Java EE的支持，幸运的是，我们不需要使用这些貌似很牛逼其实无用的功能，用社区版就好了。下载后安装。

然后启动IntelliJ IDEA，选择创建新项目->Gradle项目->next->填入GroupId（也就是你所在组织的名字，例如io.example）和ArtifactID（也就是项目的名字，例如VertxExample）->next->勾选Use auto-import（自动引入）以及Create directories for empty content roots automatically（为空内容建立文件夹结构）->next ...-> finish，随后会建立以下文件结构，跟Maven文件夹结构几乎一样：

```
├── src
│   ├── main
│   │   ├── java
│   │   └── resources
│   └── test
│       └── java
├── build.gradle
├── settings.gradle
```

随后修改build.gradle为：

```gradle
group 'io.example'//对应刚刚向导中输入的 GroupId，ArtifactId在settings.gradle中
version '1.0-SNAPSHOT'

apply plugin: 'java'

sourceCompatibility = 1.8
targetCompatibility = 1.8//新增

repositories {
    mavenCentral()
}

dependencies {
    compile 'io.vertx:vertx-core:3.4.1'//新增 vert.x core的依赖
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

//以下部分新增
jar {
    manifest {
        attributes "Main-Class": "io.example.Main"//指定Main函数所在的类
    }

    from {
        configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
```

然后我们在`src/main/java/io/example`目录（是的，您需要新建io和example两级文件夹）下新建两个java文件，分别是`Main.java`和`MyFirstVerticle.java`，代码如下：

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

随后点开IDE右边的Gradle窗口->Tasks->build->jar，双击执行，便可在build/libs文件夹下生成可执行的jar文件，我生成的是`VertxExample-1.0-SNAPSHOT.jar`，在有图形界面的操作系统中，您可双击执行，或者用以下命令：`java -jar VertxExample-1.0-SNAPSHOT.jar`执行。

随后打开浏览器，在浏览器的地址栏中输入：http://localhost:8080/ 便可看到熟悉的Hello World!啦。

## 启动器

我们也可以使用`Launcher`来替代`Main`类，这也是官方推荐的方式，修改`build.gradle`如下：
```gradle
group 'io.example'
version '1.0-SNAPSHOT'

apply plugin: 'java'

sourceCompatibility = 1.8
targetCompatibility = 1.8

repositories {
    mavenCentral()
}

dependencies {
    compile 'io.vertx:vertx-core:3.4.1'
    testCompile group: 'junit', name: 'junit', version: '4.12'
}

jar {
    manifest {
        attributes "Main-Class": "io.vertx.core.Launcher",//改为Launcher
                "Main-Verticle": "io.example.MainVerticle"//新增Main Verticle属性，对应MainVerticle类
   }

    from {
        configurations.compile.collect { it.isDirectory() ? it : zipTree(it) }
    }
}
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

至此，大功告成，欢迎来到Vert.x的世界。

## 一点知识

jar文件本质上就是一个zip文件，可以用打开zip的压缩工具解压，读者可自行解压拆开jar包，看看相关依赖的文件是否在里面。比如例子中的代码除了生成io.example.MyFirstVerticle.class和io.example.MainVerticle.class文件以外，还同时将依赖的io.vertx.core中的内容压入jar包。
