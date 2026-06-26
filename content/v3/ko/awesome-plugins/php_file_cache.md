# flightphp/cache

가볍고, 간단하며 독립적인 PHP 파일 내 캐싱 클래스. [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)에서 포크됨

**장점**
- 가볍고, 독립적이며 간단함
- 모든 코드가 하나의 파일에 포함 - 불필요한 드라이버 없음
- 안전함 - 생성된 모든 캐시 파일에는 die가 포함된 PHP 헤더가 있어, 경로를 알고 서버가 제대로 구성되지 않았더라도 직접 접근이 불가능
- 잘 문서화되고 테스트됨
- flock을 통해 동시성을 올바르게 처리
- PHP 7.4+ 지원
- MIT 라이선스 하에 무료

이 문서 사이트는 각 페이지를 캐시하기 위해 이 라이브러리를 사용합니다!

코드를 보려면 [여기](https://github.com/flightphp/cache)를 클릭하세요.

## 설치

Composer를 통해 설치:

```bash
composer require flightphp/cache
```

## 사용법

사용법은 상당히 간단합니다. 캐시 디렉토리에 캐시 파일을 저장합니다.

```php
use flight\Cache;

$app = Flight::app();

// 캐시가 저장될 디렉토리를 생성자에 전달합니다
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// 캐시는 프로덕션 모드에서만 사용되도록 보장합니다
	// ENVIRONMENT는 부트스트랩 파일이나 앱의 다른 곳에서 설정된 상수입니다
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### 캐시 값 가져오기

`get()` 메서드를 사용하여 캐시된 값을 가져옵니다. 만료된 경우 캐시를 새로 고치는 편의 메서드를 원하면 `refreshIfExpired()`를 사용할 수 있습니다.

```php

// 캐시 인스턴스 가져오기
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // 캐시할 데이터 반환
}, 10); // 10초

// 또는
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10초
}
```

### 캐시 값 저장하기

`set()` 메서드를 사용하여 캐시에 값을 저장합니다.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10초
```

### 캐시 값 삭제하기

`delete()` 메서드를 사용하여 캐시의 값을 삭제합니다.

```php
Flight::cache()->delete('simple-cache-test');
```

### 캐시 값 존재 여부 확인하기

`exists()` 메서드를 사용하여 캐시에 값이 존재하는지 확인합니다.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// 무언가 수행
}
```

### 캐시 지우기
`flush()` 메서드를 사용하여 전체 캐시를 지웁니다.

```php
Flight::cache()->flush();
```

### 캐시와 함께 메타데이터 추출하기

캐시 항목에 대한 타임스탬프 및 기타 메타데이터를 추출하려면 올바른 매개변수로 `true`를 전달해야 합니다.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // 캐시할 데이터 반환
}, 10, true); // true = 메타데이터와 함께 반환
// 또는
$data = $cache->get("simple-cache-meta-test", true); // true = 메타데이터와 함께 반환

/*
메타데이터와 함께 검색된 캐시된 항목 예시:
{
    "time":1511667506, <-- 유닉스 타임스탬프 저장
    "expire":10,       <-- 초 단위 만료 시간
    "data":"04:38:26", <-- 역직렬화된 데이터
    "permanent":false
}

메타데이터를 사용하면, 예를 들어 항목이 저장된 시점이나 만료되는 시점을 계산할 수 있습니다
"data" 키를 통해 데이터 자체에 접근할 수도 있습니다
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // 데이터가 만료되는 유닉스 타임스탬프를 가져와 현재 타임스탬프에서 빼기
$cacheddate = $data["data"]; // "data" 키를 통해 데이터 자체에 접근

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## 소스 코드

코드를 보려면 [https://github.com/flightphp/cache](https://github.com/flightphp/cache)를 방문하세요.