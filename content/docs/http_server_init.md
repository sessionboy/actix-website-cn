---
title: Http服务器初始化
menu: docs_architecture
weight: 1020
---

## 架构概述

下面是HttpServer初始化的示意图，它发生在以下代码上

```rust
#[actix_rt::main]
async fn main() -> std::io::Result<()> {
    HttpServer::new(|| {
        App::new()
            .route("/", web::to(|| HttpResponse::Ok()))
    })
    .bind("127.0.0.1:8088")?
    .run()
    .await
}
```

![](/img/diagrams/http_server.svg)
