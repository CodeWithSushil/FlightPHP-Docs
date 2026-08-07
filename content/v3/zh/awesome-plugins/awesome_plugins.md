# 超赞插件

Flight 具有极强的可扩展性。有许多插件可以用来为您的 Flight 应用程序添加功能。有些是由 Flight 团队官方支持的，有些则是帮助您快速上手的微型/轻量级库。

## AI 工具

Flight 可以通过 AI 驱动的插件变得更加酷炫。

- [Flight MCP](/awesome-plugins/mcp) - 一个用于将 MCP（模型控制协议）与 Flight 集成的插件，能够实现无缝的 AI 驱动功能。主要专注于文档页面，帮助通过提供有关 Flight 项目的最新信息来降低 token 成本。

## API 文档

API 文档对于任何 API 来说都至关重要。它帮助开发者了解如何与您的 API 交互以及返回的内容。 有几个工具可以帮助您为 Flight 项目生成 API 文档。

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Daniel Schreiber 撰写的博客文章，介绍如何使用 OpenAPI 规范和 FlightPHP 通过 API 优先的方法构建 API。
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI 是一个帮助您为 Flight 项目生成 API 文档的优秀工具。它非常易于使用，并且可以根据需要进行自定义。这是帮助您生成 Swagger 文档的 PHP 库。

## 应用性能监控 (APM)

应用性能监控 (APM) 对于任何应用都至关重要。它帮助您了解应用的性能表现以及瓶颈所在。有许多 APM 工具可以与 Flight 一起使用。
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM 是一个简单的 APM 库，可以用来监控您的 Flight 应用。它可以监控应用的性能并帮助您识别瓶颈。

## 异步处理

Flight 本身已经是一个快速的框架，但给它装上涡轮引擎会让一切变得更加有趣（也更具挑战性）！

- [flightphp/async](/awesome-plugins/async) - 官方 Flight 异步库。这个库提供了一种向应用添加异步处理的简单方法。它使用 Swoole/Openswoole 作为底层来提供一种简单有效的方法来异步运行任务。

## 授权/权限

对于任何需要控制谁可以访问什么的应用来说，授权和权限都是至关重要的。

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - 官方 Flight 权限库。这个库提供了一种向应用添加用户和应用级别权限的简单方法。

## 身份验证

对于需要验证用户身份和保护 API 端点的应用来说，身份验证是必不可少的。

- [firebase/php-jwt](/awesome-plugins/jwt) - 用于 PHP 的 JSON Web Token (JWT) 库。在 Flight 应用中实现基于令牌的身份验证的简单而安全的方式。非常适合无状态 API 身份验证、使用中间件保护路由以及实现 OAuth 风格的授权流程。

## 缓存

缓存是加速应用的绝佳方式。有许多缓存库可以与 Flight 一起使用。

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - 轻量、简单且独立的 PHP 文件内缓存类

## 命令行界面

命令行界面应用是与应用交互的绝佳方式。您可以使用它们来生成控制器、显示所有路由等等。

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway 是一个命令行应用，可以帮助您管理 Flight 应用。

## Cookie

Cookie 是在客户端存储小块数据的好方法。它们可以用来存储用户偏好、应用设置等。

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie 是一个提供简单有效管理 Cookie 方式的 PHP 库。

## 调试

在本地环境中开发时，调试至关重要。有几个插件可以提升您的调试体验。

- [tracy/tracy](/awesome-plugins/tracy) - 这是一个功能齐全的错误处理器，可以与 Flight 一起使用。它有许多面板可以帮助您调试应用。它也很容易扩展和添加您自己的面板。
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - 与 [Tracy](/awesome-plugins/tracy) 错误处理器一起使用，此插件添加了一些额外的面板，专门帮助调试 Flight 项目。

## 数据库

数据库是大多数应用的核心。这是存储和检索数据的方式。一些数据库库只是编写查询的简单封装，而另一些则是完整的 ORM。

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - 官方 Flight PDO 助手，是核心的一部分。这是一个现代的封装，带有方便的辅助方法如 `insert()`、`update()`、`delete()` 和 `transaction()` 来简化数据库操作。所有结果都作为集合返回，以实现灵活的数组/对象访问。不是 ORM，只是使用 PDO 的更好方法。
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - 官方 Flight PDO 封装，是核心的一部分（自 v3.18.0 起已弃用）。请改用 SimplePdo。
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - 官方 Flight ActiveRecord ORM/映射器。用于轻松检索和存储数据库中数据的小型库。
- [byjg/php-migration](/awesome-plugins/migrations) - 用于跟踪项目所有数据库更改的插件。
- [knifelemon/easy-query](/awesome-plugins/easy-query) - 轻量级、流畅的 SQL 查询构建器，为预处理语句生成 SQL 和参数。与 [SimplePdo](/learn/simple-pdo) 配合使用效果很好。

## 加密

对于存储敏感数据的任何应用来说，加密都是至关重要的。加密和解密数据并不太难，但正确存储加密密钥[可能](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.)[会](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/)[很困难](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key)。最重要的是永远不要将加密密钥存储在公共目录中或提交到代码仓库。

- [defuse/php-encryption](/awesome-plugins/php-encryption) - 这是一个可用于加密和解密数据的库。入门和运行相当简单，可以开始加密和解密数据。

## 任务队列

任务队列对于异步处理任务非常有帮助。这可以是发送电子邮件、处理图像或任何不需要实时完成的任务。

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue 是一个可用于异步处理任务的库。它可以与 beanstalkd、MySQL/MariaDB、SQLite 和 PostgreSQL 一起使用。

## 会话

会话对于 API 来说不是很有用，但对于构建 Web 应用来说，会话对于维护状态和登录信息至关重要。

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - 官方 Flight 会话库。这是一个简单的会话库，可以用来存储和检索会话数据。它使用 PHP 内置的会话处理。
- [Ghostff/Session](/awesome-plugins/ghost-session) - PHP 会话管理器（非阻塞、闪存、段、会话加密）。使用 PHP open_ssl 进行会话数据的可选加密/解密。

## 模板引擎

模板引擎是任何具有 UI 的 Web 应用的核心。有许多模板引擎可以与 Flight 一起使用。

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - 这是核心中非常基本的模板引擎。如果您的项目中有多个页面，不建议使用它。
- [latte/latte](/awesome-plugins/latte) - Latte 是一个功能齐全的模板引擎，非常易于使用，感觉比 Twig 或 Smarty 更接近 PHP 语法。它也很容易扩展和添加您自己的过滤器和函数。
- [twig/twig](/awesome-plugins/twig) - Twig 是一个灵活、快速且安全的模板引擎（与 Symfony 使用的相同）。AI 工具和许多 PHP 开发者都熟悉它，它默认自动转义输出，并且拥有庞大的扩展生态系统。
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate 是一个强大的 PHP 模板引擎，具有资源编译、模板继承和变量处理功能。具有自动 CSS/JS 压缩、缓存、Base64 编码和可选的 Flight PHP 框架集成。

## WordPress 集成

想在 WordPress 项目中使用 Flight？有一个方便的插件可以做到这一点！

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - 这个 WordPress 插件让您可以直接在 WordPress 旁边运行 Flight。如果您想在 WordPress 站点上添加自定义 API、微服务甚至完整应用，使用 Flight 框架非常完美。如果您想要两全其美，这非常有用！

## 贡献

有您想分享的插件吗？提交拉取请求以将其添加到列表中！