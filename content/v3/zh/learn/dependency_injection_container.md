# 依赖注入容器

## 概述

依赖注入容器（DIC）是一项强大的增强功能，可让您管理应用程序的依赖项。它也是 Flight 能与 [AI 编码工具](/learn/ai) 和单元测试良好配合的最大原因之一：控制器在构造函数中获取所需内容，而不是直接使用全局变量。

## 理解

依赖注入（DI）是现代 PHP 框架中的一个关键概念，用于管理对象的实例化和配置。一些 DIC 库的示例包括：[flightphp/container](https://github.com/flightphp/container)、[Dice](https://r.je/dice)、[Pimple](https://pimple.symfony.com/)、[PHP-DI](http://php-di.org/) 和 [league/container](https://container.thephpleague.com/)。

DIC 是一种在集中位置创建和管理类的巧妙方式。当您需要将同一个对象传递给多个类（控制器、中间件、命令等）时，这非常有用。

官方 [flightphp/skeleton](https://github.com/flightphp/skeleton) 在 `app/config/services.php` 中接入了 **Dice**，替换了共享的 `flight\Engine` 实例，并解析类似 `[App\Controller\HomeController::class, 'index']` 的路由目标。对于新项目，建议使用该模式，以便人员和 AI 代理编辑相同的位置。

## 基本用法

旧的做法可能如下所示：
```php

require 'vendor/autoload.php';

// class to manage users from the database
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// in your routes.php file

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// other UserController routes...

Flight::start();
```

从上面的代码可以看出，我们创建了一个新的 `PDO` 对象并将其传递给 `UserController` 类。对于小型应用来说这没问题，但随着应用的增长，您会发现自己在多个地方创建或传递同一个 `PDO` 对象。这时 DIC 就派上用场了。

以下是使用 DIC（使用 Dice）的相同示例：
```php

require 'vendor/autoload.php';

// same class as above. Nothing changed
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// create a new container
$container = new \Dice\Dice;

// add a rule to tell the container how to create a PDO object
// don't forget to reassign it to itself like below!
$container = $container->addRule('PDO', [
	// shared means that the same object will be returned each time
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// This registers the container handler so Flight knows to use it.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// now we can use the container to create our UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

我敢打赌您可能会认为示例中增加了许多额外代码。神奇之处在于当您有另一个需要 `PDO` 对象的控制器时。

```php

// If all your controllers have a constructor that needs a PDO object
// each of the routes below will automatically have it injected!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

使用 DIC 的额外好处是单元测试变得容易得多。您可以创建一个模拟对象并将其传递给您的类。当您为应用编写测试时，这是一个巨大的好处——而且当 AI 助手生成控制器时，构造函数注入为其提供了清晰、一致的模式（[单元测试指南](/guides/unit-testing)）。

### 创建集中的 DIC 处理器

您可以通过 [扩展](/learn/extending) 应用，在服务文件中创建集中的 DIC 处理器。示例如下：

```php
// services.php

// create a new container
$container = new \Dice\Dice;
// don't forget to reassign it to itself like below!
$container = $container->addRule('PDO', [
	// shared means that the same object will be returned each time
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// now we can create a mappable method to create any object. 
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// This registers the container handler so Flight knows to use it for controllers/middleware
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// lets say we have the following sample class that takes a PDO object in the constructor
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// code that sends an email
	}
}

// And finally you can create objects using dependency injection
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight 有一个插件，提供了一个简单的 PSR-11 兼容容器，您可以使用它来处理依赖注入。以下是一个快速使用示例：

```php

// index.php for example
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// will output this correctly!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### flightphp/container 的高级用法

您还可以递归解析依赖项。示例如下：

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Implementation ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

您也可以创建自己的 DIC 处理器。如果您有一个不是 PSR-11（如 Dice）的自定义容器，这将非常有用。请参阅 [基本用法](#basic-usage) 部分了解如何操作。

此外，在使用 Flight 时，有一些有用的默认设置可以让您的生活更轻松。

#### Engine 实例（`$app` 注入所必需）

如果您在控制器或中间件中类型提示 `flight\Engine`，**Dice 绝不能构造新的 Engine**。应从引导过程中替换同一个实例。这正是官方 skeleton 所做的，也是 `AGENTS.md` 期望 AI 生成的控制器遵循的模式：

```php
// Somewhere in your bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // or $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Critical: reuse the bootstrapped Engine — do not let Dice `new Engine()`
		Engine::class => $app,
		// Prefer SimplePdo for new code
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Optional helper for non-route code
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (skeleton layout — folder case matches namespace)
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
		// No Flight:: facade in the app layer — easier to test and clearer for AI tools
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

如果您跳过了 `Engine` 替换，Dice 可能会创建第二个 Engine，您的控制器将无法共享来自引导过程的路由、配置或映射的 Twig `render`。

#### 添加其他共享服务（SimplePdo、Config、Twig）

```php
use flight\database\SimplePdo;
use flight\Engine;

// After you create $db, $config, $twig in services.php:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

然后，控制器可以在构造函数中接收 `SimplePdo $db`（或您的配置类型），而无需再调用 `Flight::db()`。这符合 [单元测试](/guides/unit-testing) 指南和 skeleton 的房型风格（house style）。

#### 添加其他类

如果您有其他想要添加到容器中的类，使用 Dice 很容易，因为容器会自动解析它们。示例如下：

```php

$container = new \Dice\Dice;
// If you don't need to inject any dependencies into your classes
// you don't need to define anything!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flight 还可以使用任何符合 PSR-11 标准的容器。这意味着您可以使用任何实现了 PSR-11 接口的容器。以下是使用 League 的 PSR-11 容器的示例：

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// same UserController idea as above, type-hinting SimplePdo instead of raw PDO

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

这可能比之前的 Dice 示例略显冗长，但它仍然能以相同的好处完成工作！

## 另请参阅
- [安装](/install) - Skeleton 布局以及 `services.php` 的位置。
- [自动加载](/learn/autoloading) - `App\` 命名空间和文件夹**大小写**。
- [扩展 Flight](/learn/extending) - 了解如何通过扩展框架将依赖注入添加到自己的类中。
- [配置](/learn/configuration) - 了解如何为您的应用配置 Flight。
- [路由](/learn/routing) - 了解如何为应用定义路由，以及依赖注入如何与控制器配合使用。
- [中间件](/learn/middleware) - 了解如何为应用创建中间件，以及依赖注入如何与中间件配合使用。
- [单元测试](/guides/unit-testing) - 为什么构造函数注入优于 `Flight::` 全局变量。
- [AI 与开发者体验](/learn/ai) - 为人类和 AI 代理提供统一的 DI 模式。
- [SimplePdo](/learn/simple-pdo) - 推荐的注入用数据库助手。

## 故障排除
- 如果容器出现问题，请确保向容器传递了正确的类名。
- 控制器类型提示 `Engine` 但得到“空白”应用：请添加 **Engine 替换**（见上文）。Dice 绝不能 `new` 出第二个 Engine。
- 找不到 `App\Controller\…` 类：请检查 `app/Controller/` 下的文件夹大小写——参见 [自动加载](/learn/autoloading)。
- 处理器必须从 `registerContainerHandler` **返回**创建的对象（不要在缺少 `return` 的情况下调用 `Flight::make()`）。

## 更新日志
- 文档 – 记录 skeleton Dice + Engine 替换、SimplePdo，以及 `App\Controller` 布局，以便构建对 AI 友好的项目。
- v3.7.0 - 增加了向 Flight 注册 DIC 处理器的能力。