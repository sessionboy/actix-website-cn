---
title: Requests 请求
menu: docs_advanced
weight: 200
---

# Content Encoding 内容编码

Actix-web自动解压payloads。支持以下编解码器：

* Brotli
* Chunked
* Compress
* Gzip
* Deflate
* Identity
* Trailers
* EncodingExt  

如果headers包含`Content-Encoding` header，则将根据header值对请求payload进行解压缩。不支持多种编解码器，即：`Content-Encoding: br, gzip`。

# JSON Request

json body反序列化有几种选择。

第一种选择是使用*Json*提取器。
首先，您定义一个接受`Json<T>`作为参数的handler函数，然后，使用`.to()`方法注册该handler函数。
通过使用`serde_json::Value`作为类型`T`也可以接受任意有效的json对象。

{{< include-example example="requests" file="main.rs" section="json-request" >}}

您也可以将payload手动加载到内存中，然后反序列化。

在下面的示例中，我们将反序列化*MyObj*结构。我们需要先加载请求body，然后将json反序列化为一个对象。

{{< include-example example="requests" file="manual.rs" section="json-manual" >}}

> 示例目录中提供了这两个选项的[完整示例][examples]。

# 分块传输编码

Actix自动解码*chunked*(分块)编码。
[`web::Payload`][payloadextractor]提取器已经包含解码后的字节流。
如果使用支持的压缩编解码器（br，gzip，deflate）之一压缩请求payload，则将字节流解压缩。

# Multipart body 多body

Actix-web通过外部crate, [`actix-multipart`][multipartcrate]提供multipart stream(流)支持。

> 示例目录中提供了[完整的示例][multipartexample]。

# Urlencoded body

Actix-web使用[`web::Form`][formencoded]提取程序为反序列化实例解析的*application/x-www-form-urlencoded*编码bodies提供支持。实例的类型必须实现*serde*的`Deserialize` trait。

*UrlEncoded*的将来可能会在多种情况下解决错误：

* content type 不是 `application/x-www-form-urlencoded`
* transfer encoding 是 `chunked`.
* content-length 大于 256k
* payload 因错误而终止。

{{< include-example example="requests" file="urlencoded.rs" section="urlencoded" >}}

# Streaming request 流请求

*HttpRequest*是`Bytes`对象的流。它可用于读取请求body payload。

在以下示例中，我们逐块读取和打印请求的payload：

{{< include-example example="requests" file="streaming.rs" section="streaming" >}}

[examples]: https://github.com/actix/examples/tree/master/json/
[multipartstruct]: https://docs.rs/actix-multipart/0.2/actix_multipart/struct.Multipart.html
[fieldstruct]: https://docs.rs/actix-multipart/0.2/actix_multipart/struct.Field.html
[multipartexample]: https://github.com/actix/examples/tree/master/multipart/
[urlencoded]: https://docs.rs/actix-web/2/actix_web/dev/struct.UrlEncoded.html
[payloadextractor]: https://docs.rs/actix-web/2/actix_web/web/struct.Payload.html
[multipartcrate]: https://crates.io/crates/actix-multipart
[formencoded]:Jhttps://docs.rs/actix-web/2/actix_web/web/struct.Form.html
