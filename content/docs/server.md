---
title: Server 服务器
menu: docs_basics
weight: 150
---

# HTTP 服务器

[**HttpServer**][httpserverstruct]类型负责处理http请求。

`HttpServer`接受应用程序的factory作为参数，并且该应用程序factory必须具有`Send` + `Sync`边界。
有关更多信息，请参见*multi-threading*部分。

要绑定到特定的socket address(套接字地址)，必须使用[`bind()`][bindmethod]，并且可以多次调用它。
要绑定ssl socket，应使用[`bind_openssl()`][bindopensslmethod]或[`bind_rustls()`][bindrusttls]。
要运行http服务器，请使用`HttpServer::run()`方法。

{{< include-example example="server" section="main" >}}

`run()`方法返回[`Server`][server]类型的实例。
server类型的方法可用于管理http服务器

- `pause()` - 暂停接受传入的连接
- `resume()` - 恢复接受传入的连接 
- `stop()` - 停止传入连接处理，停止所有worker程序并退出 

以下示例显示如何在单独的线程中启动http服务器。

{{< include-example example="server" file="signals.rs" section="signals" >}}

## 多线程

`HttpServer`自动启动许多http *workers*，默认情况下，该数量等于系统中逻辑CPU的数量。
可以使用[`HttpServer::workers()`][workers]方法覆盖此数字。

{{< include-example example="server" file="workers.rs" section="workers" >}}


创建workers程序后，每个worker都会收到一个单独的应用程序实例来处理请求。
线程之间不共享应用程序状态，并且处理程序可以自由操作它们的state(状态)副本，而无需担心并发问题。

> 应用程序状态不需要为`Send` 或 `Sync`，但是应用程序factory必须为`Send` + `Sync`。

要在辅助线程之间共享状态，请使用`Arc`。
引入共享和synchronization(同步)后，应格外小心。
在许多情况下，由于锁定共享状态以进行修改而无意中引入了性能成本。

在某些情况下，可以使用更有效的locking(锁定)策略来减轻这些成本，例如，使用[read/write locks](https://doc.rust-lang.org/std/sync/struct.RwLock.html)而不是[mutexes](https://doc.rust-lang.org/std/sync/struct.Mutex.html)来实现非排他性锁定，但是性能最高的实现通常往往是根本不发生locking(锁定)的实现。

由于每个worker线程都按顺序处理其请求，因此阻塞当前线程的处理程序将导致当前worker线程停止处理新请求：

```rust
fn my_handler() -> impl Responder {
    std::thread::sleep(Duration::from_secs(5)); // <-- Bad practice! Will cause the current worker thread to hang!
    "response"
}
```

因此，任何长时间的，不受CPU约束的操作（例如I/O，数据库操作等）都应表示为futures(类似于js的promise)或asynchronous(异步)函数。
异步处理程序由worker线程并发执行，因此不会阻塞：

```rust
async fn my_handler() -> impl Responder {
    tokio::time::delay_for(Duration::from_secs(5)).await; // <-- Ok. Worker thread will handle other requests here
    "response"
}
```

同样的limitation(限制)也适用于提取器。
当`handler`函数接收到implements `FromRequest`的参数，并且该implementation阻塞当前线程时，worker线程将在运行处理程序时阻塞。
为此，在实现提取器时必须特别注意，并且在需要时也应implemented asynchronously(异步实现)它们。

## SSL

ssl服务器有两个功能：`rustls`和`openssl`。
`rustls`功能用于`rustls`集成，`openssl`用于`openssl`。

```toml
[dependencies]
actix-web = { version = "{{< actix-version "actix-web" >}}", features = ["openssl"] }
openssl = { version="0.10" }
```

{{< include-example example="server" file="ssl.rs" section="ssl" >}}

> **注意**：*HTTP/2.0*协议需要[tls alpn][tlsalpn]。
> 目前，只有`openssl`支持`alpn`。
> 有关完整的示例，请查看[examples/openssl][exampleopenssl]。

要创建`key.pem`和`cert.pem`，请使用命令。
**填写自己的主题**

```bash
$ openssl req -x509 -newkey rsa:4096 -keyout key.pem -out cert.pem \
  -days 365 -sha256 -subj "/C=CN/ST=Fujian/L=Xiamen/O=TVlinux/OU=Org/CN=muro.lxd"
```

要删除密码，然后将nopass.pem复制到key.pem

```bash
$ openssl rsa -in key.pem -out nopass.pem
```

## Keep-Alive

Actix可以在keep-alive连接上等待请求。

> *keep alive*连接行为是由服务器设置定义的。

- `75`, `Some(75)`, `KeepAlive::Timeout(75)` - 启用75秒*keep alive*计时器。
- `None` 或 `KeepAlive::Disabled` - 禁用 *keep alive*.
- `KeepAlive::Tcp(75)` - 使用`SO_KEEPALIVE` socket选项。

{{< include-example example="server" file="keep_alive.rs" section="keep-alive" >}}

如果选择了上面的第一个选项，则根据response的*connection-type*来计算*keep alive*状态。
默认情况下，未定义`HttpResponse::connection_type`。
在这种情况下，*keep alive*状态是由request的http版本定义的。

> 对于*HTTP/1.0*，*keep alive*为**off**状态；对于*HTTP/1.1* and *HTTP/2.0*，*keep alive*为**on**状态。

可以使用`HttpResponseBuilder::connection_type()`方法更改*Connection type*。

{{< include-example example="server" file="keep_alive_tp.rs" section="example" >}}

## Graceful shutdown 正常关机

`HttpServer`支持正常关闭。
收到stop(停止)信号后，workers将有特定的时间来完成服务请求。
超时后仍存活的所有workers都被强制撤销。
默认情况下，shutdown timeout(关机超时)设置为30秒。
您可以使用[`HttpServer::shutdown_timeout()`][shutdowntimeout]方法更改此参数。

您可以使用server address(服务器地址)向服务器发送stop(停止)消息，并指定是否要正常shutdown(关机)。
[`start()`][startmethod]方法返回服务器的地址。

`HttpServer`处理多个OS信号。
*CTRL-C*在所有操作系统上均可用，其他信号在`UNIX`系统上均可用。

- *SIGINT* - 强制shutdown workers
- *SIGTERM* - 优雅地shutdown workers
- *SIGQUIT* - 强制shutdown workers

> 可以使用[`HttpServer::disable_signals()`][disablesignals]方法禁用信号处理。

[server]: https://docs.rs/actix-web/2/actix_web/dev/struct.Server.html
[httpserverstruct]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html
[bindmethod]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.bind
[bindopensslmethod]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.bind_openssl
[bindrusttls]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.bind_rustls
[startmethod]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.start
[workers]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.workers
[tlsalpn]: https://tools.ietf.org/html/rfc7301
[exampleopenssl]: https://github.com/actix/examples/blob/master/openssl
[shutdowntimeout]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.shutdown_timeout
[disablesignals]: https://docs.rs/actix-web/2/actix_web/struct.HttpServer.html#method.disable_signals
