# Модульне тестування

## Огляд

Модульне тестування у Flight допомагає вам переконатися, що ваш застосунок поводиться очікувано, виявляти помилки на ранніх етапах і полегшувати підтримку вашої кодової бази. Flight розроблено для бездоганної роботи з [PHPUnit](https://phpunit.de/) — найпопулярнішим фреймворком для тестування PHP.

## Розуміння

Модульні тести перевіряють поведінку невеликих частин вашого застосунку (як-от контролери чи сервіси) ізольовано. У Flight це означає перевірку того, як ваші маршрути, контролери та логіка реагують на різні вхідні дані — без залежності від глобального стану чи реальних зовнішніх сервісів.

Ключові принципи:
- **Тестуйте поведінку, а не реалізацію:** Зосереджуйтеся на тому, що робить ваш код, а не на тому, як він це робить.
- **Уникайте глобального стану:** Використовуйте впровадження залежностей замість `Flight::set()` або `Flight::get()`.
- **Імітуйте зовнішні сервіси:** Замінюйте такі речі, як бази даних або поштові сервіси, тестовими дублерами.
- **Тримайте тести швидкими та зосередженими:** Модульні тести не повинні звертатися до реальних баз даних чи API.

## Базове використання

### Налаштування PHPUnit

1. Встановіть PHPUnit за допомогою Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Створіть каталог `tests` у корені вашого проєкту.
3. Додайте тестовий скрипт до вашого `composer.json`:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Створіть файл `phpunit.xml`:
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

Тепер ви можете запускати свої тести за допомогою `composer test`.

### Тестування простого обробника маршруту

Припустимо, у вас є маршрут, який перевіряє електронну пошту:

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
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Простий тест для цього контролера:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Поради:**
- Імітуйте POST-дані за допомогою `$app->request()->data`.
- Уникайте використання статичних `Flight::` у тестах — використовуйте екземпляр `$app`.

### Використання впровадження залежностей для тестованих контролерів

Впроваджуйте залежності (наприклад, базу даних або поштовий сервіс) у ваші контролери, щоб їх було легко імітувати в тестах:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

І тест із тестовими дублерами:

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Просунуте використання

- **Створення тестових дублерів:** Використовуйте вбудовані тестові дублери PHPUnit або анонімні класи для заміни залежностей.
- **Тестування контролерів безпосередньо:** Створюйте контролери з новим `Engine` та імітуйте залежності.
- **Не перестарайтеся з імітацією:** Дайте реальній логіці виконуватися там, де це можливо; імітуйте лише зовнішні сервіси.

## Дивіться також

- [Посібник з модульного тестування](/guides/unit-testing) — вичерпний посібник із найкращих практик модульного тестування.
- [Контейнер впровадження залежностей](/learn/dependency-injection-container) — як використовувати DIC для керування залежностями та покращення тестованості.
- [Розширення](/learn/extending) — як додавати власні помічники або перевизначати основні класи.
- [SimplePdo](/learn/simple-pdo) — спрощує взаємодію з базою даних і легше імітується в тестах.
- [Запити](/learn/requests) — обробка HTTP-запитів у Flight.
- [Відповіді](/learn/responses) — надсилання відповідей користувачам.
- [Модульне тестування та принципи SOLID](/learn/unit-testing-and-solid-principles) — дізнайтеся, як принципи SOLID можуть покращити ваші модульні тести.

## Усунення неполадок

- Уникайте використання глобального стану (`Flight::set()`, `$_SESSION` тощо) у вашому коді та тестах.
- Якщо ваші тести повільні, можливо, ви пишете інтеграційні тести — імітуйте зовнішні сервіси, щоб модульні тести залишалися швидкими.
- Якщо налаштування тестів складне, розгляньте можливість рефакторингу коду для використання впровадження залежностей.

## Журнал змін

- v3.15.0 — додано приклади для впровадження залежностей та створення тестових дублерів.