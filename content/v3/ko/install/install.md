# 설치 지침

Flight를 설치하기 전에 몇 가지 기본 사전 요구 사항이 있습니다. 즉, 다음이 필요합니다:

1. [시스템에 PHP 설치](#installing-php)
2. 최고의 개발자 경험을 위해 [Composer 설치](https://getcomposer.org)

## 기본 설치

[Composer](https://getcomposer.org)를 사용 중이라면 다음 명령을 실행할 수 있습니다:

```bash
composer require flightphp/core
```

이렇게 하면 Flight 코어 파일만 시스템에 설치됩니다. 프로젝트 구조, [레이아웃](/learn/templates), [의존성](/learn/dependency-injection-container), [구성](/learn/configuration), [오토로딩](/learn/autoloading) 등을 직접 정의해야 합니다. 이 방법은 Flight 외에 다른 의존성이 설치되지 않도록 보장합니다.

[파일을 직접 다운로드](https://github.com/flightphp/core/archive/master.zip)하여 웹 디렉토리에 압축을 풀 수도 있습니다.

기본 설치는 학습, 마이크로 API, 복사-붙여넣기 실험에 적합합니다. 인간과 [AI 코딩 도구](/learn/ai)가 동일한 방식으로 따를 수 있는 전체 앱 레이아웃을 원한다면 아래 권장 스켈레톤을 사용하세요.

## 권장 설치

새 프로젝트에는 [flightphp/skeleton](https://github.com/flightphp/skeleton) 앱으로 시작하는 것을 적극 권장합니다. 설치는 매우 간단합니다.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# 선택적 샘플 DB + posts 데모
php runway migrate
```

이 단계는 프로젝트 구조, Composer PSR-4 오토로딩, 구성, 그리고 [Tracy](/awesome-plugins/tracy), [Tracy Extensions](/awesome-plugins/tracy-extensions), [Runway](/awesome-plugins/runway) 같은 도구를 설정합니다. 또한 루트 **`AGENTS.md`** 파일(그리고 `app/` 아래의 범위 복사본)을 포함하여 AI 어시스턴트가 사용자와 동일한 레이아웃을 공유하도록 합니다—[AI 및 개발자 경험](/learn/ai)을 참조하세요.

### 스켈레톤이 제공하는 것

```text
project-root/
├── AGENTS.md              # AI / 에이전트 정보 원본
├── SECURITY.md            # 보안 기대 사항
├── .env.example           # 비밀 / 배포 오버레이 (.env로 복사됨)
├── public/index.php       # 웹 진입점 전용
├── app/
│   ├── config/            # bootstrap, routes, services, config_sample.php
│   ├── Controller/        # App\Controller\*  (폴더명 파스칼 표기법!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Runway CLI 명령
│   ├── views/             # Twig 템플릿 (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # SQL 마이그레이션 (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**네임스페이스는 폴더 대소문자를 따릅니다.** Composer는 `"App\\": "app/"`로 매핑하므로:

| 디스크 경로 | 네임스페이스 |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Linux에서 `app/controller/`는 `app/Controller/`와 **같지 않습니다**. 오토로딩은 대소문자를 구분하므로 스켈레톤의 PascalCase 폴더를 그대로 따르세요. 자세한 내용: [오토로딩](/learn/autoloading).

**스택 기본값(새 프로젝트):** Twig 뷰, SimplePdo + ActiveRecord, `Engine` 주입을 사용하는 Dice(앱 클래스 내부에서 `Flight::` 사용을 지양), `php runway migrate` 후 선택적 SQLite.

`create-project`는 일반적으로 `app/config/config_sample.php` → `config.php`로, `.env.example` → `.env`로 복사합니다(있는 경우). 라우트는 `app/config/routes.php`에 있고, 서비스와 DI는 `app/config/services.php`에 있습니다.

> **문서 ↔ 스켈레톤:** 이 문서는 Flight **API**를 가르칩니다(종종 짧은 `Flight::` 예제 사용). 스켈레톤은 **애플리케이션 구조**를 고정합니다. `app/` 아래에 코드를 추가할 때는 스켈레톤 트리를 따르고, 메서드 이름, 옵션, 플러그인에 대해서는 문서를 사용하세요.

## 웹 서버 구성

### PHP 내장 개발 서버

이것이 가장 간단한 실행 방법입니다. 내장 서버를 사용하여 애플리케이션을 실행하고 SQLite를 데이터베이스로 사용할 수도 있습니다(sqlite3가 시스템에 설치되어 있기만 하면). 거의 아무것도 필요하지 않습니다! PHP가 설치되면 다음 명령을 실행하세요:

```bash
php -S localhost:8000
# 또는 스켈레톤 앱 사용 시
composer start
```

그런 다음 브라우저를 열고 `http://localhost:8000`으로 이동하세요.

프로젝트의 문서 루트를 다른 디렉토리로 지정하려면(예: 프로젝트가 `~/myproject`이고 문서 루트가 `~/myproject/public/`인 경우) `~/myproject` 디렉토리에서 다음 명령을 실행할 수 있습니다:

```bash
php -S localhost:8000 -t public/
# 스켈레톤 앱에서는 이미 구성되어 있음
composer start
```

그런 다음 브라우저를 열고 `http://localhost:8000`으로 이동하세요.

### Apache

시스템에 Apache가 이미 설치되어 있는지 확인하세요. 없다면 시스템에 Apache를 설치하는 방법을 검색하세요.

Apache의 경우 `.htaccess` 파일을 다음과 같이 편집하세요:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **참고**: 하위 디렉토리에서 Flight를 사용해야 하는 경우 `RewriteEngine On` 바로 다음에
> `RewriteBase /subdir/` 줄을 추가하세요.

> **참고**: DB나 env 파일과 같은 모든 서버 파일을 보호하려면
> `.htaccess` 파일에 다음을 넣으세요:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

시스템에 Nginx가 이미 설치되어 있는지 확인하세요. 없다면 시스템에 Nginx를 설치하는 방법을 검색하세요.

Nginx의 경우 서버 선언에 다음을 추가하세요:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## `index.php` 파일 만들기

기본 설치를 진행하는 경우 시작할 수 있는 몇 가지 코드가 필요합니다.

```php
<?php

// Composer를 사용한다면 오토로더를 불러오세요.
require 'vendor/autoload.php';
// Composer를 사용하지 않는다면 프레임워크를 직접 불러오세요.
// require 'flight/Flight.php';

// 그런 다음 라우트를 정의하고 요청을 처리할 함수를 할당하세요.
Flight::route('/', function () {
  echo 'hello world!';
});

// 마지막으로 프레임워크를 시작합니다.
Flight::start();
```

스켈레톤 앱에서는 공개 진입점이 앱을 부팅만 합니다. 라우트는 `app/config/routes.php`에 등록됩니다(일반적으로 `[App\Controller\…::class, 'method']` 형태로 Dice가 의존성을 주입할 수 있게 함). 서비스, Twig, SimplePdo, 컨테이너는 `app/config/services.php`에서 연결됩니다. 이 구조는 AI 도구와 인간이 항상 동일한 위치를 편집하도록 의도된 것입니다.

## PHP 설치

시스템에 이미 `php`가 설치되어 있다면 이 지침을 건너뛰고 [다운로드 섹션](#download-the-files)으로 이동하세요.

### **macOS**

#### **Homebrew로 PHP 설치**

1. **Homebrew 설치**(아직 설치되지 않은 경우):
   - 터미널을 열고 실행:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **PHP 설치**:
   - 최신 버전 설치:
     ```bash
     brew install php
     ```
   - 특정 버전(예: PHP 8.1)을 설치하려면:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **PHP 버전 간 전환**:
   - 현재 버전을 unlink하고 원하는 버전을 link:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - 설치된 버전 확인:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **PHP 수동 설치**

1. **PHP 다운로드**:
   - [PHP for Windows](https://windows.php.net/download/)를 방문하여 최신 또는 특정 버전(예: 7.4, 8.0)을 non-thread-safe zip 파일로 다운로드하세요.

2. **PHP 압축 해제**:
   - 다운로드한 zip 파일을 `C:\php`에 압축 해제하세요.

3. **시스템 PATH에 PHP 추가**:
   - **시스템 속성** > **환경 변수**로 이동하세요.
   - **시스템 변수**에서 **Path**를 찾아 **편집**을 클릭하세요.
   - `C:\php`(또는 PHP를 압축 해제한 위치) 경로를 추가하세요.
   - **확인**을 클릭하여 모든 창을 닫으세요.

4. **PHP 구성**:
   - `php.ini-development`를 `php.ini`로 복사하세요.
   - 필요에 따라 `php.ini`를 편집하여 PHP를 구성하세요(예: `extension_dir` 설정, 확장 기능 활성화).

5. **PHP 설치 확인**:
   - 명령 프롬프트를 열고 실행:
     ```cmd
     php -v
     ```

#### **여러 PHP 버전 설치**

1. 각 버전에 대해 **위 단계를 반복**하고 각각 별도의 디렉토리(예: `C:\php7`, `C:\php8`)에 배치하세요.

2. 시스템 PATH 변수를 원하는 버전 디렉토리를 가리키도록 조정하여 **버전 간 전환**하세요.

### **Ubuntu (20.04, 22.04 등)**

#### **apt로 PHP 설치**

1. **패키지 목록 업데이트**:
   - 터미널을 열고 실행:
     ```bash
     sudo apt update
     ```

2. **PHP 설치**:
   - 최신 PHP 버전 설치:
     ```bash
     sudo apt install php
     ```
   - 특정 버전(예: PHP 8.1)을 설치하려면:
     ```bash
     sudo apt install php8.1
     ```

3. **추가 모듈 설치**(선택 사항):
   - 예를 들어 MySQL 지원을 설치하려면:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **PHP 버전 간 전환**:
   - `update-alternatives` 사용:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **설치된 버전 확인**:
   - 실행:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **yum/dnf로 PHP 설치**

1. **EPEL 저장소 활성화**:
   - 터미널을 열고 실행:
     ```bash
     sudo dnf install epel-release
     ```

2. **Remi 저장소 설치**:
   - 실행:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **PHP 설치**:
   - 기본 버전 설치:
     ```bash
     sudo dnf install php
     ```
   - 특정 버전(예: PHP 7.4)을 설치하려면:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **PHP 버전 간 전환**:
   - `dnf` 모듈 명령 사용:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **설치된 버전 확인**:
   - 실행:
     ```bash
     php -v
     ```

### **일반 참고 사항**

- 개발 환경에서는 프로젝트 요구 사항에 맞게 PHP 설정을 구성하는 것이 중요합니다.
- PHP 버전을 전환할 때는 사용하려는 특정 버전에 맞는 모든 관련 PHP 확장 기능이 설치되어 있는지 확인하세요.
- PHP 버전을 전환하거나 구성을 업데이트한 후에는 웹 서버(Apache, Nginx 등)를 다시 시작하여 변경 사항을 적용하세요.