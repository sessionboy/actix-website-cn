---
title: 入门指南
menu: docs_basics
weight: 130
---

# 入门指南

让我们编写第一个`actix-web`应用程序！

## Hello, world!

首先创建一个新的基于二进制的Cargo项目，然后切换到新目录：

```bash
cargo new hello-world
cd hello-world
```

现在，通过确保`Cargo.toml`包含以下内容，将`actix-web`添加为项目的依赖项：

```ini
[dependencies]
actix-web = "{{< actix-version "actix-web" >}}"
```

如果要使用`#[actix_rt::main]`宏，则必须将`actix-rt`添加到依赖项中。
现在，您的`Cargo.toml`应该如下所示：

```ini
[dependencies]
actix-web = "{{< actix-version "actix-web" >}}"
actix-rt = "{{< actix-version "actix-rt" >}}"
```
为了实现Web服务器，我们首先需要创建一个请求处理程序。
请求处理程序是一个异步函数，它接受零个或多个可以从请求中提取的参数（即`impl FromRequest`），并返回可以转换为`HttpResponse`的类型（即`impl Responder`）：

{{< include-example example="getting-started" section="setup" >}}

接下来，创建一个`App`实例，使用_path_和特定的_HTTP方法_向应用程序的路由注册请求处理程序。
之后，该应用程序实例可以与`HttpServer`一起使用以监听传入的连接。
服务器接受应返回给应用程序的factory函数。

{{< include-example example="getting-started" section="main" >}}

现在，使用`cargo run`编译并运行程序。
然后打开`http://localhost:8088/`查看结果。

**注意**：您可能会注意到`#[actix_rt::main]`属性宏。
该宏在actix运行时中执行标记的async函数。
任何async函数都可以通过此宏进行标记和执行。

### 使用属性宏定义路由

另外，您可以使用宏属性定义路由，这些宏属性允许您在函数上方指定路由，如下所示：

{{< include-example example="getting-started" section="macro-attributes">}}

然后，您可以使用`service()`注册route：

```rust
App::new()
    .service(index3)
```

出于一致性原因，本文档仅使用本页开头所示的显式语法。
但是，如果您更喜欢这种语法，那么在声明路由时就可以随意使用它，因为它只是语法糖。

要了解更多信息，请查看[actix-web-codegen]。

### 热更新

如果需要，您可以在开发期间拥有一个自动重新加载(热更新)服务器，该服务器可以按需重新编译。
这不是必需的，但是它使得快速开发更加方便，因为您可以在保存后立即看到更改。
要了解如何完成此操作，请查看[autoreload pattern][autoload]。

[actix-web-codegen]: https://docs.rs/actix-web-codegen/
[autoload]: ../autoreload/
