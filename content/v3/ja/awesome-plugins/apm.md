# FlightPHP APM ドキュメント

FlightPHP APMへようこそ—あなたのアプリ専用のパフォーマンスコーチです！このガイドは、FlightPHPを使ったアプリケーションパフォーマンスモニタリング（APM）のセットアップ、使用、習得への道筋です。遅いリクエストの調査やレイテンシチャートの分析など、私たちがサポートします。アプリをより速く、ユーザーをより満足させ、デバッグセッションをスムーズにしましょう！

Flight Docsサイトの[demo](https://flightphp-docs-apm.sky-9.com/apm/dashboard)を表示します。

![FlightPHP APM](/images/apm.png)

## APMが重要な理由

あなたのアプリが忙しいレストランだと想像してみてください。注文にかかる時間やキッチンの混雑状況を追跡する方法がなければ、なぜ顧客が不機嫌に帰ってしまうのかを推測することになります。APMはあなたのスーシェフのようなもので、着信リクエストからデータベースクエリまで、すべてのステップを監視し、遅延の原因を特定します。ページの表示が遅いとユーザーを失います（調査によると、サイトの読み込みに3秒以上かかると53%が離脱します！）が、APMは問題が深刻化する*前*に発見するのに役立ちます。これは積極的な安心感—「なぜこれが壊れているの？」という瞬間を減らし、「これがどれだけスムーズに動いているか！」という成功体験を増やします。

## インストール

Composerで始めましょう：

```bash
composer require flightphp/apm
```

必要なもの：
- **PHP 7.4+**: 最新のPHPをサポートしながら、LTS Linuxディストリビューションとの互換性を維持します。
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: 私たちが強化している軽量フレームワーク。

## サポートされるデータベース

FlightPHP APMは現在、メトリクスを保存するために以下のデータベースをサポートしています：

- **SQLite3**: シンプルでファイルベースで、ローカル開発や小規模アプリに最適。ほとんどのセットアップでデフォルトオプション。
- **MySQL/MariaDB**: 堅牢でスケーラブルなストレージが必要な大規模プロジェクトや本番環境に最適。

設定ステップ（以下参照）でデータベースの種類を選択できます。PHP環境に必要な拡張機能がインストールされていることを確認してください（例：`pdo_sqlite` または `pdo_mysql`）。

## はじめに

APMの素晴らしさへのステップバイステップ：

### 1. APMの登録

トラッキングを開始するために、`index.php` または `services.php` ファイルに以下を追加します：

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// データベース接続を追加する場合
// SimplePdo（または開発環境ではTracy ExtensionsのPdoQueryCapture）を推奨。
// オプション配列（5番目の引数）でAPMクエリトラッキングを有効化。
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // APM用のクエリキャプチャに必要
]);
$Apm->addPdoConnection($pdo);
```

**ここで何が起こっているか？**
- `LoggerFactory::create()` は設定（後述）を取得してロガーをセットアップします—デフォルトでSQLite。
- `Apm` が主役です—Flightのイベント（リクエスト、ルート、エラーなど）をリッスンしてメトリクスを収集します。
- `bindEventsToFlightInstance($app)` はすべてをあなたのFlightアプリに結びつけます。

**プロのヒント: サンプリング**
アプリが忙しい場合、*すべての*リクエストをログに記録するとシステムに負荷がかかる可能性があります。サンプルレート（0.0から1.0）を使用してください：

```php
$Apm = new Apm($ApmLogger, 0.1); // 10%のリクエストをログに記録
```

これによりパフォーマンスを維持しながら、信頼性の高いデータを提供します。

### 2. 設定

`.runway-config.json` を作成するために以下を実行します：

```bash
php vendor/bin/runway apm:init
```

**これは何をするか？**
- 生のメトリクスのソースと処理されたデータの宛先を尋ねるウィザードを起動します。
- デフォルトはSQLite—例：ソース用に `sqlite:/tmp/apm_metrics.sqlite`、宛先用にもう一つ。
- 以下のような設定が作成されます：
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> このプロセスでは、このセットアップのためのマイグレーションを実行するかどうかも尋ねられます。初めて設定する場合は、はいと答えてください。

**なぜ2つの場所が必要か？**
生のメトリクスは急速に蓄積されます（フィルタリングされていないログのように）。ワーカーはそれらをダッシュボード用の構造化された宛先に処理します。整理された状態を維持します！

### 3. ワーカーでメトリクスを処理

ワーカーは生のメトリクスをダッシュボード対応データに変換します。一度実行します：

```bash
php vendor/bin/runway apm:worker
```

**何をしているか？**
- ソース（例：`apm_metrics.sqlite`）から読み取ります。
- 最大100のメトリクス（デフォルトのバッチサイズ）を宛先に処理します。
- 完了するか、メトリクスが残っていない場合に停止します。

**継続的な実行**
ライブアプリの場合、継続的な処理が必要です。オプションは以下の通りです：

- **デーモンモード**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  永久に実行し、メトリクスが来るたびに処理します。開発や小規模なセットアップに最適。

- **Crontab**:
  crontab（`crontab -e`）に以下を追加：
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  毎分実行—本番環境に最適。

- **Tmux/Screen**:
  分離可能なセッションを開始：
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, then D to detach; `tmux attach -t apm-worker` to reconnect
  ```
  ログアウトしても実行を維持します。

- **カスタム調整**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: 一度に50のメトリクスを処理。
  - `--max_messages 1000`: 1000のメトリクス後に停止。
  - `--timeout 300`: 5分後に終了。

**なぜ重要か？**
ワーカーがなければ、ダッシュボードは空です。生のログと実用的な洞察の間の橋渡しです。

### 4. ダッシュボードの起動

アプリの状態を確認：

```bash
php vendor/bin/runway apm:dashboard
```

**これは何をするか？**
- `http://localhost:8001/apm/dashboard` でPHPサーバーを起動します。
- リクエストログ、遅いルート、エラー率などを表示します。

**カスタマイズ**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: 任意のIPからアクセス可能（リモート表示に便利）。
- `--port 8080`: 8001が使用中の場合に異なるポートを使用。
- `--php-path`: PHPがPATHにない場合に指定。

ブラウザでURLを開いて探索してください！

#### 本番モード

本番環境では、ファイアウォールやその他のセキュリティ対策があるため、ダッシュボードを実行するためにいくつかのテクニックを試す必要があるかもしれません。いくつかのオプション：

- **リバースプロキシの使用**: NginxまたはApacheをセットアップしてリクエストをダッシュボードに転送。
- **SSHトンネル**: サーバーにSSHできる場合、`ssh -L 8080:localhost:8001
youruser@yourserver` を使用してダッシュボードをローカルマシンにトンネル。
- **VPN**: サーバーがVPNの背後にある場合、VPNに接続してダッシュボードに直接アクセス。
- **ファイアウォールの設定**: あなたのIPまたはサーバーのネットワーク用にポート8001を開く。（または設定したポート）。
- **Apache/Nginxの設定**: アプリケーションの前にウェブサーバーがある場合、ドメインまたはサブドメインに設定できます。その場合、ドキュメントルートを `/path/to/your/project/vendor/flightphp/apm/dashboard` に設定します。

#### 異なるダッシュボードが必要ですか？

独自のダッシュボードを構築できます！独自のダッシュボード用のデータの表示方法については、vendor/flightphp/apm/src/apm/presenterディレクトリを参照してください！

## ダッシュボードの機能

ダッシュボードはAPMの本部です—ここで確認できる内容：

- **リクエストログ**: タイムスタンプ、URL、レスポンスコード、総時間を含むすべてのリクエスト。「詳細」をクリックしてミドルウェア、クエリ、エラーを表示。
- **最も遅いリクエスト**: 時間を消費しているトップ5のリクエスト（例：「/api/heavy」が2.5秒）。
- **最も遅いルート**: 平均時間によるトップ5のルート—パターンの発見に最適。
- **エラー率**: 失敗したリクエストの割合（例：2.3%の500エラー）。
- **レイテンシパーセンタイル**: 95パーセンタイル（p95）と99パーセンタイル（p99）のレスポンスタイム—最悪のケースを把握。
- **レスポンスコードチャート**: 時間の経過に伴う200、404、500の可視化。
- **長いクエリ/ミドルウェア**: トップ5の遅いデータベース呼び出しとミドルウェアレイヤー。
- **キャッシュヒット/ミス**: キャッシュが役立つ頻度。

**その他の機能**:
- 「過去1時間」「過去1日」「過去1週間」でフィルタリング。
- 深夜のセッション用のダークモード切り替え。

**例**:
`/users` へのリクエストは以下を表示する可能性があります：
- 総時間: 150ms
- ミドルウェア: `AuthMiddleware->handle` (50ms)
- クエリ: `SELECT * FROM users` (80ms)
- キャッシュ: `user_list` でヒット (5ms)

## カスタムイベントの追加

API呼び出しや支払い処理など、任意のものを追跡：

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**どこに表示されるか？**
ダッシュボードのリクエスト詳細の「カスタムイベント」セクション—見やすいJSONフォーマットで展開可能。

**ユースケース**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
これでそのAPIがアプリを遅くしているかどうかがわかります！

## データベースモニタリング

以下のようにPDOクエリを追跡：

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // APM用のクエリキャプチャに必要
]);
$Apm->addPdoConnection($pdo);
```

**取得できる内容**:
- クエリテキスト（例：`SELECT * FROM users WHERE id = ?`）
- 実行時間（例：0.015秒）
- 行数（例：42）

**注意**:
- **オプション**: DBトラッキングが必要ない場合はスキップ可能。
- **SimplePdo（推奨）**: `trackApmQueries => true` で`SimplePdo`を使用。非推奨の`PdoWrapper`も動作します（5番目のコンストラクタ引数で`true`）。生のコアPDOはまだフックされていません—お楽しみに！
- **パフォーマンス警告**: DB負荷の高いサイトで毎回のクエリをログに記録するとパフォーマンスが低下する可能性があります。サンプリング（`$Apm = new Apm($ApmLogger, 0.1)`）を使用して負荷を軽減してください。

**出力例**:
- クエリ: `SELECT name FROM products WHERE price > 100`
- 時間: 0.023秒
- 行数: 15

## ワーカーオプション

好みに合わせてワーカーを調整：

- `--timeout 300`: 5分後に停止—テストに適しています。
- `--max_messages 500`: 500メトリクスでキャップ—有限に保ちます。
- `--batch_size 200`: 一度に200を処理—速度とメモリのバランス。
- `--daemon`: ノンストップで実行—ライブモニタリングに最適。

**例**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
1時間実行し、一度に100のメトリクスを処理します。

## アプリ内のリクエストID

各リクエストには追跡用のユニークなリクエストIDがあります。アプリ内でこのIDを使用してログとメトリクスを関連付けることができます。例えば、エラーページにリクエストIDを追加できます：

```php
Flight::map('error', function($message) {
	// レスポンスヘッダーX-Flight-Request-IdからリクエストIDを取得
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// さらにFlight変数から取得することも可能
	// この方法はswooleやその他の非同期プラットフォームではうまく動作しません。
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## アップグレード

APMの新しいバージョンにアップグレードする場合、実行する必要があるデータベースマイグレーションがある可能性があります。以下のコマンドを実行することで実行できます：

```bash
php vendor/bin/runway apm:migrate
```
これにより、データベーススキーマを最新バージョンに更新するために必要なマイグレーションが実行されます。

**注意:** APMデータベースのサイズが大きい場合、これらのマイグレーションには時間がかかる可能性があります。オフピーク時間にこのコマンドを実行することをお勧めします。

### 0.4.3から0.5.0へのアップグレード

0.4.3から0.5.0にアップグレードする場合、以下のコマンドを実行する必要があります：

```bash
php vendor/bin/runway apm:config-migrate
```

これにより、古い形式の`.runway-config.json`ファイルを使用した設定を、新しい形式の`config.php`ファイルにキーと値を保存する形式に移行します。

## 古いデータの削除

データベースを整理するために、古いデータを削除できます。忙しいアプリを実行していてデータベースサイズを管理したい場合に特に便利です。以下のコマンドを実行することで実行できます：

```bash
php vendor/bin/runway apm:purge
```
これにより、30日以上前のすべてのデータがデータベースから削除されます。`--days`オプションに異なる値を渡すことで、日数を調整できます：

```bash
php vendor/bin/runway apm:purge --days 7
```
これにより、7日以上前のすべてのデータがデータベースから削除されます。

## トラブルシューティング

困った場合は以下を試してください：

- **ダッシュボードにデータがない場合？**
  - ワーカーは実行されていますか？`ps aux | grep apm:worker`で確認。
  - 設定パスは一致していますか？`.runway-config.json`のDSNが実際のファイルを指しているか確認。
  - `php vendor/bin/runway apm:worker`を手動で実行して保留中のメトリクスを処理。

- **ワーカーエラーの場合？**
  - SQLiteファイルを確認（例：`sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`）。
  - PHPログでスタックトレースを確認。

- **ダッシュボードが起動しない場合？**
  - ポート8001が使用中ですか？`--port 8080`を使用。
  - PHPが見つかりませんか？`--php-path /usr/bin/php`を使用。
  - ファイアウォールでブロックされていますか？ポートを開くか、`--host localhost`を使用。

- **遅すぎる場合？**
  - サンプルレートを下げる：`$Apm = new Apm($ApmLogger, 0.05)` (5%)。
  - バッチサイズを減らす：`--batch_size 20`。

- **例外/エラーが追跡されない場合？**
  - プロジェクトで[Tracy](https://tracy.nette.org/)が有効になっている場合、Flightのエラーハンドリングをオーバーライドします。Tracyを無効にしてから`Flight::set('flight.handle_errors', true);`が設定されていることを確認する必要があります。

- **データベースクエリが追跡されない場合？**
  - 5番目のコンストラクタ引数（オプション配列）として`['trackApmQueries' => true]`で`SimplePdo`を優先。
  - 非推奨の`PdoWrapper`を使用している場合、5番目の引数に`true`を渡す。
  - 接続作成後に`$Apm->addPdoConnection($pdo)`を呼び出す。