# 보안

## 개요

보안은 웹 애플리케이션에서 매우 중요한 문제입니다. 애플리케이션이 안전하고 사용자의 데이터가 안전한지 확인해야 합니다. Flight는 웹 애플리케이션을 보호하는 데 도움이 되는 여러 기능을 제공합니다.

## 이해

웹 애플리케이션을 구축할 때 알아야 할 여러 가지 일반적인 보안 위협이 있습니다. 가장 일반적인 위협은 다음과 같습니다:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates)는 기본적으로 출력 이스케이프를 통해 XSS를 방지하므로 이를 기억할 필요가 없습니다. [Sessions](/awesome-plugins/session)는 아래에 설명된 대로 사용자의 세션에 CSRF 토큰을 저장하여 CSRF를 방지할 수 있습니다. PDO와 함께 준비된 문을 사용하면 SQL 인젝션 공격을 방지할 수 있습니다(또는 [PdoWrapper](/learn/pdo-wrapper) 클래스의 유용한 메서드를 사용). CORS는 `Flight::start()`가 호출되기 전에 간단한 훅으로 처리할 수 있습니다.

이러한 모든 메서드는 함께 작동하여 웹 애플리케이션을 안전하게 유지하는 데 도움이 됩니다. 보안 모범 사례를 배우고 이해하는 것은 항상 가장 중요한 사항이어야 합니다.

## 기본 사용법

### 헤더

HTTP 헤더는 웹 애플리케이션을 보호하는 가장 쉬운 방법 중 하나입니다. 헤더를 사용하여 클릭재킹, XSS 및 기타 공격을 방지할 수 있습니다. 
애플리케이션에 이러한 헤더를 추가할 수 있는 방법은 여러 가지가 있습니다.

헤더의 보안을 확인할 수 있는 두 가지 훌륭한 웹사이트는 [securityheaders.com](https://securityheaders.com/)과 
[observatory.mozilla.org](https://observatory.mozilla.org/)입니다. 아래 코드를 설정한 후, 이 두 웹사이트에서 헤더가 작동하는지 쉽게 확인할 수 있습니다.

#### 수동으로 추가

`Flight\Response` 객체의 `header` 메서드를 사용하여 수동으로 이러한 헤더를 추가할 수 있습니다.
```php
// 클릭재킹을 방지하기 위해 X-Frame-Options 헤더 설정
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// XSS를 방지하기 위해 Content-Security-Policy 헤더 설정
// 참고: 이 헤더는 매우 복잡할 수 있으므로 애플리케이션에 맞는 예제를 인터넷에서 참고해야 합니다
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// XSS를 방지하기 위해 X-XSS-Protection 헤더 설정
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// MIME 스니핑을 방지하기 위해 X-Content-Type-Options 헤더 설정
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Referrer-Policy 헤더를 설정하여 리퍼러 정보 전송량 제어
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// HTTPS를 강제하기 위해 Strict-Transport-Security 헤더 설정
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// 사용 가능한 기능과 API를 제어하기 위해 Permissions-Policy 헤더 설정
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

이 코드는 `routes.php` 또는 `index.php` 파일의 상단에 추가할 수 있습니다.

#### 필터로 추가

다음과 같이 필터/훅에 추가할 수도 있습니다: 

```php
// 필터에서 헤더 추가
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

#### 미들웨어로 추가

가장 큰 유연성을 제공하는 미들웨어 클래스로도 추가할 수 있습니다. 일반적으로 이러한 헤더는 모든 HTML 및 API 응답에 적용해야 합니다.

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

// index.php 또는 라우트가 있는 곳
// 참고: 이 빈 문자열 그룹은 모든 라우트에 대한 전역 미들웨어로 작동합니다.
// 물론 특정 라우트에만 추가할 수도 있습니다.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// 더 많은 라우트
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF)는 악의적인 웹사이트가 사용자의 브라우저가 웹사이트에 요청을 보내도록 할 수 있는 공격 유형입니다. 
이것은 사용자의 인지 없이 웹사이트에서 작업을 수행하는 데 사용될 수 있습니다. Flight는 내장된 CSRF 보호 메커니즘을 제공하지 않지만, 
미들웨어를 사용하여 쉽게 구현할 수 있습니다.

#### 설정

먼저 CSRF 토큰을 생성하고 사용자의 세션에 저장해야 합니다. 그런 다음 이 토큰을 폼에 사용하고 폼이 제출될 때 확인할 수 있습니다. 세션 관리를 위해 [flightphp/session](/awesome-plugins/session) 플러그인을 사용하겠습니다.

```php
// CSRF 토큰 생성 및 사용자 세션에 저장
// (세션 객체를 생성하고 Flight에 연결했다고 가정)
// 자세한 내용은 세션 문서를 참조하세요
Flight::register('session', flight\Session::class);

// 세션당 하나의 토큰만 생성하면 됩니다(동일한 사용자의 여러 탭과 요청에서 작동함)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### 기본 PHP Flight 템플릿 사용

```html
<!-- 폼에 CSRF 토큰 사용 -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- 기타 폼 필드 -->
</form>
```

##### Latte 사용

Latte 템플릿에서 CSRF 토큰을 출력하는 커스텀 함수를 설정할 수도 있습니다.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// 기타 설정...

	// CSRF 토큰을 출력하는 커스텀 함수 설정
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

이제 Latte 템플릿에서 `csrf()` 함수를 사용하여 CSRF 토큰을 출력할 수 있습니다.

```html
<form method="post">
	{csrf()}
	<!-- 기타 폼 필드 -->
</form>
```

#### CSRF 토큰 확인

여러 방법을 사용하여 CSRF 토큰을 확인할 수 있습니다.

##### 미들웨어

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

// index.php 또는 라우트가 있는 곳
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// 더 많은 라우트
}, [ CsrfMiddleware::class ]);
```

##### 이벤트 필터

```php
// 이 미들웨어는 요청이 POST 요청인지 확인하고, 그렇다면 CSRF 토큰이 유효한지 확인합니다
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// 폼 값에서 csrf 토큰 캡처
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// 또는 JSON 응답의 경우
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS)은 악의적인 폼 입력이 웹사이트에 코드를 주입할 수 있는 공격 유형입니다. 이러한 기회의 대부분은 최종 사용자가 작성하는 폼 값에서 발생합니다. 사용자의 출력을 **절대** 신뢰해서는 안 됩니다! 모든 사용자가 세계 최고의 해커라고 가정해야 합니다. 그들은 악의적인 JavaScript 또는 HTML을 페이지에 주입할 수 있습니다. 이 코드는 사용자의 정보를 도용하거나 웹사이트에서 작업을 수행하는 데 사용될 수 있습니다. Flight의 뷰 클래스 또는 [Latte](/awesome-plugins/latte)와 같은 다른 템플릿 엔진을 사용하면 출력을 쉽게 이스케이프하여 XSS 공격을 방지할 수 있습니다.

```php
// 사용자가 자신의 이름으로 이것을 사용하려고 한다고 가정해 봅시다
$name = '<script>alert("XSS")</script>';

// 이것은 출력을 이스케이프합니다
Flight::view()->set('name', $name);
// 출력: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Latte를 뷰 클래스로 등록하여 사용하면 자동으로 이스케이프됩니다.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection은 악의적인 사용자가 데이터베이스에 SQL 코드를 주입할 수 있는 공격 유형입니다. 이는 데이터베이스에서 정보를 도용하거나 데이터베이스에서 작업을 수행하는 데 사용될 수 있습니다. 다시 한 번, 사용자의 입력을 **절대** 신뢰해서는 안 됩니다! 항상 그들이 해를 끼치려 한다고 가정해야 합니다. `PDO` 객체에서 준비된 문을 사용하면 SQL 인젝션을 방지할 수 있습니다.

```php
// Flight::db()가 PDO 객체로 등록되어 있다고 가정
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// PdoWrapper 클래스를 사용하면 한 줄로 쉽게 수행할 수 있습니다
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// PDO 객체에서 ? 플레이스홀더를 사용하여 동일한 작업을 수행할 수 있습니다
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### 안전하지 않은 예제

아래는 무해한 예제로부터 보호하기 위해 SQL 준비된 문을 사용하는 이유입니다:

```php
// 최종 사용자가 웹 폼을 작성합니다.
// 폼의 값으로 해커는 다음과 같은 것을 입력합니다:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// 쿼리가 빌드된 후에는 다음과 같이 보입니다
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// 이상해 보이지만 유효한 쿼리이며 작동합니다. 실제로,
// 이는 모든 사용자를 반환하는 매우 일반적인 SQL 인젝션 공격입니다.

var_dump($users); // 이는 단일 사용자 이름이 아닌 데이터베이스의 모든 사용자를 덤프합니다
```

### JSONP 콜백 검증

Flight의 `Flight::jsonp()` 메서드를 사용하는 경우, Flight는 JSONP 콜백 매개변수 이름을 엄격한 허용 목록 정규식(`/^[A-Za-z_$][\w$.]{0,127}$/`)에 대해 검증한다는 점에 유의하세요. 이 패턴과 일치하지 않는 콜백 이름은 Flight가 예외를 발생시켜 악의적인 콜백 값을 통한 임의 JavaScript 주입을 방지합니다.

이 검증은 기본적으로 내장되어 있으며 추가 구성이 필요하지 않지만, JSONP 엔드포인트에서 예상치 못한 오류를 디버깅할 때 알아두는 것이 좋습니다.

### CORS

Cross-Origin Resource Sharing (CORS)은 웹 페이지의 많은 리소스(예: 글꼴, JavaScript 등)가 원본 리소스가 있는 도메인 외부의 다른 도메인에서 요청될 수 있도록 하는 메커니즘입니다. Flight에는 내장 기능이 없지만, `Flight::start()` 메서드가 호출되기 전에 실행되는 훅으로 쉽게 처리할 수 있습니다.

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
		// 여기에 허용된 호스트를 사용자 정의하세요.
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

// index.php 또는 라우트가 있는 곳
$CorsUtil = new CorsUtil();

// 이것은 start가 실행되기 전에 실행되어야 합니다.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Flight 구성 강화

Flight는 보안에 직접적인 영향을 미치는 여러 엔진 설정을 노출합니다. 이러한 설정을 올바르게 지정하는 것은 애플리케이션을 강화하는 가장 쉬운 방법 중 하나입니다.

#### `flight.allow_method_override`

기본적으로 Flight는 클라이언트가 `X-HTTP-Method-Override` 헤더 또는 POST 본문의 `_method` 필드를 사용하여 요청의 HTTP 메서드를 재정의할 수 있도록 합니다. 이것은 `GET`/`POST`만 보낼 수 있는 HTML 폼에 유용하지만, 예상하지 못한 경우 위험할 수 있습니다. 공격자는 일반 폼을 통해 `DELETE` 또는 `PUT` 요청을 위조할 수 있습니다.

애플리케이션이 이 동작에 의존하지 않는 경우(예: 모든 HTTP 동사를 보낼 수 있는 최신 클라이언트 또는 JavaScript 프론트엔드가 사용하는 API를 구축하는 경우), 이를 비활성화해야 합니다:

```php
// Flight::start() 전에 index.php 또는 부트스트랩 파일에서
Flight::set('flight.allow_method_override', false);
```

기본값은 이전 버전과의 호환성을 위해 `true`이지만, 재정의 기능이 명시적으로 필요하지 않은 애플리케이션의 경우 **`false`로 설정하는 것이 강력히 권장됩니다**.

#### `flight.debug`

Flight에는 처리되지 않은 예외가 발생할 때 브라우저에 자세한 오류 정보(예외 메시지, 코드 및 전체 스택 추적)가 렌더링되는지 제어하는 `flight.debug` 설정이 있습니다. 기본값은 `false`이며, 이는 일반적인 `500 Internal Server Error` 메시지만 표시되고 클라이언트에 내부 세부 정보가 유출되지 않음을 의미합니다.

프로덕션 서버에서는 절대 활성화하지 마세요. 로컬 또는 스테이징 환경에서만 사용하세요:

```php
// 로컬 개발 전용 — 프로덕션에서는 절대 사용하지 마세요
Flight::set('flight.debug', true);
```

`flight.debug`가 `false`(기본값)인 경우, `flight.log_errors`를 활성화하여 오류를 캡처할 수 있습니다:

```php
// 클라이언트에 노출하지 않고 서버 측에서 오류 기록
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### 권장 프로덕션 구성

```php
// index.php 또는 app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### 오류 처리
민감한 오류 세부 정보가 공격자에게 유출되지 않도록 프로덕션에서는 숨기세요. 프로덕션에서는 `display_errors`를 0으로 설정하여 오류를 표시하는 대신 로그를 기록하세요.

```php
// bootstrap.php 또는 index.php에서

// app/config/config.php에 추가
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // 오류 표시 비활성화
    ini_set('log_errors', 1);     // 오류를 표시하는 대신 로그 기록
    ini_set('error_log', '/path/to/error.log');
}

// 라우트 또는 컨트롤러에서
// 제어된 오류 응답을 위해 Flight::halt() 사용
Flight::halt(403, 'Access denied');
```

### 입력 살균
사용자 입력을 절대 신뢰하지 마세요. 악의적인 데이터가 유입되지 않도록 [filter_var](https://www.php.net/manual/en/function.filter-var.php)를 사용하여 처리하기 전에 살균하세요.

```php

// $_POST['input'] 및 $_POST['email']이 있는 $_POST 요청이라고 가정

// 문자열 입력 살균
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// 이메일 살균
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### 비밀번호 해싱
[password_hash](https://www.php.net/manual/en/function.password-hash.php) 및 [password_verify](https://www.php.net/manual/en/function.password-verify.php)와 같은 PHP의 내장 함수를 사용하여 비밀번호를 안전하게 저장하고 안전하게 확인하세요. 비밀번호는 절대 일반 텍스트로 저장되어서는 안 되며, 되돌릴 수 있는 방법으로 암호화되어서도 안 됩니다. 해싱은 데이터베이스가 손상되더라도 실제 비밀번호가 보호되도록 보장합니다.

```php
$password = Flight::request()->data->password;
// 저장 시 비밀번호 해싱(예: 등록 중)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// 비밀번호 확인(예: 로그인 중)
if (password_verify($password, $stored_hash)) {
    // 비밀번호 일치
}
```

### 속도 제한
캐시를 사용하여 요청 속도를 제한하여 무차별 대입 공격 또는 서비스 거부 공격으로부터 보호하세요.

```php
// flightphp/cache가 설치 및 등록되어 있다고 가정
// 필터에서 flightphp/cache 사용
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // 60초 후 재설정
});
```

## 참고 항목
- [Sessions](/awesome-plugins/session) - 사용자 세션을 안전하게 관리하는 방법.
- [Templates](/learn/templates) - XSS를 방지하기 위해 출력을 자동으로 이스케이프하는 템플릿 사용.
- [PDO Wrapper](/learn/pdo-wrapper) - 준비된 문을 사용한 간소화된 데이터베이스 상호 작용.
- [Middleware](/learn/middleware) - 보안 헤더 추가 프로세스를 간소화하기 위한 미들웨어 사용 방법.
- [Responses](/learn/responses) - 보안 헤더로 HTTP 응답을 사용자 정의하는 방법.
- [Requests](/learn/requests) - 사용자 입력을 처리하고 살균하는 방법.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - 입력 살균을 위한 PHP 함수.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - 안전한 비밀번호 해싱을 위한 PHP 함수.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - 해시된 비밀번호를 확인하기 위한 PHP 함수.

## 문제 해결
- Flight Framework의 구성 요소와 관련된 문제에 대한 문제 해결 정보는 위의 "참고 항목" 섹션을 참조하세요.

## 변경 로그
- v3.18.1 - `flight.allow_method_override`, `flight.debug` 및 JSONP 콜백 검증을 다루는 Flight 구성 강화 섹션 추가.
- v3.1.0 - CORS, 오류 처리, 입력 살균, 비밀번호 해싱 및 속도 제한에 대한 섹션 추가.
- v2.0 - XSS를 방지하기 위해 기본 뷰에 이스케이프 추가.