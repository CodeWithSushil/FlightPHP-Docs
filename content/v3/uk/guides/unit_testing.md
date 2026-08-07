# Unit-тестування у Flight PHP з PHPUnit

Цей посібник знайомить з unit-тестуванням у Flight PHP за допомогою [PHPUnit](https://phpunit.de/), і розрахований на початківців, які хочуть зрозуміти *чому* unit-тестування важливе та як застосовувати його на практиці. Ми зосередимось на тестуванні *поведінки* — перевірці, що ваш застосунок робить те, що ви очікуєте, наприклад, надсилає електронний лист або зберігає запис, а не на тривіальних обчисленнях. Ми почнемо з простого [обробника маршрутів](/learn/routing) і перейдемо до складнішого [контролера](/learn/routing), використовуючи [впровадження залежностей](/learn/dependency-injection-container) (DI) та макетування сторонніх сервісів.

## Чому unit-тестування?

Unit-тестування гарантує, що ваш код поводиться очікувано, виявляючи помилки до того, як вони потрапляють у продакшн. Воно особливо корисне у Flight, де легка маршрутизація та гнучкість можуть призводити до складних взаємодій. Для соло-розробників або команд unit-тести слугують страхувальною сіткою, документуючи очікувану поведінку та запобігаючи регресіям, коли ви повертаєтесь до коду пізніше. Вони також покращують дизайн: код, який важко тестувати, часто сигналізує про надто складні або тісно пов'язані класи.

На відміну від спрощених прикладів (наприклад, тестування `x * y = z`), ми зосередимось на реальних поведінках, таких як перевірка вхідних даних, збереження даних або ініціювання дій на кшталт надсилання листів. Наша мета — зробити тестування доступним і значущим.

## Загальні керівні принципи

1. **Тестуйте поведінку, а не реалізацію**: Зосереджуйтесь на результатах (наприклад, «лист надіслано» або «запис збережено»), а не на внутрішніх деталях. Це робить тести стійкими до рефакторингу.
2. **Перестаньте використовувати `Flight::`**: Статичні методи Flight неймовірно зручні, але ускладнюють тестування. Вам варто звикнути використовувати змінну `$app` з `$app = Flight::app();`. `$app` має всі ті ж методи, що й `Flight::`. Ви все одно зможете використовувати `$app->route()` або `$this->app->json()` у своєму контролері тощо. Також варто використовувати справжній маршрутизатор Flight через `$router = $app->router()`, і тоді ви зможете використовувати `$router->get()`, `$router->post()`, `$router->group()` тощо. Див. [Маршрутизація](/learn/routing).
3. **Тримайте тести швидкими**: Швидкі тести спонукають до частого запуску. Уникайте повільних операцій, як-от виклики бази даних в unit-тестах. Якщо у вас повільний тест, це ознака того, що ви пишете інтеграційний тест, а не unit-тест. Інтеграційні тести — це коли ви фактично задіюєте реальні бази даних, реальні HTTP-виклики, реальне надсилання листів тощо. Вони мають своє місце, але вони повільні та можуть бути нестабільними, тобто іноді падають з невідомої причини.
4. **Використовуйте описові назви**: Назви тестів мають чітко описувати поведінку, яка тестується. Це покращує читабельність і супроводжуваність.
5. **Уникайте глобальних змінних, як чуми**: Мінімізуйте використання `$app->set()` і `$app->get()`, оскільки вони діють як глобальний стан, вимагаючи макетів у кожному тесті. Віддавайте перевагу DI або контейнеру DI (див. [Контейнер впровадження залежностей](/learn/dependency-injection-container)). Навіть використання методу `$app->map()` технічно є «глобальним» станом, і його варто уникати на користь DI. Використовуйте бібліотеку сесій, наприклад [flightphp/session](https://github.com/flightphp/session), щоб мати змогу макетувати об'єкт сесії у ваших тестах. **Не** викликайте [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) безпосередньо у вашому коді, оскільки це вносить глобальну змінну у ваш код, що ускладнює тестування.
6. **Використовуйте впровадження залежностей**: Впроваджуйте залежності (наприклад, [`PDO`](https://www.php.net/manual/en/class.pdo.php), поштові сервіси) у контролери, щоб ізолювати логіку та спростити макетування. Якщо у вас клас із забагато залежностей, розгляньте можливість рефакторингу його на менші класи, кожен з яких має єдину відповідальність згідно з [принципами SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Макетуйте сторонні сервіси**: Макетуйте бази даних, HTTP-клієнти (cURL) або поштові сервіси, щоб уникнути зовнішніх викликів. Тестуйте на один-два рівні вглиб, але дайте вашій основній логіці виконуватись. Наприклад, якщо ваш застосунок надсилає SMS, ви **НЕ** хочете реально надсилати SMS щоразу під час запуску тестів, бо ці витрати накопичуватимуться (і це буде повільніше). Натомість змакетуйте сервіс SMS і просто перевірте, що ваш код викликав сервіс SMS із правильними параметрами.
8. **Прагніть високого покриття, а не досконалості**: 100% покриття рядків — це добре, але це не означає, що все у вашому коді протестовано так, як треба (можете дослідити [покриття гілок/шляхів у PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Пріоритетними є критичні поведінки (наприклад, реєстрація користувача, відповіді API та фіксація невдалих відповідей).
9. **Використовуйте контролери для маршрутів**: У визначеннях маршрутів використовуйте контролери, а не замикання. `flight\Engine $app` впроваджується в кожен контролер через конструктор за замовчуванням. У тестах використовуйте `$app = new Flight\Engine()`, щоб створити екземпляр Flight у межах тесту, впровадіть його у ваш контролер і викликайте методи безпосередньо (наприклад, `$controller->register()`). Див. [Розширення Flight](/learn/extending) та [Маршрутизація](/learn/routing).
10. **Оберіть стиль макетування і дотримуйтесь його**: PHPUnit підтримує кілька стилів макетування (наприклад, prophecy, вбудовані макети), або ви можете використовувати анонімні класи, які мають свої переваги, як-от автодоповнення коду, ламання, якщо ви змінюєте визначення методу, тощо. Просто будьте послідовні у своїх тестах. Див. [PHPUnit Mock Objects](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Використовуйте видимість `protected` для методів/властивостей, які ви хочете тестувати в підкласах**: Це дозволяє перевизначати їх у тестових підкласах, не роблячи їх публічними; це особливо корисно для анонімних класів-макетів.

## Налаштування PHPUnit

Спершу налаштуйте [PHPUnit](https://phpunit.de/) у вашому проєкті Flight PHP за допомогою Composer для зручного тестування. Більше деталей див. у [посібнику PHPUnit для початківців](https://phpunit.readthedocs.io/en/12.3/installation.html).

1. У каталозі вашого проєкту виконайте:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Це встановить останню версію PHPUnit як залежність для розробки.

2. Створіть каталог `tests` у корені проєкту для тестових файлів.

3. Додайте тестовий скрипт до `composer.json` для зручності:
   ```json
   // інший вміст composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Створіть файл `phpunit.xml` у корені:
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

Тепер, коли ваші тести створені, ви можете запускати `composer test` для їх виконання.

## Тестування простого обробника маршруту

Почнімо з базового [маршруту](/learn/routing), який перевіряє email користувача. Ми тестуватимемо його поведінку: повернення повідомлення про успіх для коректних email та помилку для некоректних. Для перевірки email ми використовуємо [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

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

Щоб протестувати це, створіть тестовий файл. Більше про структуру тестів див. у розділі [Unit-тестування та принципи SOLID](/learn/unit-testing-and-solid-principles):

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Імітація POST-даних
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
		$request->data->email = 'invalid-email'; // Імітація POST-даних
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Ключові моменти**:
- Ми імітуємо POST-дані за допомогою класу запиту. Не використовуйте глобальні змінні, як-от `$_POST`, `$_GET` тощо, оскільки це ускладнює тестування (вам доведеться щоразу скидати ці значення, інакше інші тести можуть впасти).
- Усі контролери за замовчуванням отримують екземпляр `flight\Engine`, впроваджений у них, навіть без налаштованого DIC-контейнера. Це значно спрощує безпосереднє тестування контролерів.
- Тут взагалі немає використання `Flight::`, що робить код легшим для тестування.
- Тести перевіряють поведінку: правильний статус і повідомлення для коректних/некоректних email.

Запустіть `composer test`, щоб перевірити, що маршрут поводиться очікувано. Більше про [запити](/learn/requests) та [відповіді](/learn/responses) у Flight див. у відповідній документації.

## Використання впровадження залежностей для тестованих контролерів

Для складніших сценаріїв використовуйте [впровадження залежностей](/learn/dependency-injection-container) (DI), щоб зробити контролери тестованими. Уникайте глобальних методів Flight (наприклад, `Flight::set()`, `Flight::map()`, `Flight::register()`), оскільки вони діють як глобальний стан, вимагаючи макетів для кожного тесту. Натомість використовуйте DI-контейнер Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) або ручне DI.

Використаймо [`flight\database\SimplePdo`](/learn/simple-pdo) замість сирого PDO. Цей хелпер набагато легше макетувати та тестувати (і він є кращим вибором порівняно із застарілим `PdoWrapper`).

Ось контролер, який зберігає користувача в базу даних і надсилає вітальний лист:

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
			// додавання return тут допомагає unit-тестуванню зупинити виконання
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Ключові моменти**:
- Контролер залежить від екземпляра [`SimplePdo`](/learn/simple-pdo) та `MailerInterface` (вигаданого стороннього поштового сервісу).
- Залежності впроваджуються через конструктор, що дозволяє уникнути глобальних змінних.

### Тестування контролера з макетами

Тепер протестуємо поведінку `UserController`: перевірку email, збереження в базу даних і надсилання листів. Ми змакетуємо базу даних і поштовий сервіс, щоб ізолювати контролер.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Іноді необхідно змішувати стилі макетування
		// Тут ми використовуємо вбудований макет PHPUnit для PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Використовуємо анонімний клас для макетування SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Коли ми так макетуємо, ми насправді не викликаємо базу даних.
			// Ми можемо додатково налаштувати це, щоб змінити макет PDOStatement для симуляції помилок тощо.
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
			// Порожній конструктор обходить конструктор батьківського класу
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

		// Потрібно зіставити jsonHalt, щоб уникнути виходу
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

**Ключові моменти**:
- Ми макетуємо `SimplePdo` та `MailerInterface`, щоб уникнути реальних викликів бази даних або email.
- Тести перевіряють поведінку: коректні email ініціюють вставки в базу даних і надсилання листів; некоректні email пропускають обидві дії.
- Макетуйте сторонні залежності (наприклад, `SimplePdo`, `MailerInterface`), дозволяючи логіці контролера виконуватись.

### Надмірне макетування

Будьте обережні, щоб не макетувати занадто багато вашого коду. Наведу приклад нижче, чому це може бути погано, на прикладі нашого `UserController`. Ми змінимо цю перевірку на метод під назвою `isEmailValid` (використовуючи `filter_var`), а інші нові доповнення — в окремий метод під назвою `registerUser`.

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
			// додавання return тут допомагає unit-тестуванню зупинити виконання
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

А тепер надмірно змакетований unit-тест, який насправді нічого не тестує:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// ми пропускаємо додаткове впровадження залежностей тут, бо це «легко»
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Обходимо залежності в конструкторі
			public function __construct($app) {
				$this->app = $app;
			}

			// Просто примусово робимо це валідним.
			protected function isEmailValid($email) {
				return true; // Завжди повертаємо true, обходячи реальну перевірку
			}

			// Обходимо реальні виклики БД та поштового сервісу
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

Ура, у нас є unit-тести, і вони проходять! Але зачекайте, що як я насправді зміню внутрішню роботу `isEmailValid` або `registerUser`? Мої тести все одно проходитимуть, бо я змакетував усю функціональність. Покажу, що я маю на увазі.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... інші методи ...

	protected function isEmailValid($email) {
		// Змінена логіка
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Тепер він має мати лише певний домен
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Якщо я запущу мої наведені вище unit-тести, вони все одно пройдуть! Але оскільки я не тестував поведінку (насправді дозволяючи деякому коду виконуватися), я, можливо, написав баг, який чекає на появу в продакшні. Тест слід змінити, щоб врахувати нову поведінку, а також протилежний випадок, коли поведінка не та, яку ми очікуємо.

## Повний приклад

Повний приклад проєкту Flight PHP з unit-тестами ви можете знайти на GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide). Для глибшого розуміння див. [Unit-тестування та принципи SOLID](/learn/unit-testing-and-solid-principles).

## Типові помилки

- **Надмірне макетування**: Не макетуйте кожну залежність; дозвольте частині логіки (наприклад, валідації контролера) виконуватися, щоб тестувати реальну поведінку. Див. [Unit-тестування та принципи SOLID](/learn/unit-testing-and-solid-principles).
- **Глобальний стан**: Активне використання глобальних PHP-змінних (наприклад, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) робить тести крихкими. Те саме стосується `Flight::`. Рефакторіть код, щоб передавати залежності явно.
- **Складне налаштування**: Якщо налаштування тесту є громіздким, ваш клас, можливо, має забагато залежностей або обов'язків, що порушує [принципи SOLID](/learn/unit-testing-and-solid-principles).

## Масштабування з unit-тестами

Unit-тести особливо корисні у більших проєктах або коли ви повертаєтесь до коду через місяці. Вони документують поведінку та виявляють регресії, рятуючи вас від повторного вивчення вашого застосунку. Для соло-розробників тестуйте критичні шляхи (наприклад, реєстрацію користувача, обробку платежів). Для команд тести забезпечують узгоджену поведінку серед усіх внесків. Більше про переваги фреймворків і тестів див. у розділі [Чому фреймворки?](/learn/why-frameworks).

Долучайтеся до репозиторію документації Flight PHP зі своїми порадами щодо тестування!

_Автор: [n0nag0n](https://github.com/n0nag0n), 2025_