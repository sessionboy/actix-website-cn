---
title: Responses 响应
menu: docs_advanced
weight: 210
---

# Response

类似于构建器的模式用于构造`HttpResponse`的实例。 `HttpResponse`提供了几种返回`HttpResponseBuilder`实例的方法，该实例实现了用于构建responses的各种便捷方法。

> 检查[文档][responsebuilder]以获取类型说明。 

`.body`, `.finish`, 和 `.json`方法完成response创建并返回构造的`HttpResponse`实例。如果在同一构建器实例上多次调用此方法，则构建器将出现panic。

{{< include-example example="responses" file="main.rs" section="builder" >}}

# Content encoding 内容编码

Actix-web可以使用[Compress中间件][compressmidddleware]自动压缩payloads。支持以下编解码器：

* Brotli
* Gzip
* Deflate
* Identity

{{< include-example example="responses" file="compress.rs" section="compress" >}}

根据来自`middleware::BodyEncoding` trait的编码参数压缩Response payload。
默认情况下，使用`ContentEncoding::Auto`。
如果选择`ContentEncoding::Auto`，则压缩取决于请求的`Accept-Encoding` header。

> `ContentEncoding::Identity`可用于禁用压缩。
> 如果选择了其他content encoding，则将对该编码解码器强制执行压缩。

例如，要为单个处理程序启用`brotli`，请使用`ContentEncoding::Br`：

{{< include-example example="responses" file="brotli.rs" section="brotli" >}}

或对于整个应用程序：

{{< include-example example="responses" file="brotli_two.rs" section="brotli-two" >}}

在这种情况下，我们通过将content encoding设置为`Identity`值来显式禁用content压缩：

{{< include-example example="responses" file="identity.rs" section="identity" >}}

在处理已压缩的body时（例如，在提供assets时），请将content encoding设置为`Identity`以避免压缩已压缩的数据，并手动设置`content-encoding` header：

{{< include-example example="responses" file="identity_two.rs" section="identity-two" >}}

也可以在应用程序级别设置默认的content encoding，默认情况下使用`ContentEncoding::Auto`，这意味着自动进行content压缩协商。

{{< include-example example="responses" file="auto.rs" section="auto" >}}

# JSON Response

Json类型允许使用格式正确的JSON数据进行响应：只需返回`Json<T>`类型的值，其中`T`是要序列化为JSON的结构的类型。类型`T`必须实现serde的`Serialize`特性。

{{< include-example example="responses" file="json_resp.rs" section="json-resp" >}}

[responsebuilder]: https://docs.rs/actix-web/2/actix_web/dev/struct.HttpResponseBuilder.html
[compressmidddleware]: https://docs.rs/actix-web/2/actix_web/middleware/struct.Compress.html
