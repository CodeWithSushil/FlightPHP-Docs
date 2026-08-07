# HTMLビューとテンプレート

## 概要

Flightはデフォルトでいくつかの基本的なHTMLテンプレート機能を提供しています。テンプレートは、アプリケーションロジックをプレゼンテーション層から切り離すための非常に効果的な方法です。専用エンジン（Twig、Latteなど）は、[AIコーディングツール](/learn/ai)に馴染みのある制約された構文を提供するため、ビジネスロジックをHTMLに混ぜ込みにくくなります。

## 理解

アプリケーションを構築するとき、エンドユーザーに返したいHTMLがあるでしょう。PHP自体はテンプレート言語ですが、データベース呼び出しやAPI呼び出しなどのビジネスロジックをHTMLファイルに簡単に埋め込めてしまい、テストや疎結合化が非常に困難になります。データをテンプレートに渡し、テンプレート自体にレンダリングさせることで、コードの疎結合化とユニットテストがはるかに容易になります。テンプレートを使えば、きっと感謝されますよ！

## 基本的な使い方

Flightでは、`render`をマップする（またはビュークラスを登録する）だけで、デフォルトのビューエンジンを別のものに置き換えることができます。Twig、Latte、Smarty、Bladeなどの詳細は下にスクロールしてください。

> **スケルトンのデフォルト:** 公式の[flightphp/skeleton](https://github.com/flightphp/skeleton)は、`app/views/`（`*.twig`）配下で**Twigのみ**を使用します。コントローラーは`$this->app->render('welcome', $data)`を呼び出します（拡張子は省略可能）。これは新規プロジェクトのためのアプリケーション側の選択であり、Flightコアの要件ではありません。Latteやその他のエンジンも引き続き完全にサポートされています。

### Twig

<span class="badge bg-info">スケルトンのデフォルト</span>

[Twig](https://twig.symfony.com/)は、Symfonyや多くのPHPプロジェクトで使用されている、柔軟で高速かつ安全なテンプレートエンジンです。AIコーディングツールは特にTwigをよく知っており、デフォルトで出力を自動エスケープするためXSS対策にも役立ちます。

#### インストール

```bash
composer require twig/twig
```

（`composer create-project flightphp/skeleton`でインストールした場合は、すでに含まれています。）

#### 基本設定

`render`メソッドを上書きして、デフォルトのPHPレンダラーの代わりにTwigを使用します。

```php
// デフォルトのPHPレンダラーの代わりにTwigを使うようにrenderメソッドを上書きします
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Twigがコンパイル済みテンプレートを保存する場所
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// "welcome" または "welcome.twig" を許可します
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

スケルトンでは、この配線（設定）は`app/config/services.php`にあります（共有Twig環境、キャッシュパス、`base_url` / CSP nonceなどのグローバル変数）。コードを[AIフレンドリーかつテストフレンドリー](/learn/ai)に保つには、`Engine`を注入し、コントローラーから`$app->render()`を呼び出すことをお勧めします。

#### FlightでのTwigの使用

Twigでレンダリングできるようになったので、次のようにできます。

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

ブラウザで`/Bob`にアクセスすると、出力は次のようになります。

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### 詳細情報

Twigをレイアウトと一緒に使うより完全な例は、このドキュメントの[awesome plugins](/awesome-plugins/twig)セクションにあります。Tracyバーでレンダリング時間のメトリクスを確認するには、[Tracy ExtensionsのTwigパネル](/awesome-plugins/tracy-extensions#twig-panel-optional)を参照してください。

Twigの全機能については、[公式ドキュメント](https://twig.symfony.com/doc/3.x/)をご覧ください。

### Latte

<span class="badge bg-secondary">優れた代替案</span>

[Latte](https://latte.nette.org/)は、PHPに似た構文を持つフル機能のエンジンです。Flightアプリケーションにとっても優れた選択肢です。スケルトンは、共通のデフォルトとしてTwigを採用しているだけです（特にAIツールがテンプレートを生成する場合に便利です）。

#### インストール

```bash
composer require latte/latte
```

#### 基本設定

主なアイデアは、`render`メソッドを上書きして、デフォルトのPHPレンダラーの代わりにLatteを使用することです。

```php
// デフォルトのPHPレンダラーの代わりにLatteを使うようにrenderメソッドを上書きします
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Latteがキャッシュを保存する場所
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### FlightでのLatteの使用

Latteでレンダリングできるようになったので、次のようにできます。

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

ブラウザで`/Bob`にアクセスすると、出力は次のようになります。

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### 詳細情報

Latteをレイアウトと一緒に使うより複雑な例は、このドキュメントの[awesome plugins](/awesome-plugins/latte)セクションにあります。

翻訳や言語機能を含むLatteの全機能については、[公式ドキュメント](https://latte.nette.org/en/)をご覧ください。

### 組み込みビューエンジン

<span class="badge bg-warning">非推奨</span>

> **注:** これはまだデフォルトの機能であり、技術的にはまだ動作します。

ビューテンプレートを表示するには、テンプレートファイル名とオプションのテンプレートデータを指定して`render`メソッドを呼び出します。

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

渡したテンプレートデータは自動的にテンプレートに注入され、ローカル変数のように参照できます。テンプレートファイルは単なるPHPファイルです。`hello.php`テンプレートファイルの内容が次の場合:

```php
Hello, <?= $name ?>!
```

出力は次のようになります。

```text
Hello, Bob!
```

また、`set`メソッドを使用してビュー変数を手動で設定することもできます。

```php
Flight::view()->set('name', 'Bob');
```

これで、`name`変数はすべてのビューで使用できるようになります。したがって、単純に次のようにできます。

```php
Flight::render('hello');
```

renderメソッドでテンプレートの名前を指定するとき、`.php`拡張子は省略できることに注意してください。

デフォルトでは、Flightはテンプレートファイル用に`views`ディレクトリを探します。次の設定を行うことで、テンプレートの代替パスを設定できます。

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### レイアウト

Webサイトでは、コンテンツを差し替えられる単一のレイアウトテンプレートファイルを持つことが一般的です。レイアウトで使用するコンテンツをレンダリングするには、`render`メソッドにオプションのパラメータを渡します。

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

これで、ビューには`headerContent`および`bodyContent`という変数が保存されます。次に、次のようにしてレイアウトをレンダリングできます。

```php
Flight::render('layout', ['title' => 'Home Page']);
```

テンプレートファイルが次のようになっている場合:

`header.php`:

```php
<h1><?= $heading ?></h1>
```

`body.php`:

```php
<div><?= $body ?></div>
```

`layout.php`:

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

出力は次のようになります。
```html
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <div>World</div>
  </body>
</html>
```

### Smarty

ビューに[Smarty](http://www.smarty.net/)テンプレートエンジンを使用する方法は次のとおりです。

```php
// Smartyライブラリを読み込みます
require './Smarty/libs/Smarty.class.php';

// Smartyをビュークラスとして登録します
// また、読み込み時にSmartyを設定するコールバック関数を渡します
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// テンプレートデータを割り当てます
Flight::view()->assign('name', 'Bob');

// テンプレートを表示します
Flight::view()->display('hello.tpl');
```

完全を期すために、Flightのデフォルトのrenderメソッドも上書きする必要があります。

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

ビューに[Blade](https://laravel.com/docs/8.x/blade)テンプレートエンジンを使用する方法は次のとおりです。

まず、Composerを使用してBladeOneライブラリをインストールする必要があります。

```bash
composer require eftec/bladeone
```

次に、FlightでBladeOneをビュークラスとして設定できます。

```php
<?php
// BladeOneライブラリを読み込みます
use eftec\bladeone\BladeOne;

// BladeOneをビュークラスとして登録します
// また、読み込み時にBladeOneを設定するコールバック関数を渡します
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// テンプレートデータを共有します
Flight::view()->share('name', 'Bob');

// テンプレートを表示します
echo Flight::view()->run('hello', []);
```

完全を期すために、Flightのデフォルトのrenderメソッドも上書きする必要があります。

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

この例では、`hello.blade.php`テンプレートファイルは次のようになります。

```php
<?php
Hello, {{ $name }}!
```

出力は次のようになります。

```
Hello, Bob!
```

## 関連項目
- [インストール](/install) - 新規プロジェクト向けのスケルトンレイアウト（`app/views/*.twig`）。
- [拡張](/learn/extending) - 別のテンプレートエンジンを使用するために`render`メソッドを上書きする方法。
- [ルーティング](/learn/routing) - ルートをコントローラーにマップしてビューをレンダリングする方法。
- [レスポンス](/learn/responses) - HTTPレスポンスをカスタマイズする方法。
- [セキュリティ](/learn/security) - 自動エスケープとXSS。
- [AIと開発者体験](/learn/ai) - 単一のビューエンジンのデフォルトがコーディングエージェントに役立つ理由。
- [なぜフレームワークなのか？](/learn/why-frameworks) - テンプレートが全体像にどのように適合するか。

## トラブルシューティング
- ミドルウェアにリダイレクトがあるのに、アプリがリダイレクトされていないように見える場合は、ミドルウェアに`exit;`ステートメントを追加してください。
- Twigがテンプレートを見つけられない場合は、`flight.views.path`を確認し、そのパスに予期した拡張子のファイルが存在することを確認してください（スケルトン: `app/views/`）。

## 変更履歴
- ドキュメント - Twigが公式スケルトンのデフォルトとして記載されました。Latteは引き続き第一級の代替案です。
- v2.0 - 初回リリース。