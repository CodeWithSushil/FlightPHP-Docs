# 의존성 주입 컨테이너

## 개요

의존성 주입 컨테이너(DIC)는 애플리케이션의 의존성을 관리할 수 있게 해주는 강력한 기능입니다. 또한 Flight가 [AI 코딩 도구](/learn/ai) 및 단위 테스트와 잘 연동되는 가장 큰 이유 중 하나입니다. 컨트롤러가 전역 변수에 의존하는 대신 생성자에서 필요한 것을 주입받기 때문입니다.

## 이해

의존성 주입(DI)은 현대 PHP 프레임워크의 핵심 개념이며 객체의 생성과 구성을 관리하는 데 사용됩니다. DIC 라이브러리의 예로는 [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/), [PHP-DI](http://php-di.org/), [league/container](https://container.thephpleague.com/) 등이 있습니다.

DIC는 클래스를 중앙 집중식으로 생성하고 관리하는 멋진 방법입니다. 동일한 객체를 여러 클래스(컨트롤러, 미들웨어, 커맨드 등)에 전달해야 할 때 유용합니다.

공식 [flightphp/skeleton](https://github.com/flightphp/skeleton)은 `app/config/services.php`에 **Dice**를 연결하고, 공유 `flight\Engine` 인스턴스를 대체하며, `[App\Controller\HomeController::class, 'index']` 같은 라우트 대상을 처리합니다. 새 프로젝트에서는 사람과 에이전트가 동일한 위치를 편집할 수 있도록 이 패턴을 선호하세요.

## 기본 사용법

기존 방식은 다음과 같을 수 있습니다.

```php

require 'vendor/autoload.php';

// 데이터베이스에서 사용자를 관리하는 클래스
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

// routes.php 파일에서

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// 다른 UserController 라우트들...

Flight::start();
```

위 코드를 보면 새로운 `PDO` 객체를 생성하여 `UserController` 클래스에 전달하고 있습니다. 이는 작은 애플리케이션에는 괜찮지만, 애플리케이션이 커질수록 동일한 `PDO` 객체를 여러 곳에서 생성하거나 전달하게 되는 상황이 발생합니다. 이때 DIC가 유용하게 사용됩니다.

다음은 DIC를 사용한 동일한 예제입니다 (Dice 사용):

```php

require 'vendor/autoload.php';

// 위와 같은 클래스입니다. 변경된 것이 없습니다.
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

// 새 컨테이너 생성
$container = new \Dice\Dice;

// 컨테이너에 PDO 객체 생성 방법을 알려주는 규칙 추가
// 아래처럼 반드시 자기 자신에게 다시 할당하는 것을 잊지 마세요!
$container = $container->addRule('PDO', [
	// shared는 매번 동일한 객체가 반환된다는 의미입니다.
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// 이것은 Flight가 사용할 수 있도록 컨테이너 핸들러를 등록합니다.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// 이제 컨테이너를 사용하여 UserController를 생성할 수 있습니다.
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

예제에 불필요한 코드가 많이 추가되었다고 생각할 수도 있습니다. 마법은 다른 컨트롤러가 `PDO` 객체를 필요로 할 때 드러납니다.

```php

// 모든 컨트롤러의 생성자가 PDO 객체를 필요로 한다면
// 아래의 각 라우트는 자동으로 PDO 객체를 주입받습니다!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

DIC를 활용하는 추가적인 이점은 단위 테스트가 훨씬 쉬워진다는 것입니다. 목(mock) 객체를 만들어 클래스에 전달할 수 있습니다. 이는 애플리케이션 테스트를 작성할 때 큰 이점이며, AI 어시스턴트가 컨트롤러를 생성할 때 생성자 주입은 따를 수 있는 명확하고 일관된 패턴을 제공합니다 ([단위 테스트 가이드](/guides/unit-testing)).

### 중앙 집중식 DIC 핸들러 만들기

[앱 확장](/learn/extending)을 통해 서비스 파일에 중앙 집중식 DIC 핸들러를 만들 수 있습니다. 예시는 다음과 같습니다.

```php
// services.php

// 새 컨테이너 생성
$container = new \Dice\Dice;
// 아래처럼 반드시 자기 자신에게 다시 할당하는 것을 잊지 마세요!
$container = $container->addRule('PDO', [
	// shared는 매번 동일한 객체가 반환된다는 의미입니다.
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// 이제 모든 객체를 생성할 수 있는 매핑 가능한 메서드를 만들 수 있습니다.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// 컨트롤러/미들웨어에 사용할 수 있도록 Flight가 인식하는 컨테이너 핸들러를 등록합니다.
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// 생성자에서 PDO 객체를 받는 다음 샘플 클래스가 있다고 가정해 봅시다.
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// 이메일을 보내는 코드
	}
}

// 마지막으로 의존성 주입을 사용하여 객체를 생성할 수 있습니다.
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight에는 의존성 주입을 처리하는 데 사용할 수 있는 간단한 PSR-11 호환 컨테이너를 제공하는 플러그인이 있습니다. 사용 방법에 대한 간단한 예제는 다음과 같습니다.

```php

// 예를 들어 index.php
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
	// 정상적으로 출력될 것입니다!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### `flightphp/container` 고급 사용법

의존성을 재귀적으로 해결할 수도 있습니다. 예시는 다음과 같습니다.

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
    // 구현 ...
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

나만의 DIC 핸들러를 만들 수도 있습니다. PSR-11이 아닌 사용자 정의 컨테이너를 사용하려는 경우 유용합니다 (Dice). 이 작업을 수행하는 방법은 [기본 사용법](#basic-usage) 섹션을 참조하세요.

또한 Flight를 사용할 때 삶을 더 편하게 만들어 주는 몇 가지 유용한 기본값이 있습니다.

#### Engine 인스턴스 (`$app` 주입에 필요)

컨트롤러나 미들웨어에서 `flight\Engine`을 타입 힌트로 사용하는 경우 **Dice가 새 Engine을 생성해서는 안 됩니다**. 부트스트랩에서 동일한 인스턴스를 대체하세요. 이것이 공식 스켈레톤이 하는 방식이며, AI 생성 컨트롤러를 위해 `AGENTS.md`가 기대하는 패턴입니다.

```php
// 부트스트랩 또는 services.php의 어딘가
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // 또는 $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// 중요: 부트스트랩된 Engine을 재사용하세요. Dice가 `new Engine()`을 만들지 않도록 하세요.
		Engine::class => $app,
		// 새 코드에는 SimplePdo를 선호하세요.
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// 라우트가 아닌 코드를 위한 선택적 헬퍼
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php (스켈레톤 레이아웃 — 폴더 대소문자는 네임스페이스와 일치)
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
		// 앱 계층에는 Flight:: 파사드가 없습니다 — 테스트가 쉽고 AI 도구에 더 명확합니다.
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

만약 `Engine` 대체를 생략하면 Dice가 두 번째 Engine을 생성할 수 있으며, 컨트롤러가 부트스트랩의 라우트, 설정, 매핑된 Twig `render`를 공유하지 못할 수 있습니다.

#### 다른 공유 서비스 추가 (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// services.php에서 $db, $config, $twig를 생성한 후:
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

그러면 컨트롤러는 생성자에서 `SimplePdo $db` (또는 설정 타입)를 받을 수 있으며 `Flight::db()`를 호출할 필요가 없습니다. 이는 [단위 테스트](/guides/unit-testing) 지침 및 스켈레톤 하우스 스타일과 일치합니다.

#### 다른 클래스 추가

컨테이너에 추가하려는 다른 클래스가 있다면, Dice에서는 컨테이너가 자동으로 해결해 주기 때문에 쉽습니다. 예시는 다음과 같습니다.

```php

$container = new \Dice\Dice;
// 클래스에 의존성을 주입할 필요가 없다면
// 아무것도 정의할 필요가 없습니다!
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

Flight는 모든 PSR-11 호환 컨테이너를 사용할 수 있습니다. 즉, PSR-11 인터페이스를 구현하는 모든 컨테이너를 사용할 수 있습니다. 다음은 League의 PSR-11 컨테이너를 사용하는 예제입니다.

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// 위와 동일한 UserController 아이디어이지만, 원시 PDO 대신 SimplePdo를 타입 힌트로 사용합니다.

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

이전 Dice 예제보다 조금 더 장황할 수 있지만, 여전히 동일한 이점으로 작업을 완수합니다!

## 참고 항목

- [설치](/install) - 스켈레톤 레이아웃 및 `services.php` 위치.
- [자동 로딩](/learn/autoloading) - `App\` 네임스페이스 및 폴더 **대소문자**.
- [Flight 확장](/learn/extending) - 프레임워크를 확장하여 자신만의 클래스에 의존성 주입을 추가하는 방법을 알아보세요.
- [구성](/learn/configuration) - 애플리케이션에 맞게 Flight를 구성하는 방법을 알아보세요.
- [라우팅](/learn/routing) - 애플리케이션의 라우트를 정의하는 방법과 의존성 주입이 컨트롤러와 함께 작동하는 방식을 알아보세요.
- [미들웨어](/learn/middleware) - 애플리케이션용 미들웨어를 만드는 방법과 의존성 주입이 미들웨어와 함께 작동하는 방식을 알아보세요.
- [단위 테스트](/guides/unit-testing) - 생성자 주입이 `Flight::` 전역 변수보다 나은 이유.
- [AI 및 개발자 경험](/learn/ai) - 인간과 에이전트를 위한 하나의 DI 패턴.
- [SimplePdo](/learn/simple-pdo) - 주입에 선호되는 데이터베이스 헬퍼.

## 문제 해결

- 컨테이너에 문제가 있는 경우, 컨테이너에 올바른 클래스 이름을 전달하고 있는지 확인하세요.
- `Engine`을 타입 힌트로 사용하지만 "빈" 앱을 받는 컨트롤러: **Engine 대체**를 추가하세요 (위 참조). Dice는 두 번째 Engine을 `new`로 생성해서는 안 됩니다.
- `App\Controller\…`에 대한 클래스를 찾을 수 없는 경우: `app/Controller/` 아래의 폴더 대소문자를 확인하세요 — [자동 로딩](/learn/autoloading) 참조.
- 핸들러는 `registerContainerHandler`에서 생성된 객체를 **반환**해야 합니다 (`return` 없이 `Flight::make()`를 호출하지 마세요).

## 변경 로그

- 문서 – AI 친화적인 프로젝트를 위한 스켈레톤 Dice + Engine 대체, SimplePdo, `App\Controller` 레이아웃 문서화.
- v3.7.0 - Flight에 DIC 핸들러를 등록하는 기능이 추가되었습니다.