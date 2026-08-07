# Flight PHP と PHPUnit によるユニットテスト

このガイドでは、[PHPUnit](https://phpunit.de/) を使った Flight PHP のユニットテストを紹介します。*なぜ*ユニットテストが重要なのか、そしてそれを実際にどう適用するのかを理解したい初心者を対象としています。単純な計算ではなく、メールの送信やレコードの保存など、アプリケーションが期待どおりに動作するかという*動作*のテストに焦点を当てます。まずは単純な[ルートハンドラ](/learn/routing)から始めて、[依存性注入](/learn/dependency-injection-container)（DI）とサードパーティサービスのモックを組み込んだ、より複雑な[コントローラ](/learn/routing)へと進みます。

## なぜユニットテストを行うのか？

ユニットテストは、コードが期待どおりに動作することを保証し、本番環境に到達する前にバグを発見します。特に Flight では、軽量なルーティングと柔軟性により複雑な相互作用が生じる可能性があるため、これは非常に価値があります。個人開発者でもチームでも、ユニットテストはセーフティネットとして機能し、期待される動作を文書化し、後でコードを再訪したときにリグレッションを防ぎます。また、設計の改善にも役立ちます。テストしにくいコードは、クラスが過度に複雑であるか、結合度が高すぎることを示していることがよくあります。

単純な例（例：`x * y = z` のテスト）とは異なり、入力の検証、データの保存、メール送信などのアクションのトリガーといった、実際の世界の動作に焦点を当てます。目標は、テストを身近で有意義なものにすることです。

## 一般的な指針

1. **動作をテストし、実装をテストしない**: 内部の詳細ではなく、結果（例：「メール送信」「レコード保存」）に焦点を当てます。これにより、リファクタリングに対してテストが堅牢になります。
2. **`Flight::` の使用をやめる**: Flight の静的メソッドは非常に便利ですが、テストを困難にします。`$app = Flight::app();` から取得できる `$app` 変数を使うことに慣れてください。`$app` には `Flight::` と同じメソッドがすべてあります。コントローラ内などで `$app->route()` や `$this->app->json()` を引き続き使用できます。また、実際の Flight ルーターを `$router = $app->router()` で使い、`$router->get()`、`$router->post()`、`$router->group()` などを使用することもできます。[ルーティング](/learn/routing) を参照してください。
3. **テストを高速に保つ**: テストが高速だと頻繁に実行できます。ユニットテストではデータベース呼び出しなどの低速な操作を避けてください。テストが遅い場合は、ユニットテストではなく統合テストを書いている可能性があります。統合テストとは、実際のデータベース、実際の HTTP 呼び出し、実際のメール送信などを行うテストです。それらには役割がありますが、遅く、不明な理由で失敗することがあるため不安定になる可能性があります。
4. **説明的な名前を使う**: テスト名は、テスト対象の動作を明確に説明する必要があります。これにより、可読性と保守性が向上します。
5. **グローバル変数を避ける**: `$app->set()` と `$app->get()` の使用を最小限にしてください。これらはグローバル状態のように機能し、すべてのテストでモックが必要になります。DI または DI コンテナを優先してください（[依存性注入コンテナ](/learn/dependency-injection-container) を参照）。`$app->map()` メソッドの使用も技術的には「グローバル」であり、DI を優先して避けるべきです。テストでセッションオブジェクトをモックできるように、[flightphp/session](https://github.com/flightphp/session) などのセッションライブラリを使用してください。コード内で直接 [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) を呼び出さないでください。グローバル変数をコードに注入することになり、テストが困難になります。
6. **依存性注入を使用する**: 依存関係（例：[`PDO`](https://www.php.net/manual/en/class.pdo.php)、メーラー）をコントローラに注入して、ロジックを分離し、モックを簡単にします。依存関係が多すぎるクラスがある場合は、[SOLID 原則](https://en.wikipedia.org/wiki/SOLID)に従って、それぞれが単一の責任を持つ小さなクラスにリファクタリングすることを検討してください。
7. **サードパーティサービスをモックする**: データベース、HTTP クライアント（cURL）、メールサービスなどをモックして、外部呼び出しを避けます。1〜2層の深さをテストしますが、コアロジックは実行させてください。たとえば、アプリがテキストメッセージを送信する場合、テストを実行するたびに実際にテキストメッセージを送信したくはないはずです（コストがかさみ、遅くなるため）。代わりに、テキストメッセージサービスをモックし、コードが正しいパラメータでテキストメッセージサービスを呼び出したことを検証するだけにします。
8. **高いカバレッジを目指すが、完璧を求めない**: 100% の行カバレッジは良いことですが、コード内のすべてが期待どおりにテストされているとは限りません（[PHPUnit での分岐・パスカバレッジ](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/) を調べてみてください）。重要な動作（例：ユーザー登録、API レスポンス、失敗したレスポンスの取得）を優先してください。
9. **ルートにはコントローラを使用する**: ルート定義では、クロージャではなくコントローラを使用してください。`flight\Engine $app` はデフォルトでコンストラクタを介してすべてのコントローラに注入されます。テストでは、`$app = new Flight\Engine()` を使用してテスト内で Flight をインスタンス化し、コントローラに注入して、メソッドを直接呼び出します（例：`$controller->register()`）。[Flight の拡張](/learn/extending) と [ルーティング](/learn/routing) を参照してください。
10. **モックのスタイルを選んで一貫させる**: PHPUnit はいくつかのモックスタイル（例：prophecy、組み込みモック）をサポートしています。または、コード補完やメソッド定義を変更した場合に壊れるなどの利点がある匿名クラスを使用することもできます。テスト全体で一貫させてください。[PHPUnit モックオブジェクト](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles) を参照してください。
11. **サブクラスでテストしたいメソッドやプロパティには `protected` 可視性を使用する**: これにより、パブリックにせずにテスト用サブクラスでオーバーライドできます。これは特に匿名クラスモックに役立ちます。

## PHPUnit のセットアップ

まず、Composer を使用して Flight PHP プロジェクトに [PHPUnit](https://phpunit.de/) をセットアップします。詳細は [PHPUnit 入門ガイド](https://phpunit.readthedocs.io/en/12.3/installation.html) を参照してください。

1. プロジェクトディレクトリで次のコマンドを実行します:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   これにより、最新の PHPUnit が開発依存関係としてインストールされます。

2. プロジェクトのルートにテストファイル用の `tests` ディレクトリを作成します。

3. 利便性のために `composer.json` にテストスクリプトを追加します:
   ```json
   // composer.json の他の内容
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. ルートに `phpunit.xml` ファイルを作成します:
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

これで、テストが構築されたら、`composer test` を実行してテストを実行できます。

## 単純なルートハンドラのテスト

まず、ユーザーのメール入力を受け取る基本的な[ルート](/learn/routing)から始めましょう。その動作、つまり有効なメールには成功メッセージを返し、無効なメールにはエラーを返すことをテストします。メール検証には、[`filter_var`](https://www.php.net/manual/en/function.filter-var.php) を使用します。

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
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

これをテストするには、テストファイルを作成します。テストの構造については、[ユニットテストと SOLID 原則](/learn/unit-testing-and-solid-principles) を参照してください:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // POST データをシミュレート
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // POST データをシミュレート
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**重要なポイント**:
- リクエストクラスを使用して POST データをシミュレートします。`$_POST` や `$_GET` などのグローバルを使用しないでください。テストがより複雑になるためです（それらの値を常にリセットする必要があり、他のテストが失敗する可能性があります）。
- デフォルトでは、すべてのコントローラに `flight\Engine` インスタンスが注入されます（DIC コンテナを設定しなくても）。これにより、コントローラを直接テストするのがはるかに簡単になります。
- `Flight::` の使用は一切なく、コードをテストしやすくしています。
- テストは、有効/無効なメールに対する正しいステータスとメッセージという動作を検証します。

`composer test` を実行して、ルートが期待どおりに動作することを確認します。Flight の[リクエスト](/learn/requests)と[レスポンス](/learn/responses)の詳細については、関連ドキュメントを参照してください。

## 依存性注入を使用したテスト可能なコントローラ

より複雑なシナリオでは、[依存性注入](/learn/dependency-injection-container)（DI）を使用してコントローラをテスト可能にします。Flight のグローバル（例：`Flight::set()`、`Flight::map()`、`Flight::register()`）は、グローバル状態のように機能し、すべてのテストでモックが必要になるため避けてください。代わりに、Flight の DI コンテナ、[DICE](https://github.com/Level-2/Dice)、[PHP-DI](https://php-di.org/)、または手動 DI を使用してください。

生の PDO の代わりに [`flight\database\SimplePdo`](/learn/simple-pdo) を使用しましょう。このヘルパーはモックやユニットテストがはるかに簡単です（そして非推奨の `PdoWrapper` よりも推奨されます）。

ユーザーをデータベースに保存し、ウェルカムメールを送信するコントローラは次のとおりです:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// return を置くことで、ユニットテストの実行を停止するのに役立ちます
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**重要なポイント**:
- コントローラは、[`SimplePdo`](/learn/simple-pdo) インスタンスと `MailerInterface`（想定上のサードパーティメールサービス）に依存します。
- 依存関係はコンストラクタを介して注入され、グローバルを避けます。

### モックを使ったコントローラのテスト

次に、`UserController` の動作をテストしましょう。メールの検証、データベースへの保存、メールの送信です。コントローラを分離するために、データベースとメーラーをモックします。

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// モックスタイルを混在させなければならない場合があります
		// ここでは PHPUnit の組み込みモックを PDOStatement に使用します
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// SimplePdo をモックするための匿名クラスを使用
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// この方法でモックすると、実際のデータベース呼び出しは行われません。
			// さらに、これを設定して PDOStatement モックを変更し、障害などをシミュレートすることもできます。
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// 空のコンストラクタは親コンストラクタをバイパスします
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// 終了を避けるために jsonHalt をマップする必要があります
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**重要なポイント**:
- `SimplePdo` と `MailerInterface` をモックして、実際のデータベースやメール呼び出しを避けます。
- テストは動作を検証します。有効なメールはデータベース挿入とメール送信をトリガーし、無効なメールは両方をスキップします。
- サードパーティの依存関係（例：`SimplePdo`、`MailerInterface`）をモックし、コントローラのロジックを実行させます。

### モックしすぎる

コードをモックしすぎないように注意してください。私たちの `UserController` を使って、これがなぜ良くないのかの例を以下に示します。そのチェックを `isEmailValid`（`filter_var` を使用）というメソッドに変更し、他の新しい追加を `registerUser` という別のメソッドに変更します。

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// return を置くことで、ユニットテストの実行を停止するのに役立ちます
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

そして今度は、実際には何もテストしていない過剰にモックされたユニットテストです:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// ここでは「簡単」だからという理由で追加の依存性注入を省略しています
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// コンストラクタの依存関係をバイパス
			public function __construct($app) {
				$this->app = $app;
			}

			// これを強制的に有効にします。
			protected function isEmailValid($email) {
				return true; // 常に true を返し、実際の検証をバイパス
			}

			// 実際の DB とメーラーの呼び出しをバイパス
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

やった、ユニットテストがあって、それらはパスしています！でも待ってください。`isEmailValid` や `registerUser` の内部動作を実際に変更したらどうなるでしょうか？すべての機能をモックしてしまったので、テストはまだパスします。どういうことかお見せしましょう。

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... 他のメソッド ...

	protected function isEmailValid($email) {
		// 変更されたロジック
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// これで特定のドメインのみ許可されるようになりました
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

上記のユニットテストを実行すると、それでもパスします！しかし、動作をテストしていなかった（コードの一部を実際に実行させていなかった）ため、本番環境で発生するバグをコードに埋め込んだ可能性があります。テストは新しい動作を考慮して修正する必要があり、また動作が期待どおりでない場合の逆のケースも考慮する必要があります。

## 完全な例

ユニットテストを含む Flight PHP プロジェクトの完全な例は、GitHub にあります: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide)。より深く理解するには、[ユニットテストと SOLID 原則](/learn/unit-testing-and-solid-principles) を参照してください。

## よくある落とし穴

- **過剰なモック**: すべての依存関係をモックしないでください。実際の動作をテストするために、一部のロジック（例：コントローラの検証）は実行させてください。[ユニットテストと SOLID 原則](/learn/unit-testing-and-solid-principles) を参照してください。
- **グローバル状態**: PHP のグローバル変数（例：[`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php)、[`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)）を多用すると、テストが壊れやすくなります。`Flight::` も同様です。依存関係を明示的に渡すようにリファクタリングしてください。
- **複雑なセットアップ**: テストのセットアップが面倒な場合、クラスに依存関係や責任が多すぎて、[SOLID 原則](/learn/unit-testing-and-solid-principles)に違反している可能性があります。

## ユニットテストのスケーリング

ユニットテストは、大規模なプロジェクトや数か月後にコードを再訪する際に力を発揮します。動作を文書化し、リグレッションを検出して、アプリを再学習する手間を省きます。個人開発者の場合は、重要なパス（例：ユーザー登録、支払い処理）をテストしてください。チームの場合は、テストによって貢献全体で一貫した動作が保証されます。フレームワークとテストを使用する利点の詳細については、[フレームワークを使う理由は？](/learn/why-frameworks) を参照してください。

Flight PHP ドキュメントリポジトリにあなた自身のテストのヒントを貢献してください！

_執筆: [n0nag0n](https://github.com/n0nag0n) 2025_