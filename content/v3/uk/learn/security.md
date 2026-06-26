# Безпека

## Огляд

Безпека є дуже важливою справою, коли мова йде про веб-додатки. Ви хочете переконатися, що ваш додаток є безпечним і що дані ваших користувачів 
є захищеними. Flight надає низку функцій, які допоможуть вам захистити ваші веб-додатки.

## Розуміння

Існує низка поширених загроз безпеці, про які ви повинні знати, створюючи веб-додатки. Деякі з найпоширеніших загроз
включають:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Шаблони](/learn/templates) допомагають з XSS, екрануючи вивід за замовчуванням, тому вам не потрібно про це пам'ятати. [Сесії](/awesome-plugins/session) можуть допомогти з CSRF, зберігаючи CSRF-токен у сесії користувача, як описано нижче. Використання підготовлених запитів з PDO може допомогти запобігти атакам SQL-ін'єкцій (або використання зручних методів у класі [PdoWrapper](/learn/pdo-wrapper)). CORS можна обробити за допомогою простого хука перед викликом `Flight::start()`.

Усі ці методи працюють разом, щоб допомогти підтримувати безпеку ваших веб-додатків. Це завжди повинно бути на передньому плані вашого розуму — вивчати та розуміти найкращі практики безпеки.

## Базове використання

### Заголовки

HTTP-заголовки — один з найпростіших способів захистити ваші веб-додатки. Ви можете використовувати заголовки для запобігання clickjacking, XSS та іншим атакам. 
Є кілька способів, якими ви можете додати ці заголовки до вашого додатку.

Два чудових веб-сайти для перевірки безпеки ваших заголовків — [securityheaders.com](https://securityheaders.com/) та 
[observatory.mozilla.org](https://observatory.mozilla.org/). Після налаштування наведеного нижче коду ви легко зможете перевірити, чи працюють ваші заголовки, за допомогою цих двох веб-сайтів.

#### Додавання вручну

Ви можете вручну додати ці заголовки, використовуючи метод `header` об'єкта `Flight\Response`.
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

Їх можна додати у верхній частині файлів `routes.php` або `index.php`.

#### Додавання як фільтр

Ви також можете додати їх у фільтр/хук наступним чином: 

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

#### Додавання як проміжне ПЗ (Middleware)

Ви також можете додати їх як клас проміжного ПЗ, що забезпечує найбільшу гнучкість щодо того, до яких маршрутів це застосовувати. Загалом, ці заголовки слід застосовувати до всіх HTML- та API-відповідей.

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

Cross Site Request Forgery (CSRF) — це тип атаки, коли зловмисний веб-сайт може змусити браузер користувача надіслати запит на ваш веб-сайт. 
Це може бути використано для виконання дій на вашому веб-сайті без відома користувача. Flight не надає вбудованого механізму захисту від CSRF, 
але ви можете легко реалізувати свій власний за допомогою проміжного ПЗ.

#### Налаштування

Спочатку вам потрібно згенерувати CSRF-токен і зберегти його у сесії користувача. Потім ви можете використовувати цей токен у ваших формах і перевіряти його під час 
подання форми. Ми будемо використовувати плагін [flightphp/session](/awesome-plugins/session) для керування сесіями.

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

##### Використання стандартного шаблону PHP Flight

```html
<!-- Use the CSRF token in your form -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- other form fields -->
</form>
```

##### Використання Latte

Ви також можете встановити власну функцію для виведення CSRF-токена у ваших шаблонах Latte.

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

А тепер у ваших шаблонах Latte ви можете використовувати функцію `csrf()` для виведення CSRF-токена.

```html
<form method="post">
	{csrf()}
	<!-- other form fields -->
</form>
```

#### Перевірка CSRF-токена

Ви можете перевірити CSRF-токен кількома способами.

##### Проміжне ПЗ

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

##### Фільтри подій

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

Cross Site Scripting (XSS) — це тип атаки, коли зловмисне введення форми може впровадити код на ваш веб-сайт. Більшість таких можливостей походить 
від значень форми, які заповнюють ваші кінцеві користувачі. Ви **ніколи** не повинні довіряти виводу від ваших користувачів! Завжди припускайте, що всі вони є 
найкращими хакерами у світі. Вони можуть впроваджувати шкідливий JavaScript або HTML на вашу сторінку. Цей код може бути використаний для крадіжки інформації у ваших 
користувачів або виконання дій на вашому веб-сайті. Використовуючи клас представлення Flight або інший шаблонізатор, наприклад [Latte](/awesome-plugins/latte), ви можете легко екранувати вивід, щоб запобігти атакам XSS.

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

SQL-ін'єкція — це тип атаки, коли зловмисний користувач може впровадити SQL-код у вашу базу даних. Це може бути використано для крадіжки інформації 
з вашої бази даних або виконання дій над вашою базою даних. Знову ж таки, ви **ніколи** не повинні довіряти введенню від ваших користувачів! Завжди припускайте, що вони 
намагаються нашкодити. Ви можете використовувати підготовлені запити у ваших об'єктах `PDO`, що запобіжить SQL-ін'єкціям.

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

#### Приклад небезпечного коду

Нижче наведено приклад того, чому ми використовуємо підготовлені SQL-запити для захисту від подібних прикладів:

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

### Перевірка JSONP Callback

Якщо ви використовуєте метод `Flight::jsonp()`, зверніть увагу, що Flight перевіряє ім'я параметра callback JSONP за допомогою суворого регулярного виразу (`/^[A-Za-z_$][\w$.]{0,127}$/`). Будь-яке ім'я callback, яке не відповідає цьому шаблону, призведе до того, що Flight викине виняток, запобігаючи впровадженню довільного JavaScript через шкідливе значення callback.

Ця перевірка вбудована і не потребує додаткового налаштування, але про неї варто знати під час налагодження несподіваних помилок з JSONP-ендпоінтів.

### CORS

Cross-Origin Resource Sharing (CORS) — це механізм, який дозволяє багатьом ресурсам (наприклад, шрифтам, JavaScript тощо) на веб-сторінці бути 
запитаними з іншого домену, відмінного від того, з якого походить ресурс. У Flight немає вбудованої функціональності, 
але це легко можна обробити за допомогою хука, який запускається перед викликом методу `Flight::start()`.

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

### Зміцнення конфігурації Flight

Flight надає кілька налаштувань движка, які мають прямі наслідки для безпеки. Правильне налаштування цих параметрів — один з найпростіших способів зміцнити ваш додаток.

#### `flight.allow_method_override`

За замовчуванням Flight дозволяє клієнтам перевизначати HTTP-метод запиту за допомогою заголовка `X-HTTP-Method-Override` або поля `_method` у тілі POST-запиту. Хоча це зручно для HTML-форм, які можуть надсилати лише `GET`/`POST`, це може бути небезпечно, якщо ви цього не очікуєте — зловмисник може підробити запити `DELETE` або `PUT` через звичайну форму.

Якщо ваш додаток не покладається на таку поведінку (наприклад, ви створюєте API, яке використовується сучасними клієнтами або JavaScript-фронтендами, які можуть надсилати будь-який HTTP-метод), ви повинні вимкнути цю функцію:

```php
// In your index.php or bootstrap file, before Flight::start()
Flight::set('flight.allow_method_override', false);
```

Значення за замовчуванням — `true` для зворотної сумісності, але **встановлення значення `false` настійно рекомендується** для будь-якого додатка, якому явно не потрібна функція перевизначення.

#### `flight.debug`

Flight має налаштування `flight.debug`, яке контролює, чи буде детальна інформація про помилку (повідомлення винятку, код та повний стек викликів) відображатися у браузері, коли виникає необроблений виняток. За замовчуванням значення — `false`, що означає, що буде показано лише загальне повідомлення `500 Internal Server Error` — жодні внутрішні деталі не будуть передані клієнту.

Ніколи не вмикайте це на продакшн-сервері. Використовуйте лише локально або у середовищі для тестування:

```php
// Safe for local development only — NEVER in production
Flight::set('flight.debug', true);
```

Коли `flight.debug` має значення `false` (за замовчуванням), ви все одно можете фіксувати помилки, увімкнувши `flight.log_errors`:

```php
// Log errors server-side without exposing them to the client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Рекомендована конфігурація для продакшену

```php
// index.php or app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Обробка помилок
Приховуйте чутливі деталі помилок у продакшені, щоб уникнути витоку інформації зловмисникам. У продакшені логувати помилки замість їх відображення з параметром `display_errors`, встановленим у `0`.

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

### Санітизація вводу
Ніколи не довіряйте введенню користувача. Санітизуйте його за допомогою [filter_var](https://www.php.net/manual/en/function.filter-var.php) перед обробкою, щоб запобігти проникненню шкідливих даних.

```php

// Lets assume a $_POST request with $_POST['input'] and $_POST['email']

// Sanitize a string input
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitize an email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Хешування паролів
Зберігайте паролі безпечно та перевіряйте їх безпечно за допомогою вбудованих функцій PHP, таких як [password_hash](https://www.php.net/manual/en/function.password-hash.php) та [password_verify](https://www.php.net/manual/en/function.password-verify.php). Паролі ніколи не повинні зберігатися у відкритому вигляді, а також не повинні шифруватися зворотними методами. Хешування гарантує, що навіть якщо ваша база даних буде скомпрометована, реальні паролі залишаться захищеними.

```php
$password = Flight::request()->data->password;
// Hash a password when storing (e.g., during registration)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verify a password (e.g., during login)
if (password_verify($password, $stored_hash)) {
    // Password matches
}
```

### Обмеження швидкості запитів
Захистіть від атак методом перебору або атак типу «відмова в обслуговуванні», обмежуючи швидкість запитів за допомогою кешу.

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

## Див. також
- [Sessions](/awesome-plugins/session) - Як безпечно керувати сесіями користувачів.
- [Templates](/learn/templates) - Використання шаблонів для автоматичного екранування виводу та запобігання XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Спрощена взаємодія з базою даних за допомогою підготовлених запитів.
- [Middleware](/learn/middleware) - Як використовувати middleware для спрощення процесу додавання заголовків безпеки.
- [Responses](/learn/responses) - Як налаштовувати HTTP-відповіді з безпечними заголовками.
- [Requests](/learn/requests) - Як обробляти та санітизувати введення користувача.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Функція PHP для санітизації введення.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Функція PHP для безпечного хешування паролів.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Функція PHP для перевірки хешованих паролів.

## Усунення несправностей
- Зверніться до розділу «Див. також» вище для отримання інформації щодо усунення несправностей, пов'язаної з компонентами Flight Framework.

## Журнал змін
- v3.18.1 - Додано розділ Flight Configuration Hardening, що охоплює `flight.allow_method_override`, `flight.debug` та перевірку JSONP callback.
- v3.1.0 - Додано розділи про CORS, обробку помилок, санітизацію введення, хешування паролів та обмеження швидкості запитів.
- v2.0 - Додано екранування для стандартних представлень для запобігання XSS.