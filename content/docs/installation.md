---
title: 安装
menu: docs_intro
weight: 110
---

# 安装Rust

由于`actix-web`是Rust框架，因此您将需要Rust来开始使用。如果您还没有的话，我们建议您使用`rustup`来管理Rust安装。[官方rust指南][rustguide]包含有关入门的精彩内容。 

目前，我们至少需要Rust {{< rust-version "actix-web" >}}，因此请确保您 `rustup update`具有最新和最好的Rust版本。特别是，本指南将假定您实际运行Rust {{< rust-version "actix-web" >}}或更高版本。

# 安装 `actix-web`

感谢Rust的`cargo`包管理器，您无需显式安装`actix-web`。
只需依靠它，您就可以开始了。
对于不太可能要使用`actix-web`开发版本的情况，可以直接依赖git存储库。

发布版本:

```ini
[dependencies]
actix-web = "{{< actix-version "actix-web" >}}"
```

开发版本:

```ini
[dependencies]
actix-web = { git = "https://github.com/actix/actix-web" }
```

# Diving In

您可以在此处采取两种方法。
您可以按照指南进行操作，如果您不耐烦，则可以查看我们[广泛的示例存储库][examples]并运行其中的示例。
例如，这里是如何运行包含的基本示例：

```
git clone https://github.com/actix/examples
cd examples/basics
cargo run
```

[rustguide]: https://doc.rust-lang.org/book/ch01-01-installation.html
[examples]: https://github.com/actix/examples
