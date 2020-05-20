---
title: 中间件
menu: docs_advanced
weight: 220
---

# Middleware 中间件

Actix-web的中间件系统使我们可以在request/response处理中添加其他行为。中间件可以挂接到传入的请求过程中，使我们能够修改请求以及暂停请求处理以尽早返回响应。

中间件也可以加入响应处理。

通常，中间件参与以下操作：

* 预处理请求
* 后处理响应
* 修改应用程序状态
* 访问外部服务（redis，日志记录，会话）

中间件为每个`App`, `scope`, 或 `Resource`注册，并以与注册相反的顺序执行。
通常，中间件是一种实现[*Service trait*][servicetrait]和[*Transform trait*][transformtrait]的类型。
trait中的每个方法都有一个默认实现。
每个方法都可以立即返回结果或*future*对象。

以下演示了如何创建一个简单的中间件：

{{< include-example example="middleware" file="main.rs" section="simple" >}}

另外，对于简单的用例，可以使用[*wrap_fn*][wrap_fn]创建临时的小型中间件：

{{< include-example example="middleware" file="wrap_fn.rs" section="wrap-fn" >}}

> Actix-web提供了几种有用的中间件，例如*logging*(日志记录)，*user sessions*(用户会话)，*compress*(压缩)等。

# Logging 日志

日志记录是作为中间件实现的。通常将日志记录中间件注册为该应用程序的第一个中间件。日志中间件必须为每个应用程序注册。

`Logger`中间件使用标准日志crate记录信息。您应该为`actix_web`软件包启用`Logger`，以查看访问日志（[env_logger][envlogger]或类似记录）。

## Usage 用法

使用指定的`format`创建`Logger`中间件。可以使用`default`方法创建默认`Logger`，它使用默认格式：

```ignore
  %a %t "%r" %s %b "%{Referer}i" "%{User-Agent}i" %T
```

{{< include-example example="middleware" file="logger.rs" section="logger" >}}

以下是默认日志记录格式的示例：

```
INFO:actix_web::middleware::logger: 127.0.0.1:59934 [02/Dec/2017:00:21:43 -0800] "GET / HTTP/1.1" 302 0 "-" "curl/7.54.0" 0.000397
INFO:actix_web::middleware::logger: 127.0.0.1:59947 [02/Dec/2017:00:22:40 -0800] "GET /index.html HTTP/1.1" 200 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.13; rv:57.0) Gecko/20100101 Firefox/57.0" 0.000646
```

## Format 格式

- `%%`  百分号
- `%a`  远程IP地址（如果使用反向代理，则为代理的IP地址）
- `%t`  请求开始处理的时间
- `%P`  为请求提供服务的child的进程ID 
- `%r`  第一行request
- `%s`  响应状态码
- `%b`  响应大小（以字节为单位），包括HTTP headers(标头)
- `%T`  服务请求所用的时间，以秒为单位，浮动分数为.06f格式 
- `%D`  serve(服务)请求所花费的时间（以毫秒为单位）
- `%{FOO}i`  request.headers['FOO']
- `%{FOO}o`  response.headers['FOO']
- `%{FOO}e`  os.environ['FOO']

## Default headers

要设置默认response headers(响应头)，可以使用`DefaultHeaders`中间件。如果响应头已经包含指定的header，则`DefaultHeaders`中间件不会设置header。

{{< include-example example="middleware" file="default_headers.rs" section="default-headers" >}}

## User sessions

Actix-web提供了session管理的通用解决方案。 [**actix-session**][actixsession]中间件可以与不同的后端类型一起使用，以将session数据存储在不同的后端中。

> 默认情况下，仅实现cookie session后端。可以添加其他后端实现。

[**CookieSession**][cookiesession]使用cookie作为session存储。
`CookieSessionBackend`创建的sessions仅限于存储少于4000字节的数据，因为payload必须适合单个cookie。
如果session超过4000字节，则会生成内部服务器错误。

Cookie可能具有*signed*(签名)或 *private*(私有)安全策略。每个都有各自的`CookieSession`构造函数。

客户端可以查看但不能修改已*signed*(签名)的cookie。客户端既不能查看也不可以修改私有cookie。

构造函数将key作为参数。这是cookie session的私钥-更改此值时，所有session数据都会丢失。

通常，您将创建一个`SessionStorage`中间件，并使用特定的后端实现（例如`CookieSession`）对其进行初始化。
要访问session数据，必须使用[`Session`][requestsession]提取器。
此方法返回一个[*Session*][sessionobj]对象，该对象允许我们获取或设置session数据。

{{< include-example example="middleware" file="user_sessions.rs" section="user-session" >}}

# Error handlers 错误处理

`ErrorHandlers`中间件允许我们提供响应的自定义处理程序。

您可以使用`ErrorHandlers::handler()`方法为特定状态码注册自定义错误处理程序。
您可以修改现有的响应，也可以创建一个全新的响应。
错误处理程序可以立即返回响应，也可以返回解析为响应的Future。

{{< include-example example="middleware" file="errorhandler.rs" section="error-handler" >}}

[sessionobj]: https://docs.rs/actix-session/0.3.0/actix_session/struct.Session.html
[requestsession]: https://docs.rs/actix-session/0.3.0/actix_session/struct.Session.html
[cookiesession]: https://docs.rs/actix-session/0.3.0/actix_session/struct.CookieSession.html
[actixsession]: https://docs.rs/actix-session/0.3.0/actix_session/
[envlogger]: https://docs.rs/env_logger/*/env_logger/
[servicetrait]: https://docs.rs/actix-web/2/actix_web/dev/trait.Service.html
[transformtrait]: https://docs.rs/actix-web/2/actix_web/dev/trait.Transform.html
[wrap_fn]: https://docs.rs/actix-web/2/actix_web/struct.App.html#method.wrap_fn
