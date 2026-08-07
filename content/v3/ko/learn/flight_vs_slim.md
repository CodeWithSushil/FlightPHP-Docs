# Flight vs Slim

## Slim이란 무엇인가?
[Slim](https://slimframework.com)은 간단하면서도 강력한 웹 애플리케이션과 API를 빠르게 작성할 수 있도록 도와주는 PHP 마이크로 프레임워크입니다.

Flight의 v3 기능 중 일부는 실제로 Slim에서 많은 영감을 받았습니다. 라우트 그룹화와 특정 순서대로 미들웨어를 실행하는 것은 Slim에서 영감을 받은 두 가지 기능입니다. Slim v3는 단순함에 중점을 두고 출시되었지만, v4에 대해서는 [엇갈린 평가](https://github.com/slimphp/Slim/issues/2770)가 있습니다.

## Flight와 비교한 장점

- Slim은 더 많은 개발자 커뮤니티를 보유하고 있으며, 그 덕분에 바퀴를 다시 발명하지 않아도 되도록 도와주는 유용한 모듈이 많이 만들어지고 있습니다.
- Slim은 PHP 커뮤니티에서 흔히 사용되는 많은 인터페이스와 표준을 따르므로 상호 운용성이 높아집니다.
- Slim은 프레임워크를 배우는 데 사용할 수 있는 괜찮은 문서와 튜토리얼을 제공합니다. (물론 Laravel이나 Symfony에는 미치지 못합니다.)
- Slim에는 프레임워크를 배우는 데 활용할 수 있는 YouTube 튜토리얼과 온라인 기사 같은 다양한 자료가 있습니다.
- Slim은 PSR-7을 준수하므로 핵심 라우팅 기능을 처리하기 위해 원하는 구성 요소를 사용할 수 있습니다.

## Flight와 비교한 단점

- 놀랍게도 Slim은 마이크로 프레임워크로서 생각만큼 빠르지 않습니다. 자세한 내용은 [TechEmpower 벤치마크](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3)를 참조하세요.
- Flight는 가볍고 빠르며 사용하기 쉬운 웹 애플리케이션을 구축하려는 개발자를 대상으로 합니다.
- Flight는 의존성이 없지만, [Slim은 설치해야 하는 몇 가지 의존성](https://github.com/slimphp/Slim/blob/4.x/composer.json)이 있습니다.
- Flight는 단순함과 사용 용이성에 중점을 둡니다.
- Flight의 핵심 기능 중 하나는 하위 호환성을 유지하기 위해 최선을 다한다는 것입니다. Slim v3에서 v4로의 업그레이드는 호환성을 깨뜨리는 변경이었습니다.
- Flight는 프레임워크 세계에 처음 발을 들이는 개발자를 위한 것입니다.
- Flight는 엔터프라이즈급 애플리케이션도 처리할 수 있지만, Slim만큼 많은 예제와 튜토리얼을 제공하지는 않습니다. 또한 개발자가 코드를 체계적이고 잘 구조화된 상태로 유지하기 위해 더 많은 규율이 필요합니다.
- Flight는 개발자에게 애플리케이션에 대한 더 많은 제어권을 제공하는 반면, Slim은 백그라운드에서 마법 같은 동작을 몰래 수행할 수 있습니다.
- Flight는 데이터베이스 접근을 위한 [SimplePdo](/learn/simple-pdo)를 제공합니다 (더 이상 사용되지 않는 PdoWrapper보다 권장됩니다). Slim은 타사 라이브러리를 사용해야 합니다.
- Flight에는 애플리케이션을 보호하는 데 사용할 수 있는 [권한 플러그인](/awesome-plugins/permissions)이 있습니다. Slim은 타사 라이브러리를 사용해야 합니다.
- Flight에는 데이터베이스와 상호 작용하는 데 사용할 수 있는 [active-record](/awesome-plugins/active-record)라는 ORM이 있습니다. Slim은 타사 라이브러리를 사용해야 합니다.
- Flight에는 명령줄에서 애플리케이션을 실행하는 데 사용할 수 있는 [runway](/awesome-plugins/runway)라는 CLI 애플리케이션이 있습니다. Slim에는 없습니다.