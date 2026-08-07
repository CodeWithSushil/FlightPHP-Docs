# Flight vs Fat-Free

## Fat-Free란 무엇인가?
[Fat-Free](https://fatfreeframework.com)(애칭으로 **F3**이라고도 함)은 강력하면서도 사용하기 쉬운 PHP 마이크로프레임워크로, 빠르게 동적이고 견고한 웹 애플리케이션을 구축할 수 있도록 도와줍니다.

Flight는 여러 면에서 Fat-Free와 비교되며, 기능과 단순성 면에서 가장 가까운 사촌뻘일 것입니다. Fat-Free에는 Flight에는 없는 많은 기능이 있지만, Flight에 있는 많은 기능도 갖추고 있습니다. Fat-Free는 시대에 뒤처지기 시작했으며 예전만큼 인기가 있지는 않습니다.

업데이트 빈도가 점점 줄어들고 있으며 커뮤니티도 예전처럼 활발하지 않습니다. 코드 자체는 단순하지만, 문법적 규율이 부족한 부분이 있어 읽고 이해하기 어려울 때가 있습니다. PHP 8.3에서 작동하지만 코드 자체는 여전히 PHP 5.3 시대에 살고 있는 것처럼 보입니다.

## Flight와 비교한 장점

- Fat-Free는 GitHub에서 Flight보다 별(star)이 조금 더 많습니다.
- Fat-Free는 괜찮은 문서를 갖추고 있지만 일부 영역에서는 명확성이 부족합니다.
- Fat-Free는 YouTube 튜토리얼이나 온라인 기사처럼 프레임워크를 학습하는 데 사용할 수 있는 자료가 다소 있지만 빈약합니다.
- Fat-Free에는 때때로 유용한 [플러그인](https://fatfreeframework.com/3.8/api-reference)이 내장되어 있습니다.
- Fat-Free에는 Mapper라고 하는 내장 ORM이 있어 데이터베이스와 상호작용할 수 있습니다. Flight에는 [active-record](/awesome-plugins/active-record)가 있습니다.
- Fat-Free에는 세션, 캐싱, 지역화가 내장되어 있습니다. Flight는 타사 라이브러리를 사용해야 하지만 [문서](/awesome-plugins)에서 다루고 있습니다.
- Fat-Free에는 프레임워크를 확장하는 데 사용할 수 있는 [커뮤니티 제작 플러그인](https://fatfreeframework.com/3.8/development#Community)이 소수 있습니다. Flight는 [문서](/awesome-plugins)와 [예제](/examples) 페이지에서 일부를 다루고 있습니다.
- Fat-Free는 Flight와 마찬가지로 의존성이 없습니다.
- Fat-Free는 Flight와 마찬가지로 개발자에게 애플리케이션에 대한 제어권을 부여하고 단순한 개발자 경험을 제공하는 데 중점을 둡니다.
- Fat-Free는 Flight처럼 이전 버전과의 호환성을 유지합니다(부분적으로는 업데이트가 [점점 드물어지기](https://github.com/bcosca/fatfree/releases) 때문입니다).
- Fat-Free는 Flight와 마찬가지로 프레임워크 세계에 처음 발을 들이는 개발자를 위한 것입니다.
- Fat-Free에는 Flight의 템플릿 엔진보다 더 강력한 내장 템플릿 엔진이 있습니다. Flight는 이를 위해 [Latte](/awesome-plugins/latte)를 권장합니다.
- Fat-Free에는 Fat-Free 자체 내에서 CLI 앱을 구축하고 이를 마치 `GET` 요청처럼 취급할 수 있는 독특한 CLI 유형 "route" 명령이 있습니다. Flight는 [runway](/awesome-plugins/runway)로 이를 구현합니다.

## Flight와 비교한 단점

- Fat-Free에는 몇 가지 구현 테스트가 있고 자체 [test](https://fatfreeframework.com/3.8/test) 클래스도 있지만 매우 기본적입니다. 그러나 Flight처럼 100% 유닛 테스트가 이루어지지는 않습니다.
- 문서 사이트를 실제로 검색하려면 Google과 같은 검색 엔진을 사용해야 합니다.
- Flight 문서 사이트에는 다크 모드가 있습니다. (마이크 드롭)
- Fat-Free에는 유지 관리가 제대로 되지 않는 일부 모듈이 있습니다.
- Flight에는 데이터베이스 액세스를 위한 [SimplePdo](/learn/simple-pdo)가 있어 Fat-Free의 내장 `DB\SQL` 클래스보다 한결 간단합니다(그리고 더 이상 사용되지 않는 PdoWrapper보다 선호됩니다).
- Flight에는 애플리케이션을 보호하는 데 사용할 수 있는 [permissions 플러그인](/awesome-plugins/permissions)이 있습니다. Fat-Free는 타사 라이브러리를 사용해야 합니다.
- Flight에는 [active-record](/awesome-plugins/active-record)라는 ORM이 있는데, Fat-Free의 Mapper보다 ORM에 더 가깝게 느껴집니다.
  `active-record`의 추가 이점은 레코드 간의 관계를 정의하여 자동 조인을 할 수 있다는 점입니다. 반면 Fat-Free의 Mapper는
  [SQL 뷰](https://fatfreeframework.com/3.8/databases#ProsandCons)를 직접 만들어야 합니다.
- 놀랍게도 Fat-Free에는 루트 네임스페이스가 없습니다. Flight는 자체 코드와 충돌하지 않도록 끝까지 네임스페이스가 적용되어 있습니다.
  여기서 가장 큰 문제는 `Cache` 클래스입니다.
- Fat-Free에는 미들웨어가 없습니다. 대신 컨트롤러에서 요청과 응답을 필터링하는 데 사용할 수 있는 `beforeroute`와 `afterroute` 훅이 있습니다.
- Fat-Free는 라우트를 그룹화할 수 없습니다.
- Fat-Free에는 의존성 주입 컨테이너 핸들러가 있지만 사용 방법에 대한 문서는 매우 부족합니다.
- 기본적으로 모든 것이 [`HIVE`](https://fatfreeframework.com/3.8/quick-reference)라는 곳에 저장되기 때문에 디버깅이 약간 까다로울 수 있습니다.