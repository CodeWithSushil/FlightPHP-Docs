# 라우팅

## 개요
Flight PHP에서 라우팅은 URL 패턴을 콜백 함수나 클래스 메서드에 매핑하여 빠르고 간단한 요청 처리를 가능하게 합니다. 최소한의 오버헤드, 초보자 친화적인 사용법, 외부 의존성 없이 확장 가능하도록 설계되었습니다.

## 이해하기
라우팅은 HTTP 요청을 애플리케이션 로직에 연결하는 핵심 메커니즘입니다. 라우트를 정의하면 함수, 클래스 메서드 또는 컨트롤러 액션을 통해 서로 다른 URL이 특정 코드를 실행하도록 지정할 수 있습니다. Flight의 라우팅 시스템은 유연하며 기본 패턴, 명명된 매개변수, 정규식, 의존성 주입 및 리소스풀 라우팅과 같은 고급 기능을 지원합니다. 이 접근 방식은 코드를 체계적이고 유지 관리하기 쉽게 유지하면서 초보자에게 빠르고 간단하며 고급 사용자에게 확장 가능합니다.

> **참고:** 라우팅에 대해 더 자세히 알고 싶으신가요? 더 깊이 있는 설명은 ["프레임워크가 왜 필요한가?"](/learn/why-frameworks) 페이지를 확인하세요.

## 기본 사용법

### 간단한 라우트 정의하기
Flight의 기본 라우팅은 URL 패턴을 콜백 함수 또는 클래스와 메서드 배열과 매칭하여 수행됩니다.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> 라우트는 정의된 순서대로 매칭됩니다. 요청과 일치하는 첫 번째 라우트가 호출됩니다.

### 함수를 콜백으로 사용하기
콜백은 호출 가능한 모든 객체가 될 수 있습니다. 따라서 일반 함수를 사용할 수 있습니다:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### 클래스와 메서드를 컨트롤러로 사용하기
클래스의 메서드(정적 또는 비정적)를 사용할 수도 있습니다:

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// 또는
Flight::route('/', [ GreetingController::class, 'hello' ]); // 선호되는 방법
// 또는
Flight::route('/', [ 'GreetingController::hello' ]);
// 또는 
Flight::route('/', [ 'GreetingController->hello' ]);
```

또는 먼저 객체를 생성한 다음 메서드를 호출할 수도 있습니다:

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

> **참고:** 기본적으로 프레임워크 내에서 컨트롤러가 호출되면 [의존성 주입 컨테이너](/learn/dependency-injection-container)를 통해 지정하지 않는 한 `flight\Engine` 클래스가 항상 주입됩니다.

### 메서드별 라우팅
기본적으로 라우트 패턴은 모든 요청 메서드와 매칭됩니다. URL 앞에 식별자를 배치하여 특정 메서드에 응답할 수 있습니다.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// 라우트를 생성하는 데 Flight::get()을 사용할 수 없습니다. 
// 이는 변수를 가져오는 메서드이지 라우트를 만드는 메서드가 아닙니다.
Flight::post('/', function() { /* 코드 */ });
Flight::patch('/', function() { /* 코드 */ });
Flight::put('/', function() { /* 코드 */ });
Flight::delete('/', function() { /* 코드 */ });
```

`|` 구분자를 사용하여 여러 메서드를 단일 콜백에 매핑할 수도 있습니다:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### HEAD 및 OPTIONS 요청 특별 처리
Flight는 `HEAD` 및 `OPTIONS` HTTP 요청에 대한 기본 제공 처리를 제공합니다:

#### HEAD 요청
- **HEAD 요청**은 `GET` 요청과 동일하게 처리되지만, Flight는 응답 본문을 클라이언트에 보내기 전에 자동으로 제거합니다.
- 즉, `GET` 라우트를 정의하면 동일한 URL에 대한 HEAD 요청은 HTTP 표준에 따라 헤더만 반환하고 본문은 포함하지 않습니다.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// /info에 대한 HEAD 요청은 동일한 헤더를 반환하지만 본문은 없습니다.
```

#### OPTIONS 요청
`OPTIONS` 요청은 정의된 모든 라우트에 대해 Flight가 자동으로 처리합니다.
- OPTIONS 요청을 받으면 Flight는 `204 No Content` 상태와 해당 라우트에서 지원하는 모든 HTTP 메서드를 나열하는 `Allow` 헤더로 응답합니다.
- OPTIONS에 대한 별도 라우트를 정의할 필요가 없습니다.

```php
// 다음과 같이 정의된 라우트의 경우:
Flight::route('GET|POST /users', function() { /* ... */ });

// /users에 대한 OPTIONS 요청은 다음과 같이 응답합니다:
//
// Status: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### 라우터 객체 사용하기
추가로 사용할 수 있는 헬퍼 메서드가 포함된 Router 객체를 가져올 수 있습니다:

```php

$router = Flight::router();

// Flight::route()와 동일하게 모든 메서드를 매핑합니다
$router->map('/', function() {
	echo 'hello world!';
});

// GET 요청
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* 코드 */});
$router->put('/users/update/@id', 	function() { /* 코드 */});
$router->delete('/users/@id', 		function() { /* 코드 */});
$router->patch('/users/@id', 		function() { /* 코드 */});
```

### 정규식(Regex)
라우트에서 정규식을 사용할 수 있습니다:

```php
Flight::route('/user/[0-9]+', function () {
  // /user/1234와 일치합니다.
});
```

이 방법도 사용할 수 있지만, 명명된 매개변수 또는 명명된 매개변수와 정규식을 함께 사용하는 것이 더 읽기 쉽고 유지 관리하기 쉬우므로 권장됩니다.

### 명명된 매개변수
라우트에서 명명된 매개변수를 지정하면 콜백 함수로 전달됩니다. **이것은 무엇보다 라우트의 가독성을 위한 것입니다. 아래 중요한 주의 사항 섹션을 참조하세요.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

`:` 구분자를 사용하여 명명된 매개변수에 정규식을 포함할 수도 있습니다:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // /bob/123과 일치합니다.
  // 그러나 /bob/12345와는 일치하지 않습니다.
});
```

> **참고:** 위치 매개변수와 함께 정규식 그룹 `()`을 매칭하는 것은 지원되지 않습니다. 예: `:'\(`

#### 중요한 주의 사항
위 예제에서는 `@name`이 변수 `$name`과 직접 연결된 것처럼 보이지만 실제로는 그렇지 않습니다. 콜백 함수의 매개변수 순서가 무엇이 전달되는지를 결정합니다. 콜백 함수에서 매개변수 순서를 바꾸면 변수도 바뀝니다. 예시:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

그리고 `/bob/123` URL로 이동하면 출력은 `hello, 123 (bob)!`이 됩니다. 
_라우트와 콜백 함수를 설정할 때 주의하세요!_

### 선택적 매개변수
세그먼트를 괄호로 묶어 매칭에 선택적인 명명된 매개변수를 지정할 수 있습니다.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // 다음 URL과 일치합니다:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

일치하지 않는 선택적 매개변수는 `NULL`로 전달됩니다.

### 와일드카드 라우팅
매칭은 개별 URL 세그먼트에 대해서만 수행됩니다. 여러 세그먼트를 매칭하려면 `*` 와일드카드를 사용할 수 있습니다.

```php
Flight::route('/blog/*', function () {
  // /blog/2000/02/01과 일치합니다.
});
```

모든 요청을 단일 콜백으로 라우팅하려면 다음을 수행할 수 있습니다:

```php
Flight::route('*', function () {
  // 작업 수행
});
```

### 404 Not Found 핸들러
기본적으로 URL을 찾을 수 없으면 Flight는 매우 단순하고 평범한 `HTTP 404 Not Found` 응답을 보냅니다. 더 사용자 정의된 404 응답을 원하면 [map](/learn/extending) 메서드로 자신만의 `notFound` 메서드를 매핑할 수 있습니다:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// 사용자 정의 템플릿으로 Flight::render()를 사용할 수도 있습니다.
    $output = <<<HTML
		<h1>My Custom 404 Not Found</h1>
		<h3>The page you have requested {$url} could not be found.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### 메서드 Not Found 핸들러
기본적으로 URL은 찾았지만 메서드가 허용되지 않으면 Flight는 매우 단순하고 평범한 `HTTP 405 Method Not Allowed` 응답을 보냅니다(예: Method Not Allowed. Allowed Methods are: GET, POST). 또한 해당 URL에 허용된 메서드와 함께 `Allow` 헤더도 포함됩니다.

더 사용자 정의된 405 응답을 원하면 [map](/learn/extending) 메서드로 자신만의 `methodNotFound` 메서드를 매핑할 수 있습니다:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// 사용자 정의 템플릿으로 Flight::render()를 사용할 수도 있습니다.
	$output = <<<HTML
		<h1>My Custom 405 Method Not Allowed</h1>
		<h3>The method you have requested for {$url} is not allowed.</h3>
		<p>Allowed Methods are: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## 고급 사용법

### 라우트에서의 의존성 주입
컨테이너(PSR-11, PHP-DI, Dice 등)를 통해 의존성 주입을 사용하려면, 해당 기능을 사용할 수 있는 라우트 유형은 객체를 직접 생성하거나 컨테이너를 사용하여 객체를 생성하는 경우, 또는 호출할 클래스와 메서드를 문자열로 정의하는 경우뿐입니다. 자세한 내용은 [의존성 주입](/learn/dependency-injection-container) 페이지를 참조하세요.

간단한 예시입니다:

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
		// $this->db로 무언가를 처리합니다.
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// 필요한 매개변수로 컨테이너를 설정합니다.
// PSR-11에 대한 자세한 내용은 의존성 주입 페이지를 참조하세요.
$dice = new \Dice\Dice();

// '$dice = '로 변수를 재할당하는 것을 잊지 마세요!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// 컨테이너 핸들러를 등록합니다.
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// 일반적인 라우트처럼 사용합니다.
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// 또는
Flight::route('/hello/@id', 'Greeting->hello');
// 또는
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### 다음 라우트로 실행 전달하기
<span class="badge bg-warning">더 이상 사용되지 않음</span>
콜백 함수에서 `true`를 반환하여 실행을 다음 일치하는 라우트로 전달할 수 있습니다.

```php
Flight::route('/user/@name', function (string $name) {
  // 어떤 조건을 확인합니다.
  if ($name !== "Bob") {
    // 다음 라우트로 계속 진행합니다.
    return true;
  }
});

Flight::route('/user/*', function () {
  // 이 라우트가 호출됩니다.
});
```

이제 이와 같은 복잡한 사용 사례를 처리하려면 [미들웨어](/learn/middleware)를 사용하는 것이 권장됩니다.

### 라우트 별칭(Alias)
라우트에 별칭을 할당하면 나중에 앱에서 해당 별칭을 호출하여 코드에서 동적으로 URL을 생성할 수 있습니다(예: HTML 템플릿의 링크, 리디렉션 URL 생성).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// 또는 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// 나중에 코드 어딘가에서
class UserController {
	public function update() {

		// 사용자 저장 코드...
		$id = $user['id']; // 예: 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // '/users/5'를 반환합니다.
		Flight::redirect($redirectUrl);
	}
}

```

이 기능은 URL이 변경되는 경우 특히 유용합니다. 위 예제에서 사용자가 `/admin/users/@id`로 이동했다고 가정해 봅시다. 라우트에 별칭이 있으면 코드에서 모든 이전 URL을 찾아 변경할 필요가 없습니다. 별칭이 이제 위 예제처럼 `/admin/users/5`를 반환하기 때문입니다.

라우트 별칭은 그룹에서도 계속 작동합니다:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// 또는
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### 라우트 정보 검사
일치하는 라우트 정보를 검사하려면 두 가지 방법이 있습니다:

1. `Flight::router()` 객체의 `executedRoute` 속성을 사용할 수 있습니다.
2. 라우트 메서드의 세 번째 매개변수로 `true`를 전달하여 라우트 객체를 콜백에 전달하도록 요청할 수 있습니다. 라우트 객체는 항상 콜백 함수에 전달되는 마지막 매개변수입니다.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // $route로 무언가를 처리합니다.
  // 매칭된 HTTP 메서드 배열
  $route->methods;

  // 명명된 매개변수 배열
  $route->params;

  // 일치하는 정규식
  $route->regex;

  // URL 패턴에 사용된 '*'의 내용을 포함합니다.
  $route->splat;

  // URL 경로를 표시합니다.... 정말 필요하다면 말이죠.
  $route->pattern;

  // 이 라우트에 할당된 미들웨어를 표시합니다.
  $route->middleware;

  // 이 라우트에 할당된 별칭을 표시합니다.
  $route->alias;
});
```

> **참고:** `executedRoute` 속성은 라우트가 실행된 후에만 설정됩니다. 라우트가 실행되기 전에 접근하면 `NULL`입니다. [미들웨어](/learn/middleware)에서도 executedRoute를 사용할 수 있습니다!

#### 라우트 정의에 `true` 전달하기
```php
Flight::route('/', function(\flight\net\Route $route) {
  // 매칭된 HTTP 메서드 배열
  $route->methods;

  // 명명된 매개변수 배열
  $route->params;

  // 일치하는 정규식
  $route->regex;

  // URL 패턴에 사용된 '*'의 내용을 포함합니다.
  $route->splat;

  // URL 경로를 표시합니다.... 정말 필요하다면 말이죠.
  $route->pattern;

  // 이 라우트에 할당된 미들웨어를 표시합니다.
  $route->middleware;

  // 이 라우트에 할당된 별칭을 표시합니다.
  $route->alias;
}, true);// <-- 이 true 매개변수가 그렇게 만듭니다.
```

### 라우트 그룹화 및 미들웨어
관련된 라우트를 함께 그룹화해야 하는 경우가 있습니다(예: `/api/v1`). `group` 메서드를 사용하여 이 작업을 수행할 수 있습니다:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// /api/v1/users와 일치합니다.
  });

  Flight::route('/posts', function () {
	// /api/v1/posts와 일치합니다.
  });
});
```

그룹의 그룹을 중첩할 수도 있습니다:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get()은 변수를 가져오는 것이지 라우트를 설정하는 것이 아닙니다! 아래 객체 컨텍스트를 참조하세요.
	Flight::route('GET /users', function () {
	  // GET /api/v1/users와 일치합니다.
	});

	Flight::post('/posts', function () {
	  // POST /api/v1/posts와 일치합니다.
	});

	Flight::put('/posts/1', function () {
	  // PUT /api/v1/posts와 일치합니다.
	});
  });
  Flight::group('/v2', function () {

	// Flight::get()은 변수를 가져오는 것이지 라우트를 설정하는 것이 아닙니다! 아래 객체 컨텍스트를 참조하세요.
	Flight::route('GET /users', function () {
	  // GET /api/v2/users와 일치합니다.
	});
  });
});
```

#### 객체 컨텍스트로 그룹화
`Engine` 객체를 사용하여 다음과 같이 라우트 그룹화를 계속 사용할 수 있습니다:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // $router 변수를 사용합니다.
  $router->get('/users', function () {
	// GET /api/v1/users와 일치합니다.
  });

  $router->post('/posts', function () {
	// POST /api/v1/posts와 일치합니다.
  });
});
```

> **참고:** 이것이 `$router` 객체로 라우트와 그룹을 정의하는 선호되는 방법입니다.

#### 미들웨어로 그룹화
라우트 그룹에 미들웨어를 할당할 수도 있습니다:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// /api/v1/users와 일치합니다.
  });
}, [ MyAuthMiddleware::class ]); // 인스턴스를 사용하려면 [ new MyAuthMiddleware() ] 로도 가능합니다.
```

자세한 내용은 [그룹 미들웨어](/learn/middleware#grouping-middleware) 페이지를 참조하세요.

### 리소스 라우팅
`resource` 메서드를 사용하여 리소스에 대한 일련의 라우트를 생성할 수 있습니다. 이는 RESTful 규칙을 따르는 라우트 세트를 생성합니다.

리소스를 생성하려면 다음을 수행하세요:

```php
Flight::resource('/users', UsersController::class);
```

그러면 내부적으로 다음 라우트가 생성됩니다:

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

그리고 컨트롤러는 다음 메서드를 사용합니다:

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

> **참고:** `runway`를 사용하여 `php runway routes` 명령으로 새로 추가된 라우트를 확인할 수 있습니다.

#### 리소스 라우트 사용자 정의
리소스 라우트를 구성하는 몇 가지 옵션이 있습니다.

##### 별칭 기본값 (Alias Base)
`aliasBase`를 구성할 수 있습니다. 기본적으로 별칭은 지정된 URL의 마지막 부분입니다. 예를 들어 `/users/`는 `aliasBase`가 `users`가 됩니다. 이러한 라우트가 생성되면 별칭은 `users.index`, `users.create` 등이 됩니다. 별칭을 변경하려면 `aliasBase`를 원하는 값으로 설정하세요.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only 및 Except
`only` 및 `except` 옵션을 사용하여 생성하려는 라우트를 지정할 수도 있습니다.

```php
// 이 메서드들만 허용 목록으로 지정하고 나머지는 차단합니다.
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// 이 메서드들만 차단 목록으로 지정하고 나머지는 허용합니다.
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

기본적으로 허용 목록 및 차단 목록 옵션이므로 생성하려는 라우트를 지정할 수 있습니다.

##### 미들웨어
`resource` 메서드로 생성된 각 라우트에서 실행할 미들웨어를 지정할 수도 있습니다.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### 스트리밍 응답
이제 `stream()` 또는 `streamWithHeaders()`를 사용하여 응답을 클라이언트로 스트리밍할 수 있습니다. 대용량 파일 전송, 장기 실행 프로세스 또는 대용량 응답 생성에 유용합니다. 라우트 스트리밍은 일반 라우트와 약간 다르게 처리됩니다.

> **참고:** 스트리밍 응답은 [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering)이 `false`로 설정된 경우에만 사용할 수 있습니다.

#### 수동 헤더로 스트리밍
라우트에서 `stream()` 메서드를 사용하여 응답을 클라이언트로 스트리밍할 수 있습니다. 이 경우 클라이언트에 무언가를 출력하기 전에 모든 헤더를 직접 설정해야 합니다. 이는 PHP의 `header()` 함수 또는 `Flight::response()->setRealHeader()` 메서드로 수행됩니다.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// 당연히 경로 등을 정리해야 합니다.
	$fileNameSafe = basename($filename);

	// 라우트가 실행된 후 여기에 추가 헤더를 설정해야 한다면
	// 아무것도 출력하기 전에 정의해야 합니다.
	// 모든 헤더는 header() 함수 또는 
	// Flight::response()->setRealHeader() 호출이어야 합니다.
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// 또는
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// 원한다면 콘텐츠 길이를 직접 설정합니다.
	header('Content-Length: '.filesize($filePath));
	// 또는
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// 파일을 읽으면서 클라이언트로 스트리밍합니다.
	readfile($filePath);

// 이것이 핵심 마법 코드입니다.
})->stream();
```

#### 헤더와 함께 스트리밍
`streamWithHeaders()` 메서드를 사용하여 스트리밍을 시작하기 전에 헤더를 설정할 수도 있습니다.

```php
Flight::route('/stream-users', function() {

	// 원하는 추가 헤더를 여기에 추가할 수 있습니다.
	// header() 또는 Flight::response()->setRealHeader()를 사용해야 합니다.

	// 데이터를 가져오는 방식은 자유입니다. 예시일 뿐입니다...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// 데이터를 클라이언트로 보내려면 이 코드가 필요합니다.
		ob_flush();
	}
	echo '}';

// 이렇게 하면 스트리밍을 시작하기 전에 헤더를 설정할 수 있습니다.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// 선택적 상태 코드, 기본값은 200입니다.
	'status' => 200
]);
```

## 같이 보기
- [미들웨어](/learn/middleware) - 인증, 로깅 등에 라우트와 함께 미들웨어 사용하기.
- [의존성 주입](/learn/dependency-injection-container) - 라우트에서 객체 생성 및 관리를 단순화합니다.
- [프레임워크가 왜 필요한가?](/learn/why-frameworks) - Flight와 같은 프레임워크 사용의 이점 이해하기.
- [확장하기](/learn/extending) - `notFound` 메서드를 포함하여 Flight를 자신만의 기능으로 확장하는 방법.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - 정규식 매칭을 위한 PHP 함수.

## 문제 해결
- 라우트 매개변수는 이름이 아닌 순서로 매칭됩니다. 콜백 매개변수 순서가 라우트 정의와 일치하는지 확인하세요.
- `Flight::get()`은 라우트를 정의하지 않습니다. 라우팅에는 `Flight::route('GET /...')` 또는 그룹 내의 Router 객체 컨텍스트(예: `$router->get(...)`)를 사용하세요.
- `executedRoute` 속성은 라우트가 실행된 후에만 설정됩니다. 실행 전에는 NULL입니다.
- 스트리밍은 레거시 Flight 출력 버퍼링 기능을 비활성화해야 합니다 (`flight.v2.output_buffering = false`).
- 의존성 주입의 경우 특정 라우트 정의만 컨테이너 기반 인스턴스 생성을 지원합니다.

### 404 Not Found 또는 예기치 않은 라우트 동작
404 Not Found 오류가 표시되는 경우(하지만 정말로 라우트가 존재하고 오타가 아니라고 확신하는 경우) 라우트 엔드포인트에서 값을 echo하지 않고 반환하고 있기 때문일 수 있습니다. 그 이유는 의도적이지만 일부 개발자에게는 문제가 될 수 있습니다.

```php
Flight::route('/hello', function(){
	// 이렇게 하면 404 Not Found 오류가 발생할 수 있습니다.
	return 'Hello World';
});

// 원하는 방식
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

그 이유는 반환 출력을 "다음 라우트로 이동" 신호로 처리하는 라우터의 특수 메커니즘 때문입니다. 이 동작은 [라우팅](/learn/routing#passing) 섹션에 문서화되어 있습니다.

## 변경 로그
- v3: 리소스 라우팅, 라우트 별칭, 스트리밍 지원, 라우트 그룹 및 미들웨어 지원이 추가되었습니다.
- v1: 대부분의 기본 기능이 제공됩니다.