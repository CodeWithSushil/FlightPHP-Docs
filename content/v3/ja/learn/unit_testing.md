# ユニットテスト

## 概要

Flightでのユニットテストは、アプリケーションが期待どおりに動作することを確認し、バグを早期に発見し、コードベースを保守しやすくするのに役立ちます。Flightは、最も人気のあるPHPテストフレームワークである[PHPUnit](https://phpunit.de/)とスムーズに連携するように設計されています。

## 理解

ユニットテストは、アプリケーションの小さな部分（コントローラやサービスなど）の動作を単体でチェックします。Flightでは、ルート、コントローラ、ロジックがさまざまな入力にどのように応答するかを、グローバル状態や実際の外部サービスに依存せずにテストすることを意味します。

主要な原則：
- **実装ではなく動作をテストする：** コードが「どのように」行うかではなく、「何を」行うかに焦点を当てます。
- **グローバル状態を避ける：** `Flight::set()` や `Flight::get()` の代わりに依存性注入を使用します。
- **外部サービスをモックする：** データベースやメーラーなどの外部サービスはテストダブルに置き換えます。
- **テストを高速かつ焦点を絞ったものにする：** ユニットテストは実際のデータベースやAPIにアクセスすべきではありません。

## 基本的な使い方

### PHPUnitのセットアップ

1. ComposerでPHPUnitをインストールします。
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. プロジェクトルートに `tests` ディレクトリを作成します。
3. `composer.json` にテストスクリプトを追加します。
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. `phpunit.xml` ファイルを作成します。
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

これで `composer test` を実行してテストを実行できます。

### シンプルなルートハンドラのテスト

メールアドレスを検証するルートがあるとします。

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
    protected $app;
    public function __construct(flight\Engine $app) {
        $this->app = $app;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

このコントローラのシンプルなテスト：

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**ヒント：**
- POSTデータは `$app->request()->data` を使用してシミュレートします。
- テストでは `Flight::` の静的メソッドを使わず、`$app` インスタンスを使用してください。

### テスト可能なコントローラのための依存性注入の使用

依存関係（データベースやメーラーなど）をコントローラに注入することで、テストでモックしやすくなります。

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

モックを使ったテスト：

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## 高度な使い方

- **モッキング：** PHPUnitの組み込みモックまたは匿名クラスを使用して依存関係を置き換えます。
- **コントローラを直接テストする：** 新しい `Engine` でコントローラをインスタンス化し、依存関係をモックします。
- **過剰なモッキングを避ける：** 可能な限り実際のロジックを実行させ、外部サービスのみをモックします。

## 関連情報

- [ユニットテストガイド](/guides/unit-testing) - ユニットテストのベストプラクティスに関する包括的なガイドです。
- [依存性注入コンテナ](/learn/dependency-injection-container) - DICを使用して依存関係を管理し、テスト容易性を向上させる方法。
- [拡張](/learn/extending) - 独自のヘルパーを追加したり、コアクラスをオーバーライドする方法。
- [SimplePdo](/learn/simple-pdo) - データベース操作を簡素化し、テストでモックしやすくします。
- [リクエスト](/learn/requests) - FlightでHTTPリクエストを処理する方法。
- [レスポンス](/learn/responses) - ユーザーにレスポンスを送信する方法。
- [ユニットテストとSOLID原則](/learn/unit-testing-and-solid-principles) - SOLID原則がユニットテストを改善する方法を学びます。

## トラブルシューティング

- コードとテストでグローバル状態（`Flight::set()`、`$_SESSION` など）を使用しないようにしてください。
- テストが遅い場合は、統合テストを書いている可能性があります。外部サービスをモックしてユニットテストを高速に保ちましょう。
- テストのセットアップが複雑な場合は、依存性注入を使用するようにコードをリファクタリングすることを検討してください。

## 変更履歴

- v3.15.0 - 依存性注入とモッキングの例を追加しました。