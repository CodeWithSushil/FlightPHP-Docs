# 依存性注入コンテナ

## 概要

依存性注入コンテナ（DIC）は、アプリケーションの依存関係を管理できる強力な拡張機能です。また、Flightが [AIコーディングツール](/learn/ai) やユニットテストとうまく連携できる最大の理由の1つでもあります。コントローラーは、グローバルにアクセスする代わりに、コンストラクターで必要なものを受け取ります。

## 理解

依存性注入（DI）は、現代のPHPフレームワークにおける重要な概念であり、オブジェクトのインスタンス化と構成を管理するために使用されます。DICライブラリの例としては、[flightphp/container](https://github.com/flightphp/container)、[Dice](https://r.je/dice)、[Pimple](https://pimple.symfony.com/)、[PHP-DI](http://php-di.org/)、[league/container](https://container.thephpleague.com/) などがあります。

DICは、クラスを一元管理された場所で作成・管理するための凝った方法です。同じオブジェクトを複数のクラス（コントローラー、ミドルウェア、コマンドなど）に渡す必要がある場合に便利です。

公式の [flightphp/skeleton](https://github.com/flightphp/skeleton) は、`app/config/services.php` で **Dice** を配線し、共有の `flight\Engine` インスタンスを置き換え、`[App\Controller\HomeController::class, 'index']` のようなルートターゲットを解決します。新しいプロジェクトでは、人間とエージェントが同じ場所を編集できるように、このパターンを採用してください。

## 基本的な使い方

従来のやり方は次のようになるでしょう。

```php

require 'vendor/autoload.php';

// データベースからユーザーを管理するクラス
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// routes.php ファイル内

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// その他のUserControllerルート...
Flight::start();
```

上記のコードから、新しい `PDO` オブジェクトを作成して `UserController` クラスに渡していることがわかります。これは小規模なアプリケーションには問題ありませんが、アプリケーションが大きくなるにつれて、同じ `PDO` オブジェクトを複数の場所で作成または受け渡ししていることに気づくでしょう。ここでDICが役立ちます。

次に、DIC（Diceを使用）を使った同じ例を示します。

```php

require 'vendor/autoload.php';

// 上記と同じクラスです。何も変更されていません
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// 新しいコンテナを作成
$container = new \Dice\Dice;

// PDOオブジェクトの作成方法をコンテナに指示するルールを追加
// 下記のように必ず自分自身に再代入することを忘れないでください！
$container = $container->addRule('PDO', [
	// sharedは、毎回同じオブジェクトが返されることを意味します
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// これによりコンテナハンドラーが登録され、Flightがそれを使用することを認識します。
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// これでコンテナを使ってUserControllerを作成できます
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

きっと、この例には余分なコードがたくさん追加されたと思っているかもしれません。魔法が発揮されるのは、`PDO` オブジェクトを必要とする別のコントローラーがあるときです。

```php

// すべてのコントローラーのコンストラクターがPDOオブジェクトを必要とする場合
// 以下の各ルートには自動的にPDOが注入されます!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

DICを利用する追加の利点は、ユニットテストがはるかに簡単になることです。モックオブジェクトを作成してクラスに渡すことができます。これは、アプリケーションのテストを書く際に大きなメリットです。また、AIアシスタントがコントローラーを生成する場合、コンストラクターインジェクションにより、従うべき明確で一貫性のあるパターンが提供されます（[ユニットテストガイド](/guides/unit-testing)）。

### 集中管理型DICハンドラーの作成

アプリを[拡張](/learn/extending)することで、サービスファイルに集中管理型のDICハンドラーを作成できます。次に例を示します。

```php
// services.php

// 新しいコンテナを作成
$container = new \Dice\Dice;
// 下記のように必ず自分自身に再代入することを忘れないでください！
$container = $container->addRule('PDO', [
	// sharedは、毎回同じオブジェクトが返されることを意味します
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// これで、任意のオブジェクトを作成するためのマップ可能なメソッドを作成できます。
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// これによりコンテナハンドラーが登録され、Flightがコントローラー/ミドルウェアにそれを使用することを認識します。
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// コンストラクターでPDOオブジェクトを受け取る次のサンプルクラスがあるとします
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// メールを送信するコード
	}
}

// そして最後に、依存性注入を使用してオブジェクトを作成できます
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flightには、依存性注入を処理するために使用できるシンプルなPSR-11準拠コンテナを提供するプラグインがあります。その使用例を簡単に示します。

```php

// 例: index.php
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// これは正しく出力されます！
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### flightphp/container の高度な使い方

依存関係を再帰的に解決することもできます。次に例を示します。

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // 実装 ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

独自のDICハンドラーを作成することもできます。これは、PSR-11ではない独自のコンテナ（Diceなど）を使用したい場合に便利です。これを行う方法については、[基本的な使い方](#basic-usage) のセクションを参照してください。

さらに、Flightを使用する際に作業を容易にする便利なデフォルトがいくつかあります。

#### Engineインスタンス（`$app` インジェクションに必要）

コントローラーやミドルウェアで `flight\Engine` を型宣言する場合、**Dice は新しい Engine を構築してはなりません**。ブートストラップから同じインスタンスを置き換えてください。これが公式スケルトンのやり方であり、AI生成コントローラーで `AGENTS.md` が期待するパターンです。

```php
// ブートストラップ / services.php のどこか
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // または $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// 重要: ブートストラップ済みのEngineを再利用してください。Diceに `new Engine()` をさせないでください
		Engine::class => $app,
		// 新しいコードではSimplePdoを推奨します
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// ルート以外のコード用のオプションヘルパー
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php（スケルトンレイアウト — フォルダー名の大文字小文字は名前空間と一致します）
namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// アプリ層では Flight:: ファサードを使用しません — テストが容易で、AIツールにとっても明確です
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

`Engine` の置き換えをスキップすると、Dice が2つ目の Engine を構築し、コントローラーがブートストラップからのルート、設定、マップされた Twig `render` を共有しなくなる可能性があります。

#### 他の共有サービスを追加する（SimplePdo、Config、Twig）

```php
use flight\database\SimplePdo;
use flight\Engine;

// services.php で $db、$config、$twig を作成した後:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

これで、コントローラーはコンストラクターで `SimplePdo $db`（または設定型）を受け取り、`Flight::db()` を呼び出す必要がなくなります。これは、[ユニットテスト](/guides/unit-testing) のガイダンスとスケルトンのハウススタイルに一致します。

#### 他のクラスを追加する

コンテナに追加したい他のクラスがある場合、Dice ではコンテナによって自動的に解決されるため簡単です。次に例を示します。

```php

$container = new \Dice\Dice;
// クラスに依存関係を注入する必要がない場合
// 何も定義する必要はありません！
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flightは、PSR-11準拠の任意のコンテナも使用できます。つまり、PSR-11インターフェースを実装した任意のコンテナを使用できます。次に、LeagueのPSR-11コンテナを使用した例を示します。

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// 上記と同じUserControllerの考え方ですが、素のPDOの代わりにSimplePdoを型宣言します

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

これは以前のDiceの例よりも少し冗長かもしれませんが、同じ利点で目的を達成できます！

## 関連情報

- [インストール](/install) - スケルトレイアウトと `services.php` の場所。
- [オートローディング](/learn/autoloading) - `App\` 名前空間とフォルダーの**大文字小文字**。
- [Flightの拡張](/learn/extending) - フレームワークを拡張して独自のクラスに依存性注入を追加する方法を学ぶ。
- [設定](/learn/configuration) - アプリケーション用にFlightを設定する方法を学ぶ。
- [ルーティング](/learn/routing) - アプリケーションのルートを定義する方法と、依存性注入がコントローラーとどのように連携するかを学ぶ。
- [ミドルウェア](/learn/middleware) - アプリケーション用のミドルウェアを作成する方法と、依存性注入がミドルウェアとどのように連携するかを学ぶ。
- [ユニットテスト](/guides/unit-testing) - コンストラクターインジェクションが `Flight::` グローバルよりも優れている理由。
- [AIと開発者体験](/learn/ai) - 人間とエージェントのための単一のDIパターン。
- [SimplePdo](/learn/simple-pdo) - インジェクションに推奨されるデータベースヘルパー。

## トラブルシューティング

- コンテナで問題が発生している場合は、正しいクラス名をコンテナに渡していることを確認してください。
- `Engine` を型宣言しているのに「空の」アプリが返されるコントローラーがある場合は、**Engineの置き換え**を追加してください（上記参照）。Dice は2つ目の Engine を `new` してはいけません。
- `App\Controller\…` でクラスが見つからない場合：`app/Controller/` 配下のフォルダー名の大文字小文字を確認してください — [オートローディング](/learn/autoloading) を参照。
- ハンドラーは `registerContainerHandler` から作成したオブジェクトを**返す**必要があります（`return` なしで `Flight::make()` を呼び出さないでください）。

## 変更履歴

- ドキュメント – AIフレンドリーなプロジェクト向けに、スケルトンのDice + Engine置き換え、SimplePdo、`App\Controller` レイアウトを文書化。
- v3.7.0 - FlightにDICハンドラーを登録する機能を追加。