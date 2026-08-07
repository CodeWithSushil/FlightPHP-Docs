# 安全

## 概述

对于 Web 应用来说，安全是一件大事。你希望确保你的应用是安全的，并且用户的数据是安全的。Flight 提供了许多功能来帮助你保护你的 Web 应用。

官方 [骨架项目](https://github.com/flightphp/skeleton) 还附带了一份专门的 **`SECURITY.md`** 和安全头部中间件，以便 [AI 编码工具](/learn/ai)（以及人类）有一个专门的、审慎的位置来处理密钥、头部和 XSS/SQL 规则——与 `AGENTS.md` 中通用编码风格区分开来。

## 理解

在构建 Web 应用时，有一些常见的安全威胁你应该了解。其中最常见的一些威胁包括：
- 跨站请求伪造（CSRF）
- 跨站脚本攻击（XSS）
- SQL 注入
- 跨源资源共享（CORS）

[模板](/learn/templates) 默认会转义输出，从而帮助防范 XSS（Twig 和 Latte 会自动这样做；请利用这一优势）。[会话](/awesome-plugins/session) 可以通过在用户会话中存储 CSRF 令牌来帮助防范 CSRF，如下所述。使用带有 PDO 的预处理语句——或使用 [SimplePdo](/learn/simple-pdo) 上的辅助方法——有助于防止 SQL 注入。CORS 可以通过在调用 `Flight::start()` 之前使用一个简单的钩子来处理。

所有这些方法协同工作，帮助保护你的 Web 应用安全。学习并理解安全最佳实践应当始终是你关注的重点。不要仅仅为了让页面加载，就去要求 AI 助手“禁用 CSP”或削弱头部，而不理解其中的权衡。

## 基本用法

### 头部

HTTP 头部是保护 Web 应用最简单的方法之一。你可以使用头部来防止点击劫持、XSS 和其他攻击。有几种方法可以为你的应用添加这些头部。

两个可以检查头部安全性的优秀网站是 [securityheaders.com](https://securityheaders.com/) 和 [observatory.mozilla.org](https://observatory.mozilla.org/)。设置好下面的代码后，你可以轻松地使用这两个网站验证你的头部是否生效。

骨架项目包含 `App\Middleware\SecurityHeadersMiddleware`（CSP 带每个请求的 nonce、frame options、HSTS 等等）。最好刻意地扩展它，而不是关闭头部。

#### 手动添加

你可以使用 `Flight\Response` 对象上的 `header` 方法手动添加这些头部。
```php
// 设置 X-Frame-Options 头部以防止点击劫持
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// 设置 Content-Security-Policy 头部以防止 XSS
// 注意：这个头部可能会非常复杂，因此你需要
//  参考网上的示例来适配你的应用
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// 设置 X-XSS-Protection 头部以防止 XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// 设置 X-Content-Type-Options 头部以防止 MIME 嗅探
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// 设置 Referrer-Policy 头部以控制发送多少 referrer 信息
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// 设置 Strict-Transport-Security 头部以强制 HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// 设置 Permissions-Policy 头部以控制可以使用哪些功能和 API
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

这些可以添加在你的 `routes.php` 或 `index.php` 文件的顶部。

#### 作为过滤器添加

你也可以将它们添加在过滤器/钩子中，如下所示：

```php
// 在过滤器中添加头部
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### 作为中间件添加

你也可以将它们添加为一个中间件类，这对于将其应用到哪些路由提供了最大的灵活性。一般来说，这些头部应应用于所有 HTML 和 API 响应。

骨架风格路径和命名空间（**文件夹大小写与 `App\Middleware` 匹配**）：

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		// 当你存在内联脚本时，优先使用 bootstrap 中的 CSP nonce（骨架项目会设置 csp_nonce）
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — 空字符串分组 = 所有路由的全局中间件
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// 更多路由
}, [SecurityHeadersMiddleware::class]);
```

较旧的项目可能仍使用 `app/middlewares` 和 `app\middlewares`；只要文件夹匹配就可用。新的骨架应用使用 **`app/Middleware/`** 和 **`App\Middleware`**。参见 [自动加载](/learn/autoloading)。

### 跨站请求伪造（CSRF）

跨站请求伪造（CSRF）是一种攻击类型，恶意网站可以让用户的浏览器向你的网站发送请求。这可以用来在用户不知情的情况下在你的网站上执行操作。Flight 没有内置的 CSRF 保护机制，但你可以通过使用中间件轻松实现自己的机制。

#### 设置

首先，你需要生成一个 CSRF 令牌并将其存储在用户的会话中。然后你可以在表单中使用此令牌，并在提交表单时对其进行检查。我们将使用 [flightphp/session](/awesome-plugins/session) 插件来管理会话。

```php
// 生成 CSRF 令牌并将其存储在用户的会话中
// （假设你已经创建了一个会话对象并将其附加到 Flight）
// 有关更多信息，请参阅会话文档
Flight::register('session', flight\Session::class);

// 每个会话只需要生成一次令牌（这样它可以在多个标签页和请求之间正常工作，
// 对同一用户是共享的）
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### 使用默认的 PHP Flight 模板

```html
<!-- 在表单中使用 CSRF 令牌 -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- 其他表单字段 -->
</form>
```

##### 使用 Twig（骨架项目默认）

注册一个 Twig 函数，或将令牌传递给每个表单视图。一个最简单的示例，使用全局 + 表单字段：

```php
// 配置 Twig 时（例如 services.php）
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# 其他字段 #}
</form>
```

##### 使用 Latte

你也可以设置一个自定义函数，在你的 Latte 模板中输出 CSRF 令牌。

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// 其他配置...

	// 设置一个自定义函数来输出 CSRF 令牌
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

现在，在你的 Latte 模板中，你可以使用 `csrf()` 函数来输出 CSRF 令牌。

```html
<form method="post">
	{csrf()}
	<!-- 其他表单字段 -->
</form>
```

#### 检查 CSRF 令牌

你可以使用多种方法来检查 CSRF 令牌。

##### 中间件

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// 更多路由
}, [CsrfMiddleware::class]);
```

##### 事件过滤器

```php
// 此中间件检查请求是否为 POST 请求，如果是，则检查 CSRF 令牌是否有效
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// 从表单值中获取 csrf 令牌
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// 或者对于 JSON 响应
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### 跨站脚本攻击（XSS）

跨站脚本攻击（XSS）是一种攻击类型，恶意表单输入可以将代码注入到你的网站中。这些机会大多来自最终用户填写的表单值。你**绝不应**信任来自用户的输出！始终假设他们中的每一个人都是世界上最好的黑客。他们可以向你的页面注入恶意的 JavaScript 或 HTML。这种代码可以用来窃取用户的信息或在你网站上执行操作。使用 Flight 的视图类或像 [Twig](/awesome-plugins/twig) 或 [Latte](/awesome-plugins/latte) 这样的模板引擎，你可以轻松地转义输出以防止 XSS 攻击。

```php
// 假设用户很聪明，试图把这个作为他们的名字
$name = '<script>alert("XSS")</script>';

// 这将转义输出
Flight::view()->set('name', $name);
// 这将输出：&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig（骨架项目默认）和 Latte 默认自动转义——优先使用它们而不是直接使用 PHP echo
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → 已转义
// 除非内容完全可信，否则避免使用 |raw / 未转义的输出
```

### SQL 注入

SQL 注入是一种攻击类型，恶意用户可以向你的数据库注入 SQL 代码。这可以用来窃取你数据库中的信息或对你的数据库执行操作。同样，你**绝不应**信任用户的输入！始终假设他们是不怀好意的。请使用预处理语句——[SimplePdo](/learn/simple-pdo) 辅助方法使这成为默认路径。

```php
// 假设你已经将 Flight::db() 注册为 SimplePdo（或在控制器中注入 SimplePdo）
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo（推荐）—— 带绑定参数的一行代码
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// 同样的思路，使用 ? 占位符
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

在骨架风格的控制器中，优先使用构造器注入 `SimplePdo` 而不是 `Flight::db()`，这样测试和 AI 生成的代码可以保持一致（[依赖注入容器](/learn/dependency-injection-container)）。

#### 不安全的示例

下面是我们为什么使用 SQL 预处理语句来防止类似下面这种天真示例的原因：

```php
// 最终用户填写了一个 Web 表单。
// 对于表单的值，黑客输入了类似这样的内容：
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// 查询构建后看起来像这样
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// 看起来有点奇怪，但这是一个有效的查询，并且能够运行。事实上，
// 这是一种非常常见的 SQL 注入攻击，会返回所有用户。

var_dump($users); // 这将转储数据库中的所有用户，而不仅仅是那一个用户名
```

### 密钥和配置

- 将密钥放入 **`.env`**（或真实的环境中），而不是已提交的 `config.php` 示例中。
- 骨架项目规则：`config.php` 中写入字面默认值；在 bootstrap 阶段合并 env；**不要**在控制器中读取 `$_ENV`——应注入配置。参见 [配置](/learn/configuration)。
- 切勿提交 API 密钥、数据库密码或会话加密密钥。将 AI 工具指向 **`SECURITY.md`**，这样它们就不会发明不安全的捷径。

### JSONP 回调验证

如果你使用 Flight 的 `Flight::jsonp()` 方法，请注意 Flight 会使用严格的白名单正则表达式（`/^[A-Za-z_$][\w$.]{0,127}$/`）来验证 JSONP 回调参数名。任何不匹配此模式的回调名称都会导致 Flight 抛出异常，从而防止通过恶意回调值注入任意 JavaScript。

此验证是内置的，无需额外配置，但在调试 JSONP 端点出现的意外错误时，了解这一点是值得的。

### CORS

跨源资源共享（CORS）是一种机制，允许从资源来源域之外的另一个域请求网页上的许多资源（例如，字体、JavaScript 等）。Flight 没有内置的功能，但可以通过在调用 `Flight::start()` 方法之前运行一个钩子来轻松处理。

```php
// app/Utils/CorsUtil.php  （骨架项目：PascalCase 的 Utils 文件夹 → App\Utils）

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// 在此处定制你允许的主机。
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// bootstrap / routes —— 在 start 之前运行
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Flight 配置加固

Flight 暴露了几个直接影响安全性的引擎设置。正确设置这些是加固应用最简单的方法之一。

#### `flight.allow_method_override`

默认情况下，Flight 允许客户端使用 `X-HTTP-Method-Override` 头或 POST 正文中的 `_method` 字段来覆盖请求的 HTTP 方法。虽然这对于只能发送 `GET`/`POST` 的 HTML 表单很方便，但如果你没有预料到这一点，它可能会很危险——攻击者可能通过普通表单伪造 `DELETE` 或 `PUT` 请求。

如果你的应用不依赖此行为（例如，你正在构建一个由可以发送任意 HTTP 动词的现代客户端或 JavaScript 前端使用的 API），则应禁用它：

```php
// 在你的 index.php 或 bootstrap 文件中，在 Flight::start() 之前
Flight::set('flight.allow_method_override', false);
```

默认值为 `true` 是为了向后兼容，但**强烈建议**任何不明确需要此覆盖功能的应用将其设置为 `false`。

#### `flight.debug`

Flight 有一个 `flight.debug` 设置，控制当未捕获异常发生时，是否在浏览器中渲染详细的错误信息（异常消息、代码和完整堆栈跟踪）。默认值是 `false`，这意味着只显示通用的 `500 Internal Server Error` 消息——不会向客户端泄漏内部细节。

切勿在生产服务器上启用此功能。仅在本地或暂存环境中使用：

```php
// 仅适用于本地开发 —— 切勿在生产环境使用
Flight::set('flight.debug', true);
```

当 `flight.debug` 为 `false`（默认）时，你仍然可以通过启用 `flight.log_errors` 来捕获错误：

```php
// 在服务端记录错误，同时不向客户端暴露它们
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### 推荐的生产配置

```php
// index.php 或从应用配置 / bootstrap 应用
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### 错误处理
在生产环境中隐藏敏感的错误详细信息，以避免向攻击者泄漏信息。在生产环境中，将 `display_errors` 设置为 `0`，记录错误而不是显示错误。

```php
// 在你的 bootstrap.php 或 index.php 中

// 将此添加到你的 app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // 禁用错误显示
    ini_set('log_errors', 1);     // 改为记录错误
    ini_set('error_log', '/path/to/error.log');
}

// 在你的路由或控制器中
// 使用 Flight::halt() 进行受控的错误响应
Flight::halt(403, 'Access denied');
```

### 输入清理
永远不要信任用户输入。在处理之前使用 [filter_var](https://www.php.net/manual/en/function.filter-var.php) 对其进行清理，以防止恶意数据渗入。在应用代码中，优先通过 `$app->request()`（或 `Flight::request()`）读取输入，而不是直接使用 `$_GET` / `$_POST`。

```php

// 假设有一个 $_POST 请求，包含 $_POST['input'] 和 $_POST['email']

// 清理字符串输入
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// 清理电子邮件
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### 密码哈希
使用 PHP 内置的函数（如 [password_hash](https://www.php.net/manual/en/function.password-hash.php) 和 [password_verify](https://www.php.net/manual/en/function.password-verify.php)）来安全地存储和验证密码。密码绝不应以纯文本存储，也不应使用可逆方法加密。哈希可以确保即使你的数据库被攻破，实际密码仍然受到保护。

```php
$password = Flight::request()->data->password;
// 在存储时哈希密码（例如，注册期间）
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// 验证密码（例如，登录期间）
if (password_verify($password, $stored_hash)) {
    // 密码匹配
}
```

### 速率限制
通过使用缓存限制请求速率来防止暴力攻击或拒绝服务攻击。

```php
// 假设你已经安装并注册了 flightphp/cache
// 在过滤器中结合 flightphp/cache 使用
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // 60 秒后重置
});
```

## 另请参阅
- [会话](/awesome-plugins/session) - 如何安全地管理用户会话。
- [模板](/learn/templates) - Twig/Latte 自动转义和 XSS。
- [SimplePdo](/learn/simple-pdo) - 带有预处理语句的数据库辅助方法。
- [PdoWrapper](/learn/pdo-wrapper) - 已弃用；新代码请使用 SimplePdo。
- [中间件](/learn/middleware) - 如何使用中间件来简化添加安全头部的过程。
- [配置](/learn/configuration) - `.env` 与字面配置、生产环境标志。
- [AI 与开发者体验](/learn/ai) - 将安全策略保存在 `SECURITY.md` 中供智能体使用。
- [响应](/learn/responses) - 如何使用安全头部自定义 HTTP 响应。
- [请求](/learn/requests) - 如何处理和清理用户输入。
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - 用于输入清理的 PHP 函数。
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - 用于安全密码哈希的 PHP 函数。
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - 用于验证哈希密码的 PHP 函数。

## 故障排除
- 请参阅上面的“另请参阅”部分，获取与 Flight 框架组件问题相关的故障排除信息。
- 如果 CSP 阻止了你的脚本，请添加一个 nonce（骨架模式）或白名单特定的来源——不要在没有计划的情况下设置 `script-src *`。

## 更新日志
- 文档 – 骨架项目 `App\Middleware`、Twig CSRF/XSS 说明、SimplePdo、密钥/`.env`，以及面向 AI 友好项目的 `SECURITY.md`。
- v3.18.1 - 添加了 Flight 配置加固章节，涵盖 `flight.allow_method_override`、`flight.debug` 和 JSONP 回调验证。
- v3.1.0 - 添加了 CORS、错误处理、输入清理、密码哈希和速率限制章节。
- v2.0 - 为默认视图添加了转义以防止 XSS。