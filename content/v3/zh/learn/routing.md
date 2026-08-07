# 路由

## 概述
Flight 框架中的路由将 URL 模式映射到回调函数或类方法，使请求处理变得快速而简单。它设计为低开销、对初学者友好，并且无需外部依赖即可扩展。

## 理解
路由是 Flight 中将 HTTP 请求连接到应用逻辑的核心机制。通过定义路由，你可以指定不同的 URL 如何触发特定的代码，无论是通过函数、类方法还是控制器动作。Flight 的路由系统非常灵活，支持基本模式、命名参数、正则表达式，以及依赖注入和资源路由等高级功能。这种方法使你的代码保持组织良好且易于维护，同时对于初学者来说快速简单，对于高级用户来说可扩展。

> **注意：** 想进一步了解路由吗？请查看 [“为什么需要框架？”](/learn/why-frameworks) 页面获取更深入的解释。

## 基本用法

### 定义一个简单路由
在 Flight 中，基本的路由通过将一个 URL 模式与一个回调函数或一个类与方法的数组进行匹配来实现。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> 路由按照定义的顺序进行匹配。第一个匹配请求的路由将被调用。

### 使用函数作为回调
回调可以是任何可调用的对象。因此你可以使用普通函数：

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### 使用类和方法作为控制器
你也可以使用类的方法（静态或非静态）：

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// 或
Flight::route('/', [ GreetingController::class, 'hello' ]); // 推荐写法
// 或
Flight::route('/', [ 'GreetingController::hello' ]);
// 或 
Flight::route('/', [ 'GreetingController->hello' ]);
```

或者先创建一个对象，然后再调用方法：

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **注意：** 默认情况下，当在框架内部调用控制器时，`flight\Engine` 类总是被注入，除非你通过[依赖注入容器](/learn/dependency-injection-container)另行指定。

### 方法与路由绑定

默认情况下，路由模式会匹配所有请求方法。你可以通过在 URL 前面放置一个标识符来响应特定的方法。

```php
Flight::route('GET /', function () {
  echo '我收到一个 GET 请求。';
});

Flight::route('POST /', function () {
  echo '我收到一个 POST 请求。';
});

// 你不能使用 Flight::get() 来创建路由，因为该方法是用来获取变量的，而不是创建路由。
Flight::post('/', function() { /* 代码 */ });
Flight::patch('/', function() { /* 代码 */ });
Flight::put('/', function() { /* 代码 */ });
Flight::delete('/', function() { /* 代码 */ });
```

你也可以使用 `|` 分隔符将多个方法映射到同一个回调：

```php
Flight::route('GET|POST /', function () {
  echo '我收到一个 GET 或 POST 请求。';
});
```

### 对 HEAD 和 OPTIONS 请求的特殊处理

Flight 提供了对 `HEAD` 和 `OPTIONS` HTTP 请求的内置处理：

#### HEAD 请求

- **HEAD 请求**与 `GET` 请求的处理方式相同，但 Flight 会在发送给客户端之前自动移除响应主体。
- 这意味着你可以为 `GET` 定义一个路由，而对同一 URL 的 HEAD 请求将只返回头部（没有内容），符合 HTTP 标准。

```php
Flight::route('GET /info', function() {
    echo '这是一些信息！';
});
// 对 /info 的 HEAD 请求将返回相同的头部，但没有主体。
```

#### OPTIONS 请求

对于任何已定义的路由，Flight 会自动处理 `OPTIONS` 请求。
- 当收到 OPTIONS 请求时，Flight 会返回 `204 No Content` 状态，并带有一个 `Allow` 头，列出该路由支持的所有 HTTP 方法。
- 你不需要为 OPTIONS 单独定义路由。

```php
// 对于定义为以下形式的路由：
Flight::route('GET|POST /users', function() { /* ... */ });

// 对 /users 的 OPTIONS 请求将返回：
//
// 状态：204 No Content
// Allow：GET, POST, HEAD, OPTIONS
```

### 使用 Router 对象

此外，你可以获取 Router 对象，其中包含一些可供使用的辅助方法：

```php

$router = Flight::router();

// 像 Flight::route() 一样映射所有方法
$router->map('/', function() {
	echo 'hello world!';
});

// GET 请求
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* 代码 */});
$router->put('/users/update/@id', 	function() { /* 代码 */});
$router->delete('/users/@id', 		function() { /* 代码 */});
$router->patch('/users/@id', 		function() { /* 代码 */});
```

### 正则表达式（Regex）
你可以在路由中使用正则表达式：

```php
Flight::route('/user/[0-9]+', function () {
  // 这将匹配 /user/1234
});
```

虽然可以使用这种方法，但推荐使用命名参数，或带有正则表达式的命名参数，因为它们更具可读性和可维护性。

### 命名参数
你可以在路由中指定命名参数，这些参数将会传递给回调函数。**这更多是为了路由的可读性，而不是其他。请参阅下面关于重要注意事项的部分。**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

你还可以通过使用 `:` 分隔符在命名参数中包含正则表达式：

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // 这将匹配 /bob/123
  // 但不会匹配 /bob/12345
});
```

> **注意：** 不支持将正则分组 `()` 与位置参数匹配。例如：`:'\(`

#### 重要注意事项

虽然在上面的例子中，看起来 `@name` 直接与变量 `$name` 相关联，但事实并非如此。回调函数中参数的顺序决定了传递给它的内容。如果你交换回调函数中参数的顺序，变量也会随之交换。示例如下：

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

如果你访问以下 URL：`/bob/123`，输出将是 `hello, 123 (bob)!`。
_请务必小心_ 在设置路由和回调函数时！

### 可选参数
你可以通过将片段包裹在括号中来指定可选的命名参数。

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // 这将匹配以下 URL：
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

任何未匹配的可选参数将作为 `NULL` 传入。

### 通配符路由
匹配只在单个 URL 片段上进行。如果你想匹配多个片段，可以使用 `*` 通配符。

```php
Flight::route('/blog/*', function () {
  // 这将匹配 /blog/2000/02/01
});
```

要将所有请求路由到同一个回调，你可以这样做：

```php
Flight::route('*', function () {
  // 做某些事情
});
```

### 404 未找到处理器

默认情况下，如果找不到 URL，Flight 会发送一个非常简单的 `HTTP 404 Not Found` 响应。如果你想要更自定义的 404 响应，你可以[映射](/learn/extending)自己的 `notFound` 方法：

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// 你也可以使用 Flight::render() 配合自定义模板。
    $output = <<<HTML
		<h1>我的自定义 404 Not Found</h1>
		<h3>你请求的页面 {$url} 无法找到。</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### 方法未找到处理器

默认情况下，如果找到 URL 但方法不被允许，Flight 会发送一个非常简单的 `HTTP 405 Method Not Allowed` 响应（例如：Method Not Allowed. Allowed Methods are: GET, POST）。它还会为该 URL 包含一个带有允许方法的 `Allow` 头。

如果你想要更自定义的 405 响应，你可以[映射](/learn/extending)自己的 `methodNotFound` 方法：

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// 你也可以使用 Flight::render() 配合自定义模板。
	$output = <<<HTML
		<h1>我的自定义 405 Method Not Allowed</h1>
		<h3>你请求的方法 {$url} 不被允许。</h3>
		<p>允许的方法有：{$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## 高级用法

### 路由中的依赖注入
如果你想通过容器（PSR-11、PHP-DI、Dice 等）使用依赖注入，唯一可用的路由类型是直接自己创建对象并使用容器来创建你的对象，或者你可以使用字符串来定义要调用的类和方法。你可以在[依赖注入](/learn/dependency-injection-container)页面查看更多信息。

这里有一个快速示例：

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// 使用 $this->db 做某些事情
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// 使用你需要的参数设置容器
// 有关 PSR-11 的更多信息，请参阅依赖注入页面
$dice = new \Dice\Dice();

// 别忘了使用 '$dice = ' 重新赋值变量!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// 注册容器处理器
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// 正常定义路由
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// 或
Flight::route('/hello/@id', 'Greeting->hello');
// 或
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### 传递执行权给下一个路由
<span class="badge bg-warning">已废弃</span>
你可以通过从回调函数中返回 `true` 来将执行权传递给下一个匹配的路由。

```php
Flight::route('/user/@name', function (string $name) {
  // 检查某些条件
  if ($name !== "Bob") {
    // 继续到下一个路由
    return true;
  }
});

Flight::route('/user/*', function () {
  // 这将会被调用
});
```

现在推荐使用[中间件](/learn/middleware)来处理类似这种复杂情况。

### 路由别名
通过为路由分配别名，你可以稍后在应用中以动态方式调用该别名，以便在代码中生成（例如：HTML 模板中的链接，或生成重定向 URL）。

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// 或 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// 稍后在代码中的某个位置
class UserController {
	public function update() {

		// 保存用户的代码...
		$id = $user['id']; // 例如 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // 将返回 '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

当你的 URL 发生改变时，这会特别有帮助。在上面的例子中，假设 users 被移到了 `/admin/users/@id`。由于路由已经使用了别名，你不再需要在代码中找到所有旧的 URL 并修改它们，因为别名现在将返回 `/admin/users/5`，就像上面的例子一样。

路由别名在分组中同样有效：

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// 或
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### 检查路由信息
如果你想检查匹配的路由信息，有两种方法：

1. 你可以在 `Flight::router()` 对象上使用 `executedRoute` 属性。
2. 你可以通过在路由方法中传入 `true` 作为第三个参数，请求将路由对象传递给回调函数。路由对象将始终作为回调函数的最后一个参数传递。

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // 对 $route 做某些事情
  // 匹配的 HTTP 方法数组
  $route->methods;

  // 命名参数数组
  $route->params;

  // 匹配的正则表达式
  $route->regex;

  // 包含 URL 模式中使用的任何 '*' 的内容
  $route->splat;

  // 显示 URL 路径....如果你确实需要的话
  $route->pattern;

  // 显示分配给此路由的中间件
  $route->middleware;

  // 显示分配给此路由的别名
  $route->alias;
});
```

> **注意：** `executedRoute` 属性只会在路由执行后被设置。如果你在路由执行之前尝试访问它，它将是 `NULL`。你也可以在[中间件](/learn/middleware)中使用 executedRoute！

#### 在路由定义中传入 `true`
```php
Flight::route('/', function(\flight\net\Route $route) {
  // 匹配的 HTTP 方法数组
  $route->methods;

  // 命名参数数组
  $route->params;

  // 匹配的正则表达式
  $route->regex;

  // 包含 URL 模式中使用的任何 '*' 的内容
  $route->splat;

  // 显示 URL 路径....如果你确实需要的话
  $route->pattern;

  // 显示分配给此路由的中间件
  $route->middleware;

  // 显示分配给此路由的别名
  $route->alias;
}, true);// <-- 这个 true 参数就是实现这一点的关键
```

### 路由分组和中间件
有时你可能想要将相关的路由分组在一起（例如 `/api/v1`）。你可以使用 `group` 方法来实现：

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// 匹配 /api/v1/users
  });

  Flight::route('/posts', function () {
	// 匹配 /api/v1/posts
  });
});
```

你甚至可以嵌套分组：

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() 是获取变量的，不是设置路由的！请看下面的对象上下文
	Flight::route('GET /users', function () {
	  // 匹配 GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // 匹配 POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // 匹配 PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() 是获取变量的，不是设置路由的！请看下面的对象上下文
	Flight::route('GET /users', function () {
	  // 匹配 GET /api/v2/users
	});
  });
});
```

#### 使用对象上下文分组

你仍然可以通过以下方式使用 `Engine` 对象进行路由分组：

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // 使用 $router 变量
  $router->get('/users', function () {
	// 匹配 GET /api/v1/users
  });

  $router->post('/posts', function () {
	// 匹配 POST /api/v1/posts
  });
});
```

> **注意：** 这是使用 `$router` 对象定义路由和分组的推荐方法。

#### 使用中间件分组

你也可以为路由组分配中间件：

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// 匹配 /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // 或者 [ new MyAuthMiddleware() ] 如果你想使用实例
```

查看更多关于[分组中间件](/learn/middleware#grouping-middleware)页面的详细信息。

### 资源路由
你可以使用 `resource` 方法为一组资源创建一组路由。这将为一组遵循 RESTful 约定的资源创建路由。

要创建资源，请执行以下操作：

```php
Flight::resource('/users', UsersController::class);
```

后台将实际创建以下路由：

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

你的控制器将使用以下方法：

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **注意**：你可以通过运行 `php runway routes` 使用 `runway` 查看新添加的路由。

#### 自定义资源路由

有几个选项可以配置资源路由。

##### 别名基础名（Alias Base）

你可以配置 `aliasBase`。默认情况下，别名是所指定 URL 的最后一部分。例如，`/users/` 会生成一个 `aliasBase` 为 `users`。当创建这些路由时，别名是 `users.index`、`users.create` 等。如果你想更改别名，请将 `aliasBase` 设置为你想要的值。

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only 和 Except

你还可以通过使用 `only` 和 `except` 选项来指定要创建的路由。

```php
// 仅白名单这些方法，并黑名单其余方法
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// 仅黑名单这些方法，并白名单其余方法
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

这些基本上是白名单和黑名单选项，因此你可以指定要创建的路由。

##### 中间件

你还可以指定在 `resource` 方法创建的每个路由上运行的中间件。

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### 流式响应

你现在可以使用 `stream()` 或 `streamWithHeaders()` 向客户端流式传输响应。
这对于发送大文件、长时间运行的进程或生成大型响应非常有用。流式路由的处理方式与常规路由略有不同。

> **注意：** 仅当你有 [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) 设置为 `false` 时，流式响应才可用。

#### 手动设置头部的流式传输

你可以通过在路由上使用 `stream()` 方法向客户端流式传输响应。如果你这样做，你必须在向客户端输出任何内容之前手动设置所有头部。这可以通过 `header()` PHP 函数或 `Flight::response()->setRealHeader()` 方法来完成。

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// 显然你需要对路径进行清理等操作。
	$fileNameSafe = basename($filename);

	// 如果在这里路由执行后还有其他头部需要设置，
	// 你必须在任何输出之前定义它们。
	// 它们必须是对 header() 函数的直接调用，
	// 或者调用 Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// 或
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, '文件未找到');
	}

	// 如果你愿意，可以手动设置内容长度
	header('Content-Length: '.filesize($filePath));
	// 或
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// 在读取文件的同时将文件流式传输给客户端
	readfile($filePath);

// 这里是神奇的一行
})->stream();
```

#### 使用头部进行流式传输

你也可以使用 `streamWithHeaders()` 方法在开始流式传输之前设置头部。

```php
Flight::route('/stream-users', function() {

	// 你可以在这里添加任何你想要的额外头部
	// 但必须使用 header() 或 Flight::response()->setRealHeader()

	// 无论你如何获取数据，这里只是一个例子...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// 这是发送数据到客户端所必需的
		ob_flush();
	}
	echo '}';

// 这就是在开始流式传输之前设置头部的方式。
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// 可选状态码，默认为 200
	'status' => 200
]);
```

## 另请参阅
- [中间件](/learn/middleware) - 在路由中使用中间件进行身份验证、日志记录等。
- [依赖注入](/learn/dependency-injection-container) - 简化路由中对象的创建和管理。
- [为什么需要框架？](/learn/why-frameworks) - 了解使用像 Flight 这样的框架的好处。
- [扩展](/learn/extending) - 如何使用你自己的功能扩展 Flight，包括 `notFound` 方法。
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - 用于正则表达式匹配的 PHP 函数。

## 故障排除
- 路由参数是按顺序匹配的，而不是按名称匹配。确保回调参数的顺序与路由定义的顺序一致。
- 使用 `Flight::get()` 不会定义路由；请使用 `Flight::route('GET /...')` 进行路由，或者在分组中使用 Router 对象上下文（例如 `$router->get(...)`）。
- `executedRoute` 属性仅在路由执行后设置；在执行前为 `NULL`。
- 流式传输需要禁用旧的 Flight 输出缓冲功能（`flight.v2.output_buffering = false`）。
- 对于依赖注入，只有某些路由定义支持基于容器的实例化。

### 404 未找到或意外的路由行为

如果你看到 404 Not Found 错误（但你以性命发誓它确实存在，并且不是拼写错误），这实际上可能是你在路由端点中返回了值而不是直接输出它。这个原因是刻意的，但可能会让一些开发者措手不及。

```php
Flight::route('/hello', function(){
	// 这可能会导致 404 Not Found 错误
	return 'Hello World';
});

// 你可能想要的是
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

原因是路由器内置了一种特殊机制，将返回值视为“进入下一个路由”的信号。你可以在[路由](/learn/routing#passing)部分查看该行为的文档。

## 更新日志
- v3：新增资源路由、路由别名和流式传输支持、路由分组和中间件支持。
- v1：绝大多数基本功能可用。