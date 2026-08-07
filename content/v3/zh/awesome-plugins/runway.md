# Runway

Runway 是一个 CLI 应用程序，可帮助您管理 Flight 应用程序。它可以生成控制器、显示所有路由、运行 AI 设置助手、迁移（在骨架中），等等。它基于优秀的 [adhocore/php-cli](https://github.com/adhocore/php-cli) 库。

点击 [此处](https://github.com/flightphp/runway) 查看代码。

脚手架命令有意与 [官方骨架](https://github.com/flightphp/skeleton) 对齐，因此 [AI 编码工具](/learn/ai) 和人类每次都能获得相同的路径、命名空间和构造函数注入风格。

## 安装

使用 composer 安装。

```bash
composer require flightphp/runway
```

骨架已经依赖 Runway；从项目根目录使用 `php runway`。

## 基本配置

首次运行 Runway 时，它将尝试通过 `'runway'` 键在 `app/config/config.php` 中查找 `runway` 配置。

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// 可选；骨架也使用 index_root 作为公共入口
		'index_root' => 'public/index.php',
    ],
];
```

> **注意** - 从 **v1.2.0** 开始，`.runway-config.json` 已弃用，改用 `app/config/config.php`。升级旧项目时使用 `php runway config:migrate` 进行迁移。骨架在 create-project 时仍可能写入一个小的 `.runway-config.json` 以保持兼容性；今后请优先使用 `config.php` 中的 `runway` 键。

### 项目根目录检测

Runway 足够智能，即使从子目录运行也能检测到项目根目录。它会查找 `composer.json`、`.git` 或 `app/config/config.php` 等指标来确定项目根目录的位置。这意味着您可以从项目的任何位置运行 Runway 命令！

## 用法

Runway 有许多命令可用于管理您的 Flight 应用程序。使用 Runway 有两种简单方法。

1. 如果您使用骨架项目，可以从项目根目录运行 `php runway [命令]`。
1. 如果您将 Runway 作为通过 composer 安装的包使用，可以从项目根目录运行 `vendor/bin/runway [命令]`。

### 命令列表

您可以通过运行 `php runway` 命令查看所有可用命令的列表。

```bash
php runway
```

仅依赖您的安装中实际出现在该列表中的命令（核心 Runway 命令与项目特定命令，如骨架的 `migrate`）。

### 命令帮助

对于任何命令，您都可以传入 `--help` 标志以获取有关如何使用该命令的更多信息。

```bash
php runway routes --help
php runway make:controller --help
```

以下是一些示例：

### 生成控制器

`make:controller` 脚手架一个符合官方骨架布局的控制器：

| | |
|--|--|
| **路径** | `app/Controller/{Name}.php` |
| **命名空间** | `App\Controller` |
| **风格** | `flight\Engine` 的构造函数注入（类主体中没有 `Flight::`） |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

您应该期望的形状示例（简化版）：

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// 例如 $this->app->render('…', […]);
	}
}
```

使用类可调用对象注册，以便 Dice 可以构建控制器：

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**为什么采用这种布局？** 文件夹**大小写**必须与命名空间匹配（`Controller` 而非 `controllers`），以便 Composer 在 Linux 上进行 PSR-4 自动加载——请参阅 [自动加载](/learn/autoloading)。根目录和作用域 `AGENTS.md` 文件告诉 AI 工具使用的也是相同路径，因此生成的和手写的控制器保持一致。

> 较旧的文档和社区项目有时使用 `app/controllers/` 和 `app\controllers`。如果*您的*目录树仍然使用小写文件夹，这仍然有效。**新的骨架项目和当前的 `make:controller` 输出使用 `app/Controller/` + `App\Controller`**。

### 生成活动记录模型

首先确保您已安装 [活动记录](/awesome-plugins/active-record) 插件。

```bash
php runway make:record users
```

在官方骨架中，模型位于 **`app/Model/`** 下，命名空间为 **`App\Model`**，数据库连接是 **[SimplePdo](/learn/simple-pdo)**（注入它或将其传递给 ActiveRecord 构造函数）。生成的文件名/命名空间遵循 Runway 的当前默认值和您的 `runway` 配置——请优先将新模型与 `App\Model` 对齐，以便它们与 [自动加载](/learn/autoloading) 和 `AGENTS.md` 匹配。

与骨架帖子演示一致的模型示例：

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

如果旧的生成器仍然输出 `app/records` / `app\records`，您可以在旧版应用中保留该约定，或将文件移入 `app/Model/` 并更新命名空间以匹配文件夹大小写。

### 迁移（骨架）

官方骨架提供了一个项目命令（从 `app/commands/` 发现），例如：

```bash
php runway migrate
```

迁移是 `migrations/` 下的 SQL 文件（例如 SQLite 的 `YYYYMMDDHHMMSS_description.sql` 和 MySQL 的 `…_description.mysql.sql`），从您的数据库驱动程序配置 / 环境选择。确切的标志和行为由该项目命令定义——在您的应用中运行 `php runway migrate --help`。

### AI 助手

Runway 公开了与 [AI 与开发者体验](/learn/ai) 一起使用的面向 AI 的命令：

```bash
php runway ai:init
php runway ai:generate-instructions
```

这些命令存储 LLM 凭据并生成项目说明（主要是 **`AGENTS.md`**）。在骨架上，将 `AGENTS.md`（以及 `app/` 下的作用域副本）和 **`SECURITY.md`** 视为代理的真实来源。

### 显示所有路由

这将显示当前在 Flight 中注册的所有路由。

```bash
php runway routes
```

如果您只想查看特定路由，可以传入标志来过滤路由。

```bash
# 仅显示 GET 路由
php runway routes --get

# 仅显示 POST 路由
php runway routes --post

# 等。
```

## 向 Runway 添加自定义命令

如果您正在为 Flight 创建包，或者想在项目中添加自己的自定义命令，可以通过为您的项目/包创建 `src/commands/`、`flight/commands/`、`app/commands/` 或 `commands/` 目录来实现。如果您需要进一步的自定义，请参阅下面的配置部分。

在骨架中，项目命令位于 **`app/commands/`** 下，命名空间为 **`App\Command`**。Runway 通过路径发现它们；请保持该文件夹与 Composer 类映射/PSR-4 同步，就像您的项目已经做的那样。

要创建命令，您只需扩展 `AbstractBaseCommand` 类，并至少实现一个 `__construct` 方法和一个 `execute` 方法。

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * 构造
     *
     * @param array<string,mixed> $config 来自 app/config/config.php 的配置
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', '为文档创建示例', $config);
        $this->argument('<funny-gif>', '有趣的 gif 名称');
    }

	/**
     * 执行函数
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('正在创建示例...');

		// 在此处执行某些操作

		$io->ok('示例已创建！');
	}
}
```

有关如何将自己的自定义命令构建到 Flight 应用程序中的更多信息，请参阅 [adhocore/php-cli 文档](https://github.com/adhocore/php-cli)！

## 配置管理

由于配置已从 `v1.2.0` 开始移至 `app/config/config.php`，因此有一些辅助命令来管理配置。

> **骨架提示：** 将 `config.php` 保留为**字面** PHP 值。机密应放在 `.env` 中。避免在 `config.php` 中使用 `$_ENV[...]` 表达式——`config:set` 将该文件重写为静态数据，可能会将机密嵌入文件中。请参阅 [配置](/learn/configuration)。

### 迁移旧配置

如果您有一个旧的 `.runway-config.json` 文件，可以使用以下命令轻松将其迁移到 `app/config/config.php`：

```bash
php runway config:migrate
```

### 设置配置值

您可以使用 `config:set` 命令设置配置值。如果您想在不打开文件的情况下更新配置值，这将很有用。

```bash
php runway config:set app_root "app/"
```

### 获取配置值

您可以使用 `config:get` 命令获取配置值。

```bash
php runway config:get app_root
```

## 所有 Runway 配置

如果您需要自定义 Runway 的配置，可以在 `app/config/config.php` 中设置这些值。下面是一些您可以设置的其他配置：

```php
<?php
// app/config/config.php
return [
    // ... 其他配置值 ...

    'runway' => [
        // 这是您的应用程序目录所在的位置
        'app_root' => 'app/',

        // 这是您的根索引文件所在的位置
        'index_root' => 'public/',

        // 这些是其他项目的根路径
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // 基础路径很可能不需要配置，但如果需要可以在这里设置
        'base_paths' => [
            '/includes/libs/vendor', // 如果您的 vendor 目录或其他目录具有非常独特的路径
        ],

        // 最终路径是项目中搜索命令文件的位置
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // 如果您只想添加完整路径，请继续（相对于项目根目录的绝对或相对路径）
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### 访问配置

如果您需要有效地访问配置值，可以通过 `__construct` 方法或 `app()` 方法访问它们。同样重要的是要注意，如果您有 `app/config/services.php` 文件，这些服务也将可供您的命令使用。

```php
public function execute()
{
    $io = $this->app()->io();
    
    // 访问配置
    $app_root = $this->config['runway']['app_root'];
    
    // 访问服务，如数据库连接
    $database = $this->config['database']
    
    // ...
}
```

## AI 助手包装器

Runway 有一些辅助包装器，可以更轻松地让 AI 生成命令。您可以使用 `addOption` 和 `addArgument`，其方式类似于 Symfony Console。如果您使用 AI 工具生成命令，这将很有帮助。

```php
public function __construct(array $config)
{
    parent::__construct('make:example', '为文档创建示例', $config);
    
    // 模式参数可为空，默认完全可选
    $this->addOption('name', '示例的名称', null);
}
```

## 另请参阅

- [安装](/install) - 骨架树和 create-project 默认值
- [自动加载](/learn/autoloading) - `App\` 和文件夹大小写
- [依赖注入](/learn/dependency-injection-container) - Dice + 为生成的控制器注入 Engine
- [AI 与开发者体验](/learn/ai) - `ai:init`、`ai:generate-instructions`、`AGENTS.md`
- [活动记录](/awesome-plugins/active-record) - 与 `make:record` / 骨架 `App\Model` 一起使用的模型
- [SimplePdo](/learn/simple-pdo) - 骨架迁移和模型使用的数据库连接