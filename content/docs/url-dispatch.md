---
title: URL Dispatch
menu: docs_advanced
weight: 190
---

# URL Dispatch

URL dispatch 提供了一种使用简单的模式匹配语言将URL映射到handler(处理程序)代码的简单方法。如果其中一种模式和与请求相关联的路径信息匹配，则调用特定的处理程序对象。

请求处理程序是一个函数，该函数接受零个或多个可以从请求中提取的参数（即[*impl FromRequest*][implfromrequest]），并返回可以转换为`HttpResponse`的类型（即[*impl Responder*][implresponder]）。
有关更多信息，请查看[handler section][handlersection]。

# Resource configuration 资源配置

资源配置是向应用程序添加新资源的动作。
resource具有名称，该名称用作要用于URL生成的标识符。
该名称还允许开发人员将路由添加到现有resource。
resource还具有一种模式，旨在与URL的PATH部分（scheme和端口之后的部分，例如*URL* *http://localhost:8080/foo/bar?q=value*)中的*/foo/bar*）相匹配。
。
它与*QUERY*部分（跟在*?*之后的部分不匹配，例如*http://localhost:8080/foo/bar?q=value*中的*q=value*）。

[*App::route()*][approute]方法提供了注册路由的简单方法。
此方法将单个路由添加到应用程序路由表。
此方法接受*path pattern*(路径模式)，http方法和handler函数。
可以为同一路径多次调用`route()`方法，在这种情况下，多个路由会注册同一resource路径。

{{< include-example example="url-dispatch" section="main" >}}

虽然*App::route()*提供了注册路由的简单方法，但要访问完整的resource配置，则必须使用其他方法。
[*App::service()*][appservice]方法将单个[resource][webresource]添加到应用程序路由表。
此方法接受*path pattern*，guards，以及一个或多个routes。

{{< include-example example="url-dispatch" file="resource.rs" section="resource" >}}

如果资源不包含任何route或不具有任何匹配的route，则它将返回*NOT FOUND* http响应。

## 配置Route

Resource包含一组routes。
每个route依次具有一组`guards`和一个handler。
可以使用`Resource::route()`方法创建新route，该方法返回对新Route实例的引用。
默认情况下，route不包含任何guards，因此匹配所有请求，默认handler为`HttpNotFound`。  

该应用程序routes根据在resource注册和route注册期间定义的route标准传入requests。
Resource按照通过`Resource::route()`注册routes的顺序匹配它包含的所有路由。

> 一个*Route*可以包含任意数量的*guards*，但只能包含一个handler。

{{< include-example example="url-dispatch" file="cfg.rs" section="cfg" >}}

在此示例中，如果请求包含`Content-Type` header，此header的值为 *text/plain*且path等于`/path`，则为GET请求返回`HttpResponse::Ok()`。

如果resource无法匹配任何路由，则返回"NOT FOUND"响应。

[*ResourceHandler::route()*][resourcehandler]返回[*Route*][route]对象。可以使用类似构建器的模式来配置路由。提供以下配置方法：

* [*Route::guard()*][routeguard]  注册新的guard。每个route可以注册任意数量的guard。
* [*Route::method()*][routemethod] 注册一个方法guard。每个route可以注册任意数量的guard。
* [*Route::to()*][routeto] 注册此route的handler函数。只能注册一个handler。通常，handler注册是最后的配置操作。
* [*Route::to_async()*][routetoasync] 为该route注册一个异步handler函数。只能注册一个handler。通常，handler是最后的配置操作。  

# Route matching 路由匹配

路由配置的主要目的是使请求的route与URL路径模式匹配（或不匹配）。 `path`代表所请求URL的路径部分。

*actix-web*做到这一点的方法非常简单。
当请求进入系统时，对于系统中存在的每个resource配置声明，actix都会根据声明的模式检查请求的路径。
该检查按照通过`App::service()`方法声明routes的顺序进行。
如果找不到resource，则将默认resource用作匹配的resource。

声明路由配置时，它可能包含路由guard参数。
与路由声明关联的所有路由guard必须为true，才能在检查期间将路由配置用于给定请求。
如果在检查过程中提供给路由配置的一组路由guard参数中的任何guard返回false，则该路由将被跳过，并且路由匹配将继续通过有序路由集合。

如果有任何路由匹配，则路由匹配过程停止，并调用与该路由关联的handler(处理程序)。如果在所有路由模式都用尽之后没有路由匹配，则返回*NOT FOUND*响应。

# Resource pattern syntax 模式语法

actix在pattern参数中使用的模式匹配语言的语法很简单。
路由配置中使用的模式可以以斜杠字符开头。  

如果该模式不以斜杠字符开头，则在匹配时将在其前面添加一个隐式斜杠。
例如，以下模式是等效的：

```
{foo}/bar/baz
```

和:

```
/{foo}/bar/baz
```

以{identifier}的形式指定可变部分（替换标记），其含义是“接受直到下一个斜杠字符的任何字符，并将其用作`HttpRequest.match_info()`对象中的名称”。

模式中的替换标记与正则表达式`[^{}/]+`相匹配。

match_info是 `Params`对象，代表基于路由模式从URL提取的动态部分。
它可以作为`request.match_info`使用。
例如，以下模式定义一个文字段（foo）和两个替换标记（baz和bar）：

```
foo/{baz}/{bar}
```

上面的模式将匹配这些URL，生成以下匹配信息：

```
foo/1/2        -> Params {'baz':'1', 'bar':'2'}
foo/abc/def    -> Params {'baz':'abc', 'bar':'def'}
```

但是，它将不符合以下模式：

```
foo/1/2/        -> No match (trailing slash)
bar/abc/def     -> First segment literal mismatch
```

匹配一个segment(段)中segment替换标记将仅进行到模式中的segment中第一个非字母数字字符为止。因此，例如，如果使用此路由模式：

```
foo/{name}.html
```

文字路径*/foo/biz.html*将与上述路由模式匹配，并且匹配结果将为`Params{'name': 'biz'}`。
但是，文字路径*/foo/biz*将不匹配，因为它在由*{name}.html*表示的段末尾不包含文字*.html*（它仅包含biz，而不包含biz.html）。

要捕获两个片段，可以使用两个替换标记：

```
foo/{name}.{ext}
```

literal path(文字路径)*/foo/biz.html*将与上述路由模式匹配，匹配结果将为*Params{'name': 'biz', 'ext': 'html'}*。发生这种情况是因为*.*字面部分。 （期间）两个替换标记*{name}*和*{ext}*之间。

Replacement markers(替换标记)可以选择指定正则表达式，该正则表达式将用于确定路径段是否应与标记匹配。
若要指定替换标记仅应匹配正则表达式定义的一组特定字符，则必须使用稍微扩展形式的替换标记语法。
在大括号内，替换标记名称必须后跟冒号，然后是正则表达式。
与替换标记*[^/]+*关联的默认正则表达式匹配一个或多个非斜杠字符。
例如，在内部，替换标记{foo}可以更详细地拼写为*{foo:[^/]+}*。
您可以将其更改为任意正则表达式以匹配任意字符序列，例如*{foo:\d+}*以仅匹配数字。

Segments(段)必须至少包含一个字符才能匹配segment替换标记。例如，对于URL */abc/*：

* */abc/{foo}* 将不匹配。
* */{foo}/* 将匹配。

> **注意**：在匹配模式之前，将对路径进行URL-unquoted并将其解码为有效的Unicode字符串，并且表示匹配路径段的值也将被URL-unquoted。

因此，例如，以下模式：

```
foo/{bar}
```

匹配以下URL时：

```
http://example.com/foo/La%20Pe%C3%B1a
```

matchdict看起来像这样（值是URL-decoded）：

```
Params{'bar': 'La Pe\xf1a'}
```

路径段中的文字字符串应代表提供给actix的路径的decoded(解码)值。您不希望在模式中使用URL-encoded(URL编码)的值。例如，而不是这样：

```
/Foo%20Bar/{baz}
```

您将要使用以下内容：

```
/Foo Bar/{baz}
```

有可能获得"tail match"。为此，必须使用自定义正则表达式。

```
foo/{bar}/{tail:.*}
```

上面的模式将匹配这些URL，生成以下匹配信息：

```
foo/1/2/           -> Params{'bar':'1', 'tail': '2/'}
foo/abc/def/a/b/c  -> Params{'bar':u'abc', 'tail': 'def/a/b/c'}
```

# Scoping Routes 范围路由

范围界定可以帮助您组织共享公共根路径的路由。您可以将范围嵌套在范围内。

假设您要组织用于查看"Users"的端点的路径。这些路径可能包括：

- /users
- /users/show
- /users/show/{id}

这些路径的scoped布局如下所示

{{< include-example example="url-dispatch" file="scope.rs" section="scope" >}}

*scoped*路径可以包含可变路径段作为resources。与unscoped的路径一致。 

您可以从`HttpRequest::match_info()`获得可变路径段。[`Path` extractor][pathextractor]还能够提取范围级别的变量段。

# Match information 匹配信息

[`HttpRequest::match_info`][matchinfo]中提供了表示匹配路径段的所有值。可以使用[`Path::get()`][pathget]检索特定值。

{{< include-example example="url-dispatch" file="minfo.rs" section="minfo" >}}

对于路径'/a/1/2/'的此示例，值v1和v2将解析为“1”和“2”。

可以从尾部路径参数创建`PathBuf`。
返回的`PathBuf`是百分比解码的。
如果segment等于“..”，则跳过前一个segment（如果有）。
为了安全起见，如果segment满足以下任一条件，则返回Err表示满足条件：

* 解码后的片段以以下任意一个开头: `.` (除了 `..`), `*`
* 解码的段以以下任意一个结尾: `:`, `>`, `<`
* 解码的段包含以下任何内容: `/`
* 在Windows上，已解码的段包含以下任何内容：: '\'
* 百分比编码导致无效的UTF8.

这些条件的结果是，从请求路径参数解析的`PathBuf`可以安全地插在路径中或用作路径的后缀，而无需其他检查。

{{< include-example example="url-dispatch" file="pbuf.rs" section="pbuf" >}}

## 路径信息提取器

Actix提供了用于类型安全路径信息提取的功能。
[*Path*][pathstruct]提取信息，destination类型可以用几种不同的形式定义。
最简单的方法是使用元组类型。
元组中的每个元素必须对应于路径模式中的一个元素。
也就是说，您可以将路径模式`/{id}/{username}/`与`Path<(u32, String)>`类型进行匹配，但是`Path<(String, String, String)>`类型将始终失败。

{{< include-example example="url-dispatch" file="path.rs" section="path" >}}

也可以将路径模式信息提取到struct(结构)。在这种情况下，此结构必须实现*serde的*Deserialize特性。

{{< include-example example="url-dispatch" file="path2.rs" section="path" >}}

[*Query*][query]为请求查询参数提供了类似的功能。

# 生成resource URLs

使用[*HttpRequest.url_for()*][urlfor]方法可基于resource模式生成URL。例如，如果您使用名称“foo”和模式"{a}/{b}/{c}"配置了资源，则可以执行以下操作：

{{< include-example example="url-dispatch" file="urls.rs" section="url" >}}

这将返回类似字符串*http://example.com/test/1/2/3*的信息（至少在当前协议和主机名暗含http://example.com的情况下）。
`url_for()`方法返回[*Url object*][urlobj]，因此您可以修改此url（添加查询参数，锚点等）。
只能为*named* resources调用`url_for()`，否则将返回错误。

# 外部 resources

可以将有效URLs的Resources注册为外部Resources。它们仅用于生成URL，而在请求时从不考虑匹配。

{{< include-example example="url-dispatch" file="url_ext.rs" section="ext" >}}

# 路径规范化并重定向到斜杠附加的路由

通过规范化意味着:

* 在路径上添加斜杠。
* 用一个代替多个斜杠。

handler找到正确解析的路径后立即返回。
如果全部启用，则规范化条件的顺序为1）合并，2）合并和附加以及3）附加。
如果路径至少满足这些条件之一，则它将重定向到新路径。

{{< include-example example="url-dispatch" file="norm.rs" section="norm" >}}

在此示例中，`//resource///`将重定向到`/resource/`。

在此示例中，所有方法都注册了路径规范化处理程序，但您不应依赖此机制来重定向POST请求。
带有斜杠的*Not Found*的重定向会将POST请求转换为GET，从而丢失原始请求中的所有POST数据。
可以仅针对GET请求注册路径规范化：

{{< include-example example="url-dispatch" file="norm2.rs" section="norm" >}}

## 使用应用程序前缀编写应用程序

`web::scope()`方法允许设置特定的应用程序scope(范围)。
此scope表示resource前缀，该resource将附加到resource配置添加的所有resource模式中。
这有助于路由配置，同时仍保持相同的resource名称。
例如：

{{< include-example example="url-dispatch" file="scope.rs" section="scope" >}}

在上面的示例中，*show_users*路由的有效路由模式为*/users/show*而不是*/show*，因为应用程序的scope将位于该模式之前。
然后，仅当URL路径为*/users/show*时，路由才会匹配，并且使用路由名称show_users调用`HttpRequest.url_for()`函数时，它将生成具有相同路径的URL。

# 自定义route guard

您可以将guard视为可以接受*request*对象引用并返回*true* 或 *false*的简单函数。
形式上，guard是实现[`Guard`][guardtrait]特征的任何对象。
Actix提供了多个谓词，您可以检查api文档的[functions section][guardfuncs]。

这是一个简单的guard，用于检查请求是否包含特定的*header*：

{{< include-example example="url-dispatch" file="guard.rs" section="guard" >}}

在此示例中，仅当请求包含*CONTENT-TYPE* header时才调用*index* 处理程序。 Guards无法访问或修改请求对象，但是可以在[request extensions][requestextensions]中存储其他信息。

## 修改 guard 的值

您可以通过将任何谓词值包装在`Not`谓词中来反转其含义。例如，如果要为除“GET”以外的所有方法返回“METHOD NOT ALLOWED”响应：

{{< include-example example="url-dispatch" file="guard2.rs" section="guard2" >}}

`Any` guard均会接受guards列表，如果提供的guards中的任何一个相匹配，则他们将匹配。即：

```rust
guard::Any(guard::Get()).or(guard::Post())
```

`All` guard接受guard列表，如果所有提供的guard都匹配，则匹配。即：

```rust
guard::All(guard::Get()).and(guard::Header("content-type", "plain/text"))
```

# 更改默认的Not Found响应

如果在路由表中找不到路径模式，或者resource找不到匹配的路由，则使用默认resource。
默认响应为*NOT FOUND*。
可以使用`App::default_service()`覆盖*NOT FOUND*响应。
此方法接受与`App::service()`方法的常规resource配置相同的配置功能。

{{< include-example example="url-dispatch" file="dhandler.rs" section="default" >}}

[handlersection]: ../handlers/
[approute]: https://docs.rs/actix-web/2/actix_web/struct.App.html#method.route
[appservice]: https://docs.rs/actix-web/2/actix_web/struct.App.html?search=#method.service
[webresource]: https://docs.rs/actix-web/2/actix_web/struct.Resource.html
[resourcehandler]: https://docs.rs/actix-web/2/actix_web/struct.Resource.html#method.route
[route]: https://docs.rs/actix-web/2/actix_web/struct.Route.html
[routeguard]: https://docs.rs/actix-web/2/actix_web/struct.Route.html#method.guard
[routemethod]: https://docs.rs/actix-web/2/actix_web/struct.Route.html#method.method
[routeto]: https://docs.rs/actix-web/2/actix_web/struct.Route.html#method.to
[routetoasync]: https://docs.rs/actix-web/2/actix_web/struct.Route.html#method.to_async
[matchinfo]: https://docs.rs/actix-web/2/actix_web/struct.HttpRequest.html#method.match_info
[pathget]: https://docs.rs/actix-web/2/actix_web/dev/struct.Path.html#method.get
[pathstruct]: https://docs.rs/actix-web/2/actix_web/dev/struct.Path.html
[query]: https://docs.rs/actix-web/2/actix_web/web/struct.Query.html
[urlfor]: https://docs.rs/actix-web/2/actix_web/struct.HttpRequest.html#method.url_for
[urlobj]: https://docs.rs/url/1.7.2/url/struct.Url.html
[guardtrait]: https://docs.rs/actix-web/2/actix_web/guard/trait.Guard.html
[guardfuncs]: https://docs.rs/actix-web/2/actix_web/guard/index.html#functions
[requestextensions]: https://docs.rs/actix-web/2/actix_web/struct.HttpRequest.html#method.extensions
[implfromrequest]: https://docs.rs/actix-web/2/actix_web/trait.FromRequest.html
[implresponder]: https://docs.rs/actix-web/2/actix_web/trait.Responder.html
[pathextractor]: ../extractors
