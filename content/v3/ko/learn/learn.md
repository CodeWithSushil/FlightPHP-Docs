# Flight 알아보기

Flight는 PHP를 위한 빠르고, 간단하며, 확장 가능한 프레임워크입니다. 매우 다재다능하며 모든 종류의 웹 애플리케이션을 구축하는 데 사용할 수 있습니다. 단순함을 염두에 두고 제작되었으며, 인간과 [AI 코딩 어시스턴트](/learn/ai) 모두가 이해하고 사용하기 쉽게 작성되었습니다.

> **참고:** `Flight::`를 정적 변수로 사용하는 예제와 `$app->` Engine 객체를 사용하는 예제가 있습니다. 둘 다 서로 교환하여 사용할 수 있습니다. 컨트롤러/미들웨어에서 `$app` 및 `$this->app`은 Flight 팀이 권장하는 방식입니다(그리고 공식 스켈레톤 + `AGENTS.md`가 새 프로젝트에 대해 표준화한 방식입니다).

## 핵심 구성 요소

### [라우팅](/learn/routing)

웹 애플리케이션의 라우트를 관리하는 방법을 알아보세요. 라우트 그룹화, 라우트 매개변수 및 미들웨어도 포함됩니다.

### [미들웨어](/learn/middleware)

애플리케이션에서 요청과 응답을 필터링하기 위해 미들웨어를 사용하는 방법을 알아보세요.

### [자동 로드](/learn/autoloading)

자체 클래스를 자동 로드하는 방법을 알아보세요. 폴더 **대소문자**는 네임스페이스와 일치해야 합니다. 스켈레톤은 `App\`과 `app/Controller/` 같은 PascalCase 폴더를 사용합니다.

### [요청](/learn/requests)

애플리케이션에서 요청과 응답을 처리하는 방법을 알아보세요.

### [응답](/learn/responses)

사용자에게 응답을 보내는 방법을 알아보세요.

### [HTML 템플릿](/learn/templates)

기본 제공 PHP 뷰뿐만 아니라 Twig(스켈레톤 기본값), Latte 또는 다른 엔진으로 HTML을 렌더링하는 방법을 알아보세요.

### [보안](/learn/security)

일반적인 보안 위협으로부터 애플리케이션을 보호하는 방법을 알아보세요.

### [구성](/learn/configuration)

애플리케이션에 맞게 프레임워크를 구성하는 방법을 알아보세요.

### [이벤트 매니저](/learn/events)

이벤트 시스템을 사용하여 애플리케이션에 사용자 정의 이벤트를 추가하는 방법을 알아보세요.

### [Flight 확장](/learn/extending)

프레임워크에 자신만의 메서드와 클래스를 추가하여 확장하는 방법을 알아보세요.

### [메서드 훅과 필터링](/learn/filtering)

메서드 및 내부 프레임워크 메서드에 이벤트 훅을 추가하는 방법을 알아보세요.

### [의존성 주입 컨테이너(DIC)](/learn/dependency-injection-container)

의존성 주입 컨테이너(DIC)를 사용하여 애플리케이션의 의존성을 관리하는 방법을 알아보세요.

## 유틸리티 클래스

### [컬렉션](/learn/collections)

컬렉션은 데이터를 보관하고 쉽게 사용할 수 있도록 배열 또는 객체로 접근할 수 있게 해줍니다.

### [JSON 래퍼](/learn/json)

JSON 인코딩 및 디코딩을 일관되게 만드는 몇 가지 간단한 함수를 제공합니다.

### [SimplePdo](/learn/simple-pdo)

PDO는 때때로 필요 이상으로 골치 아플 수 있습니다. SimplePdo는 `insert()`, `update()`, `delete()`, `transaction()`과 같은 편리한 메서드를 제공하여 데이터베이스 작업을 훨씬 쉽게 만들어 주는 현대적인 PDO 헬퍼 클래스입니다.

### [PdoWrapper](/learn/pdo-wrapper) (더 이상 사용되지 않음)

원래 PDO 래퍼는 v3.18.0부터 더 이상 사용되지 않습니다. 대신 [SimplePdo](/learn/simple-pdo)를 사용하세요.

### [업로드 파일 핸들러](/learn/uploaded-file)

업로드된 파일을 관리하고 영구 위치로 이동하는 데 도움이 되는 간단한 클래스입니다.

## 중요한 개념

### [프레임워크를 사용하는 이유는 무엇인가요?](/learn/why-frameworks)

프레임워크를 사용해야 하는 이유에 대한 짧은 문서입니다. 프레임워크를 사용하기 전에 프레임워크 사용의 이점을 이해하는 것이 좋습니다.

또한 [@lubiana](https://git.php.fail/lubiana)가 훌륭한 튜토리얼을 만들었습니다. Flight에 대해 구체적으로 자세히 다루지는 않지만, 이 가이드는 프레임워크와 관련된 주요 개념과 그것들이 왜 유용한지 이해하는 데 도움이 될 것입니다. 튜토리얼은 [여기](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md)에서 찾을 수 있습니다.

### [다른 프레임워크와 Flight 비교](/learn/flight-vs-another-framework)

Laravel, Slim, Fat-Free 또는 Symfony와 같은 다른 프레임워크에서 Flight로 마이그레이션하는 경우, 이 페이지는 두 프레임워크 간의 차이점을 이해하는 데 도움이 됩니다.

## 기타 주제

### [유닛 테스트](/learn/unit-testing)

이 가이드를 따라 Flight 코드를 견고하게 유닛 테스트하는 방법을 알아보세요.

### [AI 및 개발자 경험](/learn/ai)

Flight는 코딩 LLM과 함께 사용하도록 설계되었습니다: `AGENTS.md`, Runway `ai:*` 명령, 그리고 에이전트가 패턴을 유지할 수 있는 명확한 스켈레톤 레이아웃을 제공합니다.

### [v2 -> v3 마이그레이션](/learn/migrating-to-v3)

대부분 이전 버전과의 호환성이 유지되었지만, v2에서 v3로 마이그레이션할 때 알아야 할 몇 가지 변경 사항이 있습니다.