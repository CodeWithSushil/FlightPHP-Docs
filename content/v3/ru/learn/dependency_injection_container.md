# Контейнер внедрения зависимостей

## Обзор

Контейнер внедрения зависимостей (DIC) — это мощное расширение, которое позволяет управлять
зависимостями вашего приложения. Это также одна из главных причин, по которой Flight хорошо сочетается с [инструментами ИИ для написания кода](/learn/ai) и модульными тестами: контроллеры получают то, что им нужно, через конструктор, а не обращаются к глобальным переменным.

## Понимание

Внедрение зависимостей (DI) — ключевая концепция в современных PHP-фреймворках, которая
используется для управления созданием и настройкой объектов. Вот некоторые примеры библиотек
DIC: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/),
[PHP-DI](http://php-di.org/) и [league/container](https://container.thephpleague.com/).

DIC — это удобный способ создания и управления вашими классами в
централизованном месте. Это полезно, когда вам нужно передавать один и тот же объект
нескольким классам (контроллерам, промежуточному ПО, командам и так далее).

Официальный [flightphp/skeleton](https://github.com/flightphp/skeleton) подключает **Dice** в `app/config/services.php`, заменяет общий экземпляр `flight\Engine` и разрешает цели маршрутов вида `[App\Controller\HomeController::class, 'index']`. Для новых проектов предпочтительнее использовать именно этот шаблон, чтобы и люди, и агенты редактировали одни и те же места.

## Базовое использование

Старый подход мог выглядеть так:
```php

require 'vendor/autoload.php';

// класс для управления пользователями из базы данных
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

// в вашем файле routes.php

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// другие маршруты UserController...

Flight::start();
```

Из приведённого выше кода видно, что мы создаём новый объект `PDO` и передаём его
в наш класс `UserController`. Для небольшого приложения это нормально, но по мере роста
приложения вы обнаружите, что создаёте или передаёте один и тот же объект `PDO`
в нескольких местах. Вот тут-то и пригодится DIC.

Вот тот же пример с использованием DIC (на основе Dice):
```php

require 'vendor/autoload.php';

// тот же класс, что и выше. Ничего не изменилось
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

// создаем новый контейнер
$container = new \Dice\Dice;

// добавляем правило, чтобы сообщить контейнеру, как создать объект PDO
// не забудьте переназначить его на себя, как показано ниже!
$container = $container->addRule('PDO', [
	// shared означает, что каждый раз будет возвращаться один и тот же объект
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Это регистрирует обработчик контейнера, чтобы Flight знал о его использовании.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// теперь мы можем использовать контейнер для создания нашего UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Бьюсь об заклад, вы думаете, что в пример было добавлено много лишнего кода.
Вся магия проявляется, когда появляется другой контроллер, которому нужен объект `PDO`.

```php

// Если все ваши контроллеры имеют конструктор, которому нужен объект PDO
// в каждый из маршрутов ниже он будет внедрен автоматически!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

Дополнительный бонус использования DIC в том, что модульное тестирование становится намного проще. Вы можете
создать мок-объект и передать его в ваш класс. Это огромное преимущество при написании
тестов для вашего приложения — а когда ИИ-ассистент генерирует контроллер, внедрение через конструктор даёт ему понятный и последовательный шаблон для следования ([руководство по модульному тестированию](/guides/unit-testing)).

### Создание централизованного обработчика DIC

Вы можете создать централизованный обработчик DIC в вашем файле сервисов, [расширив](/learn/extending) ваше приложение. Вот пример:

```php
// services.php

// создаем новый контейнер
$container = new \Dice\Dice;
// не забудьте переназначить его на себя, как показано ниже!
$container = $container->addRule('PDO', [
	// shared означает, что каждый раз будет возвращаться один и тот же объект
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// теперь мы можем создать отображаемый метод для создания любого объекта.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Это регистрирует обработчик контейнера, чтобы Flight знал о его использовании для контроллеров/промежуточного ПО
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// предположим, у нас есть следующий пример класса, который принимает объект PDO в конструкторе
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// код, который отправляет электронное письмо
	}
}

// И, наконец, вы можете создавать объекты с помощью внедрения зависимостей
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

У Flight есть плагин, предоставляющий простой PSR-11-совместимый контейнер, который можно использовать для управления
вашим внедрением зависимостей. Вот краткий пример его использования:

```php

// index.php, например
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
	// выведет это правильно!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Продвинутое использование flightphp/container

Вы также можете разрешать зависимости рекурсивно. Вот пример:

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
    // Реализация ...
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

Вы также можете создать свой собственный обработчик DIC. Это полезно, если у вас есть кастомный
контейнер, который вы хотите использовать и который не является PSR-11 (Dice). Смотрите
[базовое использование](#базовое-использование), чтобы узнать, как это сделать.

Кроме того,
есть несколько полезных значений по умолчанию, которые облегчат вам жизнь при работе с Flight.

#### Экземпляр Engine (требуется для внедрения `$app`)

Если вы указываете тип `flight\Engine` в контроллерах или промежуточном ПО, **Dice не должен создавать новый Engine**. Подставьте тот же экземпляр из начальной загрузки. Именно так делает официальный скелет, и это шаблон, который `AGENTS.md` ожидает для контроллеров, сгенерированных ИИ:

```php
// Где-то в вашем файле начальной загрузки / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // или $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Критически важно: используйте загруженный Engine — не позволяйте Dice создавать `new Engine()`
		Engine::class => $app,
		// Для нового кода предпочитайте SimplePdo
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Необязательный помощник для кода вне маршрутов
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php (структура скелета — регистр папки соответствует пространству имен)
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
		// Никакого фасада Flight:: в прикладном слое — проще тестировать и понятнее для ИИ-инструментов
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Если вы пропустите подстановку `Engine`, Dice может создать второй Engine, и ваш контроллер не будет использовать общие маршруты, конфигурацию или сопоставленный Twig `render` из начальной загрузки.

#### Добавление других общих сервисов (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// После того как вы создали $db, $config, $twig в services.php:
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

Тогда контроллеры могут принимать `SimplePdo $db` (или ваш тип конфигурации) в конструкторе и никогда не вызывать `Flight::db()`. Это соответствует рекомендациям из [модульного тестирования](/guides/unit-testing) и стилю проекта скелета.

#### Добавление других классов

Если у вас есть другие классы, которые вы хотите добавить в контейнер, с Dice это легко, поскольку они будут автоматически разрешены контейнером. Вот пример:

```php

$container = new \Dice\Dice;
// Если вам не нужно внедрять какие-либо зависимости в ваши классы,
// вам не нужно ничего определять!
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

Flight также может использовать любой PSR-11-совместимый контейнер. Это означает, что вы можете использовать любой
контейнер, реализующий интерфейс PSR-11. Вот пример использования PSR-11-контейнера от League:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// та же идея UserController, что и выше, с указанием типа SimplePdo вместо сырого PDO

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

Это может быть немного более многословно, чем предыдущий пример с Dice, но оно
выполняет свою задачу с теми же преимуществами!

## Смотрите также
- [Установка](/install) — Структура скелета и где находится `services.php`.
- [Автозагрузка](/learn/autoloading) — Пространства имён `App\` и **регистр** папок.
- [Расширение Flight](/learn/extending) — Узнайте, как добавить внедрение зависимостей в ваши собственные классы, расширяя фреймворк.
- [Конфигурация](/learn/configuration) — Узнайте, как настроить Flight для вашего приложения.
- [Маршрутизация](/learn/routing) — Узнайте, как определять маршруты для вашего приложения и как внедрение зависимостей работает с контроллерами.
- [Промежуточное ПО](/learn/middleware) — Узнайте, как создавать промежуточное ПО для вашего приложения и как внедрение зависимостей работает с ним.
- [Модульное тестирование](/guides/unit-testing) — Почему внедрение через конструктор лучше глобальных `Flight::`.
- [ИИ и опыт разработчика](/learn/ai) — Единый шаблон DI для людей и агентов.
- [SimplePdo](/learn/simple-pdo) — Предпочтительный помощник для работы с базой данных при внедрении.

## Устранение неполадок
- Если у вас возникают проблемы с контейнером, убедитесь, что вы передаёте в контейнер правильные имена классов.
- Контроллеры, которые указывают тип `Engine`, но получают «пустое» приложение: добавьте **подстановку Engine** (см. выше). Dice не должен создавать второй Engine через `new`.
- Класс не найден для `App\Controller\…`: проверьте регистр папки в `app/Controller/` — см. [Автозагрузка](/learn/autoloading).
- Обработчик должен **возвращать** созданный объект из `registerContainerHandler` (не вызывайте `Flight::make()` без `return`).

## Журнал изменений
- Документация — Описание скелета Dice + подстановки Engine, SimplePdo и структуры `App\Controller` для проектов, дружественных к ИИ.
- v3.7.0 — Добавлена возможность регистрации обработчика DIC во Flight.