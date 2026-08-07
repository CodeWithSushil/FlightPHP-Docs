# Контейнер впровадження залежностей

## Огляд

Контейнер впровадження залежностей (DIC) — це потужне розширення, яке дозволяє керувати
залежностями вашого застосунку. Це також одна з найбільших причин, чому Flight добре працює з [AI-інструментами кодування](/learn/ai) та модульними тестами: контролери отримують те, що їм потрібно, через конструктор, замість того щоб звертатися до глобальних змінних.

## Розуміння

Впровадження залежностей (DI) — це ключова концепція сучасних PHP-фреймворків, яка
використовується для керування створенням та конфігурацією об'єктів. Деякі приклади 
бібліотек DIC: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/), 
[PHP-DI](http://php-di.org/) та [league/container](https://container.thephpleague.com/).

DIC — це вигадливий спосіб створення та керування вашими класами в
централізованому місці. Це корисно, коли вам потрібно передати один і той самий об'єкт 
кільком класам (контролерам, проміжному ПЗ, командам тощо).

Офіційний [flightphp/skeleton](https://github.com/flightphp/skeleton) під'єднує **Dice** у `app/config/services.php`, підставляє спільний екземпляр `flight\Engine` та розв'язує цілі маршрутів як-от `[App\Controller\HomeController::class, 'index']`. Віддавайте перевагу цьому патерну для нових проєктів, щоб люди та агенти редагували одні й ті самі місця.

## Базове використання

Старий спосіб може виглядати так:
```php

require 'vendor/autoload.php';

// клас для керування користувачами з бази даних
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// у вашому файлі routes.php

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// інші маршрути UserController...

Flight::start();
```

З наведеного вище коду видно, що ми створюємо новий об'єкт `PDO` і передаємо його
нашому класу `UserController`. Це нормально для невеликого застосунку, але в міру
зростання вашого застосунку ви виявите, що створюєте або передаєте той самий об'єкт `PDO` 
в кількох місцях. Ось тут і стає в пригоді DIC.

Ось той самий приклад із використанням DIC (з Dice):
```php

require 'vendor/autoload.php';

// той самий клас, що й вище. Нічого не змінилося
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// створюємо новий контейнер
$container = new \Dice\Dice;

// додаємо правило, щоб розповісти контейнеру, як створити об'єкт PDO
// не забудьте переназначити його самому собі, як показано нижче!
$container = $container->addRule('PDO', [
	// shared означає, що щоразу повертатиметься той самий об'єкт
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Це реєструє обробник контейнера, щоб Flight знав, як його використовувати.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// тепер ми можемо використовувати контейнер для створення нашого UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Гадаю, ви можете подумати, що в приклад було додано багато зайвого коду.
Магія проявляється, коли у вас з'являється інший контролер, якому потрібен об'єкт `PDO`. 

```php

// Якщо всі ваші контролери мають конструктор, якому потрібен об'єкт PDO,
// кожен із маршрутів нижче автоматично отримає його через впровадження!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

Додаткова перевага використання DIC полягає в тому, що модульне тестування стає набагато простішим. Ви можете
створити імітаційний об'єкт і передати його у ваш клас. Це величезна перевага, коли ви
пишете тести для свого застосунку — а коли AI-асистент генерує контролер, впровадження через конструктор дає йому чіткий і послідовний патерн для наслідування ([посібник із модульного тестування](/guides/unit-testing)).

### Створення централізованого обробника DIC

Ви можете створити централізований обробник DIC у вашому файлі сервісів, [розширивши](/learn/extending) ваш застосунок. Ось приклад:

```php
// services.php

// створюємо новий контейнер
$container = new \Dice\Dice;
// не забудьте переназначити його самому собі, як показано нижче!
$container = $container->addRule('PDO', [
	// shared означає, що щоразу повертатиметься той самий об'єкт
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// тепер ми можемо створити метод для зіставлення, щоб створювати будь-який об'єкт. 
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Це реєструє обробник контейнера, щоб Flight знав, як використовувати його для контролерів/проміжного ПЗ
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// припустімо, у нас є такий приклад класу, який приймає об'єкт PDO у конструкторі
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// код, який надсилає електронний лист
	}
}

// І нарешті ви можете створювати об'єкти за допомогою впровадження залежностей
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight має плагін, який надає простий PSR-11 сумісний контейнер, який ви можете використовувати для
керування впровадженням залежностей. Ось швидкий приклад його використання:

```php

// index.php, наприклад
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// виведе це правильно!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Розширене використання flightphp/container

Ви також можете розв'язувати залежності рекурсивно. Ось приклад:

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Реалізація ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

Ви також можете створити власний обробник DIC. Це корисно, якщо у вас є власний
контейнер, який ви хочете використовувати і який не є PSR-11 (Dice). Перегляньте
розділ [базове використання](#basic-usage), щоб дізнатися, як це зробити.

Крім того,
є кілька корисних налаштувань за замовчуванням, які полегшать вам життя під час роботи з Flight.

#### Екземпляр Engine (обов'язково для впровадження `$app`)

Якщо ви вказуєте тип `flight\Engine` у контролерах або проміжному ПЗ, **Dice не повинен створювати новий Engine**. Підставте той самий екземпляр із bootstrap-коду. Саме це робить офіційний skeleton, і саме цей патерн очікує `AGENTS.md` для AI-згенерованих контролерів:

```php
// Десь у вашому bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // або $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Критично важливо: перевикористовуйте ініціалізований Engine — не дозволяйте Dice робити `new Engine()`
		Engine::class => $app,
		// Віддавайте перевагу SimplePdo для нового коду
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Додатковий помічник для коду поза маршрутами
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (структура skeleton — регістр папок відповідає неймспейсу)
namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// Жодного фасаду Flight:: у шарі застосунку — легше тестувати та зрозуміліше для AI-інструментів
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Якщо ви пропустите підстановку `Engine`, Dice може створити другий Engine, і ваш контролер не матиме спільних маршрутів, конфігурації або зіставленого Twig `render` із bootstrap-коду.

#### Додавання інших спільних сервісів (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Після створення $db, $config, $twig у services.php:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

Тоді контролери можуть приймати `SimplePdo $db` (або ваш тип конфігурації) у конструкторі й ніколи не викликати `Flight::db()`. Це відповідає рекомендаціям із [модульного тестування](/guides/unit-testing) та фірмовому стилю skeleton.

#### Додавання інших класів

Якщо у вас є інші класи, які ви хочете додати до контейнера, з Dice це просто, оскільки контейнер автоматично їх розв'яже. Ось приклад:

```php

$container = new \Dice\Dice;
// Якщо вам не потрібно впроваджувати жодних залежностей у ваші класи,
// вам не потрібно нічого визначати!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flight також може використовувати будь-який PSR-11 сумісний контейнер. Це означає, що ви можете використовувати будь-який
контейнер, який реалізує інтерфейс PSR-11. Ось приклад використання контейнера League
PSR-11:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// та сама ідея UserController, що й вище, але з типом SimplePdo замість сирого PDO

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

Це може бути трохи багатослівніше, ніж попередній приклад із Dice, але це все одно
виконує роботу з тими самими перевагами!

## Дивіться також
- [Встановлення](/install) — Структура skeleton і де знаходиться `services.php`.
- [Автозавантаження](/learn/autoloading) — Неймспейси `App\` та **регістр** папок.
- [Розширення Flight](/learn/extending) — Дізнайтеся, як додати впровадження залежностей до ваших власних класів, розширивши фреймворк.
- [Конфігурація](/learn/configuration) — Дізнайтеся, як налаштувати Flight для вашого застосунку.
- [Маршрутизація](/learn/routing) — Дізнайтеся, як визначати маршрути для вашого застосунку та як впровадження залежностей працює з контролерами.
- [Проміжне ПЗ](/learn/middleware) — Дізнайтеся, як створювати проміжне ПЗ для вашого застосунку та як впровадження залежностей працює з проміжним ПЗ.
- [Модульне тестування](/guides/unit-testing) — Чому впровадження через конструктор перевершує глобальні змінні `Flight::`.
- [AI та досвід розробника](/learn/ai) — Єдиний патерн DI для людей та агентів.
- [SimplePdo](/learn/simple-pdo) — Бажаний помічник для роботи з базою даних для впровадження.

## Усунення неполадок
- Якщо у вас виникають проблеми з вашим контейнером, переконайтеся, що ви передаєте правильні назви класів до контейнера.
- Контролери, які вказують тип `Engine`, але отримують «порожній» застосунок: додайте **підстановку Engine** (див. вище). Dice не повинен створювати другий Engine через `new`.
- Клас не знайдено для `App\Controller\…`: перевірте регістр папок у `app/Controller/` — див. [Автозавантаження](/learn/autoloading).
- Обробник повинен **повертати** створений об'єкт із `registerContainerHandler` (не викликайте `Flight::make()` без `return`).

## Журнал змін
- Документація – Документовано skeleton Dice + підстановки Engine, SimplePdo та структуру `App\Controller` для AI-дружніх проєктів.
- v3.7.0 - Додано можливість реєструвати обробник DIC у Flight.