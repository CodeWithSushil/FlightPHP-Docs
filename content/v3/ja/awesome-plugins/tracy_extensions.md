# Tracy Flight パネル拡張

これはFlightの作業をより豊かにするための拡張機能セットです。

- **Flight** - すべてのFlight変数を分析。
- **Database** - ページで実行されたすべてのクエリを分析（データベース接続を正しく初期化した場合）
- **Request** - すべての`$_SERVER`変数を分析し、すべてのグローバルペイロード（`$_GET`、`$_POST`、`$_FILES`）を調査
- **Session** - セッションがアクティブな場合、すべての`$_SESSION`変数を分析。
- **Twig** *(オプション)* - Twigテンプレートのレンダリング時間、メモリ、およびどのテンプレート/ブロック/マクロが実行されたかを分析（`twig/twig`と`twig_profile`設定が必要）

これは特に[official skeleton](https://github.com/flightphp/skeleton)で便利で、デフォルトでTwigを使用しています：同じレイアウト[AIツール](/learn/ai)に従うと、Tracyバーにも明確に表示されます。

これがパネルです

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

各パネルにはアプリケーションに関する非常に役立つ情報が表示されます！

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

コードを表示するには[こちら](https://github.com/flightphp/tracy-extensions)をクリックしてください。

## インストール

`composer require flightphp/tracy-extensions --dev`を実行すれば準備完了です！

Twigはパッケージのハード依存関係ではありません。Twigパネルが必要な場合のみ`twig/twig`をインストールしてください（スケルトンでは既にビュー用にインストールされています）。

## 設定

これを開始するために必要な設定はほとんどありません。Tracyデバッガーをこの[https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide)を使用する前に初期化する必要があります：

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// bootstrapコード
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// 環境を指定する必要があるかもしれません：Debugger::enable(Debugger::DEVELOPMENT)

// アプリでデータベース接続を使用する場合、
// 開発専用（本番環境では使用しないでください！）に使用する
// 必要なPDOラッパーがあります
// 通常のPDO接続と同じパラメータを持っています
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// またはFlightフレームワークにこれをアタッチする場合
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// これでクエリを実行するたびに時間、クエリ、パラメータがキャプチャされます

// ドットを接続します
if(Debugger::$showBar === true) {
	// Tracyが実際にレンダリングできないため、これをfalseにする必要があります :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// より多くのコード

Flight::start();
```

## 追加設定

### セッションデータ

ghostff/sessionのようなカスタムセッションハンドラを使用している場合、任意のセッションデータ配列をTracyに渡すことができ、自動的に出力されます。`TracyExtensionLoader`コンストラクタの2番目のパラメータの`session_data`キーで渡します。

```php

use Ghostff\Session\Session;
// またはflight\Sessionを使用;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Tracyが実際にレンダリングできないため、これをfalseにする必要があります :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// ルートやその他のもの...

Flight::start();
```

### Twigパネル（オプション）

アプリが[Twig](/awesome-plugins/twig)を使用している場合（公式スケルトンを含む）、Tracyバーにテンプレートメトリクスを表示できます。Twigの`Profile`を作成し、`ProfilerExtension`を環境にアタッチし、そのプロファイルを`twig_profile`キーの下のローダーに渡します。開発時のみプロファイリングをアタッチしてください。

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// オプション：テンプレートでTracyダンプヘルパーを公開
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Flight::render()をTwigにマッピング（例）
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**パネルに表示される内容**

- Twigの総レンダリング時間とメモリ
- テンプレート/ブロック/マクロの呼び出し回数
- レンダリングされた各テンプレートとその時間とメモリ

Twigタブは、リクエストでテンプレートがレンダリングされなかった場合、または`twig_profile`を省略した場合（またはTwigがインストールされていない場合）に**非表示**になります - 他のFlightパネルは引き続き動作します。

skeletonスタイルの`services.php`では、デバッグがオンの時に同じ`$profile`/`ProfilerExtension`を構築し、`twig_profile`を`TracyExtensionLoader`に渡し、`$app->render()`に共有Twig環境を使用し続けます。

### Latte

_このセクションではPHP 8.1+が必要です。_

プロジェクトにLatteがインストールされている場合、Tracyにはテンプレートを分析するためのLatteとのネイティブ統合があります。Latteインスタンスに拡張機能を登録するだけです（これは上記のTwigパネルではなく、Latte独自のTracyブリッジです）。

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// その他の設定...

	// Tracyデバッグバーが有効な場合のみ拡張機能を追加
	if(Debugger::$showBar === true) {
		// ここでLatteパネルをTracyに追加
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## 関連項目

- [Tracy](/awesome-plugins/tracy) - Flight用の基本Tracyセットアップ
- [Twig](/awesome-plugins/twig) - スケルトンとTwigパネルで使用されるテンプレートエンジン
- [Templates](/learn/templates) - Flightが`render`をTwig/Latteにマッピングする方法
- [Installation](/install) - スケルトンにはdevにtracy-extensionsが含まれています