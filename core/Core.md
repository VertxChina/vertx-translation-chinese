# Vert.x Core Manual

## 注解表

_\*：有些地方没有翻译的维持原词汇，或者在有些地方使用【】符号标注原词，（）内为语境说明_

* Client：客户端
* Server：服务端
* Writing：写（同注解为开发）
* Event Bus：事件总线
* Map：动词用法译为"映射"，名词用法翻为"哈希表"
* Authorisation：授权
* Application：应用

## **引用**

## **正文**

Vert.X内部的一个Java API的集合称为**Vert.X Core**

[Repository](https://github.com/eclipse/vert.x)

Vert.X Core提供了下列功能

* 开发TCP客户端和服务端
* 开发HTTP客户端和支持WebSocket的服务端
* 事件总线
* 共享数据【Shared Data】——本地哈希表和分布式集群哈希表
* 周期性、延迟性行为
* 部署和撤销【Undeploying】Verticle实例
* 数据报路（UDP服务器）
* DNS客户端
* 文件系统访问
* 高可用
* 集群

Core中的这些功能相当底层——你不会在这里找到类似数据库访问、授权或高层Web功能的功能【Stuff】，这一类功能可以在**Vert.X Ext**（扩展包）中找到。

**Vert.X Core**很小、很轻量级，你可以仅仅使用你想要的功能。它可以完全嵌入在已存在的应用里——你可以随便使用它，它不强制你用特殊的方式去构造应用。

