---
title: 什么是Actix
menu: docs_intro
weight: 100
---

# Actix是多方面的

Actix是几件事。它的基础是Rust强大的actor系统，`actix-web`最初是在该系统之上构建的。这是您最有可能使用的部分。`actix-web`为您提供了一个有趣且非常快速的Web开发框架。

我们称actix-web为小型实用的框架。
出于所有目的和目的，它都是一个微框架，带有一些曲折。
如果您已经是Rust程序员，那么您会发现很容易上手，但是即使您来自另一种编程语言，也应该会发现actix-web不难上手。

使用`actix-web`开发的应用程序将公开本机可执行文件中包含的HTTP服务器。
您可以将其放置在其他HTTP服务器（如nginx）之后，也可以原样提供。
即使完全没有其他HTTP服务器，`actix-web`也足以提供HTTP1和HTTP2的支持，以及SSL/TLS的支持。
这对于构建可分发的小型服务很有用。

最重要的是：`actix-web`在 {{< rust-version "actix-web" >}} 或更高版本上运行，并且可以与稳定版本一起使用。
