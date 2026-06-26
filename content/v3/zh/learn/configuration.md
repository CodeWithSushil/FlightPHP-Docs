# 配置

## 概述 

Flight 提供了一种简单的方式来配置框架的各个方面，以满足应用程序的需求。有些是默认设置的，但您可以根据需要覆盖它们。您还可以设置自己的变量，以便在整个应用程序中使用。

## 理解

您可以通过 `set` 方法设置配置值来自定义 Flight 的某些行为。

```php
Flight::set('flight.log_errors', true);
```

在 `app/config/config.php` 文件中，您可以看到所有可用的默认配置变量。

## 基本用法

### Flight 配置选项

以下是所有可用配置设置的列表：

- **flight.base_url** `?string` - 如果 Flight 在子目录中运行，则覆盖请求的基本 URL。（默认：null）
- **flight.case_sensitive** `bool` - URL 的大小写敏感匹配。（默认：false）
- **flight.handle_errors** `bool` - 允许 Flight 在内部处理所有错误。（默认：true）
  - 如果您希望 Flight 处理错误而不是默认的 PHP 行为，则需要设置为 true。
  - 如果您安装了 [Tracy](/awesome-plugins/tracy)，您需要将其设置为 false，以便 Tracy 可以处理错误。
  - 如果您安装了 [APM](/awesome-plugins/apm) 插件，您需要将其设置为 true，以便 APM 可以记录错误。
- **flight.log_errors** `bool` - 将错误记录到 Web 服务器的错误日志文件中。（默认：false）
  - 如果您安装了 [Tracy](/awesome-plugins/tracy)，Tracy 将根据 Tracy 的配置记录错误，而不是此配置。
- **flight.debug** `bool` - 当发生错误时，在浏览器中输出详细的错误信息（异常消息、代码和堆栈跟踪）。（默认：false）
  - **切勿在生产环境中启用此功能** — 它会泄漏内部应用程序细节。仅将其用于本地开发或暂存环境。
  - 当设置为 `false` 时，将显示通用的 `500 Internal Server Error`。请与 `flight.log_errors` 配合使用以在服务器端捕获错误。
- **flight.allow_method_override** `bool` - 允许通过 `X-HTTP-Method-Override` 请求头或 POST 正文中的 `_method` 字段覆盖 HTTP 方法。（默认：true）
  - 对于不需要基于 HTML 表单的方法欺骗的应用程序，**建议将其设置为 `false`**，因为它可以防止客户端通过标准 POST 表单伪造 `DELETE` 或 `PUT` 请求。
  - 有关更多详细信息，请参阅 [安全](/learn/security#flight-configuration-hardening)。
- **flight.views.path** `string` - 包含视图模板文件的目录。（默认：./views）
- **flight.views.extension** `string` - 视图模板文件扩展名。（默认：.php）
- **flight.content_length** `bool` - 设置 `Content-Length` 标头。（默认：true）
  - 如果您使用 [Tracy](/awesome-plugins/tracy)，则需要将其设置为 false，以便 Tracy 能够正确渲染。
- **flight.v2.output_buffering** `bool` - 使用旧版输出缓冲。请参阅 [迁移到 v3](migrating-to-v3)。（默认：false）

### 加载器配置

加载器还有另一个配置设置。这将允许您使用类名中带有 `_` 的类进行自动加载。

```php
// 启用带下划线的类加载
// 默认为 true
Loader::$v2ClassLoading = false;
```

### 变量

Flight 允许您保存变量，以便在应用程序的任何地方使用它们。

```php
// 保存您的变量
Flight::set('id', 123);

// 在应用程序的其他地方
$id = Flight::get('id');
```
要查看变量是否已设置，您可以执行：

```php
if (Flight::has('id')) {
  // 执行某些操作
}
```

您可以通过以下方式清除变量：

```php
// 清除 id 变量
Flight::clear('id');

// 清除所有变量
Flight::clear();
```

> **注意：** 仅仅因为您可以设置变量并不意味着您应该这样做。请谨慎使用此功能。原因在于存储在此处的任何内容都会成为全局变量。全局变量不好，因为它们可以从应用程序的任何地方更改，从而难以跟踪错误。此外，这可能会使 [单元测试](/guides/unit-testing) 等事情变得复杂。

### 错误和异常

如果 `flight.handle_errors` 设置为 true，则所有错误和异常都会被 Flight 捕获并传递给 `error` 方法。

默认行为是发送带有一些错误信息的通用 `HTTP 500 Internal Server Error` 响应。

您可以[覆盖](/learn/extending)此行为以满足自己的需求：

```php
Flight::map('error', function (Throwable $error) {
  // 处理错误
  echo $error->getTraceAsString();
});
```

默认情况下，错误不会记录到 Web 服务器。您可以通过更改配置来启用此功能：

```php
Flight::set('flight.log_errors', true);
```

#### 404 未找到

当找不到 URL 时，Flight 会调用 `notFound` 方法。默认行为是发送带有简单消息的 `HTTP 404 Not Found` 响应。

您可以[覆盖](/learn/extending)此行为以满足自己的需求：

```php
Flight::map('notFound', function () {
  // 处理未找到
});
```

## 另请参阅
- [扩展 Flight](/learn/extending) - 如何扩展和自定义 Flight 的核心功能。
- [单元测试](/guides/unit-testing) - 如何为您的 Flight 应用程序编写单元测试。
- [Tracy](/awesome-plugins/tracy) - 用于高级错误处理和调试的插件。
- [Tracy 扩展](/awesome-plugins/tracy_extensions) - 用于将 Tracy 与 Flight 集成的扩展。
- [APM](/awesome-plugins/apm) - 用于应用程序性能监控和错误跟踪的插件。

## 故障排除
- 如果您在查找配置的所有值时遇到问题，可以执行 `var_dump(Flight::get());`

## 更新日志
- v3.18.1 - 添加了 `flight.debug` 和 `flight.allow_method_override` 配置选项。
- v3.5.0 - 添加了 `flight.v2.output_buffering` 配置以支持旧版输出缓冲行为。
- v2.0 - 添加了核心配置。