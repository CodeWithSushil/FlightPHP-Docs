# 트레이시

트레이시는 Flight과 함께 사용할 수 있는 놀라운 오류 핸들러입니다. 애플리케이션 디버깅에 도움이 되는 여러 패널을 가지고 있습니다. 또한 확장하기가 매우 쉽고 자신만의 패널을 추가할 수 있습니다. Flight 팀은 [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) 플러그인으로 Flight 프로젝트를 위한 몇 가지 패널을 만들었습니다(Flight 변수, DB 쿼리, 요청, 세션, 그리고 프로파일러 프로필을 전달할 때 선택적인 **Twig** 패널 - [Tracy Extensions](/awesome-plugins/tracy-extensions) 참조).

## 설치

컴포저로 설치합니다. 그리고 Tracy는 프로덕션 오류 처리 컴포넌트를 제공하므로 실제로 개발 버전 없이 설치하는 것이 좋습니다.

```bash
composer require tracy/tracy
```

## 기본 설정

시작하기 위한 몇 가지 기본 설정 옵션이 있습니다. 자세한 내용은 [Tracy 문서](https://tracy.nette.org/en/configuring)에서 확인할 수 있습니다.

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Tracy 활성화
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // 때때로 명시적으로 지정해야 합니다 (Debugger::PRODUCTION도 마찬가지)
// Debugger::enable('23.75.345.200'); // IP 주소 배열을 제공할 수도 있습니다

// 오류와 예외가 기록될 위치입니다. 이 디렉토리가 존재하고 쓰기 가능해야 합니다.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // 모든 오류 표시
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // 더 이상 사용되지 않는 알림을 제외한 모든 오류
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // 디버거 바가 표시되면 Flight에서 content-length를 설정할 수 없습니다

	// flightphp/tracy-extensions를 포함한 경우 Flight용 Tracy 확장에 특화된 코드입니다
	// 그렇지 않으면 이 부분을 주석 처리하세요.
	new TracyExtensionLoader($app);
}
```

## 유용한 팁

코드를 디버깅할 때 데이터를 출력하는 데 매우 유용한 함수들이 있습니다.

- `bdump($var)` - 변수를 별도의 패널로 Tracy 바에 덤프합니다.
- `dumpe($var)` - 변수를 덤프한 후 즉시 종료합니다.