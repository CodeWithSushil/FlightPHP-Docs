# Маршрутизація

## Огляд
Маршрутизація у Flight PHP зіставляє URL-шаблони з функціями зворотного виклику або методами класів, що забезпечує швидку та просту обробку запитів. Вона розроблена з мінімальними накладними витратами, зручна для початківців і розширювана без зовнішніх залежностей.

## Розуміння
Маршрутизація — це основний механізм, який пов’язує HTTP-запити з логікою вашого застосунку у Flight. Визначаючи маршрути, ви задаєте, як різні URL-адреси запускають певний код — через функції, методи класів або дії контролерів. Система маршрутизації Flight є гнучкою: підтримує базові шаблони, іменовані параметри, регулярні вирази та розширені можливості, як-от впровадження залежностей і ресурсна маршрутизація. Такий підхід тримає ваш код організованим і легким у підтримці, залишаючись швидким і простим для початківців та розширюваним для досвідчених користувачів.

> **Примітка:** Хочете краще зрозуміти маршрутизацію? Перегляньте сторінку [«чому фреймворк?»](/learn/why-frameworks) для детальнішого пояснення.

## Основне використання

### Визначення простого маршруту
Базова маршрутизація у Flight виконується шляхом зіставлення URL-шаблону з функцією зворотного виклику або масивом із класом і методом.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Маршрути зіставляються в порядку їх визначення. Перший маршрут, який відповідає запиту, буде викликано.

### Використання функцій як зворотних викликів
Зворотний виклик може бути будь-яким викличним об’єктом. Тож можна використовувати звичайну функцію:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Використання класів і методів як контролера
Можна також використовувати метод класу (статичний або звичайний):

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// або
Flight::route('/', [ GreetingController::class, 'hello' ]); // рекомендований спосіб
// або
Flight::route('/', [ 'GreetingController::hello' ]);
// або 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Або створивши об’єкт спочатку, а потім викликавши метод:

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

> **Примітка:** За замовчуванням, коли контролер викликається у межах фреймворку, клас `flight\Engine` завжди впроваджується, якщо ви не вкажете інше через [контейнер впровадження залежностей](/learn/dependency-injection-container)

### Маршрутизація за методами

За замовчуванням шаблони маршрутів зіставляються з усіма методами запитів. Ви можете реагувати
на певні методи, розмістивши ідентифікатор перед URL.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// Не можна використовувати Flight::get() для маршрутів, оскільки це метод
//    для отримання змінних, а не створення маршруту.
Flight::post('/', function() { /* код */ });
Flight::patch('/', function() { /* код */ });
Flight::put('/', function() { /* код */ });
Flight::delete('/', function() { /* код */ });
```

Також можна зіставити кілька методів з одним зворотним викликом, використовуючи розділювач `|`:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### Особлива обробка HEAD і OPTIONS запитів

Flight має вбудовану обробку для HTTP-запитів `HEAD` та `OPTIONS`:

#### HEAD-запити

- **HEAD-запити** обробляються так само, як і `GET`, але Flight автоматично видаляє тіло відповіді перед відправкою клієнту.
- Це означає, що ви можете визначити маршрут для `GET`, і HEAD-запити на ту саму URL-адресу повертатимуть лише заголовки (без вмісту), як того вимагають стандарти HTTP.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// HEAD-запит до /info поверне ті самі заголовки, але без тіла.
```

#### OPTIONS-запити

`OPTIONS`-запити автоматично обробляються Flight для будь-якого визначеного маршруту.
- Коли надходить OPTIONS-запит, Flight відповідає статусом `204 No Content` та заголовком `Allow`, у якому перелічено всі підтримувані HTTP-методи для цього маршруту.
- Вам не потрібно визначати окремий маршрут для OPTIONS.

```php
// Для маршруту, визначеного як:
Flight::route('GET|POST /users', function() { /* ... */ });

// OPTIONS-запит до /users відповість:
//
// Status: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Використання об’єкта Router

Крім того, ви можете отримати об’єкт Router, який має кілька допоміжних методів:

```php

$router = Flight::router();

// зіставляє всі методи, як і Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// GET-запит
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* код */});
$router->put('/users/update/@id', 	function() { /* код */});
$router->delete('/users/@id', 		function() { /* код */});
$router->patch('/users/@id', 		function() { /* код */});
```

### Регулярні вирази (Regex)
Ви можете використовувати регулярні вирази у своїх маршрутах:

```php
Flight::route('/user/[0-9]+', function () {
  // Цей маршрут відповідатиме /user/1234
});
```

Хоча цей метод доступний, рекомендується використовувати іменовані параметри або
іменовані параметри з регулярними виразами, оскільки вони більш читабельні та легші у підтримці.

### Іменовані параметри
Ви можете вказувати іменовані параметри у маршрутах, які будуть передані
у вашу функцію зворотного виклику. **Це більше для читабельності маршруту, ніж щось інше.
Будь ласка, перегляньте важливе застереження нижче.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Ви також можете додавати регулярні вирази до іменованих параметрів, використовуючи
розділювач `:`:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Цей маршрут відповідатиме /bob/123
  // Але не відповідатиме /bob/12345
});
```

> **Примітка:** Зіставлення груп регулярних виразів `()` із позиційними параметрами не підтримується. Напр.: `:'\(`

#### Важливе застереження

Хоча у наведеному вище прикладі здається, що `@name` безпосередньо пов’язаний зі змінною `$name`, це не так. Порядок параметрів у функції зворотного виклику визначає, що буде передано. Якщо поміняти порядок параметрів у функції зворотного виклику, змінні також поміняються. Ось приклад:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

І якщо ви перейдете за наступним URL: `/bob/123`, вивід буде `hello, 123 (bob)!`.
_Будьте уважні_, коли налаштовуєте маршрути та функції зворотного виклику!

### Необов’язкові параметри
Ви можете визначити іменовані параметри як необов’язкові для зіставлення, обгорнувши
сегменти у круглі дужки.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Цей маршрут відповідатиме таким URL:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Будь-які необов’язкові параметри, які не збіглися, будуть передані як `NULL`.

### Глобальні шаблони (Wildcard Routing)
Зіставлення виконується лише для окремих сегментів URL. Якщо потрібно зіставити кілька
сегментів, можна використовувати символ `*`.

```php
Flight::route('/blog/*', function () {
  // Цей маршрут відповідатиме /blog/2000/02/01
});
```

Щоб направити всі запити на один зворотний виклик, можна зробити так:

```php
Flight::route('*', function () {
  // Зробити щось
});
```

### Обробник 404 Not Found

За замовчуванням, якщо URL не знайдено, Flight надішле дуже просту відповідь `HTTP 404 Not Found`.
Якщо ви хочете налаштувати власну відповідь 404, ви можете [змапити](/learn/extending) власний метод `notFound`:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Ви також можете використати Flight::render() з власним шаблоном.
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

### Обробник Method Not Found

За замовчуванням, якщо URL знайдено, але метод не дозволений, Flight надішле дуже просту відповідь `HTTP 405 Method Not Allowed` (напр.: Method Not Allowed. Allowed Methods are: GET, POST). Вона також міститиме заголовок `Allow` із дозволеними методами для цього URL.

Якщо ви хочете налаштувати власну відповідь 405, ви можете [змапити](/learn/extending) власний метод `methodNotFound`:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Ви також можете використати Flight::render() з власним шаблоном.
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

## Розширене використання

### Впровадження залежностей у маршрутах
Якщо ви хочете використовувати впровадження залежностей через контейнер (PSR-11, PHP-DI, Dice тощо), то
це доступно лише для таких типів маршрутів: або ви самостійно створюєте об’єкт
і використовуєте контейнер для його створення, або використовуєте рядки для визначення класу та
методу, який потрібно викликати. Більше інформації ви можете знайти на сторінці [Впровадження залежностей](/learn/dependency-injection-container).

Ось швидкий приклад:

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
		// зробити щось із $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// Налаштуйте контейнер із необхідними параметрами
// Дивіться сторінку Впровадження залежностей для отримання додаткової інформації про PSR-11
$dice = new \Dice\Dice();

// Не забудьте переназначити змінну з '$dice = '!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Зареєструйте обробник контейнера
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Маршрути як зазвичай
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// або
Flight::route('/hello/@id', 'Greeting->hello');
// або
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Передача виконання наступному маршруту
<span class="badge bg-warning">Застаріло</span>
Ви можете передати виконання наступному відповідному маршруту, повернувши `true` зі своєї
функції зворотного виклику.

```php
Flight::route('/user/@name', function (string $name) {
  // Перевірити певну умову
  if ($name !== "Bob") {
    // Перейти до наступного маршруту
    return true;
  }
});

Flight::route('/user/*', function () {
  // Цей маршрут буде викликано
});
```

Тепер для складних випадків, подібних до цього, рекомендується використовувати [мідлвару](/learn/middleware).

### Псевдоніми маршрутів
Призначивши маршруту псевдонім, ви можете пізніше динамічно викликати цей псевдонім у своєму застосунку для генерації URL (наприклад, посилання у HTML-шаблоні або для створення URL-адреси перенаправлення).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// або 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// пізніше десь у коді
class UserController {
	public function update() {

		// код для збереження користувача...
		$id = $user['id']; // наприклад, 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // поверне '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

Це особливо корисно, якщо ваш URL змінюється. У наведеному вище прикладі, скажімо, користувачів перенесли на `/admin/users/@id`.
Завдяки псевдоніму маршруту вам більше не потрібно шукати всі старі URL у коді та змінювати їх, оскільки псевдонім тепер повертатиме `/admin/users/5`, як у прикладі вище.

Псевдоніми маршрутів також працюють у групах:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// або
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Перегляд інформації про маршрут
Якщо ви хочете переглянути інформацію про відповідний маршрут, є два способи:

1. Використати властивість `executedRoute` на об’єкті `Flight::router()`.
2. Попросити передати об’єкт маршруту у ваш зворотний виклик, передавши `true` третім параметром у методі маршруту. Об’єкт маршруту завжди буде останнім параметром, переданим у вашу функцію зворотного виклику.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Зробити щось із $route
  // Масив HTTP-методів, з якими зіставлено маршрут
  $route->methods;

  // Масив іменованих параметрів
  $route->params;

  // Відповідний регулярний вираз
  $route->regex;

  // Містить вміст будь-якого '*', використаного у шаблоні URL
  $route->splat;

  // Показує шлях URL... якщо вам справді це потрібно
  $route->pattern;

  // Показує, яка мідлвара призначена цьому маршруту
  $route->middleware;

  // Показує псевдонім, призначений цьому маршруту
  $route->alias;
});
```

> **Примітка:** Властивість `executedRoute` буде встановлена лише після виконання маршруту. Якщо спробувати отримати до неї доступ до виконання маршруту, вона буде `NULL`. Ви також можете використовувати executedRoute у [мідлварі](/learn/middleware)!

#### Передача `true` у визначення маршруту
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Масив HTTP-методів, з якими зіставлено маршрут
  $route->methods;

  // Масив іменованих параметрів
  $route->params;

  // Відповідний регулярний вираз
  $route->regex;

  // Містить вміст будь-якого '*', використаного у шаблоні URL
  $route->splat;

  // Показує шлях URL... якщо вам справді це потрібно
  $route->pattern;

  // Показує, яка мідлвара призначена цьому маршруту
  $route->middleware;

  // Показує псевдонім, призначений цьому маршруту
  $route->alias;
}, true);// <-- Цей параметр true робить це можливим
```

### Групування маршрутів і мідлвара
Іноді потрібно згрупувати пов’язані маршрути (наприклад, `/api/v1`).
Це можна зробити за допомогою методу `group`:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Відповідає /api/v1/users
  });

  Flight::route('/posts', function () {
	// Відповідає /api/v1/posts
  });
});
```

Можна навіть вкладати групи в групи:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() отримує змінні, він не встановлює маршрут! Дивіться контекст об’єкта нижче
	Flight::route('GET /users', function () {
	  // Відповідає GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Відповідає POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Відповідає PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() отримує змінні, він не встановлює маршрут! Дивіться контекст об’єкта нижче
	Flight::route('GET /users', function () {
	  // Відповідає GET /api/v2/users
	});
  });
});
```

#### Групування з контекстом об’єкта

Ви також можете використовувати групування маршрутів з об’єктом `Engine` таким чином:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // використовуйте змінну $router
  $router->get('/users', function () {
	// Відповідає GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Відповідає POST /api/v1/posts
  });
});
```

> **Примітка:** Це рекомендований спосіб визначення маршрутів і груп з об’єктом `$router`.

#### Групування з мідлварою

Ви також можете призначити мідлвару групі маршрутів:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Відповідає /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // або [ new MyAuthMiddleware() ], якщо ви хочете використати екземпляр
```

Більше деталей на сторінці [групова мідлвара](/learn/middleware#grouping-middleware).

### Ресурсна маршрутизація
Ви можете створити набір маршрутів для ресурсу за допомогою методу `resource`. Це створить
набір маршрутів для ресурсу, що відповідає RESTful-конвенціям.

Щоб створити ресурс, зробіть наступне:

```php
Flight::resource('/users', UsersController::class);
```

У фоновому режимі буде створено такі маршрути:

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

А ваш контролер використовуватиме наступні методи:

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

> **Примітка:** Ви можете переглянути новостворені маршрути за допомогою `runway`, виконавши `php runway routes`.

#### Налаштування ресурсних маршрутів

Є кілька опцій для конфігурування ресурсних маршрутів.

##### База псевдоніма

Ви можете налаштувати `aliasBase`. За замовчуванням псевдонім — це остання частина вказаного URL.
Наприклад, `/users/` дасть `aliasBase` зі значенням `users`. Коли ці маршрути створюються,
псевдоніми будуть `users.index`, `users.create` тощо. Якщо ви хочете змінити псевдонім, встановіть `aliasBase`
на потрібне значення.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### only та except

Ви також можете вказати, які маршрути потрібно створювати, за допомогою опцій `only` та `except`.

```php
// Білий список лише цих методів, решта — у чорному списку
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Чорний список лише цих методів, решта — у білому списку
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

По суті, це опції білого та чорного списків, тож ви можете вказати, які маршрути створювати.

##### Мідлвара

Ви також можете вказати мідлвару, яка буде виконуватися для кожного маршруту, створеного методом `resource`.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Потокові відповіді

Тепер ви можете передавати відповіді клієнту потоково за допомогою `stream()` або `streamWithHeaders()`. 
Це корисно для надсилання великих файлів, довготривалих процесів або генерації великих відповідей. 
Потокова передача маршруту обробляється дещо інакше, ніж звичайного маршруту.

> **Примітка:** Потокові відповіді доступні лише тоді, коли [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) встановлено у `false`.

#### Потік із ручними заголовками

Ви можете передавати відповідь клієнту потоково за допомогою методу `stream()` на маршруті. Якщо ви 
це робите, ви повинні вручну встановити всі заголовки перед тим, як виводити щось клієнту.
Це робиться за допомогою PHP-функції `header()` або методу `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// очевидно, ви маєте очистити шлях тощо.
	$fileNameSafe = basename($filename);

	// Якщо вам потрібно встановити додаткові заголовки після виконання маршруту,
	// ви повинні визначити їх до того, як щось буде виведено.
	// Вони мають бути необробленим викликом функції header() або
	// викликом Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// або
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// вручну встановіть довжину вмісту, якщо бажаєте
	header('Content-Length: '.filesize($filePath));
	// або
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Потоково передайте файл клієнту, читаючи його
	readfile($filePath);

// Це магічний рядок
})->stream();
```

#### Потік із заголовками

Ви також можете використовувати метод `streamWithHeaders()`, щоб встановити заголовки перед початком потокової передачі.

```php
Flight::route('/stream-users', function() {

	// тут ви можете додати будь-які додаткові заголовки
	// ви просто повинні використовувати header() або Flight::response()->setRealHeader()

	// однак ви отримуєте свої дані, наприклад...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Це необхідно для надсилання даних клієнту
		ob_flush();
	}
	echo '}';

// Ось так ви встановите заголовки перед початком потокової передачі.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// необов’язковий код статусу, за замовчуванням 200
	'status' => 200
]);
```

## Дивіться також
- [Мідлвара](/learn/middleware) — використання мідлвари з маршрутами для автентифікації, журналювання тощо.
- [Впровадження залежностей](/learn/dependency-injection-container) — спрощення створення об’єктів та керування ними у маршрутах.
- [Чому фреймворк?](/learn/why-frameworks) — розуміння переваг використання фреймворку, такого як Flight.
- [Розширення](/learn/extending) — як розширити Flight власними функціями, зокрема методом `notFound`.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) — PHP-функція для зіставлення регулярних виразів.

## Усунення неполадок
- Параметри маршруту зіставляються за порядком, а не за ім’ям. Переконайтеся, що порядок параметрів зворотного виклику відповідає визначенню маршруту.
- Використання `Flight::get()` не визначає маршрут; для маршрутизації використовуйте `Flight::route('GET /...')` або контекст об’єкта Router у групах (напр., `$router->get(...)`).
- Властивість executedRoute встановлюється лише після виконання маршруту; до цього вона дорівнює NULL.
- Потокова передача потребує вимкнення застарілої функціональності вихідного буферизації Flight (`flight.v2.output_buffering = false`).
- Для впровадження залежностей лише певні визначення маршрутів підтримують створення через контейнер.

### 404 Not Found або неочікувана поведінка маршруту

Якщо ви бачите помилку 404 Not Found (але ви присягаєтеся, що маршрут справді існує, і це не помилка), насправді це може бути проблемою 
з тим, що ви повертаєте значення у кінцевій точці маршруту, а не просто виводите його. Причина цього навмисна, але може стати несподіванкою для деяких розробників.

```php
Flight::route('/hello', function(){
	// Це може спричинити помилку 404 Not Found
	return 'Hello World';
});

// Ймовірно, ви хочете так
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

Причина в тому, що в маршрутизатор вбудовано спеціальний механізм, який трактує повернений результат як сигнал «перейти до наступного маршруту». 
Цю поведінку задокументовано в розділі [Маршрутизація](/learn/routing#passing).

## Журнал змін
- v3: Додано ресурсну маршрутизацію, псевдоніми маршрутів, підтримку потокової передачі, групи маршрутів та підтримку мідлвари.
- v1: Доступна переважна більшість базових функцій.