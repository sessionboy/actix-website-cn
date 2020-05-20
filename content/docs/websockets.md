---
title: Websockets
menu: docs_proto
weight: 240
---

Actix-web支持带`actix-web-actors` crate的WebSocket。
可以使用[*web::Payload*][payload]将请求的`Payload`转换为[*ws::Message*][message]流，然后使用流组合器来处理实际消息，但处理与http actor的websocket通信更简单。

以下是一个简单的websocket echo服务器的示例：

{{< include-example example="websockets" file="main.rs" section="websockets" >}}

> [示例目录][examples]中提供了一个简单的websocket echo服务器示例。


> [websocket-chat directory][chat]中提供了一个示例聊天服务器，可以通过websocket或tcp连接进行聊天

[message]: https://docs.rs/actix-web-actors/2/actix_web_actors/ws/enum.Message.html
[payload]: https://docs.rs/actix-web/2/actix_web/web/struct.Payload.html
[examples]: https://github.com/actix/examples/tree/master/websocket/
[chat]: https://github.com/actix/examples/tree/master/websocket-chat/
