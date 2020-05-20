---
title: 静态文件
menu: docs_advanced
weight: 230
---

# 个别文件

可以使用自定义path模式和`NamedFile`提供静态文件。为了匹配path尾部，我们可以使用`[.*]`正则表达式。

{{< include-example example="static-files" file="main.rs" section="individual-file" >}}

# Directory 目录

要提供来自特定目录和子目录的文件，可以使用`Files`。`Files`必须使用`App::service()`方法注册，否则它将无法提供sub-paths(子路径)

{{< include-example example="static-files" file="directory.rs" section="directory" >}}

默认情况下，禁用子目录的文件列表。尝试加载目录列表将返回*404 Not Found*响应。要启用文件列表，请使用[*Files::show_files_listing()*][showfileslisting]方法。

除了显示目录的文件列表外，还可以重定向到特定的索引文件。使用[*Files::index_file()*][indexfile]方法配置此重定向。

# Configuration 配置

`NamedFiles`可以指定用于提供文件的各种选项：

- `set_content_disposition` - 用于将文件的mime映射到相应的`Content-Disposition`类型的函数
- `use_etag` - 指定是否应计算`ETag`并将其包含在headers中。
- `use_last_modified` - 指定是否应使用文件修改的timestamp(时间戳)并将其添加到`Last-Modified` header中。

上面所有方法都是可选的，并提供了最佳默认值，但是可以自定义它们中的任何一个。

{{< include-example example="static-files" file="configuration.rs" section="config-one" >}}

该配置也可以应用于目录服务：

{{< include-example example="static-files" file="configuration_two.rs" section="config-two" >}}

[showfileslisting]: https://docs.rs/actix-files/0.2/actix_files/struct.Files.html
[indexfile]: https://docs.rs/actix-files/0.2/actix_files/struct.Files.html#method.index_file
