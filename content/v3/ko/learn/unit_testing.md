# 단위 테스트

## 개요

Flight에서의 단위 테스트는 애플리케이션이 예상대로 작동하는지 확인하고, 버그를 조기에 발견하며, 코드베이스를 더 쉽게 유지보수할 수 있게 도와줍니다. Flight는 가장 널리 사용되는 PHP 테스트 프레임워크인 [PHPUnit](https://phpunit.de/)과 원활하게 작동하도록 설계되었습니다.

## 이해하기

단위 테스트는 애플리케이션의 작은 부분(컨트롤러나 서비스 등)을 격리된 상태에서 검사합니다. Flight에서 이는 전역 상태나 실제 외부 서비스에 의존하지 않고, 라우트, 컨트롤러, 로직이 다양한 입력에 어떻게 반응하는지 테스트하는 것을 의미합니다.

핵심 원칙:
- **구현이 아닌 동작을 테스트하세요:** 코드가 무엇을 하는지에 집중하고, 어떻게 하는지에는 집중하지 마세요.
- **전역 상태를 피하세요:** `Flight::set()`이나 `Flight::get()` 대신 의존성 주입을 사용하세요.
- **외부 서비스를 목(mock) 처리하세요:** 데이터베이스나 메일러 같은 것들은 테스트 더블로 대체하세요.
- **테스트를 빠르고 집중적으로 유지하세요:** 단위 테스트는 실제 데이터베이스나 API를 사용하지 않아야 합니다.

## 기본 사용법

### PHPUnit 설정하기

1. Composer로 PHPUnit을 설치합니다:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. 프로젝트 루트에 `tests` 디렉토리를 만듭니다.
3. `composer.json`에 테스트 스크립트를 추가합니다:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. `phpunit.xml` 파일을 만듭니다:
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

이제 `composer test` 명령으로 테스트를 실행할 수 있습니다.

### 간단한 라우트 핸들러 테스트하기

이메일을 검증하는 라우트가 있다고 가정해 보겠습니다:

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

이 컨트롤러에 대한 간단한 테스트입니다:

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

**팁:**
- `$app->request()->data`를 사용하여 POST 데이터를 시뮬레이션하세요.
- 테스트에서 `Flight::` 정적 메서드를 사용하지 말고 `$app` 인스턴스를 사용하세요.

### 테스트 가능한 컨트롤러를 위한 의존성 주입 사용하기

데이터베이스나 메일러 같은 의존성을 컨트롤러에 주입하면 테스트에서 쉽게 목(mock) 처리할 수 있습니다:

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

목(mock)을 사용한 테스트:

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

## 고급 사용법

- **목(Mocking):** PHPUnit의 내장 목(mock)이나 익명 클래스를 사용하여 의존성을 대체하세요.
- **컨트롤러 직접 테스트:** 새로운 `Engine`과 목(mock) 의존성으로 컨트롤러를 인스턴스화하세요.
- **과도한 목(mock) 사용을 피하세요:** 가능하면 실제 로직이 실행되도록 하고, 외부 서비스만 목(mock) 처리하세요.

## 참조

- [단위 테스트 가이드](/guides/unit-testing) - 단위 테스트 모범 사례에 대한 종합 가이드입니다.
- [의존성 주입 컨테이너](/learn/dependency-injection-container) - DIC를 사용하여 의존성을 관리하고 테스트 용이성을 향상시키는 방법입니다.
- [확장하기](/learn/extending) - 자신만의 헬퍼를 추가하거나 핵심 클래스를 재정의하는 방법입니다.
- [SimplePdo](/learn/simple-pdo) - 데이터베이스 상호작용을 단순화하고 테스트에서 목(mock) 처리를 쉽게 합니다.
- [요청(Requests)](/learn/requests) - Flight에서 HTTP 요청을 처리하는 방법입니다.
- [응답(Responses)](/learn/responses) - 사용자에게 응답을 보내는 방법입니다.
- [단위 테스트와 SOLID 원칙](/learn/unit-testing-and-solid-principles) - SOLID 원칙이 단위 테스트를 어떻게 개선할 수 있는지 알아보세요.

## 문제 해결

- 코드와 테스트에서 전역 상태(`Flight::set()`, `$_SESSION` 등)를 사용하지 마세요.
- 테스트가 느리다면 통합 테스트를 작성하고 있을 수 있습니다. 외부 서비스를 목(mock) 처리하여 단위 테스트를 빠르게 유지하세요.
- 테스트 설정이 복잡하다면 의존성 주입을 사용하도록 코드를 리팩터링하는 것을 고려하세요.

## 변경 이력

- v3.15.0 - 의존성 주입 및 목(mock) 처리에 대한 예제가 추가되었습니다.