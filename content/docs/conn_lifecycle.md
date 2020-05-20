---
title: 连接生命周期
menu: docs_architecture
weight: 1030
---


# 架构概述

Server开始侦听所有sockets后，[`Accept`][Accept]和[`Worker`][Worker]是两个主要loops，负责处理传入的客户端连接(connections)。

一旦接受连接，应用程序级别的协议处理就会在从[`Worker`][Worker]派生的协议特定的[`Dispatcher`][Dispatcher] loop中发生。

    请注意，以下图表仅概述了happy-path方案。

![](/img/diagrams/connection_overview.svg)

## 更详细的Accept loop

![](/img/diagrams/connection_accept.svg)

大多数代码实现都驻留在结构[`Accept`][Accept]的[`actix-server`][server] crate中。

## 更详细的Worker loop

![](/img/diagrams/connection_worker.svg)

大多数代码实现都驻留在struct [`Worker`][Worker]的[`actix-server`][server] crate中。

## 大致的Request loop

![](/img/diagrams/connection_request.svg)

request loop的大多数代码实现都驻留在[`actix-web`][web] 和 [`actix-http`][http] crates中。

[server]: https://crates.io/crates/actix-server
[web]: https://crates.io/crates/actix-web
[http]: https://crates.io/crates/actix-http
[Accept]: https://github.com/actix/actix-net/blob/master/actix-server/src/accept.rs
[Worker]: https://github.com/actix/actix-net/blob/master/actix-server/src/worker.rs
[Dispatcher]: https://github.com/actix/actix-web/blob/master/actix-http/src/h1/dispatcher.rs
