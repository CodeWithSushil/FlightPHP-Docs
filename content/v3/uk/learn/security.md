# Безпека

## Огляд

Безпека є дуже важливою для веб-додатків. Ви маєте переконатися, що ваш додаток захищений, а дані ваших користувачів у безпеці. Flight надає низку функцій, які допоможуть вам захистити ваші веб-додатки.

Офіційний [скелет](https://github.com/flightphp/skeleton) також містить спеціальний **`SECURITY.md`** і проміжне програмне забезпечення для заголовків безпеки, щоб [AI-інструменти для кодування](/learn/ai) (і люди) мали одне продумане місце для секретів, заголовків і правил XSS/SQL — окремо від загального стилю кодування в `AGENTS.md`.

## Розуміння

Існує кілька поширених загроз безпеці, про які слід пам’ятати під час створення веб-додатків. Деякі з найпоширеніших загроз включають:
- Міжсайтова підробка запитів (CSRF)
- Міжсайтовий скриптинг (XSS)
- SQL-ін'єкція
- Обмін ресурсами між джерелами (CORS)

[Шаблони](/learn/templates) допомагають із XSS, екрануючи вивід за замовчуванням (Twig і Latte роблять це; використовуйте цю перевагу). [Сесії](/awesome-plugins/session) можуть допомогти з CSRF, зберігаючи CSRF-токен у сесії користувача, як описано нижче. Використання підготовлених запитів із PDO або помічників на основі [SimplePdo](/learn/simple-pdo) допомагає запобігти SQL-ін'єкціям. CORS можна обробити за допомогою простого хука перед викликом `Flight::start()`.

Усі ці методи працюють разом, щоб захистити ваші веб-додатки. Завжди слід пам’ятати про вивчення та розуміння найкращих практик безпеки. Не просіть AI-асистента «вимкнути CSP» або послабити заголовки лише для того, щоб сторінка завантажилася, не розуміючи компромісів.

## Базове використання

### Заголовки

HTTP-заголовки — це один із найпростіших способів захистити ваші веб-додатки. Ви можете використовувати заголовки для запобігання клікджекінгу, XSS та інших атак. Існує кілька способів додати ці заголовки до вашого додатка.

Два чудові веб-сайти для перевірки безпеки ваших заголовків: [securityheaders.com](https://securityheaders.com/) та [observatory.mozilla.org](https://observatory.mozilla.org/). Після налаштування наведеного нижче коду ви легко зможете перевірити роботу заголовків за допомогою цих двох сайтів.

Скелет включає `App\Middleware\SecurityHeadersMiddleware` (CSP із nonce для кожного запиту, параметри фреймів, HSTS та інше). Віддавайте перевагу свідомому розширенню цього класу, а не вимкненню заголовків.

#### Додати вручну

Ви можете додати ці заголовки вручну за допомогою методу `header` на об'єкті `Flight\Response`.
```php
// Встановлюємо заголовок X-Frame-Options для запобігання клікджекінгу
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Встановлюємо заголовок Content-Security-Policy для запобігання XSS
// Зверніть увагу: цей заголовок може бути дуже складним, тому вам варто
//  переглянути приклади в інтернеті для вашого додатка
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Встановлюємо заголовок X-XSS-Protection для запобігання XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Встановлюємо заголовок X-Content-Type-Options для запобігання MIME-сніфінгу
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Встановлюємо заголовок Referrer-Policy для контролю обсягу інформації про реферера
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Встановлюємо заголовок Strict-Transport-Security для примусового HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Встановлюємо заголовок Permissions-Policy для контролю функцій і API
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Їх можна додати на початку файлів `routes.php` або `index.php`.

#### Додати як фільтр

Ви також можете додати їх у фільтр/хук, як показано нижче:

```php
// Додаємо заголовки у фільтр
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

#### Додати як проміжне програмне забезпечення

Ви також можете додати їх як клас проміжного програмного забезпечення, який забезпечує найбільшу гнучкість щодо того, до яких маршрутів це застосовувати. Загалом ці заголовки слід застосовувати до всіх HTML та API відповідей.

Шлях і простір імен у стилі скелета (**регістр папки відповідає `App\Middleware`**):

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
		// Віддавайте перевагу nonce CSP з bootstrap, якщо у вас є інлайн-скрипти (скелет встановлює csp_nonce)
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

// app/config/routes.php — порожній рядок групи = глобальне проміжне ПЗ для всіх маршрутів
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// інші маршрути
}, [SecurityHeadersMiddleware::class]);
```

Старі проєкти можуть досі використовувати `app/middlewares` і `app\middlewares`; це працює, якщо папки збігаються. Нові додатки на скелеті використовують **`app/Middleware/`** та **`App\Middleware`**. Дивіться [Автозавантаження](/learn/autoloading).

### Міжсайтова підробка запитів (CSRF)

Міжсайтова підробка запитів (CSRF) — це тип атаки, коли зловмисний веб-сайт може змусити браузер користувача надіслати запит на ваш веб-сайт. Це можна використати для виконання дій на вашому веб-сайті без відома користувача. Flight не має вбудованого механізму захисту від CSRF, але ви можете легко реалізувати власний за допомогою проміжного програмного забезпечення.

#### Налаштування

Спочатку потрібно згенерувати CSRF-токен і зберегти його в сесії користувача. Потім ви можете використовувати цей токен у ваших формах і перевіряти його під час надсилання форми. Ми використаємо плагін [flightphp/session](/awesome-plugins/session) для керування сесіями.

```php
// Генеруємо CSRF-токен і зберігаємо його в сесії користувача
// (припускаючи, що ви створили об'єкт сесії та приєднали його до Flight)
// дивіться документацію сесій для додаткової інформації
Flight::register('session', flight\Session::class);

// Ви можете згенерувати лише один токен на сесію (тож він працює 
// у кількох вкладках і запитах для одного користувача)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Використання стандартного шаблону PHP Flight

```html
<!-- Використовуємо CSRF-токен у вашій формі -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- інші поля форми -->
</form>
```

##### Використання Twig (типово для скелета)

Зареєструйте функцію Twig або передавайте токен у кожне подання форми. Мінімальний приклад із глобальною змінною та полем форми:

```php
// Під час налаштування Twig (наприклад, services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# інші поля #}
</form>
```

##### Використання Latte

Ви також можете встановити власну функцію для виведення CSRF-токена у ваших шаблонах Latte.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// інші конфігурації...

	// Встановлюємо власну функцію для виведення CSRF-токена
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

І тепер у ваших шаблонах Latte ви можете використовувати функцію `csrf()` для виведення CSRF-токена.

```html
<form method="post">
	{csrf()}
	<!-- інші поля форми -->
</form>
```

#### Перевірка CSRF-токена

Ви можете перевірити CSRF-токен кількома методами.

##### Проміжне програмне забезпечення

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
	// інші маршрути
}, [CsrfMiddleware::class]);
```

##### Фільтри подій

```php
// Це проміжне ПЗ перевіряє, чи є запит POST, і якщо так, перевіряє дійсність CSRF-токена
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// отримуємо csrf-токен із значень форми
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// або для JSON-відповіді
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Міжсайтовий скриптинг (XSS)

Міжсайтовий скриптинг (XSS) — це тип атаки, коли зловмисне введення у формі може впровадити код у ваш веб-сайт. Більшість таких можливостей походять від значень форм, які заповнюють ваші кінцеві користувачі. Вам **ніколи** не слід довіряти виводу від користувачів! Завжди вважайте, що всі вони — найкращі хакери у світі. Вони можуть впровадити зловмисний JavaScript або HTML на вашу сторінку. Цей код може бути використаний для викрадення інформації від ваших користувачів або виконання дій на вашому веб-сайті. Використовуючи клас представлення Flight або шаблонізатор, як-от [Twig](/awesome-plugins/twig) чи [Latte](/awesome-plugins/latte), ви можете легко екранувати вивід для запобігання XSS-атакам.

```php
// Припустимо, користувач розумний і намагається використати це як ім'я
$name = '<script>alert("XSS")</script>';

// Це екранує вивід
Flight::view()->set('name', $name);
// Це виведе: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (типово для скелета) і Latte автоматично екранують за замовчуванням — віддавайте їм перевагу над сирим echo у PHP
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → екрановано
// Уникайте |raw / неекранованого виводу, якщо контент повністю надійний
```

### SQL-ін'єкція

SQL-ін'єкція — це тип атаки, коли зловмисний користувач може впровадити SQL-код у вашу базу даних. Це можна використати для викрадення інформації з вашої бази даних або виконання дій із нею. Знову ж таки, вам **ніколи** не слід довіряти введенню від користувачів! Завжди вважайте, що вони прагнуть крові. Використовуйте підготовлені запити — помічники [SimplePdo](/learn/simple-pdo) роблять цей шлях типовим.

```php
// Припускаємо, що Flight::db() зареєстровано як SimplePdo (або SimplePdo впроваджено в контролер)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (рекомендовано) — однострокові запити з зв'язаними параметрами
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Та сама ідея з плейсхолдерами ?
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

У контролерах у стилі скелета віддавайте перевагу впровадженню `SimplePdo` через конструктор, а не `Flight::db()`, щоб тести та AI-згенерований код залишалися узгодженими ([DIC](/learn/dependency-injection-container)).

#### Небезпечний приклад

Нижче наведено, чому ми використовуємо підготовлені SQL-запити для захисту від таких простих прикладів:

```php
// кінцевий користувач заповнює веб-форму.
// для значення форми хакер вводить щось на кшталт цього:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Після побудови запиту він виглядає так
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Це виглядає дивно, але це дійсний запит, який спрацює. Насправді
// це дуже поширена SQL-ін'єкція, яка поверне всіх користувачів.

var_dump($users); // це виведе всіх користувачів у базі даних, а не лише одне ім'я користувача
```

### Секрети та конфігурація

- Зберігайте секрети в **`.env`** (або в реальному середовищі), а не в закомічених зразках `config.php`.
- Правило скелета: буквальні значення за замовчуванням у `config.php`; об'єднуйте з `.env` під час bootstrap; **не** читайте `$_ENV` у контролерах — натомість впроваджуйте конфігурацію. Дивіться [Конфігурація](/learn/configuration).
- Ніколи не комітьте API-ключі, паролі бази даних або ключі шифрування сесій. Вказуйте AI-інструментам на **`SECURITY.md`**, щоб вони не вигадували небезпечні скорочення.

### Перевірка зворотного виклику JSONP

Якщо ви використовуєте метод `Flight::jsonp()`, майте на увазі, що Flight перевіряє назву параметра зворотного виклику JSONP за суворим дозволеним регулярним виразом (`/^[A-Za-z_$][\w$.]{0,127}$/`). Будь-яка назва зворотного виклику, яка не відповідає цьому шаблону, призведе до винятку Flight, що запобігає впровадженню довільного JavaScript через зловмисне значення зворотного виклику.

Ця перевірка вбудована і не потребує додаткової конфігурації, але про неї варто знати під час налагодження неочікуваних помилок від JSONP-кінцевих точок.

### CORS (Обмін ресурсами між джерелами)

CORS — це механізм, який дозволяє багатьом ресурсам (наприклад, шрифтам, JavaScript тощо) на веб-сторінці запитуватися з іншого домену, ніж домен, з якого походить ресурс. Flight не має вбудованої функціональності, але це легко обробляється за допомогою хука, який виконується перед викликом методу `Flight::start()`.

```php
// app/Utils/CorsUtil.php  (скелет: тека Utils у PascalCase → App\Utils)

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
		// налаштуйте дозволені хости тут.
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

// bootstrap / routes — виконується перед start
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Посилення конфігурації Flight

Flight надає кілька налаштувань движка, які мають прямий вплив на безпеку. Правильне налаштування цих параметрів — один із найпростіших способів посилити захист вашого додатка.

#### `flight.allow_method_override`

За замовчуванням Flight дозволяє клієнтам перевизначати HTTP-метод запиту за допомогою заголовка `X-HTTP-Method-Override` або поля `_method` у POST-тілі. Хоча це зручно для HTML-форм, які можуть надсилати лише `GET`/`POST`, це може бути небезпечно, якщо ви цього не очікуєте — зловмисник може підробити `DELETE` або `PUT` запити через звичайну форму.

Якщо ваш додаток не покладається на цю поведінку (наприклад, ви створюєте API для сучасних клієнтів або JavaScript-фронтендів, які можуть надсилати будь-які HTTP-дієслова), вам слід вимкнути це:

```php
// У вашому index.php або bootstrap-файлі, перед Flight::start()
Flight::set('flight.allow_method_override', false);
```

Значення за замовчуванням — `true` для зворотної сумісності, але **встановлення `false` настійно рекомендується** для будь-якого додатка, який явно не потребує функції перевизначення.

#### `flight.debug`

Flight має налаштування `flight.debug`, яке контролює, чи відображається детальна інформація про помилку (повідомлення винятку, код і повний стек викликів) у браузері, коли виникає необроблений виняток. За замовчуванням значення `false`, що означає показ лише загального повідомлення `500 Internal Server Error` — жодні внутрішні деталі не витікають клієнту.

Ніколи не вмикайте це на продакшн-сервері. Використовуйте це лише локально або в стейджинг-середовищі:

```php
// Безпечно лише для локальної розробки — НІКОЛИ не для продакшну
Flight::set('flight.debug', true);
```

Коли `flight.debug` має значення `false` (за замовчуванням), ви все одно можете фіксувати помилки, увімкнувши `flight.log_errors`:

```php
// Логуємо помилки на сервері, не показуючи їх клієнту
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Рекомендована продакшн-конфігурація

```php
// index.php або застосовується з конфігурації додатка / bootstrap
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Обробка помилок
Приховуйте деталі чутливих помилок у продакшні, щоб не розкривати інформацію зловмисникам. У продакшні журналюйте помилки замість їх відображення, встановивши `display_errors` у `0`.

```php
// У вашому bootstrap.php або index.php

// додайте це до вашого app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Вимкнути відображення помилок
    ini_set('log_errors', 1);     // Журналювати помилки замість цього
    ini_set('error_log', '/path/to/error.log');
}

// У ваших маршрутах або контролерах
// Використовуйте Flight::halt() для контрольованих відповідей з помилками
Flight::halt(403, 'Access denied');
```

### Очищення вхідних даних
Ніколи не довіряйте введеним користувачем даним. Очищуйте їх за допомогою [filter_var](https://www.php.net/manual/en/function.filter-var.php) перед обробкою, щоб запобігти проникненню шкідливих даних. Віддавайте перевагу читанню введення через `$app->request()` (або `Flight::request()`), а не через сирі `$_GET` / `$_POST` у коді додатка.

```php

// Припустімо, є $_POST запит із $_POST['input'] та $_POST['email']

// Очищуємо введений рядок
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Очищуємо email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Хешування паролів
Зберігайте паролі безпечно та перевіряйте їх безпечно за допомогою вбудованих функцій PHP, як-от [password_hash](https://www.php.net/manual/en/function.password-hash.php) і [password_verify](https://www.php.net/manual/en/function.password-verify.php). Паролі ніколи не повинні зберігатися у відкритому вигляді, а також не повинні шифруватися оборотними методами. Хешування гарантує, що навіть якщо ваша база даних буде скомпрометована, фактичні паролі залишаться захищеними.

```php
$password = Flight::request()->data->password;
// Хешуємо пароль під час збереження (наприклад, під час реєстрації)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Перевіряємо пароль (наприклад, під час входу)
if (password_verify($password, $stored_hash)) {
    // Пароль збігається
}
```

### Обмеження швидкості (Rate Limiting)
Захищайтеся від атак перебором або атак типу «відмова в обслуговуванні», обмежуючи швидкість запитів за допомогою кешу.

```php
// Припускаємо, що у вас встановлено та зареєстровано flightphp/cache
// Використання flightphp/cache у фільтрі
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Скидання через 60 секунд
});
```

## Дивіться також
- [Сесії](/awesome-plugins/session) — Як безпечно керувати сесіями користувачів.
- [Шаблони](/learn/templates) — Автоматичне екранування Twig/Latte та XSS.
- [SimplePdo](/learn/simple-pdo) — Помічники для бази даних із підготовленими запитами.
- [PdoWrapper](/learn/pdo-wrapper) — Застарілий; для нового коду використовуйте SimplePdo.
- [Проміжне програмне забезпечення](/learn/middleware) — Як використовувати проміжне ПЗ для спрощення додавання заголовків безпеки.
- [Конфігурація](/learn/configuration) — `.env` проти буквальної конфігурації, прапорці продакшну.
- [AI та досвід розробника](/learn/ai) — Тримайте політику безпеки в `SECURITY.md` для агентів.
- [Відповіді](/learn/responses) — Як налаштовувати HTTP-відповіді з безпечними заголовками.
- [Запити](/learn/requests) — Як обробляти та очищувати введення користувача.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) — PHP-функція для очищення введення.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) — PHP-функція для безпечного хешування паролів.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) — PHP-функція для перевірки хешованих паролів.

## Усунення неполадок
- Зверніться до розділу «Дивіться також» вище для інформації про усунення неполадок, пов’язаних із компонентами Flight Framework.
- Якщо CSP блокує ваші скрипти, додайте nonce (шаблон скелета) або внесіть конкретні джерела до білого списку — не встановлюйте `script-src *` без плану.

## Журнал змін
- Документація — Скелет `App\Middleware`, нотатки Twig CSRF/XSS, SimplePdo, секрети/`.env` та `SECURITY.md` для AI-дружніх проєктів.
- v3.18.1 — Додано розділ «Посилення конфігурації Flight», що охоплює `flight.allow_method_override`, `flight.debug` та перевірку зворотного виклику JSONP.
- v3.1.0 — Додано розділи про CORS, обробку помилок, очищення введення, хешування паролів та обмеження швидкості.
- v2.0 — Додано екранування для стандартних представлень для запобігання XSS.