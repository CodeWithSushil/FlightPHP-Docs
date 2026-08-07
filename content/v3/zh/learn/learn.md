# 了解 Flight

Flight 是一个快速、简单、可扩展的 PHP 框架。它非常通用，可用于构建任何类型的 Web 应用程序。它以简单为设计理念，并且编写方式易于理解和使用——无论是人类还是 [AI 编码助手](/learn/ai)。

> **注意：** 你会看到一些示例使用 `Flight::` 作为静态变量，也有一些使用 `$app->` 引擎对象。两者可以互换使用。在控制器/中间件中，`$app` 和 `$this->app` 是 Flight 团队推荐的方式（也是官方骨架和 `AGENTS.md` 为新项目标准化的方式）。

## 核心组件

### [路由](/learn/routing)

了解如何为你的 Web 应用程序管理路由。这还包括路由分组、路由参数和中间件。

### [中间件](/learn/middleware)

了解如何在应用程序中使用中间件来过滤请求和响应。

### [自动加载](/learn/autoloading)

了解如何自动加载你自己的类。文件夹**大小写**必须与你的命名空间匹配——骨架使用 `App\` 和 PascalCase 文件夹，如 `app/Controller/`。

### [请求](/learn/requests)

了解如何在应用程序中处理请求和响应。

### [响应](/learn/responses)

了解如何向用户发送响应。

### [HTML 模板](/learn/templates)

了解如何使用 Twig（骨架默认）、Latte 或其他引擎渲染 HTML——而不仅仅是内置的 PHP 视图。

### [安全](/learn/security)

了解如何保护你的应用程序免受常见安全威胁。

### [配置](/learn/configuration)

了解如何为你的应用程序配置框架。

### [事件管理器](/learn/events)

了解如何使用事件系统为你的应用程序添加自定义事件。

### [扩展 Flight](/learn/extending)

了解如何通过添加你自己的方法和类来扩展框架。

### [方法钩子和过滤](/learn/filtering)

了解如何为你的方法和框架内部方法添加事件钩子。

### [依赖注入容器（DIC）](/learn/dependency-injection-container)

了解如何使用依赖注入容器（DIC）来管理应用程序的依赖项。

## 实用类

### [集合](/learn/collections)

集合用于保存数据，并且可以作为数组或对象访问，以便于使用。

### [JSON 包装器](/learn/json)

这里有一些简单的函数，使 JSON 的编码和解码保持一致。

### [SimplePdo](/learn/simple-pdo)

PDO 有时会带来不必要的头痛。SimplePdo 是一个现代的 PDO 辅助类，提供了 `insert()`、`update()`、`delete()` 和 `transaction()` 等便捷方法，使数据库操作更加容易。

### [PdoWrapper](/learn/pdo-wrapper)（已弃用）

原始的 PDO 包装器自 v3.18.0 起已弃用。请改用 [SimplePdo](/learn/simple-pdo)。

### [上传文件处理器](/learn/uploaded-file)

一个简单的类，用于帮助管理上传的文件并将其移动到永久位置。

## 重要概念

### [为什么使用框架？](/learn/why-frameworks)

这里有一篇短文，介绍为什么你应该使用框架。在开始使用框架之前，了解使用框架的好处是一个好主意。

此外，[@lubiana](https://git.php.fail/lubiana) 创建了一个出色的教程。虽然它没有详细深入介绍 Flight 本身，但本指南将帮助你理解围绕框架的一些主要概念以及为什么它们对你有益。你可以在[这里](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md)找到该教程。

### [Flight 与其他框架的比较](/learn/flight-vs-another-framework)

如果你正在从其他框架（如 Laravel、Slim、Fat-Free 或 Symfony）迁移到 Flight，本页将帮助你理解两者之间的差异。

## 其他主题

### [单元测试](/learn/unit-testing)

按照本指南学习如何对 Flight 代码进行单元测试，使其坚如磐石。

### [AI 与开发者体验](/learn/ai)

Flight 专为与编码 LLM 配合而构建：`AGENTS.md`、Runway `ai:*` 命令，以及清晰的骨架布局，使代理遵循既定模式。

### [从 v2 迁移到 v3](/learn/migrating-to-v3)

向后兼容性在很大程度上得到了保留，但在从 v2 迁移到 v3 时，有一些变化你应该注意。