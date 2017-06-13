# Vert.x Stack Manager

> 翻译：[Ranger Tsao](https://github.com/boliza)

Stack Manager 是一个管理 Vert.x 发行包的工具。

此工具管理维护 `lib` 目录中 jar 文件，同时采用 `YAML` 文件来描述应包含的组件依赖关系。它能自行解决了新的依赖关系，并删除了不再使用的文件。

在已经下载了一个 Vert.x 发行包中，根目录中有一个 `vertx-stack.json` ，此文件中囊括了所有官方组件。默认情况下，它只包括最小集合。

## 添加组件

如果要增加一个组件，对于文件 `vertx-stack.json` 中不存在的，将其增加至文件中，然后将描述代码中的 `included` 设置为 `true` 。代码片段例子：

```json
{
  "groupId": "io.vertx",
  "artifactId": "vertx-sync",
  "version": "${vertx.version}",
  "included": true
  }
```

此工具用 Maven 坐标来描述依赖。 其中 `groupId` 、`artifactId` 、版本属性是必需的。当然，也可以设置类型 `type`（默认为 `jar` ）和分类器 `classifier`（默认情况下为 none ）。

## 删除组件

在 `vertx-stack.json` 删除依赖描述，或者将其 `included` 设置为 `false` 便可以轻松实现删除组件。代码片段：

```json
{
  "groupId": "io.vertx",
  "artifactId": "vertx-sync",
  "version": "${vertx.version}",
  "included": false
}
```

在此操作之后，所有与依赖关系（或传递依赖关系）无关的文件都将被删除。这意味着不要手动删除文件，它们将被删除。

## 执行解析操作

执行命令 `./bin/vertx resolve --dir=lib` 便可以处理添加、删除操作。

命令 `resolve` 的执行参数或选项有：

* `--dir <value>` - 包含组件的目录。默认为 `./lib` 目录
* `--fail-on-conflict` - 设置解析器是否失败或冲突，或仅记录警告。默认禁用
* `--http-proxy <value>` - HTTP 代理地址
* `--https-proxy <value>` - HTTPS 代理地址
* `--local-repo <value>` - 本地 Maven 仓库目录。默认是 `~/.m2/repository`
* `--remote-repo <value>` - 远程 Maven 仓库地址。可以设置多个值
* `<vertx-stack.json>` - 工具描述文件路径。 默认为 `vertx-stack.json`
* `--no-cache` - 禁用缓存
* `--no-cache-for-snapshots` - 禁用 `snapshots` 版本缓存


如果配置 `VERTX_HOME` 为环境变量（或系统变量），则：

- `$VERTX_HOME/lib`作为 `lib` 目录地址
- `$VERTX_HOME/vertx-stack.json` 作为工具描述文件路径

## 排除与消除传递

同 Maven，每个依赖描述，都可以添加多个排除项。代码片段：

```json
{
  "groupId": "org.acme",
  "artifactId": "acme-lib",
  "version": "1.0.0",
  "included": true,
  "exclusions": [
    {
      "groupId": "org.acme",
      "artifactId": "acme-not-required"
    }
  ]
}
```

同时可以将依赖关系的 `transitive` 属性设置为 `false` ，以解析传递依赖关系。当使用 `fat jars` or `shaded` 组件时，是非常有用的。代码片段为：

```json
{
  "groupId": "io.vertx",
  "artifactId": "vertx-web-templ-thymeleaf",
  "version": "${vertx.version}",
  "included": true,
  "classifier": "shaded",
  "transitive": false
}
```

## 使用变量

工具描述文件中，允许自定义并使用变量。代码片段：

```json
{
  "variables": {
    "vertx.version": "3.4.1"
  }
}
```

在依赖描述中可以 `${}` 符号来引用变量。同时变量就可以用 `-D` 设置系统变量，以便于覆盖。

## JSON 格式说明

`vertx-stack.json` 中的格式支持以下三项：

* `//` 注释
* 无引号 `key` ， eg:`groupId : "org.acme"`
* 单引号 `value` ， eg:`groupId : 'org.acme'`

---

[原文档](http://vertx.io/docs/vertx-stack-manager/stack-manager/)更新于2017-03-15 15:54:14 CET
