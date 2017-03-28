# Vert.x MySQL/PostgreSQL client

**异步的MySQL/PostgreSQL的客户是负责为那些需要用MySQL或PostgreSQL数据库进行交互的应用程序Vert.x的接口。**  

它采用毛里西奥·利尼亚雷斯异步驱动程序在非阻塞的方式下与MySQL或PostgreSQL数据库进行交互
###使用MySQL和PostgreSQL客户端
本节将介绍如何配置你的项目能够使用MySQL/ PostgreSQL的客户端应用程序。
####在常规应用 
为了使用此客户端，你需要把一下jar添加到你的CLASSPATH： 

- vertx-mysql-postgresql-client 3.2.1 (the client)
- scala-library 2.11.4
- the postgress-async-2.11 and mysdql-async-2.11 from https://github.com/mauricio/postgresql-async
- joda time   
所有这些jar从Maven的重要库下载。
####封装在Fat jar的应用
如果你正在构建Maven或者Grade的Fat-jar,只需要添加一下依赖关系

-Maven(在你的pom.xml文件）:

    <dependency>
      <groupId>io.vertx</groupId>
      <artifactId>vertx-mysql-postgresql-client</artifactId>
      <version>3.2.1</version>
    </dependency>
- Gradle (在你的build.gradle文件):

    compile 'io.vertx:vertx-mysql-postgresql-client:3.2.1'
####在一个应用程序中使用分布式vert.x 
如果您使用的是分布式vert.x，将上面列出来的jar文件添加到 $VERTX_HOME/lib目录   
另外，你也可以编辑位于$VERTX_HOME的vertx-stack.json文件，设置"included":true为vertx-mysql-postgresql-client的依赖，之后运行： vertx resolve --dir=lib --stack= ./vertx-stack.json。它下载客户端和它的依赖。
***
###创建客户端 
有几种方法创建客户端，让我们看看所有的。
####使用默认的共享池 
在大多数情况下，你会想分享不同的客户端实例之间的pool。  
例如。您可以通过部署Verticle的多个实例扩展您的应用程序，并希望每个Verticle实例共享同一个pool，这样你不需要结束多个pool。  
你可以通过如下实现：

    JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
    AsyncSQLClient mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig);

    // To create a PostgreSQL client:

    JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
    AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig);
第一次调用MySQLClient.createShared或PostgreSQLClient.createShared将真正创建数据源，而且指定的配置被使用。  
后续调用将返回使用相同数据源的客户端实例，因此配置将不被使用。
####指定pool名称
您可以按如下创建一个客户端指定pool名称

    JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
    mySQLClient = MySQLClient.createShared(vertx, mySQLClientConfig, "MySQLPool1");

    // To create a PostgreSQL client:

    JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
    AsyncSQLClient postgreSQLClient = PostgreSQLClient.createShared(vertx, postgreSQLClientConfig, "PostgreSQLPool1");

如果不同的客户端都使用相同的Vert.x实例，并指定相同的pool名称创建的，它们将共享相同的数据源。  
第一次调用MySQLClient.createShared或PostgreSQLClient.createShared将真正创建数据源，而且指定的配置被使用。  
后续调用将返回使用相同数据源的客户端实例，因此配置将不被使用。  
使用这种方式创建如果你希望不同组的客户端有不同的pool,例如他们与不同的数据库进行交互.
####创建具有非共享数据源的客户端
在大多数情况下，你会想分享不同的客户端实例之间的连接池。但是，也可能要创建一个不与任何其他客户端共享连接池中的客户端实例。  
在这种情况下，你可以使用MySQLClient.createNonShared或PostgreSQLClient.createNonShared

    JsonObject mySQLClientConfig = new JsonObject().put("host", "mymysqldb.mycompany");
    AsyncSQLClient mySQLClient = MySQLClient.createNonShared(vertx, mySQLClientConfig);

    // To create a PostgreSQL client:

    JsonObject postgreSQLClientConfig = new JsonObject().put("host", "mypostgresqldb.mycompany");
    AsyncSQLClient postgreSQLClient = PostgreSQLClient.createNonShared(vertx, postgreSQLClientConfig);
这等同于调用MySQLClient.createShared或者PostgreSQLClient.createShared每次使用一个唯一的连接池名称。
###关闭客户端
你可以保持客户端很长时间（例如，你的Verticle的寿命），一旦你已经完成了它，你应该把它关闭
###获取连接
使用getConnection来获取一个连接  
当接连吃准备好的时候将返回一个在handler的连接。

    client.getConnection(res -> {
      if (res.succeeded()) {

        SQLConnection connection = res.result();

        // Got a connection

      } else {
      // Failed to get connection - deal with it
      }
    });
旦你的连接完成之后请务必将其关闭。
这个连接是一个被SQL客户端使用的普通接口的Connection的实例。  
你可以学习如何共同SQL接口文档中使用它。  
####请注意有关日期和时间戳
每当你得到数据库返回的日期数据，这项服务将隐含它们转换为ISO 8601（YYYY-MM-DDTHH：MM：SS.SSS）格式的字符串。MySQL的通常丢弃毫秒，所以你会经常看到.000。 
####注意最后插入的ID
当新行插入表中时，你可能从数据库中检索自增的id,在JDBC PAI中通常让你从一个连接检索的最后插入ID。如果你使用MySQL，它的工作确实像JDBC API的方式。在PostgreSQL，您可以添加“返回”条款，以获得最新的IDS插入。使用的查询方法来获得访问返回的列。 
####注意有关存储过程
调用和使用参数的调用目前没有实现。
###配置
无论是PostgreSQL和MySQL用户采取相同的配置

    {
      "host" : <your-host>,
      "port" : <your-port>,
      "maxPoolSize" : <maximum-number-of-open-connections>,
      "username" : <your-username>,
      "password" : <your-password>,
      "database" : <name-of-your-database>
    }
host  
数据库的主机，默认为localhost。  
port  
数据库的端口。PostgreSQL默认为5432，MySQL默认为3306.  
maxPoolSize   
可以保持打开的连接的数量。默认为10。  
username  
连接到数据库的用户名称。PostgreSQL默认为postgres，MySQL默认为root。  
password  
连接到数据库的密码，默认是没有设置的，即它不适用密码。  
database  
要连接到数据库的名称，默认是test.
# Vert.x MySQL/PostgreSQL Client
