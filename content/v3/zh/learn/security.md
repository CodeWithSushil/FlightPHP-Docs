# 安全

## 概述

在构建 Web 应用程序时，安全性非常重要。您需要确保应用程序安全，并保护用户数据的安全。Flight 提供了许多功能来帮助您保护 Web 应用程序的安全。

## 理解

在构建 Web 应用程序时，您应该了解许多常见的安全威胁。一些最常见的威胁包括：
- 跨站请求伪造 (CSRF)
- 跨站脚本 (XSS)
- SQL 注入
- 跨源资源共享 (CORS)

[Templates](/learn/templates) 通过默认转义输出来帮助防止 XSS，因此您无需记住手动操作。[Sessions](/awesome-plugins/session) 可以通过将 CSRF 令牌存储在用户会话中来帮助防止 CSRF，具体方法如下所述。使用 PDO 的预处理语句可以帮助防止 SQL 注入攻击（或使用 [PdoWrapper](/learn/pdo-wrapper) 类中的便捷方法）。CORS 可以通过在调用 `Flight::start()` 之前使用简单的钩子来处理。

所有这些方法共同作用，帮助保持 Web 应用程序的安全。学习和理解安全最佳实践始终应该是您关注的重点。

## 基本用法

### 标头

HTTP 标头是保护 Web 应用程序安全的最简单方法之一。您可以使用标头来防止点击劫持、XSS 和其他攻击。有几种方法可以将这些标头添加到您的应用程序中。

有两个很棒的网站可以检查标头的安全性：[securityheaders.com](https://securityheaders.com/) 和 [observatory.mozilla.org](https://observatory.mozilla.org/)。设置下面的代码后，您可以轻松使用这两个网站验证标头是否正常工作。

#### 手动添加

您可以使用 `Flight\Response` 对象上的 `header` 方法手动添加这些标头。
```php
// Set the X-Frame-Options header to prevent clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Set the Content-Security-Policy header to prevent XSS
// Note: this header can get very complex, so you'll want
//  to consult examples on the internet for your application
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Set the X-XSS-Protection header to prevent XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Set the X-Content-Type-Options header to prevent MIME sniffing
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Set the Referrer-Policy header to control how much referrer information is sent
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Set the Strict-Transport-Security header to force HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Set the Permissions-Policy header to control what features and APIs can be used
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

这些可以在 `routes.php` 或 `index.php` 文件的顶部添加。

#### 作为过滤器添加

您也可以在过滤器/钩子中添加它们，如下所示：

```php
// Add the headers in a filter
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

您也可以将它们添加为中间件类，这为应用到哪些路由提供了最大的灵活性。通常，这些标头应应用于所有 HTML 和 API 响应。

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

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
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php or wherever you have your routes
// FYI, this empty string group acts as a global middleware for
// all routes. Of course you could do the same thing and just add
// this only to specific routes.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ SecurityHeadersMiddleware::class ]);
```

### 跨站请求伪造 (CSRF)

跨站请求伪造 (CSRF) 是一种攻击类型，其中恶意网站可以使用户的浏览器向您的网站发送请求。这可用于在用户不知情的情况下在您的网站上执行操作。Flight 不提供内置的 CSRF 保护机制，但您可以使用中间件轻松实现自己的保护。

#### 设置

首先，您需要生成 CSRF 令牌并将其存储在用户会话中。然后，您可以在表单中使用此令牌，并在提交表单时进行检查。我们将使用 [flightphp/session](/awesome-plugins/session) 插件来管理会话。

```php
// Generate a CSRF token and store it in the user's session
// (assuming you've created a session object at attached it to Flight)
// see the session documentation for more information
Flight::register('session', flight\Session::class);

// You only need to generate a single token per session (so it works 
// across multiple tabs and requests for the same user)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### 使用默认的 PHP Flight 模板

```html
<!-- Use the CSRF token in your form -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- other form fields -->
</form>
```

##### 使用 Latte

您还可以设置一个自定义函数来在 Latte 模板中输出 CSRF 令牌。

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// other configurations...

	// Set a custom function to output the CSRF token
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

现在，您可以在 Latte 模板中使用 `csrf()` 函数来输出 CSRF 令牌。

```html
<form method="post">
	{csrf()}
	<!-- other form fields -->
</form>
```

#### 检查 CSRF 令牌

您可以使用几种方法检查 CSRF 令牌。

##### 中间件

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

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

// index.php or wherever you have your routes
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ CsrfMiddleware::class ]);
```

##### 事件过滤器

```php
// This middleware checks if the request is a POST request and if it is, it checks if the CSRF token is valid
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// capture the csrf token from the form values
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// or for a JSON response
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### 跨站脚本 (XSS)

跨站脚本 (XSS) 是一种攻击类型，其中恶意表单输入可以将代码注入您的网站。大多数此类机会来自最终用户填写的表单值。您**绝不**应该信任用户提供的输出！始终假设他们都是世界上最优秀的黑客。他们可以将恶意 JavaScript 或 HTML 注入您的页面。此代码可用于从用户那里窃取信息或在您的网站上执行操作。使用 Flight 的视图类或另一个模板引擎（如 [Latte](/awesome-plugins/latte)），您可以轻松转义输出以防止 XSS 攻击。

```php
// Let's assume the user is clever as tries to use this as their name
$name = '<script>alert("XSS")</script>';

// This will escape the output
Flight::view()->set('name', $name);
// This will output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// If you use something like Latte registered as your view class, it will also auto escape this.
Flight::view()->render('template', ['name' => $name]);
```

### SQL 注入

SQL 注入是一种攻击类型，其中恶意用户可以将 SQL 代码注入您的数据库。这可用于从数据库中窃取信息或在数据库上执行操作。同样，您**绝不**应该信任用户提供的输入！始终假设他们是来找麻烦的。您可以在 `PDO` 对象中使用预处理语句来防止 SQL 注入。

```php
// Assuming you have Flight::db() registered as your PDO object
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// If you use the PdoWrapper class, this can easily be done in one line
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// You can do the same thing with a PDO object with ? placeholders
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### 不安全的示例

以下示例说明了为什么我们使用 SQL 预处理语句来防止类似下面的无害示例：

```php
// end user fills out a web form.
// for the value of the form, the hacker puts in something like this:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// After the query is build it looks like this
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// It looks strange, but it's a valid query that will work. In fact,
// it's a very common SQL injection attack that will return all users.

var_dump($users); // this will dump all users in the database, not just the one single username
```

### JSONP 回调验证

如果您使用 Flight 的 `Flight::jsonp()` 方法，请注意 Flight 会根据严格的允许列表正则表达式（`/^[A-Za-z_$][\w$.]{0,127}$/`）验证 JSONP 回调参数名称。任何不符合此模式的回调名称都会导致 Flight 抛出异常，从而防止通过恶意回调值注入任意 JavaScript。

此验证是内置的，无需额外配置，但在调试 JSONP 端点的意外错误时值得了解。

### CORS

跨源资源共享 (CORS) 是一种机制，允许网页上的许多资源（例如，字体、JavaScript 等）从资源来源域之外的另一个域请求。Flight 没有内置功能，但这可以通过在调用 `Flight::start()` 方法之前运行的钩子轻松处理。

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
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
		// customize your allowed hosts here.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php or wherever you have your routes
$CorsUtil = new CorsUtil();

// This needs to be run before start runs.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Flight 配置加固

Flight 公开了几个对安全性有直接影响的引擎设置。正确设置这些设置是加固应用程序的最简单方法之一。

#### `flight.allow_method_override`

默认情况下，Flight 允许客户端使用 `X-HTTP-Method-Override` 标头或 POST 正文中的 `_method` 字段覆盖请求的 HTTP 方法。虽然这对于只能发送 `GET`/`POST` 的 HTML 表单很方便，但如果您没有预料到这种情况，可能会很危险——攻击者可以通过常规表单伪造 `DELETE` 或 `PUT` 请求。

如果您的应用程序不依赖此行为（例如，您正在构建由现代客户端或可以发送任何 HTTP 动词的 JavaScript 前端使用的 API），则应禁用它：

```php
// In your index.php or bootstrap file, before Flight::start()
Flight::set('flight.allow_method_override', false);
```

默认值为 `true` 以实现向后兼容性，但**强烈建议对于任何不明确需要覆盖功能的应用程序将其设置为 `false`**。

#### `flight.debug`

Flight 有一个 `flight.debug` 设置，用于控制在发生未处理的异常时是否在浏览器中呈现详细的错误信息（异常消息、代码和完整的堆栈跟踪）。默认值为 `false`，这意味着仅显示通用的 `500 Internal Server Error` 消息——不会向客户端泄露内部详细信息。

切勿在生产服务器上启用此功能。仅在本地或暂存环境中使用：

```php
// Safe for local development only — NEVER in production
Flight::set('flight.debug', true);
```

当 `flight.debug` 为 `false`（默认值）时，您仍然可以通过启用 `flight.log_errors` 来捕获错误：

```php
// Log errors server-side without exposing them to the client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### 推荐的生产配置

```php
// index.php or app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### 错误处理
在生产环境中隐藏敏感错误详细信息，以避免向攻击者泄露信息。在生产环境中，记录错误而不是显示它们，并将 `display_errors` 设置为 `0`。

```php
// In your bootstrap.php or index.php

// add this to your app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Disable error display
    ini_set('log_errors', 1);     // Log errors instead
    ini_set('error_log', '/path/to/error.log');
}

// In your routes or controllers
// Use Flight::halt() for controlled error responses
Flight::halt(403, 'Access denied');
```

### 输入清理
切勿信任用户输入。在处理之前使用 [filter_var](https://www.php.net/manual/en/function.filter-var.php) 进行清理，以防止恶意数据潜入。

```php

// Lets assume a $_POST request with $_POST['input'] and $_POST['email']

// Sanitize a string input
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitize an email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### 密码哈希
使用 PHP 的内置函数（如 [password_hash](https://www.php.net/manual/en/function.password-hash.php) 和 [password_verify](https://www.php.net/manual/en/function.password-verify.php)）安全地存储密码并安全地验证它们。密码绝不应以明文形式存储，也不应使用可逆方法加密。哈希确保即使您的数据库被破坏，实际密码也能得到保护。

```php
$password = Flight::request()->data->password;
// Hash a password when storing (e.g., during registration)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verify a password (e.g., during login)
if (password_verify($password, $stored_hash)) {
    // Password matches
}
```

### 速率限制
通过使用缓存限制请求速率来防止暴力攻击或拒绝服务攻击。

```php
// Assuming you have flightphp/cache installed and registered
// Using flightphp/cache in a filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Reset after 60 seconds
});
```

## 另请参阅
- [Sessions](/awesome-plugins/session) - 如何安全地管理用户会话。
- [Templates](/learn/templates) - 使用模板自动转义输出以防止 XSS。
- [PDO Wrapper](/learn/pdo-wrapper) - 使用预处理语句简化数据库交互。
- [Middleware](/learn/middleware) - 如何使用中间件简化添加安全标头的过程。
- [Responses](/learn/responses) - 如何使用安全标头自定义 HTTP 响应。
- [Requests](/learn/requests) - 如何处理和清理用户输入。
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - 用于输入清理的 PHP 函数。
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - 用于安全密码哈希的 PHP 函数。
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - 用于验证哈希密码的 PHP 函数。

## 故障排除
- 有关与 Flight 框架组件相关的问题的故障排除信息，请参阅上面的“另请参阅”部分。

## 更新日志
- v3.18.1 - 添加了 Flight 配置加固部分，涵盖 `flight.allow_method_override`、`flight.debug` 和 JSONP 回调验证。
- v3.1.0 - 添加了关于 CORS、错误处理、输入清理、密码哈希和速率限制的部分。
- v2.0 - 添加了默认视图的转义以防止 XSS。