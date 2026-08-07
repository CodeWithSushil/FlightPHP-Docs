# 구성

## 개요

Flight는 애플리케이션의 필요에 맞게 프레임워크의 다양한 측면을 구성할 수 있는 간단한 방법을 제공합니다. 일부는 기본적으로 설정되어 있지만 필요에 따라 재정의할 수 있습니다. 또한 애플리케이션 전반에 걸쳐 사용할 고유 변수를 설정할 수 있습니다.

명확하고 계층화된 구성(파일 기본값 + 환경 비밀값)은 [AI 코딩 도구](/learn/ai)에도 도움이 됩니다. 에이전트가 컨트롤러 내부에서 `$_ENV` 읽기를 임의로 만들지 않고 리터럴 값과 비밀값을 각각 한 곳에서 알 수 있기 때문입니다.

## 이해

`set` 메서드를 통해 구성 값을 설정하여 Flight의 특정 동작을 사용자 지정할 수 있습니다.

```php
Flight::set('flight.log_errors', true);
```

구조화된 앱([skeleton](https://github.com/flightphp/skeleton) 포함)에서는 일반적으로 `app/config/config.php`에서 프로젝트 설정을 로드한 다음 관련 키를 Engine에 적용합니다(예: `flight.base_url`, `flight.views.path`). 또한 모든 곳에서 전역 변수를 읽는 대신 작은 구성 객체를 컨트롤러에 주입할 수 있습니다. 이는 테스트와 `AGENTS.md`를 따르는 에이전트에 더 친숙합니다.

## 기본 사용법

### Flight 구성 옵션

다음은 사용 가능한 모든 구성 설정 목록입니다.

- **flight.base_url** `?string` - Flight가 하위 디렉터리에서 실행 중일 때 요청의 기본 URL을 재정의합니다. (기본값: null)
- **flight.case_sensitive** `bool` - URL에 대해 대소문자 구분 일치를 사용합니다. (기본값: false)
- **flight.handle_errors** `bool` - Flight가 모든 오류를 내부적으로 처리하도록 허용합니다. (기본값: true)
  - Flight가 기본 PHP 동작 대신 오류를 처리하도록 하려면 이 값을 true로 설정해야 합니다.
  - [Tracy](/awesome-plugins/tracy)를 설치한 경우 Tracy가 오류를 처리할 수 있도록 이 값을 false로 설정해야 합니다.
  - [APM](/awesome-plugins/apm) 플러그인을 설치한 경우 APM이 오류를 기록할 수 있도록 이 값을 true로 설정해야 합니다.
- **flight.log_errors** `bool` - 오류를 웹 서버의 오류 로그 파일에 기록합니다. (기본값: false)
  - [Tracy](/awesome-plugins/tracy)를 설치한 경우 Tracy는 이 구성이 아닌 Tracy 구성에 따라 오류를 기록합니다.
- **flight.debug** `bool` - 오류 발생 시 브라우저에 자세한 오류 정보(예외 메시지, 코드, 스택 추적)를 출력합니다. (기본값: false)
  - **프로덕션 환경에서는 절대 활성화하지 마세요** — 내부 애플리케이션 세부 정보가 유출됩니다. 로컬 개발 또는 스테이징 환경에서만 사용하세요.
  - `false`로 설정하면 일반적인 `500 Internal Server Error`가 대신 표시됩니다. 서버 측에서 오류를 캡처하려면 `flight.log_errors`와 함께 사용하세요.
- **flight.allow_method_override** `bool` - `X-HTTP-Method-Override` 요청 헤더 또는 POST 본문의 `_method` 필드를 통해 HTTP 메서드를 재정의할 수 있도록 허용합니다. (기본값: true)
  - HTML 폼 기반 메서드 스푸핑이 필요 없는 애플리케이션의 경우 **이 값을 `false`로 설정하는 것이 좋습니다.** 클라이언트가 일반 POST 폼을 통해 `DELETE` 또는 `PUT` 요청을 위조하는 것을 방지할 수 있습니다.
  - 자세한 내용은 [보안](/learn/security#flight-configuration-hardening)을 참조하세요.
- **flight.views.path** `string` - 뷰 템플릿 파일이 포함된 디렉터리입니다. (기본값: ./views)
- **flight.views.extension** `string` - 뷰 템플릿 파일 확장자입니다. (기본값: `.php`; 공식 skeleton은 Twig를 사용할 때 이 값을 `.twig`로 설정합니다)
- **flight.content_length** `bool` - `Content-Length` 헤더를 설정합니다. (기본값: true)
  - [Tracy](/awesome-plugins/tracy)를 사용하는 경우 Tracy가 올바르게 렌더링되도록 이 값을 false로 설정해야 합니다.
- **flight.v2.output_buffering** `bool` - 레거시 출력 버퍼링을 사용합니다. [v3로 마이그레이션](migrating-to-v3)을 참조하세요. (기본값: false)

### 로더 구성

로더에 대한 추가 구성 설정도 있습니다. 이를 통해 클래스 이름에 `_`가 포함된 클래스를 자동 로드할 수 있습니다.

```php
// 밑줄이 있는 클래스 로딩 활성화
// 기본값은 true
Loader::$v2ClassLoading = false;
```

[자동 로드](/learn/autoloading)는 네임스페이스와 일치하는 **폴더 대소문자**에 따라 달라진다는 점을 기억하세요. 특히 skeleton의 `App\` + `app/Controller/` 구조에서 그렇습니다.

### 프로젝트 구성 및 `.env` (skeleton 패턴)

Flight 코어는 `.env` 파일을 요구하지 않습니다. 많은 앱이 PHP 구성 배열만 사용합니다. 공식 skeleton은 Runway가 **리터럴** 구성을 안전하게 다시 쓸 수 있도록 하면서 비밀값이 git에 포함되지 않도록 구성을 계층화합니다:

1. **`.env` / 실제 환경** — 비밀값 및 배포 재정의(gitignore 처리됨).
2. **`app/config/config.php`** — 리터럴 PHP 배열 기본값(`config_sample.php`에서 복사). 이 파일 내에는 **`$_ENV[...]` 표현식을 사용하지 않는** 것이 좋습니다. `runway config:set` 같은 도구는 이 파일을 정적 값으로 다시 쓸 수 있으며 비밀값이 파일에 포함될 수 있습니다.
3. **부트스트랩에서 병합** — 매핑된 키의 경우 env가 우선합니다. 앱 코드는 컨트롤러에서 `$_ENV`를 사용하는 대신 구성 객체나 `$app->get()`을 읽습니다.

`config_sample.php` / `config.php`의 예시 구조(간소화됨):

```php
<?php
// 리터럴만 포함 — 비밀값은 skeleton 워크플로우에서 .env에 있어야 합니다.
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // 또는 mysql, 또는 비활성화하려면 ''
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env (skeleton)
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

이러한 분리는 [AI 친화적인 프로젝트](/learn/ai)를 위해 의도적으로 설계되었습니다. 지침은 “기본값은 `config.php`에, 비밀값은 `.env`에, Config / Engine을 주입하세요—컨트롤러에서 임의로 env 접근을 만들지 마세요”라고 말할 수 있습니다. 기존 앱은 `.env`를 완전히 무시하고 단일 구성 파일을 유지할 수 있습니다.

### 변수

Flight를 사용하면 애플리케이션 어디서나 사용할 수 있도록 변수를 저장할 수 있습니다.

```php
// 변수 저장
Flight::set('id', 123);

// 애플리케이션의 다른 곳에서
$id = Flight::get('id');
```

변수가 설정되었는지 확인하려면 다음을 수행할 수 있습니다:

```php
if (Flight::has('id')) {
  // 무언가를 수행
}
```

변수를 지우려면 다음을 수행할 수 있습니다:

```php
// id 변수를 지웁니다.
Flight::clear('id');

// 모든 변수를 지웁니다.
Flight::clear();
```

> **참고:** 변수를 설정할 수 있다고 해서 반드시 사용해야 하는 것은 아닙니다. 이 기능은 드물게 사용하세요. 그 이유는 여기에 저장된 모든 것은 전역 변수가 되기 때문입니다. 전역 변수는 애플리케이션 어디에서나 변경될 수 있어 버그를 추적하기 어렵게 만들기 때문에 좋지 않습니다. 또한 [단위 테스트](/guides/unit-testing)와 같은 것을 복잡하게 만들 수 있습니다. 컨트롤러에 필요한 서비스와 구성은 생성자 주입(skeleton + Dice 설정에서처럼)을 사용하는 것이 좋습니다.

### 오류 및 예외

모든 오류와 예외는 Flight에 포착되어 `error` 메서드로 전달됩니다. (`flight.handle_errors`가 true로 설정된 경우.)

기본 동작은 일부 오류 정보와 함께 일반적인 `HTTP 500 Internal Server Error` 응답을 보내는 것입니다.

자신의 필요에 따라 이 동작을 [재정의](/learn/extending)할 수 있습니다:

```php
Flight::map('error', function (Throwable $error) {
  // 오류 처리
  echo $error->getTraceAsString();
});
```

기본적으로 오류는 웹 서버에 기록되지 않습니다. 구성을 변경하여 활성화할 수 있습니다:

```php
Flight::set('flight.log_errors', true);
```

#### 404 찾을 수 없음

URL을 찾을 수 없으면 Flight는 `notFound` 메서드를 호출합니다. 기본 동작은 간단한 메시지와 함께 `HTTP 404 Not Found` 응답을 보내는 것입니다.

자신의 필요에 따라 이 동작을 [재정의](/learn/extending)할 수 있습니다:

```php
Flight::map('notFound', function () {
  // 찾을 수 없음 처리
});
```

## 같이 보기

- [설치](/install) - Skeleton 구성, `.env`, 부트스트랩 구조.
- [자동 로드](/learn/autoloading) - 네임스페이스와 폴더 대소문자.
- [Flight 확장](/learn/extending) - Flight의 핵심 기능을 확장하고 사용자 지정하는 방법.
- [단위 테스트](/guides/unit-testing) - Flight 애플리케이션에 대한 단위 테스트를 작성하는 방법.
- [AI 및 개발자 경험](/learn/ai) - `AGENTS.md`와 일관된 프로젝트 지침.
- [Tracy](/awesome-plugins/tracy) - 고급 오류 처리 및 디버깅을 위한 플러그인.
- [Tracy 확장 기능](/awesome-plugins/tracy_extensions) - Tracy를 Flight와 통합하기 위한 확장 기능.
- [APM](/awesome-plugins/apm) - 애플리케이션 성능 모니터링 및 오류 추적을 위한 플러그인.
- [보안](/learn/security) - 보안 강화 플래그 및 비밀값 처리.

## 문제 해결

- 구성의 모든 값을 확인하는 데 문제가 있는 경우 `var_dump(Flight::get());`을 실행할 수 있습니다.
- Runway 또는 배포 도구가 `config.php`를 다시 썼다면 비밀값이 커밋되지 않았는지 확인하세요. skeleton 패턴을 사용할 때는 비밀값을 `.env` 또는 실제 환경에 유지하세요.

## 변경 로그

- 문서 – skeleton 스타일 구성 / `.env` 계층화 및 새 프로젝트의 Twig 뷰 확장자 기본값을 문서화.
- v3.18.1 - `flight.debug` 및 `flight.allow_method_override` 구성 옵션 추가.
- v3.5.0 - 레거시 출력 버퍼링 동작을 지원하는 `flight.v2.output_buffering` 구성 추가.
- v2.0 - 핵심 구성 추가.