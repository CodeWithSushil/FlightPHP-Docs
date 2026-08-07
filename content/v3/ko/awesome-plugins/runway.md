# Runway

Runway은 Flight 애플리케이션을 관리하는 데 도움이 되는 CLI 애플리케이션입니다. 컨트롤러를 생성하고, 모든 라우트를 표시하고, AI 설정 도우미, 마이그레이션(스켈레톤 내) 등을 실행할 수 있습니다. 우수한 [adhocore/php-cli](https://github.com/adhocore/php-cli) 라이브러리를 기반으로 합니다.

코드를 보려면 [여기](https://github.com/flightphp/runway)를 클릭하세요.

스캐폴딩 명령은 [공식 스켈레톤](https://github.com/flightphp/skeleton)과 의도적으로 정렬되어 있어 [AI 코딩 도구](/learn/ai)와 인간이 매번 동일한 경로, 네임스페이스 및 생성자 주입 스타일을 얻을 수 있습니다.

## 설치

composer로 설치합니다.

```bash
composer require flightphp/runway
```

스켈레톤은 이미 Runway에 의존하고 있습니다. 프로젝트 루트에서 `php runway`를 사용하세요.

## 기본 구성

Runway를 처음 실행하면 `'runway'` 키를 통해 `app/config/config.php`에서 `runway` 구성을 찾으려고 시도합니다.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// 선택 사항; 스켈레톤은 공개 엔트리에 index_root도 사용합니다
		'index_root' => 'public/index.php',
    ],
];
```

> **참고** - **v1.2.0**부터 `.runway-config.json`은 `app/config/config.php`로 대체되었습니다. 이전 프로젝트를 업그레이드할 때 `php runway config:migrate`로 마이그레이션하세요. 스켈레톤은 호환성을 위해 create-project 시 작은 `.runway-config.json`을 계속 작성할 수 있습니다. 앞으로는 `config.php`의 `runway` 키를 선호하세요.

### 프로젝트 루트 감지

Runway는 하위 디렉토리에서 실행하더라도 프로젝트의 루트를 감지할 수 있을 만큼 스마트합니다. `composer.json`, `.git` 또는 `app/config/config.php`와 같은 지표를 찾아 프로젝트 루트가 어디인지 결정합니다. 이는 프로젝트의 어디에서든 Runway 명령을 실행할 수 있음을 의미합니다!

## 사용법

Runway에는 Flight 애플리케이션을 관리하는 데 사용할 수 있는 여러 명령이 있습니다. Runway를 사용하는 두 가지 쉬운 방법이 있습니다.

1. 스켈레톤 프로젝트를 사용하는 경우 프로젝트 루트에서 `php runway [명령]`을 실행할 수 있습니다.
1. composer를 통해 설치된 패키지로 Runway를 사용하는 경우 프로젝트 루트에서 `vendor/bin/runway [명령]`을 실행할 수 있습니다.

### 명령 목록

`php runway` 명령을 실행하여 사용 가능한 모든 명령 목록을 볼 수 있습니다.

```bash
php runway
```

설치에 실제로 나타나는 명령에만 의존하세요 (핵심 Runway 명령 대 스켈레톤의 `migrate`와 같은 프로젝트별 명령).

### 명령 도움말

모든 명령에 대해 `--help` 플래그를 전달하여 명령 사용 방법에 대한 자세한 정보를 얻을 수 있습니다.

```bash
php runway routes --help
php runway make:controller --help
```

몇 가지 예는 다음과 같습니다:

### 컨트롤러 생성

`make:controller`는 공식 스켈레톤 레이아웃과 일치하는 컨트롤러를 스캐폴딩합니다:

| | |
|--|--|
| **경로** | `app/Controller/{Name}.php` |
| **네임스페이스** | `App\Controller` |
| **스타일** | `flight\Engine`의 생성자 주입 (클래스 본문에 `Flight::` 없음) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

기대할 수 있는 형태의 예시(간소화됨):

```php
<?php

declare(strict_types=1);

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
		// 예: $this->app->render('…', […]);
	}
}
```

Dice가 컨트롤러를 빌드할 수 있도록 클래스 호출 가능으로 등록하세요:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**이 레이아웃이 왜 중요한가요?** 폴더 **대소문자**는 Linux에서 Composer PSR-4를 위해 네임스페이스와 일치해야 합니다 (`Controller`가 `controllers`가 아님)—[자동 로딩](/learn/autoloading) 참조. 동일한 경로는 루트 및 범위가 지정된 `AGENTS.md` 파일이 AI 도구에 사용하도록 지시하는 것이므로 생성된 컨트롤러와 손으로 작성된 컨트롤러가 동일하게 유지됩니다.

> 이전 문서와 커뮤니티 프로젝트에서는 때때로 `app/controllers/` 및 `app\controllers`를 사용했습니다. *귀하의* 트리가 여전히 소문자 폴더를 사용하는 경우 이는 유효합니다. **새 스켈레톤 프로젝트 및 현재 `make:controller` 출력은 `app/Controller/` + `App\Controller`을 사용합니다.**

### Active Record 모델 생성

먼저 [Active Record](/awesome-plugins/active-record) 플러그인을 설치했는지 확인하세요.

```bash
php runway make:record users
```

공식 스켈레톤에서 모델은 **`app/Model/`** 아래에 **`App\Model`** 네임스페이스로 존재하며, DB 연결은 **[SimplePdo](/learn/simple-pdo)**입니다 (주입하거나 ActiveRecord 생성자에 전달). 생성된 파일 이름/네임스페이스는 Runway의 현재 기본값과 `runway` 구성을 따릅니다—새 모델을 `App\Model`과 정렬하여 [자동 로딩](/learn/autoloading) 및 `AGENTS.md`와 일치하도록 하세요.

스켈레톤 posts 데모와 일치하는 모델의 예:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

이전 생성기가 여전히 `app/records` / `app\records`를 내보내는 경우 레거시 앱에서 해당 규칙을 유지하거나 파일을 `app/Model/`로 이동하고 네임스페이스를 폴더 대소문자와 일치하도록 업데이트할 수 있습니다.

### 마이그레이션 (스켈레톤)

공식 스켈레톤은 다음과 같은 프로젝트 명령을 제공합니다 (`app/commands/`에서 발견됨):

```bash
php runway migrate
```

마이그레이션은 `migrations/` 아래의 SQL 파일입니다 (예: SQLite용 `YYYYMMDDHHMMSS_description.sql`, MySQL용 `…_description.mysql.sql`). 데이터베이스 드라이버 구성 / env에서 선택됩니다. 정확한 플래그 및 동작은 해당 프로젝트 명령으로 정의됩니다—앱에서 `php runway migrate --help`를 실행하세요.

### AI 도우미

Runway는 [AI 및 개발자 경험](/learn/ai)과 함께 사용되는 AI 지향 명령을 노출합니다:

```bash
php runway ai:init
php runway ai:generate-instructions
```

이러한 명령은 LLM 자격 증명을 저장하고 프로젝트 지침(주로 **`AGENTS.md`**)을 생성합니다. 스켈레톤에서 `AGENTS.md`(및 `app/` 아래의 범위가 지정된 복사본)와 **`SECURITY.md`**를 에이전트의 진실 소스로 취급하세요.

### 모든 라우트 표시

이것은 Flight에 현재 등록된 모든 라우트를 표시합니다.

```bash
php runway routes
```

특정 라우트만 보려는 경우 플래그를 전달하여 라우트를 필터링할 수 있습니다.

```bash
# GET 라우트만 표시
php runway routes --get

# POST 라우트만 표시
php runway routes --post

# 등등.
```

## Runway에 사용자 정의 명령 추가

Flight용 패키지를 만들거나 프로젝트에 사용자 정의 명령을 추가하려는 경우 프로젝트/패키지용으로 `src/commands/`, `flight/commands/`, `app/commands/` 또는 `commands/` 디렉토리를 만들어 수행할 수 있습니다. 추가 사용자 정의가 필요한 경우 아래 구성 섹션을 참조하세요.

스켈레톤에서 프로젝트 명령은 **`app/commands/`**에 **`App\Command`** 네임스페이스로 존재합니다. Runway는 경로로 이를 발견합니다. 해당 폴더를 프로젝트가 이미 하는 Composer 클래스맵/PSR-4와 동기화된 상태로 유지하세요.

명령을 만들려면 `AbstractBaseCommand` 클래스를 확장하고 최소한 `__construct` 메서드와 `execute` 메서드를 구현하세요.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * 생성자
     *
     * @param array<string,mixed> $config app/config/config.php의 구성
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', '문서를 위한 예제 생성', $config);
        $this->argument('<funny-gif>', '재미있는 gif의 이름');
    }

	/**
     * 함수 실행
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('예제 생성 중...');

		// 여기서 무언가를 수행

		$io->ok('예제 생성 완료!');
	}
}
```

Flight 애플리케이션에 사용자 정의 명령을 구축하는 방법에 대한 자세한 정보는 [adhocore/php-cli 문서](https://github.com/adhocore/php-cli)를 참조하세요!

## 구성 관리

v1.2.0부터 구성이 `app/config/config.php`로 이동했으므로 구성을 관리하는 몇 가지 도우미 명령이 있습니다.

> **스켈레톤 팁:** `config.php`를 **리터럴** PHP 값으로 유지하세요. 비밀은 `.env`에 속합니다. `config.php` 내부에서 `$_ENV[...]` 표현식을 피하세요—`config:set`는 해당 파일을 정적 데이터로 다시 작성하며 파일에 비밀을 구울 수 있습니다. [구성](/learn/configuration) 참조.

### 이전 구성 마이그레이션

이전 `.runway-config.json` 파일이 있는 경우 다음 명령으로 `app/config/config.php`로 쉽게 마이그레이션할 수 있습니다:

```bash
php runway config:migrate
```

### 구성 값 설정

`config:set` 명령을 사용하여 구성 값을 설정할 수 있습니다. 파일을 열지 않고 구성 값을 업데이트하려는 경우 유용합니다.

```bash
php runway config:set app_root "app/"
```

### 구성 값 가져오기

`config:get` 명령을 사용하여 구성 값을 가져올 수 있습니다.

```bash
php runway config:get app_root
```

## 모든 Runway 구성

Runway의 구성을 사용자 정의해야 하는 경우 `app/config/config.php`에서 이러한 값을 설정할 수 있습니다. 설정할 수 있는 몇 가지 추가 구성은 다음과 같습니다:

```php
<?php
// app/config/config.php
return [
    // ... 기타 구성 값 ...

    'runway' => [
        // 애플리케이션 디렉토리가 있는 위치입니다
        'app_root' => 'app/',

        // 루트 인덱스 파일이 있는 디렉토리입니다
        'index_root' => 'public/',

        // 다른 프로젝트 루트의 경로입니다
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // 기본 경로는 구성할 필요가 없을 가능성이 높지만 원하는 경우 여기에 있습니다
        'base_paths' => [
            '/includes/libs/vendor', // 공급업체 디렉토리나 다른 것에 대해 정말 고유한 경로가 있는 경우
        ],

        // 최종 경로는 명령 파일을 검색할 프로젝트 내 위치입니다
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // 전체 경로를 추가하려면 바로 추가하세요 (프로젝트 루트를 기준으로 절대 또는 상대)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### 구성 접근

구성 값을 효과적으로 접근해야 하는 경우 `__construct` 메서드 또는 `app()` 메서드를 통해 접근할 수 있습니다. `app/config/services.php` 파일이 있는 경우 해당 서비스도 명령에서 사용할 수 있다는 점도 중요합니다.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // 구성 접근
    $app_root = $this->config['runway']['app_root'];
    
    // 아마도 데이터베이스 연결과 같은 서비스 접근
    $database = $this->config['database']
    
    // ...
}
```

## AI 도우미 래퍼

Runway에는 AI가 명령을 생성하는 데 더 쉽게 만드는 몇 가지 도우미 래퍼가 있습니다. Symfony Console과 유사한 방식으로 `addOption`과 `addArgument`를 사용할 수 있습니다. 이는 AI 도구를 사용하여 명령을 생성하는 경우 유용합니다.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', '문서를 위한 예제 생성', $config);
    
    // 모드 인수는 null 가능하며 완전히 선택 사항으로 기본 설정됩니다
    $this->addOption('name', '예제의 이름', null);
}
```

## 참고 항목

- [설치](/install) - 스켈레톤 트리 및 create-project 기본값
- [자동 로딩](/learn/autoloading) - `App\` 및 폴더 대소문자
- [의존성 주입](/learn/dependency-injection-container) - 생성된 컨트롤러를 위한 Dice + Engine 주입
- [AI 및 개발자 경험](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - `make:record` / 스켈레톤 `App\Model`과 함께 사용되는 모델
- [SimplePdo](/learn/simple-pdo) - 스켈레톤 마이그레이션 및 모델에 사용되는 DB 연결