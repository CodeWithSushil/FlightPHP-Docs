# 在 Flight PHP 中使用 PHPUnit 进行单元测试

本指南介绍如何使用 [PHPUnit](https://phpunit.de/) 在 Flight PHP 中进行单元测试，面向想要理解 *为什么* 单元测试很重要以及如何实际应用的初学者。我们将专注于测试 *行为*——确保你的应用做了你期望的事情，例如发送电子邮件或保存记录——而不是琐碎的计算。我们将从简单的[路由处理器](/learn/routing)开始，逐步深入到更复杂的[控制器](/learn/routing)，并融入[依赖注入](/learn/dependency-injection-container)（DI）以及模拟第三方服务。

## 为什么进行单元测试？

单元测试确保你的代码行为符合预期，在缺陷进入生产环境之前捕获它们。这在 Flight 中尤其有价值，因为轻量级路由和灵活性可能导致复杂的交互。对于独立开发者或团队而言，单元测试就像一个安全网，记录预期行为并防止你日后重新审视代码时出现回归。它们还能改进设计：难以测试的代码通常意味着类过于复杂或耦合过紧。

与过于简单的示例（例如测试 `x * y = z`）不同，我们将关注现实世界中的行为，例如验证输入、保存数据或触发发送电子邮件等操作。我们的目标是让测试更易于上手且有意义。

## 通用指导原则

1. **测试行为，而非实现**：关注结果（例如“邮件已发送”或“记录已保存”），而不是内部细节。这使测试对重构具有鲁棒性。

2. **停止使用 `Flight::`**：Flight 的静态方法极其方便，但会让测试变得困难。你应该习惯使用来自 `$app = Flight::app();` 的 `$app` 变量。`$app` 具有与 `Flight::` 相同的所有方法。你仍然可以在控制器等中使用 `$app->route()` 或 `$this->app->json()`。你还应该使用真正的 Flight 路由器，即 `$router = $app->router()`，然后就可以使用 `$router->get()`、`$router->post()`、`$router->group()` 等。参见[路由](/learn/routing)。

3. **保持测试快速**：快速的测试会鼓励频繁执行。在单元测试中避免慢操作，如数据库调用。如果你的测试很慢，这表示你正在编写集成测试，而不是单元测试。集成测试是指实际涉及真实数据库、真实 HTTP 调用、真实邮件发送等场景。它们有存在的意义，但速度慢且可能不稳定，也就是说它们有时会因未知原因失败。

4. **使用描述性名称**：测试名称应清楚描述正在测试的行为。这能提高可读性和可维护性。

5. **像躲避瘟疫一样避免全局变量**：尽量减少 `$app->set()` 和 `$app->get()` 的使用，因为它们就像全局状态，要求在每个测试中进行模拟。优先使用 DI 或 DI 容器（参见[依赖注入容器](/learn/dependency-injection-container)）。即使使用 `$app->map()` 方法，从技术上讲也属于“全局”，应避免使用而优先使用 DI。请使用诸如 [flightphp/session](https://github.com/flightphp/session) 之类的会话库，这样你可以在测试中模拟会话对象。**不要**在代码中直接调用 [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php)，因为这会向代码注入全局变量，使其难以测试。

6. **使用依赖注入**：将依赖（例如 [`PDO`](https://www.php.net/manual/en/class.pdo.php)、邮件发送器）注入控制器，以隔离逻辑并简化模拟。如果一个类有太多依赖，请考虑按照 [SOLID 原则](https://en.wikipedia.org/wiki/SOLID) 将其重构为每个只承担单一职责的较小类。

7. **模拟第三方服务**：模拟数据库、HTTP 客户端（cURL）或电子邮件服务，以避免外部调用。测试一两个深层即可，但要让你的核心逻辑运行。例如，如果你的应用发送短信，你**绝对不想**在每次运行测试时真的发送一条短信，因为费用会累积（而且速度会更慢）。相反，应模拟短信服务，只验证你的代码是否以正确的参数调用了短信服务。

8. **追求高覆盖率，而非追求完美**：100% 行覆盖率固然很好，但它实际上并不意味着你代码中的所有内容都得到了应有的测试（你可以自行研究 [PHPUnit 中的分支/路径覆盖率](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)）。优先关注关键行为（例如用户注册、API 响应以及捕获失败响应）。

9. **路由中使用控制器**：在路由定义中，使用控制器而不是闭包。默认情况下，`flight\Engine $app` 会通过构造函数注入到每个控制器中。在测试中，使用 `$app = new Flight\Engine()` 在测试中实例化 Flight，将其注入到你的控制器中，并直接调用方法（例如 `$controller->register()`）。参见[扩展 Flight](/learn/extending)和[路由](/learn/routing)。

10. **选择一种模拟风格并坚持使用**：PHPUnit 支持多种模拟风格（例如 prophecy、内置模拟），或者你可以使用匿名类，它们有自己的好处，如代码补全、在方法定义改变时会破坏测试等。只要在你的测试中保持一致即可。参见 [PHPUnit Mock Objects](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles)。

11. **对于你想在子类中测试的方法/属性，使用 `protected` 可见性**：这允许你在测试子类中覆盖它们，而无需将它们设为 public，这对于匿名类模拟尤其有用。

## 设置 PHPUnit

首先，在您的 Flight PHP 项目中使用 Composer 设置 [PHPUnit](https://phpunit.de/)，以便轻松进行测试。更多详情请参阅 [PHPUnit 入门指南](https://phpunit.readthedocs.io/en/12.3/installation.html)。

1. 在您的项目目录中运行：
   ```bash
   composer require --dev phpunit/phpunit
   ```
   这将安装最新的 PHPUnit 作为开发依赖。

2. 在项目根目录创建一个 `tests` 目录，用于存放测试文件。

3. 为方便起见，在 `composer.json` 中添加一个测试脚本：
   ```json
   // 其他 composer.json 内容
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. 在根目录创建一个 `phpunit.xml` 文件：
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

现在当你的测试构建完成后，你可以运行 `composer test` 来执行测试。

## 测试简单的路由处理器

让我们从一个基本的[路由](/learn/routing)开始，该路由验证用户的电子邮件输入。我们将测试其行为：对于有效的电子邮件返回成功消息，对于无效的电子邮件返回错误。对于电子邮件验证，我们使用 [`filter_var`](https://www.php.net/manual/en/function.filter-var.php)。

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
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

为了测试这一点，创建一个测试文件。有关测试结构化的更多信息，请参见[单元测试与 SOLID 原则](/learn/unit-testing-and-solid-principles)：

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // 模拟 POST 数据
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // 模拟 POST 数据
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**要点**：
- 我们使用请求类模拟 POST 数据。不要使用 `$_POST`、`$_GET` 等全局变量，因为这会让测试变得更加复杂（你必须始终重置这些值，否则其他测试可能会崩溃）。
- 默认情况下，即使没有设置 DIC 容器，所有控制器也会注入 `flight\Engine` 实例。这使得直接测试控制器变得容易得多。
- 完全没有使用 `Flight::`，使代码更容易测试。
- 测试验证行为：针对有效/无效电子邮件返回正确的状态和消息。

运行 `composer test` 以验证路由行为是否符合预期。有关 Flight 中[请求](/learn/requests)和[响应](/learn/responses)的更多信息，请参阅相关文档。

## 使用依赖注入实现可测试的控制器

对于更复杂的场景，请使用[依赖注入](/learn/dependency-injection-container)（DI）使控制器可测试。避免使用 Flight 的全局方法（例如 `Flight::set()`、`Flight::map()`、`Flight::register()`），因为它们就像全局状态，要求每个测试都进行模拟。相反，使用 Flight 的 DI 容器、[DICE](https://github.com/Level-2/Dice)、[PHP-DI](https://php-di.org/) 或手动 DI。

让我们使用 [`flight\database\SimplePdo`](/learn/simple-pdo) 而不是原始 PDO。这个辅助类更容易进行模拟和单元测试（并且比已弃用的 `PdoWrapper` 更受推荐）。

下面是一个将用户保存到数据库并发送欢迎电子邮件的控制器：

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// 在这里添加 return 有助于单元测试停止执行
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**要点**：
- 该控制器依赖一个 [`SimplePdo`](/learn/simple-pdo) 实例和一个 `MailerInterface`（一个模拟的第三方电子邮件服务）。
- 依赖通过构造函数注入，避免了全局变量。

### 使用模拟对象测试控制器

现在，让我们测试 `UserController` 的行为：验证电子邮件、保存到数据库以及发送电子邮件。我们将模拟数据库和邮件发送器，以隔离控制器。

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// 有时混合使用模拟风格是必要的
		// 这里我们使用 PHPUnit 内置的模拟对象来处理 PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// 使用匿名类来模拟 SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// 当我们这样模拟时，实际上并没有进行数据库调用。
			// 我们可以进一步设置它来改变 PDOStatement 模拟对象，以模拟失败等情况。
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// 空的构造函数会绕过父构造函数
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// 需要映射 jsonHalt 以避免退出
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**要点**：
- 我们模拟 `SimplePdo` 和 `MailerInterface`，以避免真实的数据库或电子邮件调用。
- 测试验证行为：有效的电子邮件会触发数据库插入和邮件发送；无效的电子邮件会跳过这两者。
- 模拟第三方依赖（例如 `SimplePdo`、`MailerInterface`），让控制器的逻辑运行。

### 过度模拟

注意不要过度模拟你的代码。下面我举个例子，说明为什么这可能是件坏事，就以我们的 `UserController` 为例。我们会将那个检查改为一个名为 `isEmailValid` 的方法（使用 `filter_var`），将其他新增内容改为一个单独的方法 `registerUser`。

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// 在这里添加 return 有助于单元测试停止执行
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

现在来看这个过度模拟的单元测试，它实际上什么也没有测试：

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// 我们在这里跳过了额外的依赖注入，因为它“很简单”
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// 在构造函数中绕过依赖
			public function __construct($app) {
				$this->app = $app;
			}

			// 我们只需强制其有效。
			protected function isEmailValid($email) {
				return true; // 始终返回 true，绕过真实的验证
			}

			// 绕过实际的数据库和邮件发送调用
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

太好了，我们有单元测试而且它们通过了！但是等等，如果我实际上更改了 `isEmailValid` 或 `registerUser` 的内部实现呢？我的测试仍然会通过，因为我已经模拟掉了所有功能。让我展示一下我的意思。

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... 其他方法 ...

	protected function isEmailValid($email) {
		// 更改后的逻辑
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// 现在它应该只允许特定的域名
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

如果我运行上面的单元测试，它们仍然会通过！但因为我并没有测试行为（实际上让部分代码运行），我可能已经编写了一个潜伏在生产环境中的 bug。测试应该被修改以考虑到新的行为，以及当行为不符合我们预期时的相反情况。

## 完整示例

你可以在 GitHub 上找到一个包含单元测试的 Flight PHP 项目完整示例：[n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide)。
如需更深入的理解，请参见[单元测试与 SOLID 原则](/learn/unit-testing-and-solid-principles)。

## 常见陷阱

- **过度模拟**：不要模拟每一个依赖；让一些逻辑（例如控制器验证）运行以测试真实行为。参见[单元测试与 SOLID 原则](/learn/unit-testing-and-solid-principles)。
- **全局状态**：大量使用 PHP 全局变量（例如 [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php)、[`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)）会使测试变得脆弱。`Flight::` 也是如此。应重构为显式传递依赖。
- **复杂的设置**：如果测试设置很繁琐，你的类可能有太多依赖或职责，违反了 [SOLID 原则](/learn/unit-testing-and-solid-principles)。

## 单元测试的规模化应用

单元测试在较大的项目中或数月后重新审视代码时尤为出色。它们记录行为并捕获回归，避免你重新学习自己的应用。对于独立开发者，请测试关键路径（例如用户注册、支付处理）。对于团队，测试可确保不同贡献之间行为一致。更多关于使用框架和测试的好处的信息，请参见[为什么使用框架？](/learn/why-frameworks)。

欢迎为 Flight PHP 文档仓库贡献你自己的测试技巧！

_由 [n0nag0n](https://github.com/n0nag0n) 于 2025 年撰写_