---
title: 热更新
menu: docs_patterns
weight: 1000
---

# 自动重新加载开发服务器

在开发过程中，让cargo在更改时自动重新编译代码会非常方便。
这可以通过使用[cargo-watch][cargowatch]来完成。
因为actix应用程序通常将绑定到端口以侦听传入的HTTP请求，所以将其与[listenfd][listenfd] crate和[systemfd][systemfd]实用程序结合使用以确保socket在应用程序编译和重新加载时保持打开状态是有意义的。

`systemfd`将打开一个socket，并将其传递给`cargo-watch`，它将监听更改，然后调用编译器并运行actix应用。然后，actix应用程序将使用`listenfd`来拾取`systemfd`打开的socket。

## Binaries Necessary 必要的二进制文件

为了获得热更新的体验，您需要安装`cargo-watch`和`systemfd`。两者都是用Rust编写的，可以与`cargo install`一起安装：

```
cargo install systemfd cargo-watch
```

## 代码变更

另外，您需要稍微修改一下actix应用程序，以便它可以使用`systemfd`打开的外部socket。将`listenfd`依赖项添加到您的应用程序：

```ini
[dependencies]
listenfd = "0.3"
```

然后修改您的服务器代码以仅调用`bind`作为fallback：

{{< include-example example="autoreload" file="main.rs" section="autoreload" >}}

## 运行服务器

现在要运行开发服务器，请调用以下命令：

```
systemfd --no-pid -s http::3000 -- cargo watch -x run
```

[cargowatch]: https://github.com/passcod/cargo-watch
[listenfd]: https://crates.io/crates/listenfd
[systemfd]: https://github.com/mitsuhiko/systemfd
