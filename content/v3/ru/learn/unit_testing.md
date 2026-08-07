# Модульное тестирование

## Обзор

Модульное тестирование во Flight помогает вам убедиться, что ваше приложение ведёт себя ожидаемо, выявлять ошибки на ранних стадиях и упрощать поддержку кодовой базы. Flight разработан для бесперебойной работы с [PHPUnit](https://phpunit.de/) — самым популярным фреймворком для тестирования PHP.

## Понимание

Модульные тесты проверяют поведение небольших частей вашего приложения (таких как контроллеры или сервисы) изолированно. Во Flight это означает проверку того, как ваши маршруты, контроллеры и логика реагируют на различные входные данные — без зависимости от глобального состояния или реальных внешних сервисов.

Ключевые принципы:
- **Тестируйте поведение, а не реализацию:** Сосредоточьтесь на том, что делает ваш код, а не на том, как он это делает.
- **Избегайте глобального состояния:** Используйте внедрение зависимостей вместо `Flight::set()` или `Flight::get()`.
- **Мокайте внешние сервисы:** Заменяйте такие вещи, как базы данных или почтовые сервисы, двойниками тестов.
- **Поддерживайте тесты быстрыми и сфокусированными:** Модульные тесты не должны обращаться к реальным базам данных или API.

## Базовое использование

### Настройка PHPUnit

1. Установите PHPUnit через Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Создайте каталог `tests` в корне вашего проекта.
3. Добавьте тестовый скрипт в ваш `composer.json`:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Создайте файл `phpunit.xml`:
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

Теперь вы можете запускать тесты с помощью `composer test`.

### Тестирование простого обработчика маршрута

Предположим, у вас есть маршрут, который проверяет email:

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

Простой тест для этого контроллера:

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

**Советы:**
- Имитируйте POST-данные с помощью `$app->request()->data`.
- Избегайте использования статических методов `Flight::` в тестах — используйте экземпляр `$app`.

### Использование внедрения зависимостей для тестируемых контроллеров

Внедряйте зависимости (например, базу данных или почтовый сервис) в ваши контроллеры, чтобы их можно было легко мокать в тестах:

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

И тест с моками:

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

## Продвинутое использование

- **Мокание:** Используйте встроенные моки PHPUnit или анонимные классы для замены зависимостей.
- **Прямое тестирование контроллеров:** Создавайте экземпляры контроллеров с новым `Engine` и мокайте зависимости.
- **Избегайте чрезмерного мокания:** Позволяйте реальной логике выполняться там, где это возможно; мокайте только внешние сервисы.

## Смотрите также

- [Руководство по модульному тестированию](/guides/unit-testing) — Полное руководство по лучшим практикам модульного тестирования.
- [Контейнер внедрения зависимостей](/learn/dependency-injection-container) — Как использовать DIC для управления зависимостями и улучшения тестируемости.
- [Расширение](/learn/extending) — Как добавить собственные хелперы или переопределить основные классы.
- [SimplePdo](/learn/simple-pdo) — Упрощает взаимодействие с базой данных и упрощает мокание в тестах.
- [Запросы](/learn/requests) — Обработка HTTP-запросов во Flight.
- [Ответы](/learn/responses) — Отправка ответов пользователям.
- [Модульное тестирование и принципы SOLID](/learn/unit-testing-and-solid-principles) — Узнайте, как принципы SOLID могут улучшить ваши модульные тесты.

## Устранение неполадок

- Избегайте использования глобального состояния (`Flight::set()`, `$_SESSION` и т.д.) в вашем коде и тестах.
- Если ваши тесты выполняются медленно, возможно, вы пишете интеграционные тесты — мокайте внешние сервисы, чтобы модульные тесты оставались быстрыми.
- Если настройка тестов сложна, рассмотрите рефакторинг кода для использования внедрения зависимостей.

## История изменений

- v3.15.0 — Добавлены примеры для внедрения зависимостей и мокания.