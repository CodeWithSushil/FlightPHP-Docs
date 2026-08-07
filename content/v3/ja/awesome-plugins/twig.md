# Twig

[Twig](https://twig.symfony.com/) は、PHP向けの柔軟で高速かつ安全なテンプレートエンジンです。Symfonyやその他の多くのプロジェクトで使用されているテンプレート言語であり、AIコーディングツールや大多数のPHP開発者がその構文をよく理解しています。Twigはテンプレートを最適化されたPHPにコンパイルし、デフォルトで出力を自動エスケープ（XSS対策に有効）し、フィルタ、関数、拡張機能で簡単に拡張できます。

## インストール

composerでインストールします。

```bash
composer require twig/twig
```

## 基本設定

開始するための基本的な設定オプションがいくつかあります。詳細は[Twigドキュメント](https://twig.symfony.com/doc/3.x/)でご覧いただけます。

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Twigがコンパイル済みテンプレートを保存する場所
		'cache' => __DIR__ . '/../cache/twig',
		// ソースが変更されたときにテンプレートを再コンパイル（開発時に便利）
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Twigをビュークラスとして登録する

単一のTwig環境を再利用したい場合（本番環境で推奨）、それを登録して`render`をそれに向けることができます：

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

## シンプルなレイアウト例

これはレイアウトファイルの簡単な例です。このファイルは他のすべてのビューをラップするために使用されます。

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
				{# ここにナビゲーション要素を配置 #}
			</nav>
		</header>
		<div id="content">
			{# ここが魔法です #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

そして、コンテンツブロック内にレンダリングされるファイルを作成します：

```html
{# app/views/home.twig #}
{# これはTwigにこのファイルが「layout.twigの中にある」ことを伝えます #}
{% extends 'layout.twig' %}

{# これはコンテンツブロック内のレイアウト内にレンダリングされるコンテンツです #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

その後、関数またはコントローラー内でこれをレンダリングする場合、以下のようにします：

```php
// シンプルなルート
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// またはコントローラーを使用する場合
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

Twigを最大限に活用する方法の詳細については、[Twigドキュメント](https://twig.symfony.com/doc/3.x/)をご覧ください！

## デバッグ

Twigには、テンプレート内で使用できる`dump()`関数を追加する[デバッグ拡張機能](https://twig.symfony.com/doc/3.x/functions/dump.html)が付属しています。開発時のみ有効にします：

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // dump()関数に必要
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

テンプレート内では：

```html
{{ dump(user) }}
```

また、PHPレベルのデバッグ用に[Tracy](/awesome-plugins/tracy)とTwigを組み合わせることもできます。テンプレートレベルのメトリクス（レンダリング時間、メモリ、実行されたテンプレート/ブロック）については、[flightphp/tracy-extensions](/awesome-plugins/tracy-extensions)のオプションの**Twigパネル**を使用します：`Twig\Profiler\Profile`を`twig_profile`として`TracyExtensionLoader`に渡します。オプションの`TwigTracyExtension`は、Tracyがオンのときにテンプレート内で`{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}`を公開します。

## セキュリティに関する注意

Twigはデフォルトで出力を自動エスケープするため、XSS攻撃から保護するのに役立ちます。テキストには`{{ variable }}`を優先してください。HTMLコンテンツを意図的に信頼する場合（例：サーバーサイドで既に処理済みのサニタイズされたMarkdown）のみ、`|raw`フィルタを使用してください。