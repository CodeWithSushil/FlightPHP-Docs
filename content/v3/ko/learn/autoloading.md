# 오토로딩

## 개요

오토로딩은 PHP에서 클래스를 로드할 디렉터리(하나 또는 여러 개)를 지정하는 개념입니다. `require`나 `include`를 사용해 클래스를 로드하는 것보다 훨씬 유용합니다. 또한 Composer 패키지를 사용하기 위한 필수 조건이기도 합니다.

오토로딩을 올바르게 설정하는 것은 [AI 지원 개발](/learn/ai)에도 중요합니다. 에이전트는 네임스페이스가 가리키는 위치에 파일을 배치하기 때문입니다. 폴더 **대소문자**와 네임스페이스의 대소문자가 일치하지 않으면, 대소문자를 구분하지 않는 Mac 디스크에서는 "잘 작동"했더라도 Linux에서는 클래스를 찾을 수 없는 오류가 발생할 수 있습니다.

## 이해하기

기본적으로 모든 `Flight` 클래스는 Composer 덕분에 자동으로 오토로딩됩니다. **여러분의** 애플리케이션 클래스의 경우 일반적으로 두 가지 방법이 있습니다.

1. **Composer PSR-4** ([공식 스켈레톤](https://github.com/flightphp/skeleton)이 사용하는 방식): `composer.json`에서 네임스페이스 접두사를 디렉터리에 매핑한 다음 `composer dump-autoload`를 실행합니다.
2. **`Flight::path()`**: Flight의 로더에 디렉터리를 지정합니다. (간단한 앱이나 앱 코드에 Composer를 사용하지 않을 때 유용합니다.)

오토로더를 사용하면 코드가 훨씬 간결해집니다. 모든 파일 상단에 긴 `include` / `require` 목록을 작성하는 대신, 클래스를 처음 사용할 때 로드됩니다.

### 대소문자 구분 (두 번 읽어보세요)

**네임스페이스는 디렉터리 구조 및 해당 디렉터리의 대소문자와 일치해야 합니다.**

| 동작함 | Linux에서 깨짐 |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` 인데 폴더가 `app/controllers/` 인 경우 |
| `app\controllers\MyController` → `app/controllers/MyController.php` | `App\`과 소문자 `controllers`를 혼용한 경우 |

PHP 네임스페이스는 일부 상황에서 대소문자를 구분하지 않지만, **Composer와 파일 시스템은 그렇지 않습니다.** 공식 스켈레톤은 다음을 표준으로 사용합니다.

- Composer: `"App\\": "app/"`
- 폴더: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase) – `controllers` / `middlewares`가 아님

이전 문서와 커뮤니티 예제에서는 소문자 `app\controllers`를 사용하기도 했습니다. 폴더가 소문자라면 여전히 작동하지만, **새 스켈레톤 프로젝트는 `App\` + PascalCase 폴더를 사용합니다.** 프로젝트마다 하나의 규칙을 선택하고 일관되게 유지하여 사람과 AI 도구가 두 번째 레이아웃을 만들지 않도록 하세요.

## 스켈레톤 (새 프로젝트에 권장)

`composer create-project flightphp/skeleton` 실행 후에는 앱 코드가 Composer를 통해 오토로딩됩니다. `App\` 클래스에 대해 `Flight::path()`가 필요하지 않습니다.

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Dice가 컨테이너를 통해 App\Controller\…을 해석합니다.
$router->get('/', [HomeController::class, 'index']);
```

전체 트리는 [설치](/install)를 참조하고, `AGENTS.md`가 코딩 어시스턴트를 위해 이 레이아웃을 문서화하는 방법은 [AI 및 개발자 경험](/learn/ai)을 참조하세요.

## 기본 사용법 (`Flight::path()`)

다음과 같은 디렉터리 트리가 있다고 가정해 봅시다.

```text
# 예제 경로
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - 이 프로젝트의 컨트롤러를 포함합니다.
│   ├── translations
│   ├── UTILS - 이 애플리케이션만을 위한 클래스를 포함합니다. (나중에 예시를 위해 의도적으로 모두 대문자입니다.)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

이것이 일반적인 앱 트리와 유사하다는 것을 눈치채셨을 것입니다. (문서 사이트 자체도 구조화된 레이아웃을 사용합니다.) 여기서 소문자 `controllers`는 유효한 *선택*입니다. 다만 스켈레톤의 현재 기본값은 아닙니다.

각 디렉터리를 다음과 같이 지정할 수 있습니다.

```php

/**
 * public/index.php
 */

// 오토로더에 경로 추가
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// 네임스페이스 불필요

// 오토로딩되는 모든 클래스는 Pascal Case(각 단어의 첫 글자를 대문자로, 공백 없음)를 권장합니다.
class MyController {

	public function index() {
		// do something
	}
}
```

## `Flight::path()`와 네임스페이스

네임스페이스가 있다면 구현이 매우 쉬워집니다. `Flight::path()` 메서드를 사용하여 애플리케이션의 루트 디렉터리(문서 루트나 `public/` 폴더가 아님)를 지정해야 합니다.

```php

/**
 * public/index.php
 */

// 오토로더에 경로 추가
Flight::path(__DIR__.'/../');
```

이제 컨트롤러는 다음과 같을 수 있습니다. 아래 예제를 보되, 중요한 정보가 담긴 주석에 주의하세요.

```php
/**
 * app/controllers/MyController.php
 */

// 네임스페이스 필수
// 네임스페이스는 디렉터리 구조와 동일해야 합니다.
// 네임스페이스는 디렉터리 구조와 대소문자가 일치해야 합니다.
// 네임스페이스와 디렉터리에는 밑줄을 사용할 수 없습니다. (Loader::setV2ClassLoading(false)를 설정한 경우는 제외)
namespace app\controllers;

// 오토로딩되는 모든 클래스는 Pascal Case(각 단어의 첫 글자를 대문자로, 공백 없음)를 권장합니다.
// 3.7.2부터는 Loader::setV2ClassLoading(false); 를 실행하여 클래스 이름에 Pascal_Snake_Case를 사용할 수 있습니다.
class MyController {

	public function index() {
		// do something
	}
}
```

그리고 utils 디렉터리의 클래스를 오토로딩하려면 기본적으로 동일한 방식으로 하면 됩니다.

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// 네임스페이스는 디렉터리 구조 및 대소문자와 일치해야 합니다. (위 파일 트리에서처럼 UTILS 디렉터리는 모두 대문자임을 주의하세요.)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// do something
	}
}
```

### 스켈레톤 스타일 네임스페이스 (동일한 규칙, 다른 대소문자)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

규칙은 변하지 않았습니다. 단지 스켈레톤이 선택한 폴더/네임스페이스 대소문자가 다를 뿐입니다. **폴더가 어떤 대소문자를 사용하든 `namespace` 줄은 동일하게 맞춰야 합니다.**

## 클래스 이름의 밑줄

3.7.2부터는 `Loader::setV2ClassLoading(false);`를 실행하여 클래스 이름에 Pascal_Snake_Case를 사용할 수 있습니다.
이렇게 하면 클래스 이름에 밑줄을 사용할 수 있습니다.
권장되지는 않지만, 필요한 사람들을 위해 제공됩니다.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// 오토로더에 경로 추가
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// 네임스페이스 불필요

class My_Controller {

	public function index() {
		// do something
	}
}
```

## 같이 보기
- [설치](/install) - 스켈레톤 트리 및 새 프로젝트의 `App\` 기본값.
- [라우팅](/learn/routing) - 컨트롤러에 경로를 매핑하고 뷰를 렌더링하는 방법.
- [의존성 주입](/learn/dependency-injection-container) - 컨트롤러가 `Engine` 및 서비스를 얻는 방법.
- [AI 및 개발자 경험](/learn/ai) - `AGENTS.md`를 통해 에이전트를 여러분의 레이아웃에 맞추는 방법.
- [프레임워크를 사용하는 이유?](/learn/why-frameworks) - Flight 같은 프레임워크 사용의 이점 이해.

## 문제 해결
- 네임스페이스 클래스를 찾을 수 없는 이유를 모르겠다면, `Flight::path()` 사용 시 **프로젝트 루트**(또는 네임스페이스에 맞는 올바른 기준 경로)를 가리키는지 확인하세요. 네임스페이스에 반영하는 것을 잊은 중첩 폴더만 가리키지 마세요.
- Composer PSR-4를 사용하는 경우 `composer.json` 매핑을 변경한 후 `composer dump-autoload`를 실행하세요.
- Linux CI 또는 프로덕션 환경에서 폴더 대소문자 오류는 매우 흔한 "내 컴퓨터에서는 작동하는데" 실패 원인입니다.

### 클래스를 찾을 수 없음 (오토로딩이 작동하지 않음)

여기에는 몇 가지 이유가 있을 수 있습니다. 아래에 몇 가지 예시가 있습니다.

#### 잘못된 파일 이름
가장 흔한 원인은 클래스 이름이 파일 이름과 일치하지 않는 경우입니다.

클래스 이름이 `MyClass`라면 파일 이름은 `MyClass.php`여야 합니다. 클래스 이름이 `MyClass`인데 파일 이름이 `myclass.php`라면 오토로더가 해당 클래스를 찾을 수 없습니다.

#### 잘못된 네임스페이스 또는 폴더 대소문자
네임스페이스를 사용하는 경우, 네임스페이스는 디렉터리 구조와 **대소문자까지 포함하여** 일치해야 합니다.

```php
// ...code...

// MyController가 app/Controller (스켈레톤)에 있고 App\Controller 네임스페이스를 사용한다면
// 아래는 작동하지 않습니다:
Flight::route('/hello', 'MyController->hello');

// 스켈레톤 스타일:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// 이전 소문자 레이아웃 (폴더가 실제로 app/controllers인 경우에만):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// 또는 완전히 자격을 갖춘 이름:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()`가 정의되지 않음 (Composer를 사용하지 않는 앱 코드)

애플리케이션 클래스에 Composer 대신 `Flight::path()`를 사용한다면, 해당 클래스를 참조하는 라우트보다 먼저(보통 부트스트랩 시작 부분이나 `public/index.php`에서) 경로를 정의하세요.

```php
// 오토로더에 경로 추가 (네임스페이스 앱의 경우 프로젝트 루트)
Flight::path(__DIR__.'/../');
```

공식 스켈레톤은 `App\`에 대해 주로 **Composer PSR-4**를 사용하므로, 컨트롤러와 모델에 `Flight::path()`가 필요하지 않은 경우가 대부분입니다.

## 변경 로그
- 문서 – 스켈레톤 `App\` + PascalCase 폴더 및 사람과 AI 도구를 위한 대소문자 구분 주의 사항 문서화.
- v3.7.2 - `Loader::setV2ClassLoading(false);`를 실행하여 클래스 이름에 Pascal_Snake_Case를 사용할 수 있습니다.
- v2.0 - 오토로딩 기능 추가.