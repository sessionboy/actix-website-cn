---
title: 应用
menu: docs_basics
weight: 140
---

# 写一个应用

`actix-web`提供了多种原语以使用Rust构建Web服务器和应用程序。
它提供路由，中间件，请求的预处理，响应的后处理等。

所有`actix-web`服务器均围绕[`App`][app]实例构建。
它用于注册resources(资源)和middlewares(中间件)的路由。
它还存储同一scope(范围/作用域)内所有处理程序之间共享的应用程序状态。

应用程序的[`scope`][scope]充当所有路由的命名空间，即特定应用程序作用域的所有路由都具有相同的url路径前缀。
应用程序前缀始终包含一个“/”斜杠。
如果提供的前缀不包含斜杠，则会自动将其插入。
前缀应包含值路径段。

> 对于范围为`/app`的应用程序
> 路径为`/app`, `/app/`, 或 `/app/test`的任何请求都将匹配；
> 但是，路径`/application`不匹配。

{{< include-example example="application" file="app.rs" section="setup" >}}

在此示例中，将创建一个带有`/app`前缀和`index.html` resource(资源)的应用程序。
该resource可通过`/app/index.html`网址获得。

> 有关更多信息，请查看[URL Dispatch][usingappprefix]部分。

## State 状态

应用程序状态与同一scope(范围)内的所有路由和resources(资源)共享。
可以使用[`web::Data<T>`][data]提取器访问状态，其中`T`是状态类型。
状态也可用于中间件。

让我们编写一个简单的应用程序，并将应用程序名称存储在以下状态：

{{< include-example example="application" file="state.rs" section="setup" >}}

并在初始化应用程序时传递状态，然后启动应用程序：

{{< include-example example="application" file="state.rs" section="start_app" >}}

可以在应用程序中注册任意数量的状态类型。

## Shared Mutable State 共享可变状态

`HttpServer`接受应用程序的factory(工厂函数)而不是应用程序实例。
Http服务器为每个线程构造一个应用程序实例，因此必须多次构造应用程序数据。
如果要在不同线程之间共享数据，则应使用可共享对象，例如
`Send + Sync`。

在内部，[`web::Data`][data]使用`Arc`。
因此，为了避免double Arc，我们应该在使用[`App::app_data()`][appdata]注册数据之前创建数据。

在下面的示例中，我们将编写一个具有可变共享状态的应用程序。
首先，我们定义状态并创建处理程序：

{{< include-example example="application" file="state.rs" section="setup_mutable" >}}

并在应用程序中注册数据：

{{< include-example example="application" file="state.rs" section="make_app_mutable" >}}

## 使用应用程序的Scope来编写应用程序

[`web::scope()`][webscope]方法允许设置特定的应用程序前缀。
此scope表示资源前缀，该前缀将附加到资源配置添加的所有资源patterns(模式)中。
这有利于路由的配置。

例如:

{{< include-example example="application" file="scope.rs" section="scope" >}}

在上面的示例中，*show_users*路由的有效路由模式为*/users/show*，而不是*/show*，因为应用程序的scope参数将添加到该模式之前。
然后，仅当URL路径为*/users/show*时，路由才会匹配，并且使用路由名称show_users调用[`HttpRequest.url_for()`][urlfor]函数时，它将生成具有相同路径的URL。

## 应用程序guards(守卫)和虚拟主机

您可以将guard视为可以接受*request*对象引用并返回*true* 或 *false*的简单函数。
形式上，guard是实现[`Guard`][guardtrait] trait的任何对象。
`Actix-web`提供了几种guards，您可以检查api文档的[functions section][guardfuncs]。

[`Header`][guardheader]提供了一种guards，它可以根据请求的header作为应用程序的过滤器。

{{< include-example example="application" file="vh.rs" section="vh" >}}

# Configure 配置

为了简单和可重用，[`App`][appconfig] 和 [`web::Scope`][webscopeconfig]都提供了`configure`方法。
该函数对于将配置的部分移动到其他模块甚至库中很有用。
例如，某些resource的配置可以移至其他模块。

{{< include-example example="application" file="config.rs" section="config" >}}

上面示例的结果将是：

```
/         -> "/"
/app      -> "app"
/api/test -> "test"
```
每个[`ServiceConfig`][serviceconfig]可以拥有自己的`data`, `routes`, 和 `services`。

[usingappprefix]: /docs/url-dispatch/index.html#using-an-application-prefix-to-compose-applications
[stateexample]: https://github.com/actix/examples/blob/master/state/src/main.rs
[guardtrait]: https://docs.rs/actix-web/2/actix_web/guard/trait.Guard.html
[guardfuncs]: https://docs.rs/actix-web/2/actix_web/guard/index.html#functions
[guardheader]: https://docs.rs/actix-web/2/actix_web/guard/fn.Header.html
[data]: https://docs.rs/actix-web/2/actix_web/web/struct.Data.html
[app]: https://docs.rs/actix-web/2/actix_web/struct.App.html
[appconfig]: https://docs.rs/actix-web/2/actix_web/struct.App.html#method.configure
[appdata]: https://docs.rs/actix-web/2/actix_web/struct.App.html#method.app_data
[scope]: https://docs.rs/actix-web/2/actix_web/struct.Scope.html
[webscopeconfig]: https://docs.rs/actix-web/2/actix_web/struct.Scope.html#method.configure
[webscope]: https://docs.rs/actix-web/2/actix_web/web/fn.scope.html
[urlfor]: https://docs.rs/actix-web/2/actix_web/struct.HttpRequest.html#method.url_for
[serviceconfig]: https://docs.rs/actix-web/2/actix_web/web/struct.ServiceConfig.html
