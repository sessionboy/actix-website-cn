---
title: 提取器
menu: docs_basics
weight: 170
---

# 类型安全的信息提取

Actix-web提供了一种用于类型安全的访问请求信息的工具，称为提取器（即`impl FromRequest`）。
默认情况下，actix-web提供了几种提取器实现。

提取器可以作为handler函数的参数进行访问。
Actix-web每个handler函数最多支持10个提取器。
参数位置无关紧要。

{{< include-example example="extractors" file="main.rs" section="option-one" >}}

# Path 路径

[*Path*][pathstruct]提供可以从请求路径中提取的信息。
您可以从path反序列化任何变量段。

例如，对于`/users/{userid}/{friend}`路径注册的resource(资源)，可以反序列化两个segments(段)，即`userid`和`friend`。
这些段可以提取到一个元组中，即`Path<(u32, String)>`或从*serde* crate实现`Deserialize`特性的任何structure(结构体)。

{{< include-example example="extractors" file="path_one.rs" section="path-one" >}}

也有可能从serde提取path信息到实现`Deserialize`(反序列化)特征的特定类型。
这是一个使用*serde*而不是*tuple*类型的等效示例。

{{< include-example example="extractors" file="path_two.rs" section="path-two" >}}

也可以通过name来`get` 或 `query`路径参数的请求：

{{< include-example example="extractors" file="path_three.rs" section="path-three" >}}

# Query

[*Query*][querystruct]类型为请求的查询参数提供提取功能。
它的下面使用了*serde_urlencoded* crate。

{{< include-example example="extractors" file="query.rs" section="query" >}}

# Json

[*Json*][jsonstruct]允许将请求body反序列化为struct(结构)。
要从请求的正文中(body)提取类型化的信息，类型`T`必须从*serde*实现`Deserialize`(反序列化) trait(特征)。

{{< include-example example="extractors" file="json_one.rs" section="json-one" >}}

一些提取程序提供了一种配置提取过程的方法。
Json提取器的[*JsonConfig*][jsonconfig]类型用于configuration(配置)。
要配置提取程序，请将其配置对象传递到resource的`.data()`方法。
如果是Json提取器，它将返回*JsonConfig*。
您可以配置json payload的最大大小以及自定义错误处理函数。

以下示例将payload的大小限制为4kb，并使用自定义错误处理程序。

{{< include-example example="extractors" file="json_two.rs" section="json-two" >}}

# Form 表单

目前仅支持url-encoded形式。
可以将url-encoded的正文提取为特定类型。
此类型必须从*serde* crate中执行`Deserialize` trait。

[*FormConfig*][formconfig] 允许配置提取过程。

{{< include-example example="extractors" file="form.rs" section="form" >}}

# 其他 

Actix-web还提供了其他几种提取器：

* [*Data*][datastruct] - 如果您需要访问应用程序状态。
* `HttpRequest` - HttpRequest本身是一个提取程序，如果需要访问request，它会返回self。
* `String` - 您可以将request(请求)的payload转换为字符串。文档strings中提供了[示例][stringexample]。
* `bytes::Bytes` - 您可以将request(请求)的payload转换为Bytes(字节)。文档strings中提供了[示例][stringexample]。
* `Payload` - 您可以访问request的payload。示例[payloadexample]

# Application state extractor 应用状态提取器

可通过`web::Data` extractor(提取器)从处理程序访问应用程序状态；
但是，状态可以作为只读引用进行访问。
如果您需要对状态的可变访问，则必须实现它。

**当心**，actix将创建应用程序状态和handlers(处理程序)的多个副本，这对于每个线程都是唯一的。
如果您在多个线程中运行应用程序，actix将创建与应用程序state对象和handler对象的线程数相同的数量。

这是存储处理的请求数的处理程序的示例：

{{< include-example example="request-handlers" file="main.rs" section="data" >}}

尽管此handler将起作用，但`self.0`会有所不同，具体取决于线程数和每个线程处理的请求数。
适当的implementation(实现)将使用`Arc`和`AtomicUsize`。

{{< include-example example="request-handlers" file="handlers_arc.rs" section="arc" >}}

> 注意诸如`Mutex`或`RwLock`之类的同步原语。
> actix-web框架异步处理请求。
> 通过阻止线程执行，所有并发请求处理过程都将被阻止。
> 如果需要从多个线程共享或更新某些状态，请考虑使用tokio synchronization primitives (同步原语)。

[pathstruct]: https://docs.rs/actix-web/2/actix_web/dev/struct.Path.html
[querystruct]: https://docs.rs/actix-web/2/actix_web/web/struct.Query.html
[jsonstruct]: https://docs.rs/actix-web/2/actix_web/web/struct.Json.html
[jsonconfig]: https://docs.rs/actix-web/2/actix_web/web/struct.JsonConfig.html
[formconfig]: https://docs.rs/actix-web/2/actix_web/web/struct.FormConfig.html
[datastruct]: https://docs.rs/actix-web/2/actix_web/web/struct.Data.html
[stringexample]: https://docs.rs/actix-web/2/actix_web/trait.FromRequest.html#example-2
[bytesexample]: https://docs.rs/actix-web/2/actix_web/trait.FromRequest.html#example-4
[payloadexample]: https://docs.rs/actix-web/2/actix_web/web/struct.Payload.html
[actix]: https://actix.github.io/actix/actix/
