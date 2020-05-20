---
title: 错误处理
menu: docs_advanced
weight: 180
---

# 错误处理

Actix-web使用自己的[`actix_web::error::Error`][actixerror]类型和[`actix_web::error::ResponseError`][responseerror] trait来从Web处理程序处理错误。

如果handler在还实现了`ResponseError` trait的`Result`中返回`Error`（指的是[一般的Rust trait `std::error::Error`][stderror]），则actix-web会将错误与相应的[`actix_web::http::StatusCode`][status_code]一起呈现为HTTP响应。

默认情况下会生成内部服务器错误：

```rust
pub trait ResponseError {
    fn error_response(&self) -> HttpResponse;
    fn status_code(&self) -> StatusCode;
}
```

`Responder`将兼容`Result`强制转换为HTTP响应：

```rust
impl<T: Responder, E: Into<Error>> Responder for Result<T, E>
```

上面代码中的`Error`是actix-web的error定义，任何实现`ResponseError`的errors都可以自动转换为一个。

Actix-web提供了一些常见的非actix错误的`ResponseError`实现。例如，如果handler以`io::Error`响应，则该错误将转换为`HttpInternalServerError`：

```rust
use std::io;

fn index(_req: HttpRequest) -> io::Result<fs::NamedFile> {
    Ok(fs::NamedFile::open("static/index.html")?)
}
```

有关`ResponseError`的外部实现的完整列表，请参见[actix-web API文档][responseerrorimpls]。

## 自定义错误响应的示例 

这是`ResponseError`的示例实现：

{{< include-example example="errors" file="main.rs" section="response-error" >}}

`ResponseError`具有`error_response()`的默认实现，它将呈现_500_ （服务器内部错误），并且在上面执行`index`处理程序时会发生这种情况。 

覆盖`error_response()`以产生更多有用的结果：

{{< include-example example="errors" file="override_error.rs" section="override" >}}

# Error 助手

Actix-web提供了一组error helper函数，这些函数可用于从其他错误中生成特定的HTTP错误代码。
在这里，我们使用`map_err`将未实现`ResponseError`trait的`MyError`转换为_400_（错误请求）：

{{< include-example example="errors" file="helpers.rs" section="helpers" >}}

有关可用的错误帮助程序的完整列表，请查看[actix-web错误模块的API文档][actixerror]。

# 兼容故障

Actix-web提供与[failure](故障)库的自动兼容性，以便将导致故障的错误自动转换为actix错误。
请记住，这些错误将使用默认的_500_状态码呈现，除非您还为其提供了自己的`error_response()`实现。

# Error logging 错误日志

Actix在`WARN`日志级别记录所有错误。如果应用程序的日志级别设置为`DEBUG`，并且启用了`RUST_BACKTRACE`，则还将记录backtrace(回溯)。这些可以使用环境变量进行配置：

```
>> RUST_BACKTRACE=1 RUST_LOG=actix_web=debug cargo run
```

`Error` 类型使用cause的错误回溯（如果有）。如果根本的故障不提供回溯，则构造一个新的回溯，指向发生转换的点（而不是错误的根源）。

# 错误处理的推荐做法

考虑将应用程序产生的错误分为两大类可能是有用的：一类是面向用户的，二类不是面向用户的。

前者的一个示例是，我可能使用failure(失败)指定`UserError`枚举，该枚举封装了一个`ValidationError`，以便在用户发送错误的输入时返回：

{{< include-example example="errors" file="recommend_one.rs" section="recommend-one" >}}

这将完全按照预期的方式运行，因为用`display`定义的错误消息是在明确意图下被用户读取的。

但是，并非所有错误都希望发送回错误消息-在服务器环境中会发生很多故障，我们可能希望向用户隐藏具体信息。
例如，如果数据库关闭并且客户端库开始产生连接超时错误，或者HTML模板格式不正确，并且渲染时发生错误。
在这些情况下，最好将错误映射为适合用户使用的一般错误。

这是一个使用自定义消息将内部错误映射到面向用户的`InternalError`的示例：

{{< include-example example="errors" file="recommend_two.rs" section="recommend-two" >}}

通过将错误分为面对用户的错误和不面对用户的错误，我们可以确保我们不会意外地使用户暴露于应用程序内部抛出的错误中，而这些错误并非本应引起的。

# Error Logging 错误日志

这是使用`middleware::Logger`的基本示例：

{{< include-example example="errors" file="logging.rs" section="logging" >}}

[actixerror]: https://docs.rs/actix-web/2/actix_web/error/struct.Error.html
[errorhelpers]: https://docs.rs/actix-web/2/actix_web/trait.ResponseError.html
[failure]: https://github.com/rust-lang-nursery/failure
[responseerror]: https://docs.rs/actix-web/2/actix_web/error/trait.ResponseError.html
[responseerrorimpls]: https://docs.rs/actix-web/2/actix_web/error/trait.ResponseError.html#foreign-impls
[stderror]: https://doc.rust-lang.org/std/error/trait.Error.html
[status_code]: https://docs.rs/actix-web/2.0.0/actix_web/http/struct.StatusCode.html
