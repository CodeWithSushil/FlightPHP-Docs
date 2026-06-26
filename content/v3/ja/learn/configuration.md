# 設定

## 概要

Flightは、アプリケーションのニーズに合わせてフレームワークのさまざまな側面を設定する簡単な方法を提供します。一部はデフォルトで設定されていますが、必要に応じて上書きすることができます。また、アプリケーション全体で使用する独自の変数を設定することもできます。

## 理解

`set`メソッドを使用して設定値を設定することで、Flightの特定の動作をカスタマイズできます。

```php
Flight::set('flight.log_errors', true);
```

`app/config/config.php`ファイルでは、使用可能なすべてのデフォルト設定変数を確認できます。

## 基本的な使用方法

### Flight設定オプション

以下は、利用可能なすべての設定のリストです：

- **flight.base_url** `?string` - Flightがサブディレクトリで実行されている場合、リクエストのベースURLを上書きします。（デフォルト: null）
- **flight.case_sensitive** `bool` - URLの大文字小文字を区別したマッチング。（デフォルト: false）
- **flight.handle_errors** `bool` - Flightが内部ですべてのエラーを処理できるようにします。（デフォルト: true）
  - FlightにデフォルトのPHP動作ではなくエラーを処理させたい場合は、これをtrueにする必要があります。
  - [Tracy](/awesome-plugins/tracy)をインストールしている場合は、Tracyがエラーを処理できるようにfalseに設定してください。
  - [APM](/awesome-plugins/apm)プラグインをインストールしている場合は、APMがエラーをログに記録できるようにtrueに設定してください。
- **flight.log_errors** `bool` - Webサーバーのエラーログファイルにエラーを記録します。（デフォルト: false）
  - [Tracy](/awesome-plugins/tracy)をインストールしている場合、TracyはTracyの設定に基づいてエラーをログに記録し、この設定は使用されません。
- **flight.debug** `bool` - エラーが発生したときにブラウザに詳細なエラー情報（例外メッセージ、コード、スタックトレース）を出力します。（デフォルト: false）
  - **本番環境では絶対に有効にしないでください** — 内部アプリケーションの詳細が漏洩します。ローカル開発またはステージングでのみ使用してください。
  - `false`の場合、代わりに一般的な`500 Internal Server Error`が表示されます。エラーをサーバー側でキャプチャするために`flight.log_errors`と組み合わせます。
- **flight.allow_method_override** `bool` - `X-HTTP-Method-Override`リクエストヘッダーまたはPOSTボディの`_method`フィールドを介してHTTPメソッドを上書きできるようにします。（デフォルト: true）
  - HTMLフォームベースのメソッドスプーフィングを必要としないアプリケーションでは、`false`に設定することを**推奨**します。これにより、クライアントが標準のPOSTフォームを介して`DELETE`または`PUT`リクエストを偽造することを防ぎます。
  - 詳細については[セキュリティ](/learn/security#flight-configuration-hardening)を参照してください。
- **flight.views.path** `string` - ビューテンプレートファイルを含むディレクトリ。（デフォルト: ./views）
- **flight.views.extension** `string` - ビューテンプレートファイルの拡張子。（デフォルト: .php）
- **flight.content_length** `bool` - `Content-Length`ヘッダーを設定します。（デフォルト: true）
  - [Tracy](/awesome-plugins/tracy)を使用している場合は、Tracyが正しくレンダリングできるようにfalseに設定する必要があります。
- **flight.v2.output_buffering** `bool` - レガシー出力バッファリングを使用します。[v3への移行](migrating-to-v3)を参照してください。（デフォルト: false）

### ローダー設定

ローダー用の追加の設定もあります。これにより、クラス名に`_`を含むクラスをオートロードできます。

```php
// アンダースコアを使用したクラスの読み込みを有効にする
// デフォルトはtrue
Loader::$v2ClassLoading = false;
```

### 変数

Flightでは、アプリケーションのどこでも使用できるように変数を保存できます。

```php
// 変数を保存する
Flight::set('id', 123);

// アプリケーションの別の場所で
$id = Flight::get('id');
```
変数が設定されているかどうかを確認するには：

```php
if (Flight::has('id')) {
  // 何かする
}
```

変数をクリアするには：

```php
// id変数をクリアする
Flight::clear('id');

// すべての変数をクリアする
Flight::clear();
```

> **注:** 変数を設定できるからといって、必ずしも設定すべきではありません。この機能は控えめに使用してください。その理由は、ここに保存されたものはすべてグローバル変数になるからです。グローバル変数は、アプリケーションのどこからでも変更できるため、バグの追跡が困難になるため悪いです。また、[ユニットテスト](/guides/unit-testing)などの作業も複雑になる可能性があります。

### エラーと例外

`flight.handle_errors`がtrueに設定されている場合、すべてのエラーと例外はFlightによってキャッチされ、`error`メソッドに渡されます。

デフォルトの動作は、いくつかのエラー情報を含む一般的な`HTTP 500 Internal Server Error`レスポンスを送信することです。

必要に応じて[上書き](/learn/extending)できます：

```php
Flight::map('error', function (Throwable $error) {
  // エラーを処理する
  echo $error->getTraceAsString();
});
```

デフォルトでは、エラーはWebサーバーに記録されません。以下のように設定を変更することでこれを有効にできます：

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

URLが見つからない場合、Flightは`notFound`メソッドを呼び出します。デフォルトの動作は、簡単なメッセージを含む`HTTP 404 Not Found`レスポンスを送信することです。

必要に応じて[上書き](/learn/extending)できます：

```php
Flight::map('notFound', function () {
  // 見つからない場合の処理
});
```

## 関連項目
- [Flightの拡張](/learn/extending) - Flightのコア機能を拡張およびカスタマイズする方法。
- [ユニットテスト](/guides/unit-testing) - Flightアプリケーションのユニットテストの書き方。
- [Tracy](/awesome-plugins/tracy) - 高度なエラーハンドリングとデバッグ用のプラグイン。
- [Tracy拡張機能](/awesome-plugins/tracy_extensions) - TracyをFlightと統合するための拡張機能。
- [APM](/awesome-plugins/apm) - アプリケーションパフォーマンス監視とエラートラッキング用のプラグイン。

## トラブルシューティング
- 設定のすべての値を見つけるのに問題がある場合は、`var_dump(Flight::get());`を実行できます。

## 変更履歴
- v3.18.1 - `flight.debug`および`flight.allow_method_override`設定オプションを追加。
- v3.5.0 - レガシー出力バッファリング動作をサポートするために`flight.v2.output_buffering`の設定を追加。
- v2.0 - コア設定を追加。