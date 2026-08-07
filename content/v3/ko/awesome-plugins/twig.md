# Twig

[Twig](https://twig.symfony.com/)는 PHP를 위한 유연하고 빠르며 안전한 템플릿 엔진입니다. Symfony와 많은 다른 프로젝트에서 사용되는 템플릿 언어로, AI 코딩 도구와 대부분의 PHP 개발자들이 이미 그 문법에 익숙합니다. Twig는 템플릿을 최적화된 PHP로 컴파일하고, 기본적으로 출력 자동 이스케이프를 지원하여(XSS 보호에 유용) 필터, 함수, 확장을 통해 쉽게 확장할 수 있습니다.

## 설치

컴포저로 설치합니다.

```bash
composer require twig/twig
```

## 기본 설정

시작하기 위한 몇 가지 기본 설정 옵션이 있습니다. 이에 대한 자세한 내용은 [Twig 문서](https://twig.symfony.com/doc/3.x/)에서 확인할 수 있습니다.

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Twig가 컴파일된 템플릿을 저장하는 위치
		'cache' => __DIR__ . '/../cache/twig',
		// 소스가 변경될 때 템플릿을 다시 컴파일(개발 환경에 유용)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Twig를 뷰 클래스로 등록

단일 Twig 환경을 재사용하려면(프로덕션에 권장), 이를 등록하고 `render`가 해당 환경을 가리키도록 합니다:

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	],
]);

$app->map('render', function(string $template, array $data): void {
	echo Flight::view()->render($template, $data);
});
```

## 간단한 레이아웃 예제

다음은 레이아웃 파일의 간단한 예제입니다. 이 파일은 다른 모든 뷰를 감싸는 데 사용됩니다.

```html
{# app/views/layout.twig #}
<!doctype html>
<html lang="en">
	<head>
		<title>{% if title %}{{ title }} - {% endif %}My App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				{# 여기에 네비게이션 요소를 추가하세요 #}
			</nav>
		</header>
		<div id="content">
			{# 여기가 핵심입니다 #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

이제 해당 콘텐츠 블록 안에 렌더링할 파일입니다:

```html
{# app/views/home.twig #}
{# 이 파일이 "layout.twig" 파일 안에 있음을 Twig에 알립니다 #}
{% extends 'layout.twig' %}

{# 콘텐츠 블록 안에 렌더링될 내용입니다 #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

그리고 함수나 컨트롤러에서 이를 렌더링하려면 다음과 같이 합니다:

```php
// 간단한 라우트
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// 컨트롤러를 사용하는 경우
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Home Page'
		]);
	}
}
```

Twig를 최대한 활용하는 방법에 대한 자세한 내용은 [Twig 문서](https://twig.symfony.com/doc/3.x/)를 참조하세요!

## 디버깅

Twig에는 템플릿 내에서 사용할 수 있는 `dump()` 함수를 추가하는 [디버그 확장](https://twig.symfony.com/doc/3.x/functions/dump.html)이 포함되어 있습니다. 개발 환경에서만 활성화하세요:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // dump() 함수에 필요
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

그런 다음 템플릿에서:

```html
{{ dump(user) }}
```

또한 Twig를 [Tracy](/awesome-plugins/tracy)와 함께 사용하여 PHP 수준 디버깅을 수행할 수도 있습니다. 템플릿 수준 메트릭(렌더링 시간, 메모리, 실행된 템플릿/블록)을 위해 [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions)의 선택적 **Twig 패널**을 사용하세요: `twig_profile`로 `Twig\Profiler\Profile`을 `TracyExtensionLoader`에 전달하세요. 선택적 `TwigTracyExtension`은 Tracy가 켜져 있을 때 템플릿에서 `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}`를 노출합니다.

## 보안 참고 사항

Twig는 기본적으로 출력을 자동 이스케이프하여 XSS 공격으로부터 보호합니다. 텍스트에는 `{{ variable }}`을 사용하세요. HTML 콘텐츠를 의도적으로 신뢰하는 경우(예: 서버 측에서 이미 처리한 정제된 마크다운)에만 `|raw` 필터를 사용하세요.