# Tracy

TracyはFlightで使用できる素晴らしいエラーハンドラです。アプリケーションのデバッグに役立つ複数のパネルを備えています。また、非常に簡単に拡張でき、独自のパネルを追加することも可能です。Flightチームは、[flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions)プラグインを使用してFlightプロジェクト向けにいくつかのパネルを作成しました（Flight変数、DBクエリ、リクエスト、セッション、およびプロファイラープロファイルを渡す場合のオプションの**Twig**パネル—[Tracy Extensions](/awesome-plugins/tracy-extensions)を参照）。

## インストール

Composerでインストールします。また、Tracyには本番環境用のエラーハンドリングコンポーネントが付属しているため、dev版ではなく通常版をインストールすることをお勧めします。

```bash
composer require tracy/tracy
```

## 基本設定

開始するための基本的な設定オプションがあります。詳細については[Tracy Documentation](https://tracy.nette.org/en/configuring)をご覧ください。

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Tracyを有効化
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // 明示的に指定する必要がある場合もあります（Debugger::PRODUCTIONも同様）
// Debugger::enable('23.75.345.200'); // IPアドレスの配列を指定することもできます

// エラーと例外が記録される場所です。このディレクトリが存在し、書き込み可能であることを確認してください。
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // すべてのエラーを表示
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // 非推奨通知を除くすべてのエラー
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // デバッガーバーが表示されている場合、Flightはcontent-lengthを設定できません

	// 含めている場合はFlight用のTracy Extensionに固有です
	// それ以外の場合はコメントアウトしてください。
	new TracyExtensionLoader($app);
}
```

## 役立つヒント

コードをデバッグする際、データを表示するための非常に便利な関数があります。

- `bdump($var)` - 変数をTracy Barの別パネルにダンプします。
- `dumpe($var)` - 変数をダンプした後、すぐに終了します。