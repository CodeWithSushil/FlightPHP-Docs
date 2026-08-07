# Flight PHP Framework

Flight 是一个快速、简单、可扩展的 PHP 框架——专为希望快速完成任务且无需繁琐设置的开发者而构建。无论您是在构建经典的 Web 应用程序、极速 API，还是与 AI 编程助手搭配使用，Flight 的低占用和直观设计都使其成为完美选择。Flight 旨在精简，但也能满足企业架构需求。

## 为什么选择 Flight？

- **适合初学者：** Flight 是新手 PHP 开发者的绝佳起点。其清晰的结构和简洁的语法帮助您学习 Web 开发，而不会陷入冗余代码。
- **深受专业人士喜爱：** 经验丰富的开发者喜爱 Flight 的灵活性和控制力。您可以从小型原型扩展到功能齐全的应用程序，而无需切换框架。
- **向后兼容：** 我们珍惜您的时间。Flight v3 是 v2 的增强版本，保留了几乎相同的 API。我们相信进化而非革命——不会在每次大版本更新时“破坏一切”。
- **零依赖：** Flight 的核心完全无依赖——没有 polyfill、没有外部包，甚至没有 PSR 接口。这意味着更少的攻击向量、更小的占用，以及不会因上游依赖导致意外的破坏性变更。可选插件可能包含依赖，但核心始终保持精简和安全。
- **AI 友好：** Flight 的小 API 表面积和[官方骨架](https://github.com/flightphp/skeleton)（一个布局、`AGENTS.md`、构造函数注入）使 AI 编码工具易于保持模式一致。无论是手动输入每一行还是与代理协作，代码库都相同。[了解更多关于使用 AI 与 Flight 搭配的信息](/learn/ai)。

## 视频概览

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">很简单，对吧？</span>
      <br>
      <a href="https://docs.flightphp.com/learn">了解更多</a> 关于 Flight 的文档！
    </div>
  </div>
</div>

## 快速开始

要进行快速的裸机安装，请使用 Composer 安装：

```bash
composer require flightphp/core
```

或者您可以从[此处](https://github.com/flightphp/core)下载仓库的 zip 文件。然后您将拥有一个基本的 `index.php` 文件，如下所示：

```php
<?php

// 如果使用 composer 安装
require 'vendor/autoload.php';
// 或者如果通过 zip 文件手动安装
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

就是这样！您已经拥有一个基本的 Flight 应用程序。现在您可以使用 `php -S localhost:8000` 运行此文件，并在浏览器中访问 `http://localhost:8000` 查看输出。

像这样简短的 `Flight::` 示例非常适合学习和微型应用。对于人类和 AI 工具共享的完整项目布局，请使用下面的骨架。

## 骨架/样板应用程序

有一个官方启动器可帮助您开始任何新的 Flight 项目。它从一开始就设置了结构、配置、Composer 脚本和 AI 友好指令。

查看 [flightphp/skeleton](https://github.com/flightphp/skeleton) 获取一个即用的项目，或访问[示例](examples)页面获取灵感。想要 AI 工作流程的详细信息？[探索 AI 与开发者体验](/learn/ai)。

您将获得的内容（高级概述）：

- **`App\` 命名空间**带有 PascalCase 文件夹（`app/Controller/`、`app/Middleware/`、`app/Model/` 等）——文件夹**大小写**必须与命名空间匹配（参见[自动加载](/learn/autoloading)）
- **Dice + `Engine` 注入**使控制器保持可测试性（在应用代码中优先使用 `$this->app` 而非 `Flight::`）
- **Twig** 视图、**SimplePdo** + ActiveRecord 示例、Runway **migrate**
- 根目录 **`AGENTS.md`**（加上作用域副本）和 **`SECURITY.md`** 用于助手和安全策略

## 安装骨架应用程序

很简单！

```bash
# 创建新项目
composer create-project flightphp/skeleton my-project/
# 进入您的新项目目录
cd my-project/
# 启动本地开发服务器立即开始！
composer start
```

它创建项目结构，将 `config_sample.php` 复制到 `config.php`（以及将 `.env.example` 复制到 `.env`，如果存在），然后您就可以开始工作了。可选的示例数据：

```bash
php runway migrate
# 然后访问 /posts 和 /api/posts
```

## 高性能

Flight 是最快的 PHP 框架之一。其轻量级核心意味着更少的开销和更快的速度——完美适合传统应用和现代 AI 辅助工作流程。您可以在 [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks) 查看所有基准测试

请参阅下面与其他流行 PHP 框架的基准测试对比。

| 框架 | 纯文本请求/秒 | JSON 请求/秒 |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight 与 AI

好奇 Flight 如何与编码 LLM 搭配？[探索](/learn/ai) `AGENTS.md`、Runway `ai:*` 命令以及骨架布局如何让助手保持在正确的轨道上。

## 稳定性和向后兼容性

我们珍惜您的时间。我们都见过每隔几年就彻底重塑自己的框架，让开发者留下损坏的代码和昂贵的迁移。Flight 不同。Flight v3 被设计为 v2 的增强，这意味着您熟悉和喜爱的 API 没有被剥离。事实上，大多数 v2 项目无需任何更改即可在 v3 中工作。

我们致力于保持 Flight 的稳定，这样您就可以专注于构建您的应用程序，而不是修复您的框架。对于*新*项目，骨架可能是有主见的；核心 API 对其他人来说保持熟悉。

# 社区

我们在 Matrix 聊天

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

以及 Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# 贡献

您可以通过两种方式为 Flight 做出贡献：

1. 通过访问[核心仓库](https://github.com/flightphp/core)为框架核心做出贡献。
2. 帮助改进文档！本文档网站托管在 [Github](https://github.com/flightphp/docs) 上。如果您发现错误或想改进某些内容，请随时提交拉取请求。我们喜欢更新和新想法——特别是围绕 AI 和新技术！

# 要求

Flight 需要 PHP 7.4 或更高版本。

**注意：** PHP 7.4 受到支持，因为在撰写本文时（2024 年），PHP 7.4 是一些 LTS Linux 发行版的默认版本。强制迁移到 PHP >8 会给这些用户带来很多麻烦。该框架也支持 PHP >8。

# 许可证

Flight 在 [MIT](https://github.com/flightphp/core/blob/master/LICENSE) 许可证下发布。# Flight PHP Framework

Flight 是一个快速、简单、可扩展的 PHP 框架——专为希望快速完成任务且无需繁琐设置的开发者而构建。无论您是在构建经典的 Web 应用程序、极速 API，还是与 AI 编程助手搭配使用，Flight 的低占用和直观设计都使其成为完美选择。Flight 旨在精简，但也能满足企业架构需求。

## 为什么选择 Flight？

- **适合初学者：** Flight 是新手 PHP 开发者的绝佳起点。其清晰的结构和简洁的语法帮助您学习 Web 开发，而不会陷入冗余代码。
- **深受专业人士喜爱：** 经验丰富的开发者喜爱 Flight 的灵活性和控制力。您可以从小型原型扩展到功能齐全的应用程序，而无需切换框架。
- **向后兼容：** 我们珍惜您的时间。Flight v3 是 v2 的增强版本，保留了几乎相同的 API。我们相信进化而非革命——不会在每次大版本更新时“破坏一切”。
- **零依赖：** Flight 的核心完全无依赖——没有 polyfill、没有外部包，甚至没有 PSR 接口。这意味着更少的攻击向量、更小的占用，以及不会因上游依赖导致意外的破坏性变更。可选插件可能包含依赖，但核心始终保持精简和安全。
- **AI 友好：** Flight 的小 API 表面积和[官方骨架](https://github.com/flightphp/skeleton)（一个布局、`AGENTS.md`、构造函数注入）使 AI 编码工具易于保持模式一致。无论是手动输入每一行还是与代理协作，代码库都相同。[了解更多关于使用 AI 与 Flight 搭配的信息](/learn/ai)。

## 视频概览

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">很简单，对吧？</span>
      <br>
      <a href="https://docs.flightphp.com/learn">了解更多</a> 关于 Flight 的文档！
    </div>
  </div>
</div>

## 快速开始

要进行快速的裸机安装，请使用 Composer 安装：

```bash
composer require flightphp/core
```

或者您可以从[此处](https://github.com/flightphp/core)下载仓库的 zip 文件。然后您将拥有一个基本的 `index.php` 文件，如下所示：

```php
<?php

// 如果使用 composer 安装
require 'vendor/autoload.php';
// 或者如果通过 zip 文件手动安装
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

就是这样！您已经拥有一个基本的 Flight 应用程序。现在您可以使用 `php -S localhost:8000` 运行此文件，并在浏览器中访问 `http://localhost:8000` 查看输出。

像这样简短的 `Flight::` 示例非常适合学习和微型应用。对于人类和 AI 工具共享的完整项目布局，请使用下面的骨架。

## 骨架/样板应用程序

有一个官方启动器可帮助您开始任何新的 Flight 项目。它从一开始就设置了结构、配置、Composer 脚本和 AI 友好指令。

查看 [flightphp/skeleton](https://github.com/flightphp/skeleton) 获取一个即用的项目，或访问[示例](examples)页面获取灵感。想要 AI 工作流程的详细信息？[探索 AI 与开发者体验](/learn/ai)。

您将获得的内容（高级概述）：

- **`App\` 命名空间**带有 PascalCase 文件夹（`app/Controller/`、`app/Middleware/`、`app/Model/` 等）——文件夹**大小写**必须与命名空间匹配（参见[自动加载](/learn/autoloading)）
- **Dice + `Engine` 注入**使控制器保持可测试性（在应用代码中优先使用 `$this->app` 而非 `Flight::`）
- **Twig** 视图、**SimplePdo** + ActiveRecord 示例、Runway **migrate**
- 根目录 **`AGENTS.md`**（加上作用域副本）和 **`SECURITY.md`** 用于助手和安全策略

## 安装骨架应用程序

很简单！

```bash
# 创建新项目
composer create-project flightphp/skeleton my-project/
# 进入您的新项目目录
cd my-project/
# 启动本地开发服务器立即开始！
composer start
```

它创建项目结构，将 `config_sample.php` 复制到 `config.php`（以及将 `.env.example` 复制到 `.env`，如果存在），然后您就可以开始工作了。可选的示例数据：

```bash
php runway migrate
# 然后访问 /posts 和 /api/posts
```

## 高性能

Flight 是最快的 PHP 框架之一。其轻量级核心意味着更少的开销和更快的速度——完美适合传统应用和现代 AI 辅助工作流程。您可以在 [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks) 查看所有基准测试

请参阅下面与其他流行 PHP 框架的基准测试对比。

| 框架 | 纯文本请求/秒 | JSON 请求/秒 |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight 与 AI

好奇 Flight 如何与编码 LLM 搭配？[探索](/learn/ai) `AGENTS.md`、Runway `ai:*` 命令以及骨架布局如何让助手保持在正确的轨道上。

## 稳定性和向后兼容性

我们珍惜您的时间。我们都见过每隔几年就彻底重塑自己的框架，让开发者留下损坏的代码和昂贵的迁移。Flight 不同。Flight v3 被设计为 v2 的增强，这意味着您熟悉和喜爱的 API 没有被剥离。事实上，大多数 v2 项目无需任何更改即可在 v3 中工作。

我们致力于保持 Flight 的稳定，这样您就可以专注于构建您的应用程序，而不是修复您的框架。对于*新*项目，骨架可能是有主见的；核心 API 对其他人来说保持熟悉。

# 社区

我们在 Matrix 聊天

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

以及 Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# 贡献

您可以通过两种方式为 Flight 做出贡献：

1. 通过访问[核心仓库](https://github.com/flightphp/core)为框架核心做出贡献。
2. 帮助改进文档！本文档网站托管在 [Github](https://github.com/flightphp/docs) 上。如果您发现错误或想改进某些内容，请随时提交拉取请求。我们喜欢更新和新想法——特别是围绕 AI 和新技术！

# 要求

Flight 需要 PHP 7.4 或更高版本。

**注意：** PHP 7.4 受到支持，因为在撰写本文时（2024 年），PHP 7.4 是一些 LTS Linux 发行版的默认版本。强制迁移到 PHP >8 会给这些用户带来很多麻烦。该框架也支持 PHP >8。

# 许可证

Flight 在 [MIT](https://github.com/flightphp/core/blob/master/LICENSE) 许可证下发布。