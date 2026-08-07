# 自动加载

## 概述

自动加载是 PHP 中的一个概念，你可以指定一个或多个目录来加载类。这比使用 `require` 或 `include` 加载类要有益得多。这也是使用 Composer 包的一项要求。

正确实现自动加载对于 [AI 辅助开发](/learn/ai) 也很重要：AI 代理会将文件放置在命名空间指向的位置。如果文件夹**大小写**和命名空间大小写不一致，即使在大小写不敏感的 Mac 磁盘上“没问题”，在 Linux 上也会出现类找不到的错误。

## 理解

默认情况下，任何 `Flight` 类都会由 Composer 自动加载。对于**你自己的**应用类，有两种常用方法：

1. **Composer PSR-4**（[官方骨架](https://github.com/flightphp/skeleton) 使用的方式）：在 `composer.json` 中将命名空间前缀映射到目录，然后运行 `composer dump-autoload`。
2. **`Flight::path()`**：让 Flight 的加载器指向目录（对于简单的应用或当你没有为应用代码使用 Composer 时很方便）。

使用自动加载器可以大大简化你的代码。无需在每个文件顶部写一堆 `include` / `require`，类会在你首次使用时才加载。

### 大小写敏感性（请读两遍）

**命名空间必须与目录结构以及这些目录的字母大小写匹配。**

| 有效 | 在 Linux 上会出错 |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` 搭配文件夹 `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | 将 `App\` 与小写 `controllers` 混用 |

PHP 命名空间在某些上下文中不区分大小写，但 **Composer 和文件系统是区分大小写的**。官方骨架统一使用以下规范：

- Composer：`"App\\": "app/"`
- 文件夹：**`Controller`**、**`Middleware`**、**`Model`**、**`Utils`**（PascalCase），而不是 `controllers` / `middlewares`

旧文档和社区示例有时使用小写的 `app\controllers`。如果你的文件夹是小写的，那仍然有效——但**新的骨架项目使用 `App\` + PascalCase 文件夹**。请为每个项目选择一种约定并保持一致，这样人类和 AI 工具就不会凭空发明出第二种布局。

## 骨架（推荐用于新项目）

运行 `composer create-project flightphp/skeleton` 之后，应用代码会通过 Composer 自动加载——`App\` 类无需使用 `Flight::path()`：

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php —— Dice 通过容器解析 App\Controller\…
$router->get('/', [HomeController::class, 'index']);
```

查看 [安装](/install) 了解完整目录结构，查看 [AI 与开发者体验](/learn/ai) 了解 `AGENTS.md` 如何为编码助手记录此布局。

## 基本用法（`Flight::path()`）

假设我们有以下目录结构：

```text
# 示例路径
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - 包含此项目的控制器
│   ├── translations
│   ├── UTILS - 仅包含此应用程序的类（这里特意全大写，稍后的示例会用到）
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

你可能已经注意到这与典型的应用目录结构相似（文档站点本身也使用结构化布局）。这里的小写 `controllers` 是有效的*选择*——只是不是骨架当前的默认设置。

你可以像这样指定要加载的每个目录：

```php

/**
 * public/index.php
 */

// 向自动加载器添加一个路径
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// 不需要命名空间

// 建议所有自动加载的类使用 Pascal 命名法（每个单词首字母大写，无空格）
class MyController {

	public function index() {
		// 做一些事情
	}
}
```

## 使用 `Flight::path()` 时的命名空间

如果你的代码确实使用了命名空间，实现起来其实非常简单。你应该使用 `Flight::path()` 方法指定应用的根目录（不是文档根目录或 `public/` 文件夹）。

```php

/**
 * public/index.php
 */

// 向自动加载器添加一个路径
Flight::path(__DIR__.'/../');
```

下面就是你的控制器可能的样子。请看下面的示例，但请注意注释中的重要信息。

```php
/**
 * app/controllers/MyController.php
 */

// 命名空间是必需的
// 命名空间与目录结构保持一致
// 命名空间必须与目录结构的大小写保持一致
// 命名空间和目录不能包含下划线（除非设置了 Loader::setV2ClassLoading(false)）
namespace app\controllers;

// 建议所有自动加载的类使用 Pascal 命名法（每个单词首字母大写，无空格）
// 自 3.7.2 起，你可以通过运行 Loader::setV2ClassLoading(false); 来对类名使用 Pascal_Snake_Case
class MyController {

	public function index() {
		// 做一些事情
	}
}
```

如果你想自动加载 utils 目录中的类，操作基本相同：

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// 命名空间必须与目录结构和大小写匹配（注意 UTILS 目录是全大写，
//     就像上面的目录树一样）
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// 做一些事情
	}
}
```

### 骨架风格命名空间（规则相同，大小写不同）

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

规则没有改变——只是骨架选择的文件夹/命名空间大小写不同。**无论你的文件夹使用什么大小写，`namespace` 行都必须与之匹配。**

## 类名中的下划线

从 3.7.2 开始，你可以通过运行 `Loader::setV2ClassLoading(false);` 来对类名使用 Pascal_Snake_Case。
这将允许你在类名中使用下划线。
虽然不推荐，但可为有需要的人提供此功能。

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// 向自动加载器添加一个路径
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// 不需要命名空间

class My_Controller {

	public function index() {
		// 做一些事情
	}
}
```

## 另请参阅
- [安装](/install) - 新项目的骨架目录结构和 `App\` 默认设置。
- [路由](/learn/routing) - 如何将路由映射到控制器并渲染视图。
- [依赖注入](/learn/dependency-injection-container) - 控制器如何获取 `Engine` 和服务。
- [AI 与开发者体验](/learn/ai) - 通过 `AGENTS.md` 让 AI 代理与你的布局保持一致。
- [为什么使用框架？](/learn/why-frameworks) - 了解使用 Flight 这类框架的好处。

## 故障排除

如果你始终找不到命名空间类的原因，请记住：使用 `Flight::path()` 时，要指向**项目根目录**（或命名空间对应的正确基础目录），而不仅仅是一个你忘记在命名空间中镜像的嵌套文件夹。

使用 Composer PSR-4 时，在更改 `composer.json` 映射后运行 `composer dump-autoload`。

在 Linux CI 或生产环境中，文件夹大小写错误是一种非常常见的“在我的机器上没问题”的失败原因。

### 找不到类（自动加载不起作用）

可能有几个原因导致这种情况。下面是一些示例。

#### 文件名不正确

最常见的原因是类名与文件名不匹配。

如果你有一个名为 `MyClass` 的类，那么文件名应为 `MyClass.php`。如果你有一个名为 `MyClass` 的类，但文件名是 `myclass.php`，那么自动加载器将无法找到它。

#### 命名空间或文件夹大小写不正确

如果你使用命名空间，那么命名空间应与目录结构匹配**包括大小写**。

```php
// ...代码...

// 如果你的 MyController 位于 app/Controller（骨架）并且命名空间为 App\Controller
// 下面这种方式不会生效：
Flight::route('/hello', 'MyController->hello');

// 骨架风格：
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// 旧的小写布局（仅当你的文件夹确实为 app/controllers 时）：
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// 或者使用完全限定名称：
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### 未定义 `path()`（非 Composer 应用代码）

如果你依赖 `Flight::path()` 而不是 Composer 来加载应用类，请在引用这些类的路由之前定义路径（通常在引导过程 / `public/index.php` 的早期）：

```php
// 向自动加载器添加一个路径（对于使用命名空间的应用，指向项目根目录）
Flight::path(__DIR__.'/../');
```

官方骨架主要使用 **Composer PSR-4** 来加载 `App\`，因此在那里你通常不需要为控制器和模型使用 `Flight::path()`。

## 更新日志

- 文档 – 记录骨架 `App\` + PascalCase 文件夹以及人类和 AI 工具可能遇到的大小写敏感性陷阱。
- v3.7.2 - 你可以通过运行 `Loader::setV2ClassLoading(false);` 来对类名使用 Pascal_Snake_Case。
- v2.0 - 新增自动加载功能。