# Маршрутизация

## Обзор

Маршрутизация во Flight PHP сопоставляет URL-шаблоны с функциями обратного вызова или методами классов, обеспечивая быструю и простую обработку запросов. Она спроектирована с минимальными накладными расходами, удобна для начинающих и расширяема без внешних зависимостей.

## Понимание

Маршрутизация — это основной механизм, который связывает HTTP-запросы с логикой вашего приложения во Flight. Определяя маршруты, вы указываете, как разные URL-адреса запускают конкретный код — через функции, методы классов или действия контроллеров. Система маршрутизации Flight гибкая: она поддерживает базовые шаблоны, именованные параметры, регулярные выражения и расширенные возможности, такие как внедрение зависимостей и ресурсная маршрутизация. Такой подход поддерживает организованность и простоту сопровождения кода, оставаясь быстрым и простым для новичков и расширяемым для продвинутых пользователей.

> **Примечание:** Хотите узнать больше о маршрутизации? Посмотрите страницу ["зачем нужен фреймворк?"](/learn/why-frameworks) для более подробного объяснения.

## Базовое использование

### Определение простого маршрута
Базовая маршрутизация во Flight выполняется путём сопоставления URL-шаблона с функцией обратного вызова или массивом из класса и метода.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Маршруты сопоставляются в порядке их определения. Будет вызван первый маршрут, соответствующий запросу.

### Использование функций в качестве обратных вызовов
Обратный вызов может быть любым вызываемым объектом. Таким образом, вы можете использовать обычную функцию:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Использование классов и методов в качестве контроллера
Вы также можете использовать метод класса (статический или обычный):

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// или
Flight::route('/', [ GreetingController::class, 'hello' ]); // предпочтительный способ
// или
Flight::route('/', [ 'GreetingController::hello' ]);
// или 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Или создав объект сначала, а затем вызвав метод:

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

> **Примечание:** По умолчанию, когда контроллер вызывается внутри фреймворка, класс `flight\Engine` всегда внедряется, если вы не укажете иное через [контейнер внедрения зависимостей](/learn/dependency-injection-container).

### Маршрутизация по конкретным методам

По умолчанию шаблоны маршрутов сопоставляются со всеми методами запросов. Вы можете отвечать на конкретные методы, размещая идентификатор перед URL.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// Вы не можете использовать Flight::get() для маршрутов, так как это метод
//    для получения переменных, а не для создания маршрута.
Flight::post('/', function() { /* код */ });
Flight::patch('/', function() { /* код */ });
Flight::put('/', function() { /* код */ });
Flight::delete('/', function() { /* код */ });
```

Вы также можете сопоставить несколько методов с одним обратным вызовом, используя разделитель `|`:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### Особая обработка HEAD и OPTIONS запросов

Flight предоставляет встроенную обработку для HTTP-запросов `HEAD` и `OPTIONS`:

#### HEAD-запросы

- **HEAD-запросы** обрабатываются так же, как `GET`-запросы, но Flight автоматически удаляет тело ответа перед отправкой клиенту.
- Это означает, что вы можете определить маршрут для `GET`, и HEAD-запросы к тому же URL будут возвращать только заголовки (без содержимого), как того требуют стандарты HTTP.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// HEAD-запрос к /info вернёт те же заголовки, но без тела.
```

#### OPTIONS-запросы

`OPTIONS`-запросы автоматически обрабатываются Flight для любого определённого маршрута.
- Когда получен OPTIONS-запрос, Flight отвечает со статусом `204 No Content` и заголовком `Allow`, перечисляющим все поддерживаемые HTTP-методы для этого маршрута.
- Вам не нужно определять отдельный маршрут для OPTIONS.

```php
// Для маршрута, определённого как:
Flight::route('GET|POST /users', function() { /* ... */ });

// OPTIONS-запрос к /users ответит:
//
// Статус: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Использование объекта Router

Кроме того, вы можете получить объект Router, который содержит несколько вспомогательных методов для использования:

```php

$router = Flight::router();

// сопоставляет все методы, как Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// GET-запрос
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* код */});
$router->put('/users/update/@id', 	function() { /* код */});
$router->delete('/users/@id', 		function() { /* код */});
$router->patch('/users/@id', 		function() { /* код */});
```

### Регулярные выражения (Regex)
Вы можете использовать регулярные выражения в ваших маршрутах:

```php
Flight::route('/user/[0-9]+', function () {
  // Это будет соответствовать /user/1234
});
```

Хотя этот метод доступен, рекомендуется использовать именованные параметры или именованные параметры с регулярными выражениями, так как они более читаемы и проще в сопровождении.

### Именованные параметры
Вы можете указать именованные параметры в ваших маршрутах, которые будут переданы в вашу функцию обратного вызова. **Это нужно скорее для читаемости маршрута, чем для чего-либо ещё. Пожалуйста, обратите внимание на важное предостережение ниже.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Вы также можете включить регулярные выражения в ваши именованные параметры, используя разделитель `:`:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Это будет соответствовать /bob/123
  // Но не будет соответствовать /bob/12345
});
```

> **Примечание:** Сопоставление групп регулярных выражений `()` с позиционными параметрами не поддерживается. Например: `:'\(`

#### Важное предостережение

Хотя в приведённом выше примере кажется, что `@name` напрямую связан с переменной `$name`, это не так. Порядок параметров в функции обратного вызова определяет, что именно будет в неё передано. Если вы поменяете порядок параметров в функции обратного вызова, переменные также поменяются. Вот пример:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

И если вы перейдёте по следующему URL: `/bob/123`, вывод будет `hello, 123 (bob)!`. 
_Пожалуйста, будьте внимательны_ при настройке маршрутов и функций обратного вызова!

### Необязательные параметры
Вы можете указать именованные параметры как необязательные для сопоставления, обернув сегменты в круглые скобки.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Это будет соответствовать следующим URL:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Любые необязательные параметры, которые не были сопоставлены, будут переданы как `NULL`.

### Подстановочная маршрутизация (Wildcard)
Сопоставление выполняется только по отдельным сегментам URL. Если вы хотите сопоставить несколько сегментов, вы можете использовать подстановочный знак `*`.

```php
Flight::route('/blog/*', function () {
  // Это будет соответствовать /blog/2000/02/01
});
```

Чтобы направить все запросы на один обратный вызов, вы можете сделать следующее:

```php
Flight::route('*', function () {
  // Сделать что-то
});
```

### Обработчик 404 Not Found

По умолчанию, если URL не найден, Flight отправляет очень простой ответ `HTTP 404 Not Found`.
Если вы хотите настроить ответ 404, вы можете [сопоставить](/learn/extending) собственный метод `notFound`:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Вы также можете использовать Flight::render() с пользовательским шаблоном.
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

### Обработчик 405 Method Not Allowed

По умолчанию, если URL найден, но метод не разрешён, Flight отправляет очень простой ответ `HTTP 405 Method Not Allowed` (например: Method Not Allowed. Allowed Methods are: GET, POST). Он также включает заголовок `Allow` с разрешёнными методами для этого URL.

Если вы хотите более настраиваемый ответ 405, вы можете [сопоставить](/learn/extending) собственный метод `methodNotFound`:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Вы также можете использовать Flight::render() с пользовательским шаблоном.
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

## Продвинутое использование

### Внедрение зависимостей в маршрутах
Если вы хотите использовать внедрение зависимостей через контейнер (PSR-11, PHP-DI, Dice и т. д.), это доступно только для тех типов маршрутов, где вы либо сами создаёте объект, а контейнер используете для создания вашего объекта, либо используете строки для определения класса и вызываемого метода. Вы можете перейти на страницу [Внедрение зависимостей](/learn/dependency-injection-container) для получения дополнительной информации.

Вот краткий пример:

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
		// сделать что-то с $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// Настройте контейнер с необходимыми параметрами
// Смотрите страницу о внедрении зависимостей для получения дополнительной информации о PSR-11
$dice = new \Dice\Dice();

// Не забудьте переназначить переменную через '$dice = '!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Зарегистрируйте обработчик контейнера
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Маршруты как обычно
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// или
Flight::route('/hello/@id', 'Greeting->hello');
// или
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Передача выполнения следующему маршруту
<span class="badge bg-warning">Устарело</span>
Вы можете передать выполнение следующему подходящему маршруту, вернув `true` из вашей функции обратного вызова.

```php
Flight::route('/user/@name', function (string $name) {
  // Проверка некоторого условия
  if ($name !== "Bob") {
    // Перейти к следующему маршруту
    return true;
  }
});

Flight::route('/user/*', function () {
  // Этот маршрут будет вызван
});
```

Теперь рекомендуется использовать [промежуточное ПО (middleware)](/learn/middleware) для обработки сложных случаев, подобных этому.

### Псевдонимы маршрутов
Назначив маршруту псевдоним, вы можете позже динамически вызывать этот псевдоним в вашем приложении для генерации URL в коде (например, ссылка в HTML-шаблоне или создание URL для перенаправления).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// или 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// позже в коде где-нибудь
class UserController {
	public function update() {

		// код для сохранения пользователя...
		$id = $user['id']; // например, 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // вернёт '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

Это особенно полезно, если ваш URL изменился. В приведённом выше примере предположим, что пользователи были перемещены на `/admin/users/@id`.
Благодаря псевдониму маршрута вам больше не нужно искать все старые URL в вашем коде и изменять их, потому что псевдоним теперь вернёт `/admin/users/5`, как в примере выше.

Псевдонимы маршрутов также работают в группах:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// или
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Просмотр информации о маршруте
Если вы хотите просмотреть информацию о сопоставленном маршруте, есть два способа:

1. Используйте свойство `executedRoute` на объекте `Flight::router()`.
2. Вы можете запросить передачу объекта маршрута в ваш обратный вызов, передав `true` в качестве третьего параметра в методе маршрута. Объект маршрута всегда будет последним параметром, переданным в вашу функцию обратного вызова.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Сделать что-то с $route
  // Массив сопоставленных HTTP-методов
  $route->methods;

  // Массив именованных параметров
  $route->params;

  // Соответствующее регулярное выражение
  $route->regex;

  // Содержит содержимое любого '*', использованного в URL-шаблоне
  $route->splat;

  // Показывает путь URL... если вам действительно это нужно
  $route->pattern;

  // Показывает, какое промежуточное ПО назначено этому маршруту
  $route->middleware;

  // Показывает псевдоним, назначенный этому маршруту
  $route->alias;
});
```

> **Примечание:** Свойство `executedRoute` будет установлено только после выполнения маршрута. Если вы попытаетесь получить к нему доступ до выполнения маршрута, оно будет `NULL`. Вы также можете использовать `executedRoute` в [промежуточном ПО](/learn/middleware)!

#### Передача `true` в определение маршрута
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Массив сопоставленных HTTP-методов
  $route->methods;

  // Массив именованных параметров
  $route->params;

  // Соответствующее регулярное выражение
  $route->regex;

  // Содержит содержимое любого '*', использованного в URL-шаблоне
  $route->splat;

  // Показывает путь URL... если вам действительно это нужно
  $route->pattern;

  // Показывает, какое промежуточное ПО назначено этому маршруту
  $route->middleware;

  // Показывает псевдоним, назначенный этому маршруту
  $route->alias;
}, true); // <-- Этот параметр true обеспечивает это
```

### Группировка маршрутов и промежуточное ПО
Могут быть случаи, когда вы хотите сгруппировать связанные маршруты (например, `/api/v1`).
Это можно сделать с помощью метода `group`:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Соответствует /api/v1/users
  });

  Flight::route('/posts', function () {
	// Соответствует /api/v1/posts
  });
});
```

Вы даже можете вкладывать группы в группы:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() получает переменные, а не устанавливает маршрут! Смотрите объектный контекст ниже
	Flight::route('GET /users', function () {
	  // Соответствует GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Соответствует POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Соответствует PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() получает переменные, а не устанавливает маршрут! Смотрите объектный контекст ниже
	Flight::route('GET /users', function () {
	  // Соответствует GET /api/v2/users
	});
  });
});
```

#### Группировка с объектным контекстом

Вы всё ещё можете использовать группировку маршрутов с объектом `Engine` следующим образом:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // используем переменную $router
  $router->get('/users', function () {
	// Соответствует GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Соответствует POST /api/v1/posts
  });
});
```

> **Примечание:** Это предпочтительный способ определения маршрутов и групп с объектом `$router`.

#### Группировка с промежуточным ПО

Вы также можете назначить промежуточное ПО группе маршрутов:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Соответствует /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // или [ new MyAuthMiddleware() ], если вы хотите использовать экземпляр
```

Подробнее на странице [промежуточное ПО для групп](/learn/middleware#grouping-middleware).

### Ресурсная маршрутизация
Вы можете создать набор маршрутов для ресурса с помощью метода `resource`. Это создаст набор маршрутов для ресурса, следующих RESTful-соглашениям.

Чтобы создать ресурс, сделайте следующее:

```php
Flight::resource('/users', UsersController::class);
```

И что произойдёт в фоновом режиме, так это создание следующих маршрутов:

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

Ваш контроллер будет использовать следующие методы:

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

> **Примечание:** Вы можете просмотреть добавленные маршруты с помощью `runway`, выполнив `php runway routes`.

#### Настройка ресурсных маршрутов

Есть несколько параметров для настройки ресурсных маршрутов.

##### Базовый псевдоним (Alias Base)

Вы можете настроить `aliasBase`. По умолчанию псевдонимом является последняя часть указанного URL.
Например, `/users/` приведёт к `aliasBase` значению `users`. Когда эти маршруты создаются, псевдонимы будут `users.index`, `users.create` и т. д. Если вы хотите изменить псевдоним, установите `aliasBase` в нужное значение.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only и Except

Вы также можете указать, какие маршруты создавать, используя параметры `only` и `except`.

```php
// Разрешить только эти методы и запретить остальные
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Запретить только эти методы и разрешить остальные
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

По сути, это параметры белого и чёрного списков, позволяющие указать, какие маршруты вы хотите создать.

##### Промежуточное ПО

Вы также можете указать промежуточное ПО для выполнения на каждом из маршрутов, созданных методом `resource`.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Потоковые ответы (Streaming)

Теперь вы можете отправлять потоковые ответы клиенту с помощью `stream()` или `streamWithHeaders()`.
Это полезно для отправки больших файлов, длительных процессов или создания больших ответов. Потоковая передача маршрута обрабатывается немного иначе, чем обычный маршрут.

> **Примечание:** Потоковые ответы доступны только в том случае, если для параметра [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) установлено значение `false`.

#### Потоковая передача с ручной установкой заголовков

Вы можете отправить потоковый ответ клиенту, используя метод `stream()` на маршруте. Если вы это делаете, вы должны вручную установить все заголовки до того, как выведете что-либо клиенту.
Это делается с помощью PHP-функции `header()` или метода `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// очевидно, вы должны очистить путь и т.д.
	$fileNameSafe = basename($filename);

	// Если вам нужно установить дополнительные заголовки после выполнения маршрута,
	// вы должны определить их до любого вывода.
	// Все они должны быть прямым вызовом функции header() или
	// вызовом Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// или
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// вручную установите длину содержимого, если хотите
	header('Content-Length: '.filesize($filePath));
	// или
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Потоковая передача файла клиенту по мере его чтения
	readfile($filePath);

// Это волшебная строка здесь
})->stream();
```

#### Потоковая передача с заголовками

Вы также можете использовать метод `streamWithHeaders()` для установки заголовков перед началом потоковой передачи.

```php
Flight::route('/stream-users', function() {

	// вы можете добавить любые дополнительные заголовки здесь
	// просто необходимо использовать header() или Flight::response()->setRealHeader()

	// как бы вы ни получали данные, просто для примера...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Это необходимо для отправки данных клиенту
		ob_flush();
	}
	echo '}';

// Вот так вы установите заголовки перед началом потоковой передачи.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// необязательный код состояния, по умолчанию 200
	'status' => 200
]);
```

## Смотрите также
- [Промежуточное ПО](/learn/middleware) — Использование промежуточного ПО с маршрутами для аутентификации, логирования и т. д.
- [Внедрение зависимостей](/learn/dependency-injection-container) — Упрощение создания и управления объектами в маршрутах.
- [Зачем нужен фреймворк?](/learn/why-frameworks) — Понимание преимуществ использования такого фреймворка, как Flight.
- [Расширение](/learn/extending) — Как расширить Flight с помощью собственных функций, включая метод `notFound`.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) — PHP-функция для сопоставления регулярных выражений.

## Устранение неполадок
- Параметры маршрута сопоставляются по порядку, а не по имени. Убедитесь, что порядок параметров в функции обратного вызова соответствует определению маршрута.
- Использование `Flight::get()` не определяет маршрут; для маршрутизации используйте `Flight::route('GET /...')` или объект Router в группах (например, `$router->get(...)`).
- Свойство `executedRoute` устанавливается только после выполнения маршрута; до выполнения оно равно `NULL`.
- Потоковая передача требует отключения устаревшей функциональности буферизации вывода Flight (`flight.v2.output_buffering = false`).
- Для внедрения зависимостей только некоторые определения маршрутов поддерживают создание через контейнер.

### 404 Not Found или неожиданное поведение маршрута

Если вы видите ошибку 404 Not Found (но вы готовы поклясться, что маршрут действительно существует и это не опечатка), на самом деле это может быть проблемой того, что вы возвращаете значение в конечной точке маршрута вместо простого вывода. Причина этого намеренная, но может застать врасплох некоторых разработчиков.

```php
Flight::route('/hello', function(){
	// Это может вызвать ошибку 404 Not Found
	return 'Hello World';
});

// Вероятно, вам нужно это
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

Причина в том, что в роутер встроен специальный механизм, который интерпретирует возвращаемый вывод как сигнал «перейти к следующему маршруту». Это поведение задокументировано в разделе [Маршрутизация](/learn/routing#passing).

## История изменений
- v3: Добавлены ресурсная маршрутизация, псевдонимы маршрутов и поддержка потоковой передачи, группы маршрутов и поддержка промежуточного ПО.
- v1: Доступно подавляющее большинство базовых функций.