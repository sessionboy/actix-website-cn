---
title: HTTP/2.0
menu: docs_proto
weight: 250
---

如果可能，`actix-web`会自动将连接升级到*HTTP/2.0*。

# Negotiation 协商

在没有先验知识的情况下，基于tls的*HTTP/2.0*协议需要[tls alpn][tlsalpn]。 

> 目前，只有`rust-openssl`支持。

`alpn`协商需要启用该功能。启用后，`HttpServer`将提供[bind_openssl][bindopenssl]方法。

```toml
[dependencies]
actix-web = { version = "{{< actix-version "actix-web" >}}", features = ["openssl"] }
actix-rt = "1.0.0"
openssl = { version = "0.10", features = ["v110"] }
```
{{< include-example example="http2" file="main.rs" section="main" >}}

不支持升级到[rfc section 3.2][rfcsection32]中描述的*HTTP/2.0*模式。明文连接和tls连接都支持使用prior knowledge(先验知识)启动*HTTP/2*。 [rfc section 3.4][rfcsection34]。

> 请查看[examples/tls][examples]以获取具体示例。

[rfcsection32]: https://http2.github.io/http2-spec/#rfc.section.3.2
[rfcsection34]: https://http2.github.io/http2-spec/#rfc.section.3.4
[bindopenssl]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.bind_openssl
[tlsalpn]: https://tools.ietf.org/html/rfc7301
[examples]: https://github.com/actix/examples/tree/master/rustls
