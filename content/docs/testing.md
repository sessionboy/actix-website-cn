---
title: 测试
menu: docs_advanced
weight: 215
---

# 测试

每个应用程序都应该经过良好的测试。 Actix-web提供了执行单元和集成测试的工具。

# Unit Tests 单元测试

对于单元测试，actix-web提供了请求构建器类型。 [*TestRequest*][testrequest]实现类似构建器的模式。您可以使用`to_http_request()`生成`HttpRequest`实例，并使用它调用处理程序。

{{< include-example example="testing" file="main.rs" section="unit-tests" >}}

# Integration tests 集成测试

有几种测试应用程序的方法。 Actix-web可用于在实际的http服务器中通过特定的handlers运行应用程序。

`TestRequest::get()`, `TestRequest::post()`和其他方法可用于将请求发送到测试服务器。

要创建用于测试的`Service`，请使用`test::init_service`方法，该方法接受常规的`App`构建器。

查看[api文档][actixdocs]以获取更多信息。

{{< include-example example="testing" file="integration_one.rs" section="integration-one" >}}

如果需要更复杂的应用程序，则配置testing应该与创建普通应用程序非常相似。
例如，您可能需要初始化应用程序状态。
就像使用普通应用程序一样，使用`data`方法创建一个`App`并附加状态。

{{< include-example example="testing" file="integration_two.rs" section="integration-two" >}}

# Stream response tests 流响应测试

如果您需要测试流，则只需调用`take_body()`并将生成的[*ResponseBody*][responsebody]转换为future并执行它就足够了。例如，测试[*Server Sent Events*(服务器sent事件)][serversentevents]。

{{< include-example example="testing" file="stream_response.rs" section="stream-response" >}}

[serversentevents]: https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events
[responsebody]: https://docs.rs/actix-web/2/actix_web/body/enum.ResponseBody.html
[actixdocs]: https://docs.rs/actix-web/2/actix_web/test/index.html
[testrequest]: https://docs.rs/actix-web/2/actix_web/error/trait.ResponseError.html#foreign-impls
