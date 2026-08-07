# Flight PHP에서 PHPUnit을 사용한 단위 테스트

이 가이드는 [PHPUnit](https://phpunit.de/)을 사용하여 Flight PHP에서 단위 테스트를 수행하는 방법을 소개하며, 단위 테스트가 *왜* 중요한지와 실제로 어떻게 적용하는지 이해하려는 초보자를 대상으로 합니다. 단순한 계산 테스트가 아니라 이메일 전송이나 레코드 저장과 같은 애플리케이션의 *동작*을 테스트하는 데 초점을 맞춥니다. 간단한 [라우트 핸들러](/learn/routing)에서 시작하여 [의존성 주입](/learn/dependency-injection-container)(DI)과 타사 서비스 모킹을 포함한 더 복잡한 [컨트롤러](/learn/routing)로 진행합니다.

## 왜 단위 테스트를 해야 하나요?

단위 테스트는 코드가 예상대로 작동하는지 확인하여 버그가 프로덕션에 도달하기 전에 잡아냅니다. 특히 Flight의 가벼운 라우팅과 유연성으로 인해 복잡한 상호작용이 발생할 수 있으므로 단위 테스트는 매우 유용합니다. 개인 개발자든 팀이든 단위 테스트는 안전망 역할을 하며 예상 동작을 문서화하고 나중에 코드를 다시 볼 때 회귀를 방지합니다. 또한 테스트는 설계를 개선합니다. 테스트하기 어려운 코드는 종종 지나치게 복잡하거나 강하게 결합된 클래스를 나타냅니다.

단순한 예제(예: `x * y = z` 테스트)와 달리 입력 검증, 데이터 저장, 이메일 전송과 같은 트리거 등 실제 세계의 동작에 초점을 맞출 것입니다. 목표는 테스트를 접근하기 쉽고 의미 있게 만드는 것입니다.

## 일반적인 지침 원칙

1. **구현이 아닌 동작을 테스트하세요**: 결과(예: "이메일 전송됨" 또는 "레코드 저장됨")에 초점을 맞추고 내부 세부 사항은 무시하세요. 이렇게 하면 리팩터링에도 테스트가 견고해집니다.
2. **`Flight::` 사용을 중단하세요**: Flight의 정적 메서드는 매우 편리하지만 테스트를 어렵게 만듭니다. `$app = Flight::app();`의 `$app` 변수에 익숙해져야 합니다. `$app`에는 `Flight::`가 가진 모든 메서드가 동일하게 있습니다. 컨트롤러에서 `$app->route()` 또는 `$this->app->json()`을 계속 사용할 수 있습니다. 또한 실제 Flight 라우터를 `$router = $app->router()`로 사용하고 `$router->get()`, `$router->post()`, `$router->group()` 등을 사용할 수 있습니다. [라우팅](/learn/routing)을 참조하세요.
3. **테스트를 빠르게 유지하세요**: 빠른 테스트는 자주 실행하게 만듭니다. 단위 테스트에서 데이터베이스 호출과 같은 느린 작업을 피하세요. 느린 테스트가 있다면 그것은 통합 테스트를 작성하고 있다는 신호입니다. 통합 테스트는 실제 데이터베이스, 실제 HTTP 호출, 실제 이메일 전송 등을 포함할 때입니다. 통합 테스트도 필요하지만 느리고 알 수 없는 이유로 실패할 수 있는 플레이키한 특성이 있습니다.
4. **설명적인 이름을 사용하세요**: 테스트 이름은 테스트하려는 동작을 명확하게 설명해야 합니다. 이는 가독성과 유지보수성을 향상시킵니다.
5. **전역 변수를 벼락처럼 피하세요**: `$app->set()`과 `$app->get()`은 전역 상태처럼 작동하므로 모든 테스트에서 목이 필요하게 됩니다. 사용을 최소화하고 DI 또는 DI 컨테이너를 선호하세요 ([의존성 주입 컨테이너](/learn/dependency-injection-container)). `$app->map()` 메서드도 기술적으로 "전역"이며 DI를 위해 피해야 합니다. [flightphp/session](https://github.com/flightphp/session)과 같은 세션 라이브러리를 사용하여 테스트에서 세션 객체를 목으로 만들 수 있습니다. [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php)을 코드에서 직접 호출하지 마세요. 이는 코드에 전역 변수를 주입하여 테스트를 어렵게 만듭니다.
6. **의존성 주입을 사용하세요**: 컨트롤러에 의존성(예: [`PDO`](https://www.php.net/manual/en/class.pdo.php), 메일러)을 주입하여 로직을 분리하고 모킹을 단순화하세요. 너무 많은 의존성을 가진 클래스가 있다면 [SOLID 원칙](https://en.wikipedia.org/wiki/SOLID)에 따라 각각 단일 책임을 갖는 더 작은 클래스로 리팩터링하는 것을 고려하세요.
7. **타사 서비스를 모킹하세요**: 외부 호출을 피하기 위해 데이터베이스, HTTP 클라이언트(cURL), 이메일 서비스를 모킹하세요. 테스트는 한두 계층 깊이 수행하되 핵심 로직은 실제로 실행되도록 두세요. 예를 들어 앱이 문자 메시지를 보낸다면, 테스트를 실행할 때마다 실제로 문자 메시지를 보내고 싶지 않을 것입니다. 비용이 누적되고 느려지기 때문입니다. 대신 문자 메시지 서비스를 목으로 만들어 코드가 문자 메시지 서비스를 올바른 매개변수로 호출했는지만 확인하세요.
8. **높은 커버리지를 목표로 하되 완벽함은 추구하지 마세요**: 100% 라인 커버리지는 좋지만 실제로 코드의 모든 것이 제대로 테스트되고 있다는 뜻은 아닙니다([PHPUnit에서 라인/브랜치/경로 커버리지 수집](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)을 연구해 보세요). 중요한 동작(예: 사용자 등록, API 응답 및 실패 응답 캡처)을 우선시하세요.
9. **라우트에 컨트롤러를 사용하세요**: 라우트 정의에서 클로저 대신 컨트롤러를 사용하세요. `flight\Engine $app`은 기본적으로 생성자를 통해 모든 컨트롤러에 주입됩니다. 테스트에서는 `$app = new Flight\Engine()`을 사용하여 테스트 내에서 Flight를 인스턴스화하고 컨트롤러에 주입한 다음 메서드를 직접 호출하세요(예: `$controller->register()`). [Flight 확장](/learn/extending) 및 [라우팅](/learn/routing)을 참조하세요.
10. **모킹 스타일을 정하고 일관되게 유지하세요**: PHPUnit은 여러 모킹 스타일(예: prophecy, 내장 목)을 지원하며, 코드 완성, 메서드 정의 변경 시 깨짐 감지 등의 장점이 있는 익명 클래스를 사용할 수도 있습니다. 테스트 전반에 걸쳐 일관성을 유지하세요. [PHPUnit 목 객체](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles)를 참조하세요.
11. **하위 클래스에서 테스트하려는 메서드/속성에는 `protected` 가시성을 사용하세요**: 이렇게 하면 공개로 만들지 않고도 테스트 하위 클래스에서 재정의할 수 있습니다. 익명 클래스 목에서 특히 유용합니다.

## PHPUnit 설정

먼저 Composer를 사용하여 Flight PHP 프로젝트에 [PHPUnit](https://phpunit.de/)을 설치하여 쉽게 테스트할 수 있게 하세요. 자세한 내용은 [PHPUnit 시작 가이드](https://phpunit.readthedocs.io/en/12.3/installation.html)를 참조하세요.

1. 프로젝트 디렉터리에서 다음을 실행하세요:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   이 명령은 최신 PHPUnit을 개발 종속성으로 설치합니다.

2. 프로젝트 루트에 테스트 파일용 `tests` 디렉터리를 만드세요.

3. 편의를 위해 `composer.json`에 테스트 스크립트를 추가하세요:
   ```json
   // 기타 composer.json 내용
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. 루트에 `phpunit.xml` 파일을 만드세요:
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

이제 테스트가 준비되면 `composer test`를 실행하여 테스트를 실행할 수 있습니다.

## 간단한 라우트 핸들러 테스트

사용자의 이메일 입력을 검증하는 기본적인 [라우트](/learn/routing)부터 시작하겠습니다. 유효한 이메일에는 성공 메시지를, 유효하지 않은 이메일에는 오류를 반환하는 동작을 테스트합니다. 이메일 검증에는 [`filter_var`](https://www.php.net/manual/en/function.filter-var.php)를 사용합니다.

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

테스트를 위해 테스트 파일을 만드세요. 테스트 구조에 대한 자세한 내용은 [단위 테스트와 SOLID 원칙](/learn/unit-testing-and-solid-principles)을 참조하세요:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // POST 데이터 시뮬레이션
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
		$request->data->email = 'invalid-email'; // POST 데이터 시뮬레이션
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**핵심 사항**:
- `$_POST`, `$_GET` 같은 전역 변수를 사용하지 않고 요청 클래스를 사용하여 POST 데이터를 시뮬레이션합니다. 전역 변수를 사용하면 다른 테스트가 깨질 수 있으므로 항상 값을 재설정해야 해서 테스트가 더 복잡해집니다.
- 모든 컨트롤러는 기본적으로 DIC(의존성 주입 컨테이너)가 설정되지 않아도 `flight\Engine` 인스턴스가 주입됩니다. 이렇게 하면 컨트롤러를 직접 훨씬 쉽게 테스트할 수 있습니다.
- `Flight::`를 전혀 사용하지 않으므로 코드를 테스트하기가 더 쉽습니다.
- 테스트는 유효/무효 이메일에 대한 올바른 상태와 메시지라는 동작을 검증합니다.

`composer test`를 실행하여 라우트가 예상대로 작동하는지 확인하세요. Flight의 [요청](/learn/requests) 및 [응답](/learn/responses)에 대한 자세한 내용은 관련 문서를 참조하세요.

## 테스트 가능한 컨트롤러를 위한 의존성 주입 사용

더 복잡한 시나리오에서는 [의존성 주입](/learn/dependency-injection-container)(DI)을 사용하여 컨트롤러를 테스트 가능하게 만드세요. Flight의 전역 메서드(예: `Flight::set()`, `Flight::map()`, `Flight::register()`)는 전역 상태처럼 작동하여 모든 테스트에서 목이 필요하므로 피하세요. 대신 Flight의 DI 컨테이너인 [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) 또는 수동 DI를 사용하세요.

원시 PDO 대신 [`flight\database\SimplePdo`](/learn/simple-pdo)를 사용해 보겠습니다. 이 헬퍼는 목으로 만들고 단위 테스트하기 훨씬 쉬우며(더 이상 사용되지 않는 `PdoWrapper`보다 권장됩니다).

다음은 사용자를 데이터베이스에 저장하고 환영 이메일을 보내는 컨트롤러입니다:

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
			// 여기서 return을 추가하면 단위 테스트에서 실행을 중단시키는 데 도움이 됩니다.
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**핵심 사항**:
- 컨트롤러는 [`SimplePdo`](/learn/simple-pdo) 인스턴스와 `MailerInterface`(가상의 타사 이메일 서비스)에 의존합니다.
- 의존성은 생성자를 통해 주입되므로 전역 변수를 피합니다.

### 목(Mock)을 사용하여 컨트롤러 테스트

이제 `UserController`의 동작(이메일 검증, 데이터베이스 저장, 이메일 전송)을 테스트해 보겠습니다. 데이터베이스와 메일러를 목으로 만들어 컨트롤러를 격리합니다.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// 때로는 모킹 스타일을 혼합하는 것이 필요합니다.
		// 여기서는 PHPUnit 내장 목을 PDOStatement에 사용합니다.
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// 익명 클래스를 사용하여 SimplePdo를 모킹합니다.
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// 이렇게 모킹하면 실제 데이터베이스 호출이 이루어지지 않습니다.
			// PDOStatement 목을 추가로 설정하여 실패 등을 시뮬레이션할 수 있습니다.
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
			// 빈 생성자는 부모 생성자를 우회합니다.
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

		// 종료를 피하기 위해 jsonHalt를 매핑해야 합니다.
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

**핵심 사항**:
- 실제 데이터베이스 또는 이메일 호출을 피하기 위해 `SimplePdo`와 `MailerInterface`를 목으로 만듭니다.
- 테스트는 동작을 검증합니다: 유효한 이메일은 데이터베이스 삽입과 이메일 전송을 트리거하고, 유효하지 않은 이메일은 둘 다 건너뜁니다.
- 타사 의존성(예: `SimplePdo`, `MailerInterface`)만 목으로 만들고 컨트롤러의 로직은 실제로 실행되게 합니다.

### 너무 많이 모킹하기

코드를 너무 많이 모킹하지 않도록 주의하세요. `UserController`를 예로 들어 이것이 왜 나쁜 일인지 아래에서 설명하겠습니다. 해당 검증을 `isEmailValid`( `filter_var` 사용)라는 메서드로 바꾸고, 다른 새 추가 사항은 `registerUser`라는 별도의 메서드로 분리하겠습니다.

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
			// 여기서 return을 추가하면 단위 테스트에서 실행을 중단시키는 데 도움이 됩니다.
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

이제 실제로 아무것도 테스트하지 않는 과도하게 모킹된 단위 테스트를 보겠습니다:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// "쉽다"는 이유로 추가 의존성 주입을 건너뜁니다.
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// 생성자의 의존성을 우회합니다.
			public function __construct($app) {
				$this->app = $app;
			}

			// 그냥 유효하다고 강제합니다.
			protected function isEmailValid($email) {
				return true; // 항상 true를 반환하여 실제 검증을 우회합니다.
			}

			// 실제 DB 및 메일러 호출을 우회합니다.
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

훌륭합니다. 단위 테스트가 있고 통과합니다! 하지만 잠깐, `isEmailValid`나 `registerUser`의 내부 동작을 실제로 변경하면 어떻게 될까요? 모든 기능을 모킹했기 때문에 테스트는 여전히 통과할 것입니다. 그 의미를 보여드리겠습니다.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... 기타 메서드 ...

	protected function isEmailValid($email) {
		// 로직 변경
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// 이제 특정 도메인만 허용해야 합니다.
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

위의 단위 테스트를 실행하면 여전히 통과합니다! 그러나 동작을 테스트하지 않았기 때문에(일부 코드가 실제로 실행되도록 두지 않았기 때문에) 프로덕션에서 발생할 수 있는 버그를 코딩했을 가능성이 있습니다. 테스트는 새 동작을 반영하도록 수정되어야 하며, 동작이 기대와 다를 때의 반대 경우도 수정되어야 합니다.

## 전체 예제

단위 테스트가 포함된 Flight PHP 프로젝트의 전체 예제는 GitHub에서 확인할 수 있습니다: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide).
더 깊이 이해하려면 [단위 테스트와 SOLID 원칙](/learn/unit-testing-and-solid-principles)을 참조하세요.

## 흔한 함정

- **과도한 모킹**: 모든 의존성을 모킹하지 마세요. 실제 동작을 테스트하려면 일부 로직(예: 컨트롤러 검증)은 실행되도록 두세요. [단위 테스트와 SOLID 원칙](/learn/unit-testing-and-solid-principles)을 참조하세요.
- **전역 상태**: PHP 전역 변수(예: [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php))를 많이 사용하면 테스트가 취약해집니다. `Flight::` 사용도 마찬가지입니다. 의존성을 명시적으로 전달하도록 리팩터링하세요.
- **복잡한 설정**: 테스트 설정이 번거롭다면 클래스에 너무 많은 의존성이나 책임이 있어 [SOLID 원칙](/learn/unit-testing-and-solid-principles)을 위반했을 수 있습니다.

## 단위 테스트로 확장하기

단위 테스트는 더 큰 프로젝트나 몇 달 후 코드를 다시 볼 때 특히 빛을 발합니다. 동작을 문서화하고 회귀를 잡아내므로 앱을 다시 배우는 수고를 덜어줍니다. 개인 개발자는 중요한 경로(예: 사용자 가입, 결제 처리)를 테스트하세요. 팀의 경우 테스트는 기여 전반에 걸쳐 일관된 동작을 보장합니다. 프레임워크와 테스트의 이점에 대한 자세한 내용은 [프레임워크가 왜 필요한가요?](/learn/why-frameworks)를 참조하세요.

Flight PHP 문서 저장소에 자신만의 테스트 팁을 기여하세요!

_Written by [n0nag0n](https://github.com/n0nag0n) 2025_