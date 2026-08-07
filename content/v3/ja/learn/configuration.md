# 設定

## 概要

Flightは、アプリケーションのニーズに合わせてフレームワークのさまざまな側面を設定する簡単な方法を提供します。一部はデフォルトで設定されていますが、必要に応じて上書きできます。また、アプリケーション全体で使用する独自の変数を設定することもできます。

明確で階層化された設定（ファイルのデフォルト + 環境シークレット）は、[AIコーディングツール](/learn/ai)にも役立ちます。エージェントは、コントローラー内で `$_ENV` 読み取りを独自に作り出す代わりに、リテラルのための場所とシークレットのための場所を1つずつ学ぶことができます。

## 理解

Flightの特定の動作は、`set`メソッドを通じて設定値を設定することでカスタマイズできます。

```php
Flight::set('flight.log_errors', true);
```

構造化されたアプリ（[スケルトン](https://github.com/flightphp/skeleton)を含む）では、通常 `app/config/config.php` からプロジェクト設定を読み込み、関連するキーをエンジンに適用します（例: `flight.base_url`、`flight.views.path`）。また、グローバルをあちこちで読み取る代わりに、小さな設定オブジェクトをコントローラーに注入することもできます。これにより、テストや `AGENTS.md` に従うエージェントにとってより親しみやすくなります。

## 基本的な使い方

### Flight設定オプション

以下は、利用可能なすべての設定項目のリストです。

- **flight.base_url** `?string` - Flightがサブディレクトリで実行されている場合、リクエストのベースURLを上書きします。（デフォルト: null）
- **flight.case_sensitive** `bool` - URLの大文字小文字を区別したマッチングを行います。（デフォルト: false）
- **flight.handle_errors** `bool` - Flightがすべてのエラーを内部的に処理できるようにします。（デフォルト: true）
  - デフォルトのPHPの動作ではなくFlightにエラーを処理させたい場合は、これをtrueにする必要があります。
  - [Tracy](/awesome-plugins/tracy)をインストールしている場合は、Tracyがエラーを処理できるようにこれをfalseに設定します。
  - [APM](/awesome-plugins/apm)プラグインをインストールしている場合は、APMがエラーをログに記録できるようにこれをtrueに設定します。
- **flight.log_errors** `bool` - エラーをWebサーバーのエラーログファイルに記録します。（デフォルト: false）
  - [Tracy](/awesome-plugins/tracy)をインストールしている場合、Tracyはこの設定ではなくTracyの設定に基づいてエラーを記録します。
- **flight.debug** `bool` - エラー発生時に、詳細なエラー情報（例外メッセージ、コード、スタックトレース）をブラウザに出力します。（デフォルト: false）
  - **本番環境では絶対に有効にしないでください** — 内部のアプリケーション詳細が漏洩します。ローカル開発またはステージング環境でのみ使用してください。
  - `false`の場合、代わりに一般的な `500 Internal Server Error` が表示されます。サーバー側でエラーを記録するには、`flight.log_errors` と組み合わせてください。
- **flight.allow_method_override** `bool` - `X-HTTP-Method-Override`リクエストヘッダーまたはPOST本文の`_method`フィールドを介してHTTPメソッドを上書きできるようにします。（デフォルト: true）
  - HTMLフォームベースのメソッド偽装を必要としないアプリケーションでは、**これを`false`に設定することをお勧めします**。これにより、クライアントが標準のPOSTフォームを介して`DELETE`や`PUT`リクエストを偽装することを防ぎます。
  - 詳細については、[セキュリティ](/learn/security#flight-configuration-hardening)を参照してください。
- **flight.views.path** `string` - ビューテンプレートファイルを含むディレクトリ。（デフォルト: ./views）
- **flight.views.extension** `string` - ビューテンプレートファイルの拡張子。（デフォルト: `.php`。公式スケルトンではTwigを使用する場合、これを`.twig`に設定します）
- **flight.content_length** `bool` - `Content-Length`ヘッダーを設定します。（デフォルト: true）
  - [Tracy](/awesome-plugins/tracy)を使用している場合、Tracyが正しくレンダリングされるようにこれをfalseに設定する必要があります。
- **flight.v2.output_buffering** `bool` - レガシー出力バッファリングを使用します。[v3への移行](migrating-to-v3)を参照してください。（デフォルト: false）

### ローダー設定

ローダーにはもう1つの設定項目があります。これにより、クラス名に`_`を含むクラスをオートロードできます。

```php
// アンダースコアを使用したクラス読み込みを有効にする
// デフォルトはtrue
Loader::$v2ClassLoading = false;
```

[オートローディング](/learn/autoloading)は、名前空間と一致する**フォルダーの大文字小文字**にも依存することを忘れないでください。特にスケルトンの `App\` + `app/Controller/` レイアウトでは重要です。

### プロジェクト設定と`.env`（スケルトンパターン）

Flightのコアは`.env`ファイルを必要としません。多くのアプリはPHPの設定配列のみを使用します。公式スケルトンは設定を階層化しているため、シークレットをgitの管理外に保ちながら、Runwayが**リテラル**設定を安全に書き換えることができます。

1. **`.env` / 実際の環境** — シークレットとデプロイ時の上書き（gitignoreされます）。
2. **`app/config/config.php`** — リテラルなPHP配列のデフォルト（`config_sample.php`からコピー）。このファイル内では **`$_ENV[...]` 式を使わない**ことをお勧めします。`runway config:set` のようなツールはこれを静的値として書き換え、シークレットをファイルに焼き付ける可能性があります。
3. **ブートストラップ時にマージ** — マッピングされたキーでは環境変数が優先されます。アプリコードはコントローラー内の`$_ENV`ではなく、設定オブジェクトまたは`$app->get()`を読み取ります。

`config_sample.php` / `config.php` の例（簡略版）:

```php
<?php
// リテラルのみ — シークレットはスケルトンワークフローでは .env に置く
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // または mysql、または無効にする場合は ''
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env（スケルトン）
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

この分割は、[AIフレンドリーなプロジェクト](/learn/ai)のために意図的に行われています。手順書には「デフォルトは `config.php`、シークレットは `.env`、Config / Engine を注入し、コントローラーで env アクセスを独自に作らないこと」と記載できます。既存のアプリは `.env` を完全に無視して、単一の設定ファイルを維持することもできます。

### 変数

Flightを使用すると、アプリケーションのどこでも使用できる変数を保存できます。

```php
// 変数を保存
Flight::set('id', 123);

// アプリケーション内の別の場所
$id = Flight::get('id');
```

変数が設定されているかどうかを確認するには、次のようにします。

```php
if (Flight::has('id')) {
  // 何かを行う
}
```

変数をクリアするには、次のようにします。

```php
// id変数をクリア
Flight::clear('id');

// すべての変数をクリア
Flight::clear();
```

> **注:** 変数を設定できるからといって、それを使うべきとは限りません。この機能は控えめに使用してください。ここに保存されたものはすべてグローバル変数になるためです。グローバル変数は、アプリケーションのどこからでも変更できるため、バグの追跡が難しくなります。さらに、[ユニットテスト](/guides/unit-testing)などを複雑にする可能性があります。コントローラーが必要とするサービスや設定には、コンストラクター注入（スケルトン + Dice設定のように）を優先してください。

### エラーと例外

すべてのエラーと例外はFlightによってキャッチされ、`error`メソッドに渡されます（`flight.handle_errors`がtrueに設定されている場合）。

デフォルトの動作は、いくつかのエラー情報を含む一般的な `HTTP 500 Internal Server Error` レスポンスを送信することです。

この動作は、必要に応じて[上書き](/learn/extending)できます。

```php
Flight::map('error', function (Throwable $error) {
  // エラーを処理
  echo $error->getTraceAsString();
});
```

デフォルトでは、エラーはWebサーバーに記録されません。設定を変更することで有効にできます。

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

URLが見つからない場合、Flightは`notFound`メソッドを呼び出します。デフォルトの動作は、簡単なメッセージを含む`HTTP 404 Not Found`レスポンスを送信することです。

この動作は、必要に応じて[上書き](/learn/extending)できます。

```php
Flight::map('notFound', function () {
  // 見つからない場合の処理
});
```

## 関連項目
- [インストール](/install) - スケルトン設定、`.env`、ブートストラップの構成。
- [オートローディング](/learn/autoloading) - 名前空間とフォルダーの大文字小文字。
- [Flightの拡張](/learn/extending) - Flightのコア機能を拡張およびカスタマイズする方法。
- [ユニットテスト](/guides/unit-testing) - Flightアプリケーションのユニットテストの書き方。
- [AIと開発者エクスペリエンス](/learn/ai) - `AGENTS.md`と一貫したプロジェクト指示。
- [Tracy](/awesome-plugins/tracy) - 高度なエラー処理とデバッグのためのプラグイン。
- [Tracy拡張機能](/awesome-plugins/tracy_extensions) - TracyをFlightと統合するための拡張機能。
- [APM](/awesome-plugins/apm) - アプリケーションパフォーマンス監視とエラートラッキングのためのプラグイン。
- [セキュリティ](/learn/security) - セキュリティ強化フラグとシークレットの取り扱い。

## トラブルシューティング
- 設定のすべての値を確認するのに問題がある場合は、`var_dump(Flight::get());`を実行できます。
- Runwayまたはデプロイツールが`config.php`を書き換えた場合は、シークレットがコミットされていないことを確認してください。スケルトンパターンを使用する場合は、シークレットを`.env`または実際の環境に保持してください。

## 変更履歴
- ドキュメント – スケルトンスタイルの設定 / `.env`の階層化と、新しいプロジェクト向けのTwigビュー拡張子のデフォルトを文書化。
- v3.18.1 - `flight.debug`および`flight.allow_method_override`設定オプションを追加。
- v3.5.0 - レガシー出力バッファリング動作をサポートするための`flight.v2.output_buffering`設定を追加。
- v2.0 - コア設定を追加。