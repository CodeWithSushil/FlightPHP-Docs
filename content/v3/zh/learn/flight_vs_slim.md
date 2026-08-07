# Flight 对比 Slim

## 什么是 Slim？
[Slim](https://slimframework.com) 是一个 PHP 微框架，帮助你快速编写简单而强大的 Web 应用和 API。

Flight 的 v3 功能中有不少灵感实际上来自 Slim。路由分组以及按特定顺序执行中间件，这两个功能就是受 Slim 启发的。Slim v3 发布时主打简洁，但关于 v4 的[评价褒贬不一](https://github.com/slimphp/Slim/issues/2770)。

## 相比 Flight 的优点

- Slim 拥有更大的开发者社区，这些开发者制作了实用的模块，帮助你避免重复造轮子。
- Slim 遵循许多 PHP 社区中常见的接口和标准，这提高了互操作性。
- Slim 有不错的文档和教程，可以用来学习该框架（不过与 Laravel 或 Symfony 相比还是有所不及）。
- Slim 有一些各种资源，比如 YouTube 教程和在线文章，可以用来学习该框架。
- Slim 允许你使用任何你想要的组件来处理核心路由功能，因为它符合 PSR-7 标准。

## 相比 Flight 的缺点

- 令人惊讶的是，Slim 作为一个微框架，并没有你想象中那么快。更多信息请参阅 [TechEmpower 基准测试](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3)。
- Flight 面向的是希望构建轻量、快速且易于使用的 Web 应用的开发者。
- Flight 没有依赖，而 [Slim 有几个必须安装的依赖](https://github.com/slimphp/Slim/blob/4.x/composer.json)。
- Flight 致力于简洁和易用。
- Flight 的核心特性之一是它尽量保持向后兼容性。Slim 从 v3 到 v4 是一次破坏性变更。
- Flight 适合初次踏入框架领域的开发者。
- Flight 也能胜任企业级应用，但它没有像 Slim 那样多的示例和教程。
  这也要求开发者具备更强的自律性，以保持代码的组织性和良好结构。
- Flight 赋予开发者对应用的更多控制权，而 Slim 可能会在幕后悄悄引入一些“魔法”。
- Flight 有 [SimplePdo](/learn/simple-pdo) 用于数据库访问（比已弃用的 PdoWrapper 更推荐）。Slim 则需要你使用第三方库。
- Flight 有[权限插件](/awesome-plugins/permissions)，可以用来保护你的应用。Slim 则需要你使用第三方库。
- Flight 有一个名为 [active-record](/awesome-plugins/active-record) 的 ORM，可以用来与数据库交互。Slim 则需要你使用第三方库。
- Flight 有一个名为 [runway](/awesome-plugins/runway) 的 CLI 应用，可以在命令行中运行你的应用。Slim 没有。