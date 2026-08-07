# Tracy

Tracy 是一个出色的错误处理程序，可以与 Flight 一起使用。它有许多面板可以帮助您调试应用程序。它也非常易于扩展并添加您自己的面板。Flight 团队创建了一些专门用于 Flight 项目的面板，使用 [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) 插件（Flight 变量、数据库查询、请求、会话，以及当您传递分析器配置文件时可选的 **Twig** 面板——请参阅 [Tracy 扩展](/awesome-plugins/tracy-extensions)）。

## 安装

使用 composer 安装。您实际上需要安装非开发版本，因为 Tracy 带有生产环境错误处理组件。

```bash
composer require tracy/tracy
```

## 基本配置

有一些基本配置选项可以开始使用。您可以在 [Tracy 文档](https://tracy.nette.org/en/configuring) 中阅读更多相关信息。

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// 启用 Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // 有时您必须明确指定（还有 Debugger::PRODUCTION）
// Debugger::enable('23.75.345.200'); // 您也可以提供 IP 地址数组

// 这是错误和异常将被记录的位置。确保此目录存在且可写。
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // 显示所有错误
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // 除已弃用通知之外的所有错误
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // 如果 Debugger 栏可见，则 content-length 不能由 Flight 设置

	// 这是 Flight 的 Tracy 扩展的特定配置（如果您已包含该扩展）
	// 否则请注释掉此行。
	new TracyExtensionLoader($app);
}
```

## 有用的提示

当您调试代码时，有一些非常有用的函数可以为您输出数据。

- `bdump($var)` - 这会将变量转储到 Tracy 栏的单独面板中。
- `dumpe($var)` - 这会转储变量然后立即终止。