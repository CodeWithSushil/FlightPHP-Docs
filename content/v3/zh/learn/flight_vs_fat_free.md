# Flight vs Fat-Free

## 什么是 Fat-Free？
[Fat-Free](https://fatfreeframework.com)（通常被称为 **F3**）是一个功能强大且易于使用的 PHP 微框架，旨在帮助你快速构建动态且健壮的 Web 应用！

Flight 在许多方面与 Fat-Free 相当，并且在功能和简洁性方面可以说是最接近的表亲。Fat-Free 拥有许多 Flight 没有的功能，但它也有许多 Flight 已经具备的功能。Fat-Free 正在逐渐显现出老态，并且已经不如从前那样流行。

更新变得越来越少，社区也不再像以前那样活跃。代码本身足够简单，但有时缺乏语法上的严谨性，导致阅读和理解起来比较困难。它确实适用于 PHP 8.3，但代码本身看起来仍然停留在 PHP 5.3 时代。

## 与 Flight 相比的优点

- Fat-Free 在 GitHub 上的星标数比 Flight 多。
- Fat-Free 有一些不错的文档，但在某些领域的清晰度上有所欠缺。
- Fat-Free 有一些零散资源，例如 YouTube 教程和在线文章，可以用来学习该框架。
- Fat-Free 内置了一些[有用的插件](https://fatfreeframework.com/3.8/api-reference)，有时会很有帮助。
- Fat-Free 内置了一个名为 Mapper 的 ORM，可用于与数据库交互。Flight 则使用 [active-record](/awesome-plugins/active-record)。
- Fat-Free 内置了 Sessions、缓存和本地化功能。Flight 需要你使用第三方库，但这一点在[文档](/awesome-plugins)中有所覆盖。
- Fat-Free 有一小组[社区创建的插件](https://fatfreeframework.com/3.8/development#Community)可用于扩展框架。Flight 在[文档](/awesome-plugins)和[示例](/examples)页面中介绍了一些。
- Fat-Free 和 Flight 一样，没有任何依赖。
- Fat-Free 和 Flight 一样，致力于让开发者掌控自己的应用，并提供简单的开发者体验。
- Fat-Free 和 Flight 一样保持了向后兼容性（部分原因是更新变得[不那么频繁](https://github.com/bcosca/fatfree/releases)）。
- Fat-Free 和 Flight 一样，适合那些第一次踏入框架领域的开发者。
- Fat-Free 内置的模板引擎比 Flight 的模板引擎更强大。Flight 推荐使用 [Latte](/awesome-plugins/latte) 来实现类似效果。
- Fat-Free 有一种独特的 CLI 类型“路由”命令，你可以在 Fat-Free 内部构建 CLI 应用，并像处理 `GET` 请求一样对待它。Flight 则通过 [runway](/awesome-plugins/runway) 来实现这一点。

## 与 Flight 相比的缺点

- Fat-Free 有一些实现测试，甚至还有自己的非常基础的 [test](https://fatfreeframework.com/3.8/test) 类。然而，它并没有像 Flight 那样做到 100% 单元测试。
- 你只能通过 Google 等搜索引擎来实际搜索文档站点。
- Flight 的文档站点支持深色模式。（麦克风掉落）
- Fat-Free 有一些模块长期缺乏维护。
- Flight 提供了用于数据库访问的 [SimplePdo](/learn/simple-pdo)，这比 Fat-Free 内置的 `DB\SQL` 类更简单一些（并且比已弃用的 PdoWrapper 更受推荐）。
- Flight 有一个[权限插件](/awesome-plugins/permissions)，可用于保护你的应用。Fat-Free 则需要你使用第三方库。
- Flight 有一个名为 [active-record](/awesome-plugins/active-record) 的 ORM，使用起来比 Fat-Free 的 Mapper 更接近 ORM 的体验。
  `active-record` 的额外好处是你可以定义记录之间的关系，从而实现自动联表，而 Fat-Free 的 Mapper 需要你创建 [SQL 视图](https://fatfreeframework.com/3.8/databases#ProsandCons)。
- 令人惊讶的是，Fat-Free 没有根命名空间。Flight 则全程使用命名空间，以避免与你自己的代码发生冲突。
  其中 `Cache` 类是最严重的“罪魁祸首”。
- Fat-Free 没有中间件。取而代之的是 `beforeroute` 和 `afterroute` 钩子，可用于在控制器中过滤请求和响应。
- Fat-Free 无法对路由进行分组。
- Fat-Free 有一个依赖注入容器处理器，但关于如何使用它的文档却少得可怜。
- 调试可能会有点棘手，因为基本上所有内容都存储在所谓的 [`HIVE`](https://fatfreeframework.com/3.8/quick-reference) 中。