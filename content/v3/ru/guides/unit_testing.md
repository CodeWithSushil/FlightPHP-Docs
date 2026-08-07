# Модульное тестирование в Flight PHP с PHPUnit

Это руководство знакомит с модульным тестированием во Flight PHP с использованием [PHPUnit](https://phpunit.de/), предназначено для начинающих, которые хотят понять *почему* модульное тестирование важно и как применять его на практике. Мы сосредоточимся на тестировании *поведения* — проверке, что ваше приложение делает то, что вы ожидаете, например, отправляет электронное письмо или сохраняет запись, — а не на тривиальных вычислениях. Мы начнем с простого [обработчика маршрута](/learn/routing) и перейдем к более сложному [контроллеру](/learn/routing), используя [внедрение зависимостей](/learn/dependency-injection-container) (DI) и имитацию сторонних сервисов.

## Зачем нужно модульное тестирование?

Модульное тестирование гарантирует, что ваш код ведет себя ожидаемо, выявляя ошибки до попадания в продакшн. Это особенно ценно во Flight, где легковесная маршрутизация и гибкость могут приводить к сложным взаимодействиям. Для разработчиков-одиночек или команд модульные тесты действуют как страховочная сеть, документируя ожидаемое поведение и предотвращая регрессии при возвращении к коду позже. Они также улучшают дизайн: код, который трудно тестировать, часто сигнализирует о чрезмерно сложных или тесно связанных классах.

В отличие от упрощенных примеров (например, проверка `x * y = z`), мы сосредоточимся на реальных сценариях, таких как проверка ввода, сохранение данных или запуск действий вроде отправки писем. Наша цель — сделать тестирование доступным и осмысленным.

## Общие руководящие принципы

1. **Тестируйте поведение, а не реализацию**: Сосредоточьтесь на результатах (например, «письмо отправлено» или «запись сохранена»), а не на внутренних деталях. Это делает тесты устойчивыми к рефакторингу.
2. **Перестаньте использовать `Flight::`**: Статические методы Flight чрезвычайно удобны, но затрудняют тестирование. Привыкайте использовать переменную `$app` из `$app = Flight::app();`. `$app` имеет все те же методы, что и `Flight::`. Вы по-прежнему сможете использовать `$app->route()` или `$this->app->json()` в контроллере и т.д. Также следует использовать настоящий роутер Flight через `$router = $app->router()`, и тогда вы сможете использовать `$router->get()`, `$router->post()`, `$router->group()` и т.д. См. [Маршрутизация](/learn/routing).
3. **Поддерживайте тесты быстрыми**: Быстрые тесты побуждают к частому выполнению. Избегайте медленных операций, таких как вызовы баз данных, в модульных тестах. Если у вас медленный тест, это признак того, что вы пишете интеграционный тест, а не модульный. Интеграционные тесты — это когда вы реально подключаете базы данных, реальные HTTP-вызовы, реальную отправку писем и т.д. У них есть свое место, но они медленные и могут быть нестабильными, то есть иногда падать по неизвестной причине.
4. **Используйте описательные имена**: Имена тестов должны четко описывать тестируемое поведение. Это улучшает читаемость и поддерживаемость.
5. **Избегайте глобальных переменных как чумы**: Минимизируйте использование `$app->set()` и `$app->get()`, так как они действуют как глобальное состояние, требуя имитаций в каждом тесте. Предпочитайте DI или контейнер внедрения зависимостей (см. [Контейнер внедрения зависимостей](/learn/dependency-injection-container)). Даже использование метода `$app->map()` технически является «глобальным» и его следует избегать в пользу DI. Используйте библиотеку сессий, например [flightphp/session](https://github.com/flightphp/session), чтобы можно было имитировать объект сессии в тестах. **Не** вызывайте [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) напрямую в коде, так как это внедряет глобальную переменную в ваш код, что затрудняет тестирование.
6. **Используйте внедрение зависимостей**: Внедряйте зависимости (например, [`PDO`](https://www.php.net/manual/en/class.pdo.php), почтовые сервисы) в контроллеры, чтобы изолировать логику и упростить имитацию. Если у класса слишком много зависимостей, рассмотрите возможность рефакторинга на более мелкие классы, каждый из которых имеет одну ответственность в соответствии с [принципами SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Имитируйте сторонние сервисы**: Имитируйте базы данных, HTTP-клиенты (cURL) или почтовые сервисы, чтобы избежать внешних вызовов. Тестируйте на один-два уровня вглубь, но давайте основной логике выполняться. Например, если ваше приложение отправляет текстовое сообщение, вам **НЕ** нужно реально отправлять сообщение каждый раз при запуске тестов, потому что расходы будут расти (и это будет медленнее). Вместо этого имитируйте сервис отправки сообщений и просто проверяйте, что ваш код вызвал этот сервис с правильными параметрами.
8. **Стремитесь к высокому покрытию, а не к совершенству**: 100% покрытие строк — это хорошо, но на самом деле не означает, что весь код протестирован так, как нужно (почитайте о [покрытии ветвей/путей в PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Приоритезируйте критически важное поведение (например, регистрацию пользователя, ответы API и фиксацию неудачных ответов).
9. **Используйте контроллеры для маршрутов**: В определениях маршрутов используйте контроллеры, а не замыкания. Экземпляр `flight\Engine $app` внедряется в каждый контроллер через конструктор по умолчанию. В тестах используйте `$app = new Flight\Engine()` для создания экземпляра Flight внутри теста, внедряйте его в контроллер и вызывайте методы напрямую (например, `$controller->register()`). См. [Расширение Flight](/learn/extending) и [Маршрутизация](/learn/routing).
10. **Выберите стиль имитации и придерживайтесь его**: PHPUnit поддерживает несколько стилей имитации (например, prophecy, встроенные имитации), или вы можете использовать анонимные классы, у которых есть свои преимущества, такие как автодополнение кода, поломка при изменении сигнатуры метода и т.д. Просто будьте последовательны в своих тестах. См. [PHPUnit Mock Objects](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Используйте видимость `protected` для методов/свойств, которые вы хотите тестировать в подклассах**: Это позволяет переопределять их в тестовых подклассах без открытия доступа, что особенно полезно для имитаций анонимных классов.

## Настройка PHPUnit

Сначала настройте [PHPUnit](https://phpunit.de/) в вашем проекте Flight PHP с помощью Composer для удобного тестирования. См. [Руководство по началу работы с PHPUnit](https://phpunit.readthedocs.io/en/12.3/installation.html) для подробностей.

1. В каталоге вашего проекта выполните:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Это установит последнюю версию PHPUnit как зависимость для разработки.

2. Создайте каталог `tests` в корне вашего проекта для файлов тестов.

3. Добавьте скрипт тестирования в `composer.json` для удобства:
   ```json
   // остальное содержимое composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Создайте файл `phpunit.xml` в корне:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Теперь, когда ваши тесты написаны, вы можете запустить `composer test` для их выполнения.

## Тестирование простого обработчика маршрута

Начнем с простого [маршрута](/learn/routing), который проверяет ввод email пользователя. Мы протестируем его поведение: возврат сообщения об успехе для корректных email и ошибки для некорректных. Для проверки email мы используем [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
	protected $app;

	public function __construct(flight\Engine $app) {
		$this->app = $app;
	}

	public function register() {
		$email = $this->app->request()->data->email;
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

Для тестирования создайте файл теста. См. [Модульное тестирование и принципы SOLID](/learn/unit-testing-and-solid-principles) для получения дополнительной информации о структуре тестов:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Имитация POST-данных
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Имитация POST-данных
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Ключевые моменты**:
- Мы имитируем POST-данные с помощью класса запроса. Не используйте глобальные переменные, такие как `$_POST`, `$_GET` и т.д., так как это усложняет тестирование (вам придется постоянно сбрасывать эти значения, иначе другие тесты могут упасть).
- Все контроллеры по умолчанию получают экземпляр `flight\Engine`, внедренный в них, даже без настройки контейнера DI. Это значительно упрощает прямое тестирование контроллеров.
- Здесь вообще не используется `Flight::`, что делает код более простым для тестирования.
- Тесты проверяют поведение: правильный статус и сообщение для корректных/некорректных email.

Запустите `composer test`, чтобы убедиться, что маршрут ведет себя ожидаемо. Дополнительную информацию о [запросах](/learn/requests) и [ответах](/learn/responses) во Flight см. в соответствующей документации.

## Использование внедрения зависимостей для тестируемых контроллеров

Для более сложных сценариев используйте [внедрение зависимостей](/learn/dependency-injection-container) (DI), чтобы сделать контроллеры тестируемыми. Избегайте глобальных переменных Flight (например, `Flight::set()`, `Flight::map()`, `Flight::register()`), так как они действуют как глобальное состояние, требуя имитаций для каждого теста. Вместо этого используйте контейнер DI Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) или ручное внедрение зависимостей.

Давайте использовать [`flight\database\SimplePdo`](/learn/simple-pdo) вместо сырого PDO. Этот помощник гораздо проще имитировать и тестировать (и он предпочтительнее устаревшего `PdoWrapper`).

Вот контроллер, который сохраняет пользователя в базу данных и отправляет приветственное письмо:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// добавление return здесь помогает модульному тестированию остановить выполнение
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Ключевые моменты**:
- Контроллер зависит от экземпляра [`SimplePdo`](/learn/simple-pdo) и `MailerInterface` (предполагаемый сторонний почтовый сервис).
- Зависимости внедряются через конструктор, что позволяет избежать глобальных переменных.

### Тестирование контроллера с имитациями

Теперь давайте протестируем поведение `UserController`: проверку email, сохранение в базу данных и отправку писем. Мы имитируем базу данных и почтовый сервис, чтобы изолировать контроллер.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Иногда необходимо смешивать стили имитации
		// Здесь мы используем встроенную имитацию PHPUnit для PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Использование анонимного класса для имитации SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// При такой имитации мы на самом деле не обращаемся к базе данных.
			// Мы можем дополнительно настроить имитацию PDOStatement для имитации сбоев и т.д.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Пустой конструктор обходит родительский конструктор
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// Необходимо сопоставить jsonHalt, чтобы избежать выхода
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**Ключевые моменты**:
- Мы имитируем `SimplePdo` и `MailerInterface`, чтобы избежать реальных вызовов базы данных или отправки писем.
- Тесты проверяют поведение: корректные email запускают вставки в базу данных и отправку писем; некорректные email пропускают оба действия.
- Имитируйте сторонние зависимости (например, `SimplePdo`, `MailerInterface`), позволяя логике контроллера выполняться.

### Чрезмерная имитация

Будьте осторожны, не имитируйте слишком большую часть вашего кода. Приведу пример, почему это может быть плохо, на основе нашего `UserController`. Мы изменим проверку на метод `isEmailValid` (используя `filter_var`), а остальные новые добавления в отдельный метод `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// добавление return здесь помогает модульному тестированию остановить выполнение
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

А теперь пере-имитированный модульный тест, который на самом деле ничего не тестирует:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// мы пропускаем дополнительное внедрение зависимостей здесь, потому что это "легко"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Обходим зависимости в конструкторе
			public function __construct($app) {
				$this->app = $app;
			}

			// Просто принудительно сделаем это валидным.
			protected function isEmailValid($email) {
				return true; // Всегда возвращает true, обходя реальную проверку
			}

			// Обходим реальные вызовы БД и почтового сервиса
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

Ура, у нас есть модульные тесты, и они проходят! Но подождите, что если я действительно изменю внутреннюю работу `isEmailValid` или `registerUser`? Мои тесты все равно будут проходить, потому что я имитировал все функциональности. Позвольте показать, что я имею в виду.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... другие методы ...

	protected function isEmailValid($email) {
		// Измененная логика
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Теперь должен быть только определенный домен
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Если я запущу свои вышеуказанные модульные тесты, они все равно пройдут! Но поскольку я не тестировал поведение (фактически позволяя некоторому коду выполняться), я потенциально запрограммировал ошибку, которая ожидает возможности попасть в продакшн. Тест должен быть изменен с учетом нового поведения, а также противоположного случая, когда поведение не соответствует ожиданиям.

## Полный пример

Полный пример проекта Flight PHP с модульными тестами можно найти на GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide).
Для более глубокого понимания см. [Модульное тестирование и принципы SOLID](/learn/unit-testing-and-solid-principles).

## Частые ошибки

- **Чрезмерная имитация**: Не имитируйте каждую зависимость; позвольте некоторой логике (например, проверке контроллера) выполняться, чтобы тестировать реальное поведение. См. [Модульное тестирование и принципы SOLID](/learn/unit-testing-and-solid-principles).
- **Глобальное состояние**: Активное использование глобальных PHP-переменных (например, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) делает тесты хрупкими. То же самое касается `Flight::`. Выполните рефакторинг, чтобы передавать зависимости явно.
- **Сложная настройка**: Если настройка теста громоздка, возможно, у вашего класса слишком много зависимостей или обязанностей, что нарушает [принципы SOLID](/learn/unit-testing-and-solid-principles).

## Масштабирование с помощью модульных тестов

Модульные тесты особенно полезны в крупных проектах или при возвращении к коду спустя месяцы. Они документируют поведение и выявляют регрессии, избавляя вас от необходимости заново изучать приложение. Для разработчиков-одиночек тестируйте критические пути (например, регистрацию пользователей, обработку платежей). Для команд тесты обеспечивают согласованное поведение при внесении изменений. См. [Зачем нужны фреймворки?](/learn/why-frameworks) для получения дополнительной информации о преимуществах использования фреймворков и тестов.

Внесите свои собственные советы по тестированию в репозиторий документации Flight PHP!

_Автор: [n0nag0n](https://github.com/n0nag0n) 2025_