# Безопасность

## Обзор

Безопасность очень важна для веб-приложений. Вы хотите убедиться, что ваше приложение защищено, а данные пользователей в безопасности. Flight предоставляет ряд функций, которые помогут вам защитить ваши веб-приложения.

Официальный [скелет](https://github.com/flightphp/skeleton) также включает специальный **`SECURITY.md`** и middleware для заголовков безопасности, чтобы [инструменты ИИ-кодирования](/learn/ai) (и люди) имели одно продуманное место для секретов, заголовков и правил XSS/SQL — отдельно от общего стиля кодирования в `AGENTS.md`.

## Понимание

Существует ряд распространённых угроз безопасности, о которых следует знать при создании веб-приложений. Некоторые из наиболее распространённых угроз включают:
- Межсайтовая подделка запросов (CSRF)
- Межсайтовый скриптинг (XSS)
- SQL-инъекции
- Совместное использование ресурсов между источниками (CORS)

[Шаблоны](/learn/templates) помогают с XSS, экранируя вывод по умолчанию (Twig и Latte делают это; используйте это преимущество). [Сессии](/awesome-plugins/session) могут помочь с CSRF, сохраняя CSRF-токен в сессии пользователя, как описано ниже. Использование подготовленных запросов с PDO — или помощников в [SimplePdo](/learn/simple-pdo) — помогает предотвратить SQL-инъекции. CORS можно обработать с помощью простого хука перед вызовом `Flight::start()`.

Все эти методы работают вместе, чтобы поддерживать безопасность ваших веб-приложений. Всегда следует помнить о необходимости изучения и понимания лучших практик безопасности. Не просите ИИ-ассистента «отключить CSP» или ослабить заголовки только для того, чтобы страница загрузилась, не понимая компромиссов.

## Базовое использование

### Заголовки

HTTP-заголовки — один из самых простых способов защитить ваши веб-приложения. Вы можете использовать заголовки для предотвращения кликджекинга, XSS и других атак. Существует несколько способов добавить эти заголовки в ваше приложение.

Два отличных сайта для проверки безопасности ваших заголовков: [securityheaders.com](https://securityheaders.com/) и [observatory.mozilla.org](https://observatory.mozilla.org/). После настройки кода ниже вы сможете легко проверить, что ваши заголовки работают, с помощью этих двух сайтов.

Скелет включает `App\Middleware\SecurityHeadersMiddleware` (CSP с nonce для каждого запроса, frame options, HSTS и другое). Предпочтительнее осознанно расширять его, а не отключать заголовки.

#### Добавление вручную

Вы можете вручную добавить эти заголовки с помощью метода `header` объекта `Flight\Response`.

```php
// Устанавливает заголовок X-Frame-Options для предотвращения кликджекинга
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Устанавливает заголовок Content-Security-Policy для предотвращения XSS
// Примечание: этот заголовок может быть очень сложным, поэтому вам понадобится
//  обратиться к примерам в интернете для вашего приложения
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Устанавливает заголовок X-XSS-Protection для предотвращения XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Устанавливает заголовок X-Content-Type-Options для предотвращения определения MIME-типа по содержимому
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Устанавливает заголовок Referrer-Policy для контроля объёма передаваемой информации о источнике
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Устанавливает заголовок Strict-Transport-Security для принудительного использования HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Устанавливает заголовок Permissions-Policy для управления доступными функциями и API
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Их можно добавить в начало ваших файлов `routes.php` или `index.php`.

#### Добавление в качестве фильтра

Вы также можете добавить их в фильтр/хук следующим образом:

```php
// Добавить заголовки в фильтре
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

#### Добавление в качестве middleware

Вы также можете добавить их как класс middleware, что обеспечивает наибольшую гибкость в выборе маршрутов, к которым это применяется. В целом эти заголовки должны применяться ко всем HTML и API ответам.

Путь и пространство имён в стиле скелета (**регистр папки соответствует `App\Middleware`**):

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
		// Предпочитайте CSP nonce из bootstrap при наличии встроенных скриптов (скелет устанавливает csp_nonce)
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

// app/config/routes.php — группа с пустой строкой = глобальный middleware для всех маршрутов
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// другие маршруты
}, [SecurityHeadersMiddleware::class]);
```

Старые проекты могут по-прежнему использовать `app/middlewares` и `app\middlewares`; это работает, если папки совпадают. Новые приложения на скелете используют **`app/Middleware/`** и **`App\Middleware`**. См. [Автозагрузка](/learn/autoloading).

### Межсайтовая подделка запросов (CSRF)

Межсайтовая подделка запросов (CSRF) — это тип атаки, при которой вредоносный веб-сайт может заставить браузер пользователя отправить запрос на ваш сайт. Это может быть использовано для выполнения действий на вашем сайте без ведома пользователя. Flight не предоставляет встроенного механизма защиты от CSRF, но вы можете легко реализовать свой собственный с помощью middleware.

#### Настройка

Сначала вам нужно сгенерировать CSRF-токен и сохранить его в сессии пользователя. Затем вы можете использовать этот токен в своих формах и проверять его при отправке формы. Мы будем использовать плагин [flightphp/session](/awesome-plugins/session) для управления сессиями.

```php
// Генерируем CSRF-токен и сохраняем его в сессии пользователя
// (предполагая, что вы создали объект сессии и прикрепили его к Flight)
// см. документацию по сессиям для получения дополнительной информации
Flight::register('session', flight\Session::class);

// Вам нужно генерировать только один токен на сессию (чтобы он работал
// для нескольких вкладок и запросов одного пользователя)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Использование стандартного PHP шаблона Flight

```html
<!-- Используйте CSRF-токен в вашей форме -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- остальные поля формы -->
</form>
```

##### Использование Twig (по умолчанию в скелете)

Зарегистрируйте функцию Twig или передавайте токен в каждое представление формы. Минимальный пример с глобальной переменной + полем формы:

```php
// При настройке Twig (например, services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# остальные поля #}
</form>
```

##### Использование Latte

Вы также можете настроить пользовательскую функцию для вывода CSRF-токена в ваших шаблонах Latte.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// другие настройки...

	// Устанавливаем пользовательскую функцию для вывода CSRF-токена
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

И теперь в ваших шаблонах Latte вы можете использовать функцию `csrf()` для вывода CSRF-токена.

```html
<form method="post">
	{csrf()}
	<!-- остальные поля формы -->
</form>
```

#### Проверка CSRF-токена

Вы можете проверить CSRF-токен несколькими способами.

##### Middleware

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
	// другие маршруты
}, [CsrfMiddleware::class]);
```

##### Событийные фильтры

```php
// Этот middleware проверяет, является ли запрос POST-запросом, и если да, проверяет валидность CSRF-токена
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// получаем csrf-токен из значений формы
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// или для JSON-ответа
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Межсайтовый скриптинг (XSS)

Межсайтовый скриптинг (XSS) — это тип атаки, при которой вредоносный ввод в форме может внедрить код на ваш сайт. Большинство этих возможностей исходят из значений форм, которые заполняют ваши конечные пользователи. Вам **никогда** не следует доверять выводу от ваших пользователей! Всегда предполагайте, что все они — лучшие хакеры в мире. Они могут внедрить вредоносный JavaScript или HTML на вашу страницу. Этот код может быть использован для кражи информации у ваших пользователей или выполнения действий на вашем сайте. Используя класс представления Flight или шаблонизатор, такой как [Twig](/awesome-plugins/twig) или [Latte](/awesome-plugins/latte), вы можете легко экранировать вывод для предотвращения XSS-атак.

```php
// Предположим, пользователь умный и пытается использовать это в качестве своего имени
$name = '<script>alert("XSS")</script>';

// Это экранирует вывод
Flight::view()->set('name', $name);
// Это выведет: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (по умолчанию в скелете) и Latte экранируют по умолчанию — предпочитайте их обычному PHP echo
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → экранировано
// Избегайте |raw / неэкранированного вывода, если только контент полностью не доверенный
```

### SQL-инъекции

SQL-инъекция — это тип атаки, при которой злоумышленник может внедрить SQL-код в вашу базу данных. Это может быть использовано для кражи информации из вашей базы данных или выполнения действий с вашей базой данных. Опять же, вам **никогда** не следует доверять вводу от ваших пользователей! Всегда предполагайте, что они жаждут крови. Используйте подготовленные запросы — помощники [SimplePdo](/learn/simple-pdo) делают это стандартным путём.

```php
// Предполагая, что Flight::db() зарегистрирован как SimplePdo (или вы внедряете SimplePdo в контроллер)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (предпочтительно) — однострочные запросы со связанными параметрами
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Та же идея с плейсхолдерами ?
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

В контроллерах в стиле скелета предпочитайте внедрение `SimplePdo` через конструктор вместо `Flight::db()`, чтобы тесты и ИИ-сгенерированный код оставались согласованными ([DIC](/learn/dependency-injection-container)).

#### Небезопасный пример

Ниже показано, почему мы используем подготовленные SQL-запросы для защиты от таких безобидных примеров, как этот:

```php
// конечный пользователь заполняет веб-форму.
// в поле формы хакер вводит что-то вроде:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// После сборки запроса он выглядит так
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Это выглядит странно, но это валидный запрос, который будет работать. На самом деле,
// это очень распространённая SQL-инъекция, которая вернёт всех пользователей.

var_dump($users); // этот выведет всех пользователей в базе данных, а не только одно имя пользователя
```

### Секреты и конфигурация

- Помещайте секреты в **`.env`** (или в реальное окружение), а не в коммитируемые образцы `config.php`.
- Правило скелета: буквальные значения по умолчанию в `config.php`; объединяйте окружение в bootstrap; **не** читайте `$_ENV` внутри контроллеров — вместо этого внедряйте конфигурацию. См. [Конфигурация](/learn/configuration).
- Никогда не коммитьте API-ключи, пароли БД или ключи шифрования сессий. Направляйте ИИ-инструменты на **`SECURITY.md`**, чтобы они не придумывали небезопасные обходные пути.

### Проверка JSONP-колбэка

Если вы используете метод `Flight::jsonp()` во Flight, учтите, что Flight проверяет имя параметра callback в JSONP на соответствие строгому регулярному выражению из белого списка (`/^[A-Za-z_$][\w$.]{0,127}$/`). Любое имя callback, не соответствующее этому шаблону, приведёт к исключению во Flight, предотвращая внедрение произвольного JavaScript через вредоносное значение callback.

Эта проверка встроена и не требует дополнительной настройки, но о ней полезно знать при отладке неожиданных ошибок от JSONP-эндпоинтов.

### CORS

Совместное использование ресурсов между источниками (CORS) — это механизм, который позволяет запрашивать многие ресурсы (например, шрифты, JavaScript и т.д.) на веб-странице с другого домена, отличного от домена, на котором ресурс был создан. Flight не имеет встроенной функциональности, но это легко можно обработать с помощью хука, который выполняется перед вызовом метода `Flight::start()`.

```php
// app/Utils/CorsUtil.php (скелет: папка Utils в PascalCase → App\Utils)

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
		// настройте здесь ваши разрешённые хосты.
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

// bootstrap / маршруты — запускается перед стартом
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Усиление конфигурации Flight

Flight предоставляет несколько настроек движка, которые имеют прямое влияние на безопасность. Правильная их настройка — один из самых простых способов усилить защиту вашего приложения.

#### `flight.allow_method_override`

По умолчанию Flight позволяет клиентам переопределять HTTP-метод запроса с помощью заголовка `X-HTTP-Method-Override` или поля `_method` в теле POST. Хотя это удобно для HTML-форм, которые могут отправлять только `GET`/`POST`, это может быть опасно, если вы этого не ожидаете — злоумышленник может подделать `DELETE` или `PUT` запросы через обычную форму.

Если ваше приложение не полагается на это поведение (например, вы создаёте API, потребляемое современными клиентами или JavaScript-фронтендами, которые могут отправлять любые HTTP-глаголы), вам следует отключить его:

```php
// В вашем файле index.php или bootstrap, перед Flight::start()
Flight::set('flight.allow_method_override', false);
```

Значение по умолчанию — `true` для обратной совместимости, но **настоятельно рекомендуется устанавливать `false`** для любого приложения, которому явно не нужна функция переопределения.

#### `flight.debug`

Во Flight есть настройка `flight.debug`, которая управляет отображением подробной информации об ошибке (сообщение исключения, код и полный стек-трейс) в браузере при возникновении необработанного исключения. По умолчанию `false`, что означает показ только общего сообщения `500 Internal Server Error` — никакие внутренние детали не утекают клиенту.

Никогда не включайте это на production-сервере. Используйте только локально или в staging-окружении:

```php
// Безопасно только для локальной разработки — НИКОГДА в production
Flight::set('flight.debug', true);
```

Когда `flight.debug` имеет значение `false` (по умолчанию), вы по-прежнему можете перехватывать ошибки, включив `flight.log_errors`:

```php
// Логировать ошибки на стороне сервера, не раскрывая их клиенту
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Рекомендуемая production-конфигурация

```php
// index.php или применяется из конфигурации приложения / bootstrap
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Обработка ошибок

Скрывайте чувствительные детали ошибок в production, чтобы избежать утечки информации злоумышленникам. В production логируйте ошибки вместо их отображения, установив `display_errors` в `0`.

```php
// В вашем bootstrap.php или index.php

// добавьте это в ваш app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Отключить отображение ошибок
    ini_set('log_errors', 1);     // Вместо этого логировать ошибки
    ini_set('error_log', '/path/to/error.log');
}

// В ваших маршрутах или контроллерах
// Используйте Flight::halt() для контролируемых ответов с ошибками
Flight::halt(403, 'Access denied');
```

### Санитизация ввода

Никогда не доверяйте пользовательскому вводу. Санитизируйте его с помощью [filter_var](https://www.php.net/manual/en/function.filter-var.php) перед обработкой, чтобы предотвратить проникновение вредоносных данных. Предпочитайте чтение ввода через `$app->request()` (или `Flight::request()`) вместо сырых `$_GET` / `$_POST` в коде приложения.

```php

// Предположим, есть $_POST запрос с $_POST['input'] и $_POST['email']

// Санитизируем строковый ввод
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Санитизируем email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Хеширование паролей

Храните пароли безопасно и проверяйте их с помощью встроенных функций PHP, таких как [password_hash](https://www.php.net/manual/en/function.password-hash.php) и [password_verify](https://www.php.net/manual/en/function.password-verify.php). Пароли никогда не должны храниться в открытом виде, а также не должны шифроваться обратимыми методами. Хеширование гарантирует, что даже в случае компрометации вашей базы данных фактические пароли останутся защищёнными.

```php
$password = Flight::request()->data->password;
// Хешируйте пароль при сохранении (например, при регистрации)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Проверяйте пароль (например, при входе)
if (password_verify($password, $stored_hash)) {
    // Пароль совпадает
}
```

### Ограничение частоты запросов

Защититесь от атак методом перебора или отказов в обслуживании, ограничивая частоту запросов с помощью кэша.

```php
// Предполагая, что у вас установлен и зарегистрирован flightphp/cache
// Использование flightphp/cache в фильтре
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Сброс через 60 секунд
});
```

## Смотрите также

- [Сессии](/awesome-plugins/session) — Как безопасно управлять пользовательскими сессиями.
- [Шаблоны](/learn/templates) — Twig/Latte авто-экранирование и XSS.
- [SimplePdo](/learn/simple-pdo) — Помощники для работы с БД с подготовленными запросами.
- [PdoWrapper](/learn/pdo-wrapper) — Устарел; используйте SimplePdo для нового кода.
- [Middleware](/learn/middleware) — Как использовать middleware для упрощения процесса добавления заголовков безопасности.
- [Конфигурация](/learn/configuration) — `.env` против буквальной конфигурации, production-флаги.
- [AI и опыт разработчика](/learn/ai) — Держите политику безопасности в `SECURITY.md` для агентов.
- [Ответы](/learn/responses) — Как настраивать HTTP-ответы с безопасными заголовками.
- [Запросы](/learn/requests) — Как обрабатывать и санитизировать пользовательский ввод.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) — PHP-функция для санитизации ввода.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) — PHP-функция для безопасного хеширования паролей.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) — PHP-функция для проверки хешированных паролей.

## Устранение неполадок

- Обратитесь к разделу «Смотрите также» выше для получения информации по устранению неполадок, связанных с компонентами Flight Framework.
- Если CSP блокирует ваши скрипты, добавьте nonce (паттерн скелета) или добавьте конкретные источники в белый список — не устанавливайте `script-src *` без плана.

## Журнал изменений

- Документация — скелет `App\Middleware`, заметки по Twig CSRF/XSS, SimplePdo, секреты/`.env` и `SECURITY.md` для AI-ориентированных проектов.
- v3.18.1 — Добавлен раздел усиления конфигурации Flight, охватывающий `flight.allow_method_override`, `flight.debug` и проверку JSONP-колбэка.
- v3.1.0 — Добавлены разделы о CORS, обработке ошибок, санитизации ввода, хешировании паролей и ограничении частоты запросов.
- v2.0 — Добавлено экранирование для стандартных представлений для предотвращения XSS.