# Безопасность

## Обзор

Безопасность — это важный аспект при разработке веб-приложений. Вы хотите убедиться, что ваше приложение безопасно, а данные пользователей защищены. Flight предоставляет ряд функций, которые помогут вам защитить ваши веб-приложения.

## Понимание

Существует ряд распространённых угроз безопасности, о которых следует знать при создании веб-приложений. Наиболее распространённые угрозы включают:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates) помогают предотвратить XSS, экранируя вывод по умолчанию, так что вам не нужно об этом помнить. [Sessions](/awesome-plugins/session) могут помочь с CSRF, сохраняя CSRF-токен в сессии пользователя, как описано ниже. Использование подготовленных запросов с PDO помогает предотвратить атаки SQL-инъекций (или с использованием удобных методов в классе [PdoWrapper](/learn/pdo-wrapper)). CORS можно обработать с помощью простого хука перед вызовом `Flight::start()`.

Все эти методы работают вместе, помогая обеспечить безопасность ваших веб-приложений. Всегда следует помнить о необходимости изучать и понимать лучшие практики безопасности.

## Основное использование

### Заголовки

HTTP-заголовки — один из самых простых способов защиты веб-приложений. С их помощью можно предотвратить clickjacking, XSS и другие атаки. 
Существует несколько способов добавления этих заголовков в приложение.

Два отличных сайта для проверки безопасности ваших заголовков — [securityheaders.com](https://securityheaders.com/) и 
[observatory.mozilla.org](https://observatory.mozilla.org/). После настройки приведённого ниже кода вы легко сможете убедиться, что заголовки работают, с помощью этих двух сайтов.

#### Добавление вручную

Вы можете вручную добавить эти заголовки, используя метод `header` объекта `Flight\Response`.
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

Их можно добавить в начало файлов `routes.php` или `index.php`.

#### Добавление в виде фильтра

Вы также можете добавить их в фильтр/хук следующим образом: 

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

#### Добавление в виде middleware

Вы также можете добавить их в виде класса middleware, что обеспечивает наибольшую гибкость в выборе маршрутов для применения. В общем случае эти заголовки следует применять ко всем HTML и API-ответам.

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

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) — это тип атаки, при которой вредоносный сайт может заставить браузер пользователя отправить запрос на ваш сайт. 
Это может быть использовано для выполнения действий на вашем сайте без ведома пользователя. Flight не предоставляет встроенного механизма защиты от CSRF, 
но вы легко можете реализовать собственный с помощью middleware.

#### Настройка

Сначала необходимо сгенерировать CSRF-токен и сохранить его в сессии пользователя. Затем вы можете использовать этот токен в формах и проверять его при 
отправке формы. Мы будем использовать плагин [flightphp/session](/awesome-plugins/session) для управления сессиями.

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

##### Использование шаблона по умолчанию PHP Flight

```html
<!-- Use the CSRF token in your form -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- other form fields -->
</form>
```

##### Использование Latte

Вы также можете задать пользовательскую функцию для вывода CSRF-токена в шаблонах Latte.

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

Теперь в шаблонах Latte вы можете использовать функцию `csrf()` для вывода CSRF-токена.

```html
<form method="post">
	{csrf()}
	<!-- other form fields -->
</form>
```

#### Проверка CSRF-токена

Проверить CSRF-токен можно несколькими способами.

##### Middleware

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

##### Фильтры событий

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

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) — это тип атаки, при которой вредоносный ввод из формы может внедрить код на ваш сайт. Большинство таких возможностей возникает 
из значений формы, которые заполняют конечные пользователи. Вы **никогда** не должны доверять выводу от пользователей! Всегда исходите из того, что все они — 
лучшие хакеры в мире. Они могут внедрить вредоносный JavaScript или HTML на вашу страницу. Этот код может быть использован для кражи информации у ваших 
пользователей или выполнения действий на вашем сайте. Используя класс представлений Flight или другой шаблонизатор, такой как [Latte](/awesome-plugins/latte), вы легко можете экранировать вывод для предотвращения XSS-атак.

```php
// Let's assume the user is clever as tries to use this as their name
$name = '<script>alert("XSS")</script>';

// This will escape the output
Flight::view()->set('name', $name);
// This will output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// If you use something like Latte registered as your view class, it will also auto escape this.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection — это тип атаки, при которой злоумышленник может внедрить SQL-код в вашу базу данных. Это может быть использовано для кражи информации 
из базы данных или выполнения действий с ней. Опять же, вы **никогда** не должны доверять вводу от пользователей! Всегда исходите из того, что они 
намерены навредить. Использование подготовленных запросов в объектах `PDO` поможет предотвратить SQL-инъекции.

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

#### Небезопасный пример

Ниже показано, почему мы используем SQL-подготовленные запросы для защиты от безобидных примеров вроде следующего:

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

### Проверка JSONP Callback

Если вы используете метод `Flight::jsonp()`, имейте в виду, что Flight проверяет имя параметра JSONP callback по строгому allowlist regex (`/^[A-Za-z_$][\w$.]{0,127}$/`). Любое имя callback, не соответствующее этому шаблону, вызовет исключение во Flight, предотвращая внедрение произвольного JavaScript через вредоносное значение callback.

Эта проверка встроена и не требует дополнительной настройки, но о ней стоит знать при отладке неожиданных ошибок из JSONP-эндпоинтов.

### CORS

Cross-Origin Resource Sharing (CORS) — это механизм, который позволяет запрашивать множество ресурсов (например, шрифты, JavaScript и т.д.) на веб-странице 
с другого домена, отличного от того, откуда произошёл ресурс. Flight не имеет встроенной функциональности, 
но это легко можно обработать с помощью хука, который запускается перед вызовом метода `Flight::start()`.

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

### Усиление конфигурации Flight

Flight предоставляет несколько настроек движка, которые напрямую влияют на безопасность. Правильная настройка этих параметров — один из самых простых способов усилить защиту приложения.

#### `flight.allow_method_override`

По умолчанию Flight позволяет клиентам переопределять HTTP-метод запроса с помощью заголовка `X-HTTP-Method-Override` или поля `_method` в теле POST-запроса. Хотя это удобно для HTML-форм, которые могут отправлять только `GET`/`POST`, это может быть опасно, если вы этого не ожидаете — злоумышленник может подделать `DELETE` или `PUT` запросы через обычную форму.

Если ваше приложение не полагается на такое поведение (например, вы создаёте API, потребляемое современными клиентами или JavaScript-фронтендами, которые могут отправлять любой HTTP-метод), вы должны отключить его:

```php
// In your index.php or bootstrap file, before Flight::start()
Flight::set('flight.allow_method_override', false);
```

Значение по умолчанию — `true` для обратной совместимости, но **рекомендуется установить значение `false`** для любого приложения, которому явно не требуется функция переопределения.

#### `flight.debug`

Flight имеет настройку `flight.debug`, которая управляет тем, отображается ли подробная информация об ошибке (сообщение исключения, код и полный стек вызовов) в браузере при возникновении необработанного исключения. По умолчанию значение `false`, что означает, что клиенту показывается только общее сообщение `500 Internal Server Error` — внутренние детали не передаются.

Никогда не включайте эту опцию на продакшен-сервере. Используйте её только локально или в staging-окружении:

```php
// Safe for local development only — NEVER in production
Flight::set('flight.debug', true);
```

Когда `flight.debug` равно `false` (по умолчанию), вы всё равно можете фиксировать ошибки, включив `flight.log_errors`:

```php
// Log errors server-side without exposing them to the client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Рекомендуемая конфигурация для продакшена

```php
// index.php or app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Обработка ошибок
Скрывайте конфиденциальные детали ошибок в продакшене, чтобы избежать утечки информации злоумышленникам. В продакшене логируйте ошибки вместо их отображения, установив `display_errors` в `0`.

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

### Санитизация ввода
Никогда не доверяйте пользовательскому вводу. Санитизируйте его с помощью [filter_var](https://www.php.net/manual/en/function.filter-var.php) перед обработкой, чтобы предотвратить попадание вредоносных данных.

```php

// Lets assume a $_POST request with $_POST['input'] and $_POST['email']

// Sanitize a string input
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitize an email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Хеширование паролей
Храните пароли безопасно и проверяйте их с помощью встроенных функций PHP, таких как [password_hash](https://www.php.net/manual/en/function.password-hash.php) и [password_verify](https://www.php.net/manual/en/function.password-verify.php). Пароли никогда не должны храниться в открытом виде или шифроваться обратимыми методами. Хеширование гарантирует, что даже при компрометации базы данных реальные пароли останутся защищёнными.

```php
$password = Flight::request()->data->password;
// Hash a password when storing (e.g., during registration)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verify a password (e.g., during login)
if (password_verify($password, $stored_hash)) {
    // Password matches
}
```

### Ограничение частоты запросов
Защищайтесь от brute force-атак или атак типа «отказ в обслуживании», ограничивая частоту запросов с помощью кэша.

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

## См. также
- [Sessions](/awesome-plugins/session) - Как безопасно управлять пользовательскими сессиями.
- [Templates](/learn/templates) - Использование шаблонов для автоматического экранирования вывода и предотвращения XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Упрощённое взаимодействие с базой данных с помощью подготовленных запросов.
- [Middleware](/learn/middleware) - Как использовать middleware для упрощения добавления security-заголовков.
- [Responses](/learn/responses) - Как настраивать HTTP-ответы с безопасными заголовками.
- [Requests](/learn/requests) - Как обрабатывать и санитизировать пользовательский ввод.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - PHP-функция для санитизации ввода.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - PHP-функция для безопасного хеширования паролей.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - PHP-функция для проверки хешированных паролей.

## Устранение неполадок
- Обратитесь к разделу «См. также» выше для получения информации по устранению неполадок, связанных с компонентами Flight Framework.

## Журнал изменений
- v3.18.1 - Добавлен раздел Flight Configuration Hardening, охватывающий `flight.allow_method_override`, `flight.debug` и проверку JSONP callback.
- v3.1.0 - Добавлены разделы о CORS, обработке ошибок, санитизации ввода, хешировании паролей и ограничении частоты запросов.
- v2.0 - Добавлено экранирование для представлений по умолчанию для предотвращения XSS.