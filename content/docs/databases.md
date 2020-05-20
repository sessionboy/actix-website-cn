---
title: 数据库
menu: docs_patterns
weight: 1010
---

# Diesel

{{% alert %}}
NOTE: The `actix-web` 1.0 version of this section is still
[being updated](https://github.com/cldershem/actix-website/tree/update1.0-db). Checkout
this [example](https://github.com/actix/examples/tree/master/async_db) until then.
{{% /alert %}}

目前，Diesel 1.0不支持异步操作，但是可以将`actix` synchronous actor系统用作数据库接口api。

从技术上讲，sync actors是worker风格的actors。多个sync actors可以并行运行，并处理来自同一队列的消息。Sync actors在mpsc模式下工作。

让我们创建一个简单的数据库api，可以将新的user行插入SQLite表。我们必须定义一个sync actor和该actor将使用的连接。相同的方法可以用于其他数据库。

{{< include-example example="og_databases" file="main.rs" section="actor" >}}

这是我们actor的定义。现在，我们必须定义*create user*消息和响应。

{{< include-example example="og_databases" file="main.rs" section="message" >}}

我们可以将`CreateUser`消息发送给`DbExecutor` actor，结果，我们将收到一个`User`模型实例。接下来，我们必须为此消息定义handler实现。

{{< include-example example="og_databases" file="main.rs" section="handler" >}}

现在，我们可以从任何http handler或中间件中使用*DbExecutor* actor。我们需要做的就是启动*DbExecutor* actor并将地址存储在http handler可以访问它的状态下。

{{< include-example example="og_databases" file="main.rs" section="main" >}}

我们将在请求handler中使用该地址。handle(句柄)返回一个future对象；因此，我们异步接收消息响应。 `Route::a()`必须用于 async handler注册。

{{< include-example example="og_databases" file="main.rs" section="index" >}}

> [示例目录][examples]中提供了完整的示例。

> 可以在[actix文档][actixdocs]中找到有关sync actors的更多信息。

[examples]: https://github.com/actix/examples/tree/master/diesel/
[actixdocs]: https://docs.rs/actix/0.7.0/actix/sync/index.html
