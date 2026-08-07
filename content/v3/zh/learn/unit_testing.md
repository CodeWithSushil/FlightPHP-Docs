# 单元测试

## 概述

Flight 中的单元测试可帮助您确保应用程序按预期运行，及早发现错误，并使代码库更易于维护。Flight 旨在与 [PHPUnit](https://phpunit.de/) 顺畅配合，PHPUnit 是最流行的 PHP 测试框架。

## 理解

单元测试会隔离地检查应用程序中各个小部分（如控制器或服务）的行为。在 Flight 中，这意味着测试您的路由、控制器和逻辑如何响应不同的输入——而不依赖全局状态或真实的外部服务。

关键原则：
- **测试行为而非实现：** 关注代码做什么，而不是如何做。
- **避免全局状态：** 使用依赖注入，而不是 `Flight::set()` 或 `Flight::get()`。
- **模拟外部服务：** 使用测试替身来替换数据库或邮件程序等内容。
- **保持测试快速且专注：** 单元测试不应访问真实的数据库或 API。

## 基本用法

### 设置 PHPUnit

1. 使用 Composer 安装 PHPUnit：
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. 在项目根目录中创建一个 `tests` 目录。
3. 将测试脚本添加到您的 `composer.json`：
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. 创建一个 `phpunit.xml` 文件：
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

现在您可以使用 `composer test` 运行测试。

### 测试简单的路由处理器

假设您有一条用于验证电子邮件的路由：

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
    protected $app;
    public function __construct(flight\Engine $app) {
        $this->app = $app;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

针对此控制器的简单测试：

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**提示：**
- 使用 `$app->request()->data` 模拟 POST 数据。
- 在测试中避免使用 `Flight::` 静态方法——请使用 `$app` 实例。

### 使用依赖注入来使控制器可测试

将依赖项（如数据库或邮件程序）注入控制器中，以便在测试中轻松模拟它们：

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

以及一个使用 mock 的测试：

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## 高级用法

- **模拟（Mocking）：** 使用 PHPUnit 内置的 mock 或匿名类来替换依赖项。
- **直接测试控制器：** 使用新的 `Engine` 实例化控制器，并模拟依赖项。
- **避免过度模拟：** 尽可能让真实逻辑运行；只模拟外部服务。

## 另请参阅

- [单元测试指南](/guides/unit-testing) - 全面介绍单元测试最佳实践的指南。
- [依赖注入容器](/learn/dependency-injection-container) - 如何使用 DIC 管理依赖并提高可测试性。
- [扩展](/learn/extending) - 如何添加自己的助手或覆盖核心类。
- [SimplePdo](/learn/simple-pdo) - 简化数据库交互，并且更易于在测试中模拟。
- [请求](/learn/requests) - 在 Flight 中处理 HTTP 请求。
- [响应](/learn/responses) - 向用户发送响应。
- [单元测试与 SOLID 原则](/learn/unit-testing-and-solid-principles) - 了解 SOLID 原则如何改进您的单元测试。

## 故障排除

- 避免在代码和测试中使用全局状态（`Flight::set()`、`$_SESSION` 等）。
- 如果测试运行缓慢，则可能是在编写集成测试——请模拟外部服务以保持单元测试快速运行。
- 如果测试设置过于复杂，请考虑重构代码以使用依赖注入。

## 更新日志

- v3.15.0 - 添加了依赖注入和模拟的示例。