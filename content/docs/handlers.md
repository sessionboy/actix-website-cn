---
title: Handlers 处理程序
menu: docs_basics
weight: 160
---

# Request Handlers 请求处理程序

request handler(请求处理程序)是一个async function(异步函数)，它接受零个或多个可以从请求中提取的参数（即[*impl FromRequest*][implfromrequest]），并返回可以转换为HttpResponse的类型（即[*impl Responder*][respondertrait]）。

请求处理分为两个阶段。
首先，调用handler对象，返回实现[*Responder*][respondertrait]特性的任何对象。
然后，在返回的对象上调用`respond_to()`，将其自身转换为`HttpResponse`或`Error`。

默认情况下，`actix-web`为某些标准类型（例如`&'static str`, `String`等）提供`Responder`实现。

> 有关implementations的完整列表，请查看[*Responder 文档*][responderimpls]。

有效处理程序的示例：

```rust
async fn index(_req: HttpRequest) -> &'static str {
    "Hello world!"
}
```

```rust
async fn index(_req: HttpRequest) -> String {
    "Hello world!".to_owned()
}
```

您还可以将签名更改为return `impl Responder`，如果涉及到更复杂的类型，它可以很好地工作。

```rust
async fn index(_req: HttpRequest) -> impl Responder {
    Bytes::from_static(b"Hello world!")
}
```

```rust
async fn index(req: HttpRequest) -> Box<Future<Item=HttpResponse, Error=Error>> {
    ...
}
```

## 自定义类型的Response

要直接从handler函数返回自定义类型，该类型需要实现`Responder`特性。

让我们为自定义类型创建一个response(响应)，该response序列化为`application/json` response：

{{< include-example example="responder-trait" file="main.rs" section="responder-trait" >}}

## Streaming response body 流式响应体

Response body(响应体)可以异步生成。
在这种情况下，body必须实现stream特征`Stream<Item=Bytes, Error=Error>`，即：

{{< include-example example="async-handlers" file="stream.rs" section="stream" >}}

## 不同的return类型（任一种）

有时，您需要返回不同类型的responses。
例如，您可以进行错误检查并返回错误，返回异步responses或需要两种不同类型的任何结果。

在这种情况下，可以使用[*Either*][either]类型。
可以将两种不同的responder(响应者)类型组合为一个类型。

{{< include-example example="either" file="main.rs" section="either" >}}

[implfromrequest]: https://docs.rs/actix-web/2/actix_web/trait.FromRequest.html
[respondertrait]: https://docs.rs/actix-web/2/actix_web/trait.Responder.html
[responderimpls]: https://docs.rs/actix-web/2/actix_web/trait.Responder.html#foreign-impls
[either]: https://docs.rs/actix-web/2/actix_web/enum.Either.html
