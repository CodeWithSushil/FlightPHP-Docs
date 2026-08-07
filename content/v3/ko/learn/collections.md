# 컬렉션

## 개요

Flight의 `Collection` 클래스는 데이터 집합을 관리하기 위한 유용한 유틸리티입니다. 배열 표기법과 객체 표기법을 모두 사용하여 데이터에 접근하고 조작할 수 있으므로 코드가 더 깔끔하고 유연해집니다.

## 이해

`Collection`은 기본적으로 배열을 감싸는 래퍼이지만, 몇 가지 추가 기능이 있습니다. 배열처럼 사용할 수 있고, 반복할 수 있으며, 항목 수를 셀 수 있고, 객체 속성처럼 항목에 접근할 수도 있습니다. 이는 앱에서 구조화된 데이터를 전달하거나 코드를 더 읽기 쉽게 만들고 싶을 때 특히 유용합니다.

컬렉션은 여러 PHP 인터페이스를 구현합니다:
- `ArrayAccess` (배열 구문을 사용할 수 있음)
- `Iterator` (`foreach`로 반복할 수 있음)
- `Countable` (`count()`를 사용할 수 있음)
- `JsonSerializable` (쉽게 JSON으로 변환할 수 있음)

## 기본 사용법

### 컬렉션 생성

생성자에 배열을 전달하면 컬렉션을 만들 수 있습니다:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### 항목 접근

배열 표기법이나 객체 표기법을 사용하여 항목에 접근할 수 있습니다:

```php
// 배열 표기법
echo $collection['name']; // 출력: FlightPHP

// 객체 표기법
echo $collection->version; // 출력: 3
```

존재하지 않는 키에 접근하려고 하면 오류 대신 `null`을 반환합니다.

### 항목 설정

두 표기법 모두를 사용하여 항목을 설정할 수 있습니다:

```php
// 배열 표기법
$collection['author'] = 'Mike Cao';

// 객체 표기법
$collection->license = 'MIT';
```

### 항목 확인 및 제거

항목이 존재하는지 확인:

```php
if (isset($collection['name'])) {
  // 작업 수행
}

if (isset($collection->version)) {
  // 작업 수행
}
```

항목 제거:

```php
unset($collection['author']);
unset($collection->license);
```

### 컬렉션 반복

컬렉션은 반복 가능하므로 `foreach` 루프에서 사용할 수 있습니다:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### 항목 개수 세기

컬렉션의 항목 수를 셀 수 있습니다:

```php
echo count($collection); // 출력: 4
```

### 모든 키 또는 데이터 가져오기

모든 키 가져오기:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

모든 데이터를 배열로 가져오기:

```php
$data = $collection->getData();
```

### 컬렉션 비우기

모든 항목 제거:

```php
$collection->clear();
```

### JSON 직렬화

컬렉션은 쉽게 JSON으로 변환할 수 있습니다:

```php
echo json_encode($collection);
// 출력: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## 고급 사용법

필요한 경우 내부 데이터 배열을 완전히 교체할 수 있습니다:

```php
$collection->setData(['foo' => 'bar']);
```

컬렉션은 컴포넌트 간에 구조화된 데이터를 전달하거나 배열 데이터에 더 객체 지향적인 인터페이스를 제공하려 할 때 특히 유용합니다.

## 참고 항목

- [요청](/learn/requests) - HTTP 요청을 처리하는 방법과 컬렉션을 사용하여 요청 데이터를 관리하는 방법을 알아보세요.
- [SimplePdo](/learn/simple-pdo) - 쿼리 결과 행을 컬렉션으로 반환하는 데이터베이스 헬퍼입니다.

## 문제 해결

- 존재하지 않는 키에 접근하려고 하면 오류 대신 `null`을 반환합니다.
- 컬렉션은 재귀적이지 않다는 점을 기억하세요: 중첩 배열은 자동으로 컬렉션으로 변환되지 않습니다.
- 컬렉션을 재설정해야 한다면 `$collection->clear()` 또는 `$collection->setData([])`를 사용하세요.

## 변경 내역

- v3.0 - 타입 힌트 개선 및 PHP 8+ 지원.
- v1.0 - Collection 클래스 최초 릴리스.