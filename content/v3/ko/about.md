# Flight PHP 프레임워크

Flight은 빠르고 간단하며 확장 가능한 PHP 프레임워크로, 빠르게 작업을 완료하고자 하는 개발자들을 위해 설계되었습니다. 클래식 웹 앱, 초고속 API 구축, 또는 AI 코딩 어시스턴트와의 협업까지, Flight의 경량성과 직관적인 설계는 완벽한 선택이 될 것입니다. Flight은 경량성을 지향하지만, 엔터프라이즈 아키텍처 요구사항도 충분히 처리할 수 있습니다.

## 왜 Flight을 선택해야 할까요?

- **초보자 친화적:** Flight은 PHP를 처음 접하는 개발자에게 훌륭한 시작점입니다. 명확한 구조와 간단한 문법으로 복잡한 보일러플레이트 없이 웹 개발을 배울 수 있습니다.
- **전문가들에게 사랑받는:** 숙련된 개발자들은 Flight의 유연성과 제어 능력에 매료됩니다. 작은 프로토타입에서부터 완전한 기능을 갖춘 앱까지, 프레임워크를 바꾸지 않고도 확장할 수 있습니다.
- **하위 호환성:** 우리는 여러분의 시간을 소중히 여깁니다. Flight v3는 v2의 확장으로, 기존 API를 거의 그대로 유지합니다. 우리는 진화가 아닌 혁명을 믿습니다—메이저 버전이 나올 때마다 "모든 것을 깨는" 일은 없습니다.
- **의존성 없음:** Flight의 핵심은 완전히 의존성 없이 구성되어 있습니다—폴리필, 외부 패키지, PSR 인터페이스조차 없습니다. 이는 더 적은 공격 벡터, 더 작은 풋프린트, 그리고 상위 의존성으로 인한 예기치 않은 호환성 문제를 의미합니다. 선택적 플러그인은 의존성을 포함할 수 있지만, 핵심은 항상 경량성과 보안을 유지합니다.
- **AI 친화적:** Flight의 작은 API 표면과 [공식 스켈레톤](https://github.com/flightphp/skeleton)(단일 레이아웃, `AGENTS.md`, 생성자 주입)은 AI 코딩 도구가 패턴을 유지하기 쉽게 합니다. 모든 코드를 직접 입력하든, 에이전트와 협업하든 동일한 코드베이스를 사용합니다. [Flight과 AI 사용에 대해 더 알아보기](/learn/ai).

## 비디오 개요

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">간단하지 않나요?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">문서에서</a> Flight에 대해 더 알아보세요!
    </div>
  </div>
</div>

## 빠른 시작

빠른 기본 설치의 경우, Composer로 설치하세요:

```bash
composer require flightphp/core
```

또는 [여기](https://github.com/flightphp/core)에서 저장소의 zip 파일을 다운로드할 수 있습니다. 그런 다음 다음과 같은 기본 `index.php` 파일을 가지게 됩니다:

```php
<?php

// composer로 설치한 경우
require 'vendor/autoload.php';
// 또는 zip 파일로 수동 설치한 경우
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

이게 전부입니다! 기본적인 Flight 애플리케이션이 완성되었습니다. 이제 `php -S localhost:8000`로 이 파일을 실행하고 브라우저에서 `http://localhost:8000`을 방문하여 결과를 확인할 수 있습니다.

이와 같은 짧은 `Flight::` 예제는 학습과 마이크로 앱에 적합합니다. 인간과 AI 도구가 공유하는 전체 프로젝트 레이아웃의 경우, 아래 스켈레톤을 사용하세요.

## 스켈레톤/보일러플레이트 앱

새로운 Flight 프로젝트를 시작할 수 있도록 공식 스타터가 있습니다. 구조, 설정, Composer 스크립트, AI 친화적인 지침을 처음부터 설정합니다.

준비된 프로젝트는 [flightphp/skeleton](https://github.com/flightphp/skeleton)을 확인하거나, 영감을 얻으려면 [예제](examples) 페이지를 방문하세요. AI 워크플로우 세부 사항이 필요하신가요? [AI와 개발자 경험 탐색](/learn/ai).

얻을 수 있는 것들 (고급 수준):

- **`App\` 네임스페이스**와 PascalCase 폴더 (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—폴더 **대소문자**는 네임스페이스와 일치해야 함 ([자동 로딩](/learn/autoloading) 참조)
- **Dice + `Engine` 주입**으로 컨트롤러가 테스트 가능하게 유지 (앱 코드에서 `Flight::` 대신 `$this->app` 선호)
- **Twig** 뷰, **SimplePdo** + ActiveRecord 샘플, Runway **migrate**
- 어시스턴트와 보안 정책을 위한 루트 **`AGENTS.md`** (및 범위 지정된 복사본)와 **`SECURITY.md`**

## 스켈레톤 앱 설치

간단합니다!

```bash
# 새 프로젝트 생성
composer create-project flightphp/skeleton my-project/
# 새 프로젝트 디렉토리로 이동
cd my-project/
# 바로 시작할 수 있도록 로컬 개발 서버 실행!
composer start
```

프로젝트 구조를 생성하고, `config_sample.php` → `config.php` (그리고 `.env.example` → `.env`가 있는 경우)로 복사하며, 준비가 완료됩니다. 선택적 샘플 데이터:

```bash
php runway migrate
# 그 다음 /posts와 /api/posts 방문
```

## 고성능

Flight는 가장 빠른 PHP 프레임워크 중 하나입니다. 경량화된 코어는 오버헤드를 줄이고 속도를 높여—전통적인 앱과 현대적인 AI 지원 워크플로우 모두에 적합합니다. [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)에서 모든 벤치마크를 확인할 수 있습니다.

다른 인기 있는 PHP 프레임워크들과의 벤치마크를 아래에서 확인하세요.

| Framework | Plaintext Reqs/sec | JSON Reqs/sec |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight과 AI

코딩 LLM과 Flight이 어떻게 연동되는지 궁금하신가요? `AGENTS.md`, Runway `ai:*` 명령어, 스켈레톤 레이아웃이 어시스턴트들을 올바른 길로 유지하는 방법을 [발견](/learn/ai)하세요.

## 안정성과 하위 호환성

우리는 여러분의 시간을 소중히 여깁니다. 우리는 몇 년마다 완전히 재설계되어 개발자들에게 깨진 코드와 비용이 많이 드는 마이그레이션을 남기는 프레임워크들을 모두 보아왔습니다. Flight은 다릅니다. Flight v3는 v2의 확장으로 설계되어, 여러분이 알고 사랑하는 API가 제거되지 않았습니다. 실제로 대부분의 v2 프로젝트는 v3에서 아무런 변경 없이 작동할 것입니다.

여러분이 프레임워크를 고치는 대신 앱 구축에 집중할 수 있도록 Flight을 안정적으로 유지하는 데 전념하고 있습니다. 스켈레톤은 *새로운* 프로젝트에 대해 독단적일 수 있습니다; 코어 API는 다른 모든 사람들에게 친숙하게 유지됩니다.

# 커뮤니티

Matrix 채팅에 있습니다

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

그리고 Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# 기여하기

Flight에 기여할 수 있는 두 가지 방법이 있습니다:

1. [코어 저장소](https://github.com/flightphp/core)를 방문하여 코어 프레임워크에 기여하세요.
2. 문서를 더 좋게 만들어주세요! 이 문서 웹사이트는 [Github](https://github.com/flightphp/docs)에 호스팅되어 있습니다. 오류를 발견하거나 개선하고 싶은 부분이 있으면 풀 리퀘스트를 제출해 주세요. 우리는 업데이트와 새로운 아이디어를 사랑합니다—특히 AI와 새로운 기술에 관한 것들을!

# 요구사항

Flight은 PHP 7.4 이상이 필요합니다.

**참고:** PHP 7.4가 지원되는 이유는 현재 작성 시점(2024)에서 PHP 7.4가 일부 LTS Linux 배포판의 기본 버전이기 때문입니다. PHP >8로의 이동을 강제하면 해당 사용자들에게 많은 문제를 야기할 것입니다. 프레임워크는 PHP >8도 지원합니다.

# 라이선스

Flight은 [MIT](https://github.com/flightphp/core/blob/master/LICENSE) 라이선스로 배포됩니다.