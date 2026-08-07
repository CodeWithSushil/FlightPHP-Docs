# FlightPHP APM 문서

FlightPHP APM에 오신 것을 환영합니다—앱의 개인 성능 코치입니다! 이 가이드는 FlightPHP를 사용한 애플리케이션 성능 모니터링(APM)의 설정, 사용 및 마스터링을 위한 로드맵입니다. 느린 요청을 찾거나 지연 시간 차트를 보고 싶으시다면, 저희가 도와드리겠습니다. 앱을 더 빠르게 만들고, 사용자를 더 행복하게 하며, 디버깅 세션을 더 쉽게 만들어 보세요!

Flight Docs 사이트의 대시보드 [데모](https://flightphp-docs-apm.sky-9.com/apm/dashboard)를 확인하세요.

![FlightPHP APM](/images/apm.png)

## APM이 중요한 이유

상상해 보세요: 앱이 바쁜 레스토랑입니다. 주문이 얼마나 걸리는지, 주방이 어디에서 지연되는지 추적할 방법이 없다면, 고객이 왜 불만을 품고 떠나는지 추측하게 됩니다. APM은 소셰프입니다—들어오는 요청부터 데이터베이스 쿼리까지 모든 단계를 감시하고, 속도를 늦추는 모든 것을 표시합니다. 느린 페이지는 사용자를 잃게 만듭니다(연구에 따르면 사이트 로딩이 3초 이상 걸리면 53%가 이탈한다고 합니다!), APM은 문제가 발생하기 *전에* 이를 잡아줍니다. 이는 선제적 안심입니다—"이게 왜 고장났지?" 순간이 줄어들고, "이게 얼마나 매끄럽게 실행되는지 보세요!"라는 승리가 늘어납니다.

## 설치

Composer로 시작하세요:

```bash
composer require flightphp/apm
```

필요한 것:
- **PHP 7.4+**: 현대 PHP를 지원하면서 LTS Linux 배포판과 호환됩니다.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: 우리가 개선하고 있는 경량 프레임워크.

## 지원되는 데이터베이스

FlightPHP APM은 현재 메트릭 저장을 위해 다음 데이터베이스를 지원합니다:

- **SQLite3**: 간단하고 파일 기반이며 로컬 개발이나 소규모 앱에 적합합니다. 대부분의 설정에서 기본 옵션입니다.
- **MySQL/MariaDB**: 견고하고 확장 가능한 저장소가 필요한 대규모 프로젝트나 프로덕션 환경에 이상적입니다.

구성 단계(아래 참조)에서 데이터베이스 유형을 선택할 수 있습니다. PHP 환경에 필요한 확장(예: `pdo_sqlite` 또는 `pdo_mysql`)이 설치되어 있는지 확인하세요.

## 시작하기

APM의 훌륭함을 위한 단계별 가이드입니다:

### 1. APM 등록

추적을 시작하려면 `index.php` 또는 `services.php` 파일에 다음을 추가하세요:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// 데이터베이스 연결을 추가하는 경우
// SimplePdo(또는 개발 시 Tracy Extensions의 PdoQueryCapture)를 선호하세요.
// 옵션 배열(5번째 인수)을 통해 APM 쿼리 추적을 활성화하세요.
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // APM을 위한 쿼리 캡처에 필요
]);
$Apm->addPdoConnection($pdo);
```

**여기서 무슨 일이 일어나고 있나요?**
- `LoggerFactory::create()`는 설정(곧 자세히 설명)을 가져와 로거를 설정합니다—기본값은 SQLite입니다.
- `Apm`이 스타입니다—Flight의 이벤트(요청, 경로, 오류 등)를 수신하고 메트릭을 수집합니다.
- `bindEventsToFlightInstance($app)`는 모든 것을 Flight 앱에 연결합니다.

**전문가 팁: 샘플링**
앱이 바쁘다면 *모든* 요청을 로깅하면 과부하가 걸릴 수 있습니다. 샘플 비율(0.0에서 1.0)을 사용하세요:

```php
$Apm = new Apm($ApmLogger, 0.1); // 요청의 10%를 로깅
```

이렇게 하면 성능을 빠르게 유지하면서도 견고한 데이터를 제공합니다.

### 2. 구성하기

`.runway-config.json`을 생성하려면 다음을 실행하세요:

```bash
php vendor/bin/runway apm:init
```

**이게 무슨 역할을 하나요?**
- 원시 메트릭이 어디에서 오는지(소스)와 처리된 데이터가 어디로 가는지(대상)를 묻는 마법사를 시작합니다.
- 기본값은 SQLite입니다—예를 들어 소스의 경우 `sqlite:/tmp/apm_metrics.sqlite`, 대상의 경우 다른 것.
- 다음과 같은 설정이 생성됩니다:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> 이 과정은 또한 이 설정에 대한 마이그레이션을 실행할지 묻습니다. 처음 설정하는 경우, 답은 예입니다.

**왜 두 개의 위치가 필요한가요?**
원시 메트릭은 빠르게 쌓입니다(필터링되지 않은 로그를 생각해 보세요). 워커는 대시보드를 위한 구조화된 대상으로 처리합니다. 깔끔하게 유지됩니다!

### 3. 워커로 메트릭 처리하기

워커는 원시 메트릭을 대시보드 준비 데이터로 변환합니다. 한 번 실행하세요:

```bash
php vendor/bin/runway apm:worker
```

**무슨 일을 하나요?**
- 소스(예: `apm_metrics.sqlite`)에서 읽습니다.
- 최대 100개의 메트릭(기본 배치 크기)을 대상으로 처리합니다.
- 완료되거나 남은 메트릭이 없으면 중지됩니다.

**계속 실행하기**
라이브 앱의 경우, 지속적인 처리가 필요합니다. 옵션은 다음과 같습니다:

- **데몬 모드**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  영원히 실행되며, 메트릭이 들어올 때마다 처리합니다. 개발이나 소규모 설정에 적합합니다.

- **Crontab**:
  crontab에 다음을 추가하세요(`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  매분 실행됩니다—프로덕션에 완벽합니다.

- **Tmux/Screen**:
  분리 가능한 세션을 시작하세요:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, 그 다음 D로 분리; `tmux attach -t apm-worker`로 재연결
  ```
  로그아웃해도 계속 실행됩니다.

- **사용자 정의 조정**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: 한 번에 50개의 메트릭을 처리합니다.
  - `--max_messages 1000`: 1000개의 메트릭 후 중지합니다.
  - `--timeout 300`: 5분 후 종료합니다.

**왜 신경 써야 하나요?**
워커 없이는 대시보드가 비어 있습니다. 원시 로그와 실행 가능한 인사이트 사이의 다리입니다.

### 4. 대시보드 시작하기

앱의 상태를 확인하세요:

```bash
php vendor/bin/runway apm:dashboard
```

**이게 무슨 역할을 하나요?**
- `http://localhost:8001/apm/dashboard`에서 PHP 서버를 시작합니다.
- 요청 로그, 느린 경로, 오류율 등을 표시합니다.

**사용자 정의하기**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: 모든 IP에서 접근 가능(원격 보기용으로 편리함).
- `--port 8080`: 8001이 사용 중인 경우 다른 포트를 사용합니다.
- `--php-path`: PHP가 PATH에 없는 경우 PHP를 가리킵니다.

브라우저에서 URL을 클릭하고 탐색하세요!

#### 프로덕션 모드

프로덕션의 경우, 방화벽 및 기타 보안 조치가 있을 가능성이 높으므로 대시보드를 실행하기 위해 몇 가지 기술을 시도해야 할 수 있습니다. 몇 가지 옵션은 다음과 같습니다:

- **역방향 프록시 사용**: Nginx 또는 Apache를 설정하여 요청을 대시보드로 전달합니다.
- **SSH 터널**: 서버에 SSH할 수 있는 경우, `ssh -L 8080:localhost:8001
youruser@yourserver`를 사용하여 대시보드를 로컬 머신으로 터널링합니다.
- **VPN**: 서버가 VPN 뒤에 있는 경우, VPN에 연결하고 대시보드에 직접 접근합니다.
- **방화벽 구성**: IP 또는 서버의 네트워크에 대해 포트 8001을 엽니다. (또는 설정한 포트).
- **Apache/Nginx 구성**: 애플리케이션 앞에 웹 서버가 있는 경우, 도메인 또는 하위 도메인으로 구성할 수 있습니다. 이렇게 하는 경우, 문서 루트를 `/path/to/your/project/vendor/flightphp/apm/dashboard`로 설정합니다.

#### 다른 대시보드를 원하시나요?

원하는 경우 직접 대시보드를 만들 수 있습니다! `vendor/flightphp/apm/src/apm/presenter` 디렉토리를 확인하여 자체 대시보드용 데이터를 표시하는 방법에 대한 아이디어를 얻으세요!

## 대시보드 기능

대시보드는 APM HQ입니다—여기서 볼 수 있는 것들입니다:

- **요청 로그**: 타임스탬프, URL, 응답 코드 및 총 시간이 포함된 모든 요청. 미들웨어, 쿼리 및 오류에 대한 "세부 정보"를 클릭하세요.
- **가장 느린 요청**: 시간을 많이 소모하는 상위 5개 요청(예: 2.5초의 "/api/heavy").
- **가장 느린 경로**: 평균 시간 기준 상위 5개 경로—패턴을 찾는 데 좋습니다.
- **오류율**: 실패하는 요청의 백분율(예: 2.3%의 500 오류).
- **지연 시간 백분위수**: 95번째(p95) 및 99번째(p99) 응답 시간—최악의 시나리오를 파악하세요.
- **응답 코드 차트**: 시간 경과에 따른 200, 404, 500 오류를 시각화합니다.
- **긴 쿼리/미들웨어**: 상위 5개의 느린 데이터베이스 호출 및 미들웨어 계층.
- **캐시 히트/미스**: 캐시가 얼마나 자주 문제를 해결하는지.

**추가 기능**:
- "지난 1시간", "지난 1일" 또는 "지난 1주"로 필터링.
- 야간 세션을 위한 다크 모드 전환.

**예시**:
`/users`에 대한 요청은 다음을 표시할 수 있습니다:
- 총 시간: 150ms
- 미들웨어: `AuthMiddleware->handle` (50ms)
- 쿼리: `SELECT * FROM users` (80ms)
- 캐시: `user_list` 히트 (5ms)

## 사용자 정의 이벤트 추가하기

API 호출이나 결제 프로세스와 같은 모든 것을 추적하세요:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**어디에 표시되나요?**
대시보드의 요청 세부 정보에서 "사용자 정의 이벤트" 아래에 표시됩니다—예쁜 JSON 형식으로 확장 가능합니다.

**사용 사례**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
이제 해당 API가 앱을 지연시키는지 확인할 수 있습니다!

## 데이터베이스 모니터링

PDO 쿼리를 다음과 같이 추적하세요:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // APM을 위한 쿼리 캡처에 필요
]);
$Apm->addPdoConnection($pdo);
```

**얻을 수 있는 것**:
- 쿼리 텍스트(예: `SELECT * FROM users WHERE id = ?`)
- 실행 시간(예: 0.015s)
- 행 수(예: 42)

**주의사항**:
- **선택 사항**: DB 추적이 필요하지 않은 경우 건너뛰세요.
- **SimplePdo(선호됨)**: `trackApmQueries => true`와 함께 `SimplePdo`를 사용하세요. 더 이상 사용되지 않는 `PdoWrapper`도 여전히 작동합니다(5번째 생성자 인수 `true`). 원시 코어 PDO는 아직 연결되지 않았습니다—기다려 주세요!
- **성능 경고**: DB가 많은 사이트에서 모든 쿼리를 로깅하면 속도가 느려질 수 있습니다. 부하를 줄이기 위해 샘플링(`$Apm = new Apm($ApmLogger, 0.1)`)을 사용하세요.

**예시 출력**:
- 쿼리: `SELECT name FROM products WHERE price > 100`
- 시간: 0.023s
- 행: 15

## 워커 옵션

워커를 원하는 대로 조정하세요:

- `--timeout 300`: 5분 후 중지—테스트용으로 좋습니다.
- `--max_messages 500`: 500개의 메트릭으로 제한—유한하게 유지합니다.
- `--batch_size 200`: 한 번에 200개 처리—속도와 메모리의 균형을 맞춥니다.
- `--daemon`: 중단 없이 실행—실시간 모니터링에 이상적입니다.

**예시**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
한 시간 동안 실행되며, 한 번에 100개의 메트릭을 처리합니다.

## 앱의 요청 ID

각 요청에는 추적을 위한 고유한 요청 ID가 있습니다. 이 ID를 사용하여 로그와 메트릭을 상호 연관시킬 수 있습니다. 예를 들어 오류 페이지에 요청 ID를 추가할 수 있습니다:

```php
Flight::map('error', function($message) {
	// 응답 헤더 X-Flight-Request-Id에서 요청 ID를 가져옵니다
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// 추가로 Flight 변수에서 가져올 수도 있습니다
	// 이 방법은 swoole 또는 기타 비동기 플랫폼에서 잘 작동하지 않습니다.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## 업그레이드

APM의 최신 버전으로 업그레이드하는 경우, 실행해야 할 데이터베이스 마이그레이션이 있을 수 있습니다. 다음 명령을 실행하여 이 작업을 수행할 수 있습니다:

```bash
php vendor/bin/runway apm:migrate
```
이 명령은 데이터베이스 스키마를 최신 버전으로 업데이트하는 데 필요한 마이그레이션을 실행합니다.

**참고:** APM 데이터베이스가 크기가 큰 경우, 이러한 마이그레이션에 시간이 걸릴 수 있습니다. 피크 시간이 아닌 시간에 이 명령을 실행하는 것이 좋습니다.

### 0.4.3 -> 0.5.0으로 업그레이드

0.4.3에서 0.5.0으로 업그레이드하는 경우, 다음 명령을 실행해야 합니다:

```bash
php vendor/bin/runway apm:config-migrate
```

이 명령은 `.runway-config.json` 파일을 사용하는 이전 형식에서 `config.php` 파일에 키/값을 저장하는 새 형식으로 설정을 마이그레이션합니다.

## 오래된 데이터 삭제

데이터베이스를 깔끔하게 유지하려면 오래된 데이터를 삭제할 수 있습니다. 이는 바쁜 앱을 실행하고 데이터베이스 크기를 관리 가능한 상태로 유지하려는 경우 특히 유용합니다.
다음 명령을 실행하여 이 작업을 수행할 수 있습니다:

```bash
php vendor/bin/runway apm:purge
```
이 명령은 데이터베이스에서 30일 이상 된 모든 데이터를 제거합니다. `--days` 옵션에 다른 값을 전달하여 일 수를 조정할 수 있습니다:

```bash
php vendor/bin/runway apm:purge --days 7
```
이 명령은 데이터베이스에서 7일 이상 된 모든 데이터를 제거합니다.

## 문제 해결

막혔나요? 다음을 시도해 보세요:

- **대시보드 데이터가 없나요?**
  - 워커가 실행 중인가요? `ps aux | grep apm:worker`를 확인하세요.
  - 설정 경로가 일치하나요? `.runway-config.json` DSN이 실제 파일을 가리키는지 확인하세요.
  - 보류 중인 메트릭을 처리하기 위해 `php vendor/bin/runway apm:worker`를 수동으로 실행하세요.

- **워커 오류?**
  - SQLite 파일을 확인하세요(예: `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - PHP 로그에서 스택 트레이스를 확인하세요.

- **대시보드가 시작되지 않나요?**
  - 포트 8001이 사용 중인가요? `--port 8080`을 사용하세요.
  - PHP를 찾을 수 없나요? `--php-path /usr/bin/php`를 사용하세요.
  - 방화벽이 차단하나요? 포트를 열거나 `--host localhost`를 사용하세요.

- **너무 느린가요?**
  - 샘플 비율을 낮추세요: `$Apm = new Apm($ApmLogger, 0.05)` (5%).
  - 배치 크기를 줄이세요: `--batch_size 20`.

- **예외/오류를 추적하지 않나요?**
  - 프로젝트에 [Tracy](https://tracy.nette.org/)가 활성화되어 있는 경우, Flight의 오류 처리를 재정의합니다. Tracy를 비활성화한 다음 `Flight::set('flight.handle_errors', true);`가 설정되어 있는지 확인해야 합니다.

- **데이터베이스 쿼리를 추적하지 않나요?**
  - 5번째 생성자 인수(옵션 배열)로 `['trackApmQueries' => true]`와 함께 `SimplePdo`를 선호하세요.
  - 더 이상 사용되지 않는 `PdoWrapper`를 여전히 사용하는 경우, 5번째 인수로 `true`를 전달하세요.
  - 연결을 생성한 후 `$Apm->addPdoConnection($pdo)`를 호출하세요.