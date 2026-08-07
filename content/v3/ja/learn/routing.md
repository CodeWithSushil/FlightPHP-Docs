# ルーティング

## 概要
Flight PHP のルーティングは、URL パターンをコールバック関数やクラスのメソッドにマッピングし、高速かつシンプルなリクエスト処理を可能にします。最小限のオーバーヘッド、初心者にも使いやすく、外部依存なしで拡張できるように設計されています。

## 理解
ルーティングは、HTTP リクエストをアプリケーションのロジックに接続する中核的な仕組みです。ルートを定義することで、関数、クラスメソッド、コントローラアクションのいずれを通しても、異なる URL が特定のコードをトリガーする方法を指定できます。Flight のルーティングシステムは柔軟で、基本的なパターン、名前付きパラメータ、正規表現、依存性注入やリソースフルルーティングなどの高度な機能をサポートしています。このアプローチにより、コードは整理されて保守しやすく保たれ、初心者には高速でシンプル、上級者には拡張可能です。

> **注:** ルーティングについてもっと理解したいですか？ 詳しい説明は [「なぜフレームワークを使うのか？」](/learn/why-frameworks) のページを参照してください。

## 基本的な使い方

### シンプルなルートの定義
Flight の基本的なルーティングは、URL パターンをコールバック関数またはクラスとメソッドの配列とマッチングさせることで行います。

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> ルートは定義された順序でマッチングされます。リクエストに最初にマッチしたルートが呼び出されます。

### コールバックとして関数を使う
コールバックは、呼び出し可能なオブジェクトであれば何でもかまいません。通常の関数も使用できます:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### コントローラーとしてクラスとメソッドを使う
クラスのメソッド（静的または非静的）も使用できます:

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// または
Flight::route('/', [ GreetingController::class, 'hello' ]); // 推奨される方法
// または
Flight::route('/', [ 'GreetingController::hello' ]);
// または 
Flight::route('/', [ 'GreetingController->hello' ]);
```

または、先にオブジェクトを作成してからメソッドを呼び出します:

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **注:** デフォルトでは、フレームワーク内でコントローラーが呼び出されるとき、[依存性注入コンテナ](/learn/dependency-injection-container) で指定しない限り、`flight\Engine` クラスが常に注入されます。

### メソッドごとのルーティング

デフォルトでは、ルートパターンはすべてのリクエストメソッドに対してマッチングされます。URL の前に識別子を置くことで、特定のメソッドに応答できます。

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// ルートには Flight::get() を使用できません。これは変数を取得するメソッドであり、
// ルートを作成するものではないためです。
Flight::post('/', function() { /* code */ });
Flight::patch('/', function() { /* code */ });
Flight::put('/', function() { /* code */ });
Flight::delete('/', function() { /* code */ });
```

`|` 区切り文字を使用して、複数のメソッドを単一のコールバックにマッピングすることもできます:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### HEAD および OPTIONS リクエストの特別処理

Flight は、`HEAD` および `OPTIONS` HTTP リクエストに対する組み込みの処理を提供します:

#### HEAD リクエスト

- **HEAD リクエスト** は `GET` リクエストと同様に扱われますが、Flight はレスポンスをクライアントに送信する前にレスポンスボディを自動的に削除します。
- つまり、`GET` 用のルートを定義すれば、同じ URL への HEAD リクエストは HTTP 標準に従ってヘッダーのみ（コンテンツなし）を返します。

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// /info への HEAD リクエストは同じヘッダーを返しますが、ボディは返しません。
```

#### OPTIONS リクエスト

`OPTIONS` リクエストは、定義された任意のルートに対して Flight によって自動的に処理されます。
- OPTIONS リクエストを受信すると、Flight は `204 No Content` ステータスと、そのルートでサポートされているすべての HTTP メソッドを一覧表示する `Allow` ヘッダーで応答します。
- OPTIONS 用に別途ルートを定義する必要はありません。

```php
// 次のように定義されたルートの場合:
Flight::route('GET|POST /users', function() { /* ... */ });

// /users への OPTIONS リクエストは次のように応答します:
//
// Status: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Router オブジェクトを使う

さらに、使用できるヘルパーメソッドを持つ Router オブジェクトを取得できます:

```php

$router = Flight::router();

// Flight::route() と同様にすべてのメソッドをマッピングする
$router->map('/', function() {
	echo 'hello world!';
});

// GET リクエスト
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* code */});
$router->put('/users/update/@id', 	function() { /* code */});
$router->delete('/users/@id', 		function() { /* code */});
$router->patch('/users/@id', 		function() { /* code */});
```

### 正規表現（Regex）
ルートで正規表現を使用できます:

```php
Flight::route('/user/[0-9]+', function () {
  // これは /user/1234 にマッチします
});
```

この方法も利用可能ですが、名前付きパラメータ、または正規表現を使用した名前付きパラメータを使用することをお勧めします。読みやすく、保守が容易だからです。

### 名前付きパラメータ
ルート内に名前付きパラメータを指定でき、コールバック関数に渡されます。**これは何よりもルートの読みやすさのためのものです。重要な注意点については以下のセクションを参照してください。**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

名前付きパラメータに正規表現を含める場合は、`:` 区切り文字を使用します:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // これは /bob/123 にマッチします
  // ただし /bob/12345 にはマッチしません
});
```

> **注:** 位置パラメータを持つ正規表現グループ `()` のマッチングはサポートされていません。例: `:'\(`

#### 重要な注意点

上記の例では、`@name` が変数 `$name` に直接結びついているように見えますが、実際はそうではありません。コールバック関数のパラメータの順序が、渡される内容を決定します。コールバック関数のパラメータの順序を入れ替えると、変数も入れ替わります。次に例を示します:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

そして、`/bob/123` という URL にアクセスすると、出力は `hello, 123 (bob)!` になります。_ルートとコールバック関数を設定するときは注意してください!_

### オプションパラメータ
セグメントを括弧で囲むことで、マッチングにオプションの名前付きパラメータを指定できます。

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // これは次の URL にマッチします:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

マッチしなかったオプションパラメータは `NULL` として渡されます。

### ワイルドカードルーティング
マッチングは個々の URL セグメントに対してのみ行われます。複数のセグメントにマッチさせたい場合は、`*` ワイルドカードを使用できます。

```php
Flight::route('/blog/*', function () {
  // これは /blog/2000/02/01 にマッチします
});
```

すべてのリクエストを単一のコールバックにルーティングするには、次のようにします:

```php
Flight::route('*', function () {
  // 何か処理をする
});
```

### 404 Not Found ハンドラ

デフォルトでは、URL が見つからない場合、Flight は非常にシンプルで素朴な `HTTP 404 Not Found` レスポンスを送信します。よりカスタマイズされた 404 レスポンスが必要な場合は、独自の `notFound` メソッドを [マップ](/learn/extending) できます:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// カスタムテンプレートで Flight::render() を使用することもできます。
    $output = <<<HTML
		<h1>My Custom 404 Not Found</h1>
		<h3>The page you have requested {$url} could not be found.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Method Not Found ハンドラ

デフォルトでは、URL は見つかったがメソッドが許可されていない場合、Flight は非常にシンプルで素朴な `HTTP 405 Method Not Allowed` レスポンスを送信します（例: Method Not Allowed. Allowed Methods are: GET, POST）。また、その URL で許可されているメソッドを含む `Allow` ヘッダーも含まれます。

よりカスタマイズされた 405 レスポンスが必要な場合は、独自の `methodNotFound` メソッドを [マップ](/learn/extending) できます:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// カスタムテンプレートで Flight::render() を使用することもできます。
	$output = <<<HTML
		<h1>My Custom 405 Method Not Allowed</h1>
		<h3>The method you have requested for {$url} is not allowed.</h3>
		<p>Allowed Methods are: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## 上級の使い方

### ルートでの依存性注入
コンテナ（PSR-11、PHP-DI、Dice など）を介した依存性注入を使用したい場合、それが利用できるルート定義は、オブジェクトを自分で直接作成してコンテナでオブジェクトを作成する方法か、呼び出すクラスとメソッドを文字列で定義する方法のいずれかです。詳細については、[依存性注入](/learn/dependency-injection-container) のページを参照してください。

簡単な例を示します:

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// $this->db を使って何か処理をする
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// 必要なパラメータを指定してコンテナをセットアップします
// PSR-11 の詳細については依存性注入のページを参照してください
$dice = new \Dice\Dice();

// 変数を '$dice = ' で再代入することを忘れないでください!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// コンテナハンドラを登録する
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// 通常どおりルートを定義する
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// または
Flight::route('/hello/@id', 'Greeting->hello');
// または
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### 次のルートへの実行の受け渡し
<span class="badge bg-warning">非推奨</span>
コールバック関数から `true` を返すことで、次にマッチするルートに実行を渡すことができます。

```php
Flight::route('/user/@name', function (string $name) {
  // 何らかの条件をチェックする
  if ($name !== "Bob") {
    // 次のルートに進む
    return true;
  }
});

Flight::route('/user/*', function () {
  // これは呼び出されます
});
```

このような複雑なユースケースを処理するには、[ミドルウェア](/learn/middleware) を使用することをお勧めします。

### ルートエイリアス
ルートにエイリアスを割り当てることで、後でアプリ内でそのエイリアスを動的に呼び出して URL を生成できます（例: HTML テンプレート内のリンク、リダイレクト URL の生成など）。

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// または 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// 後でコード内のどこかで
class UserController {
	public function update() {

		// ユーザーを保存するコード...
		$id = $user['id']; // 例: 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // '/users/5' を返します
		Flight::redirect($redirectUrl);
	}
}

```

これは、URL が変更される場合に特に役立ちます。上記の例で、users が代わりに `/admin/users/@id` に移動されたとします。ルートにエイリアスを設定しておけば、エイリアスが上記の例のように `/admin/users/5` を返すため、コード内の古い URL をすべて見つけて変更する必要はもうありません。

ルートエイリアスはグループ内でも機能します:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// または
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### ルート情報の確認
マッチしたルート情報を確認したい場合、2 つの方法があります:

1. `Flight::router()` オブジェクトの `executedRoute` プロパティを使用する。
2. ルートメソッドの第 3 パラメータに `true` を渡すことで、ルートオブジェクトをコールバックに渡すようリクエストする。ルートオブジェクトは常にコールバック関数の最後のパラメータとして渡されます。

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // $route を使って何か処理をする
  // マッチした HTTP メソッドの配列
  $route->methods;

  // 名前付きパラメータの配列
  $route->params;

  // マッチした正規表現
  $route->regex;

  // URL パターンで使用された '*' の内容を含む
  $route->splat;

  // URL パスを表示する...本当に必要な場合
  $route->pattern;

  // このルートに割り当てられたミドルウェアを表示する
  $route->middleware;

  // このルートに割り当てられたエイリアスを表示する
  $route->alias;
});
```

> **注:** `executedRoute` プロパティは、ルートが実行された後にのみ設定されます。ルートが実行される前にアクセスしようとすると、`NULL` になります。また、executedRoute は [ミドルウェア](/learn/middleware) 内でも使用できます!

#### ルート定義に `true` を渡す
```php
Flight::route('/', function(\flight\net\Route $route) {
  // マッチした HTTP メソッドの配列
  $route->methods;

  // 名前付きパラメータの配列
  $route->params;

  // マッチした正規表現
  $route->regex;

  // URL パターンで使用された '*' の内容を含む
  $route->splat;

  // URL パスを表示する...本当に必要な場合
  $route->pattern;

  // このルートに割り当てられたミドルウェアを表示する
  $route->middleware;

  // このルートに割り当てられたエイリアスを表示する
  $route->alias;
}, true);// <-- この true パラメータによってそれが実現されます
```

### ルートのグループ化とミドルウェア
関連するルートをグループ化したい場合があります（例: `/api/v1`）。これを行うには、`group` メソッドを使用します:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// /api/v1/users にマッチする
  });

  Flight::route('/posts', function () {
	// /api/v1/posts にマッチする
  });
});
```

グループをネストすることもできます:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() は変数を取得するもので、ルートを設定するものではありません。以下のオブジェクトコンテキストを参照してください
	Flight::route('GET /users', function () {
	  // GET /api/v1/users にマッチする
	});

	Flight::post('/posts', function () {
	  // POST /api/v1/posts にマッチする
	});

	Flight::put('/posts/1', function () {
	  // PUT /api/v1/posts にマッチする
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() は変数を取得するもので、ルートを設定するものではありません。以下のオブジェクトコンテキストを参照してください
	Flight::route('GET /users', function () {
	  // GET /api/v2/users にマッチする
	});
  });
});
```

#### オブジェクトコンテキストでのグループ化

`Engine` オブジェクトを使用してルートグループを使うこともできます:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // $router 変数を使用する
  $router->get('/users', function () {
	// GET /api/v1/users にマッチする
  });

  $router->post('/posts', function () {
	// POST /api/v1/posts にマッチする
  });
});
```

> **注:** これはルートとグループを $router オブジェクトで定義する推奨方法です。

#### ミドルウェアでのグループ化

ルートのグループにミドルウェアを割り当てることもできます:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// /api/v1/users にマッチする
  });
}, [ MyAuthMiddleware::class ]); // インスタンスを使用する場合は [ new MyAuthMiddleware() ] も可
```

詳細については、[グループミドルウェア](/learn/middleware#grouping-middleware) のページを参照してください。

### リソースルーティング
`resource` メソッドを使用して、リソース用のルートセットを作成できます。これにより、RESTful 規約に従ったリソース用のルートセットが作成されます。

リソースを作成するには、次のようにします:

```php
Flight::resource('/users', UsersController::class);
```

バックグラウンドでは、次のルートが作成されます:

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

コントローラーでは、次のメソッドを使用します:

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **注**: 新しく追加されたルートは、`php runway routes` を実行することで `runway` で確認できます。

#### リソースルートのカスタマイズ

リソースルートを設定するためのオプションがいくつかあります。

##### エイリアスベース

`aliasBase` を設定できます。デフォルトでは、エイリアスは指定された URL の最後の部分です。たとえば、`/users/` の場合、`aliasBase` は `users` になります。これらのルートが作成されるとき、エイリアスは `users.index`、`users.create` などになります。エイリアスを変更したい場合は、`aliasBase` を希望する値に設定します。

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only と Except

`only` および `except` オプションを使用して、作成するルートを指定することもできます。

```php
// これらのメソッドのみをホワイトリスト化し、残りをブラックリスト化する
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// これらのメソッドのみをブラックリスト化し、残りをホワイトリスト化する
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

これらは基本的にホワイトリストおよびブラックリストのオプションであり、作成するルートを指定できます。

##### ミドルウェア

`resource` メソッドによって作成された各ルートで実行されるミドルウェアを指定することもできます。

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### ストリーミングレスポンス

`stream()` または `streamWithHeaders()` を使用して、クライアントにレスポンスをストリーミングできるようになりました。これは、大きなファイルの送信、長時間実行プロセス、または大きなレスポンスの生成に役立ちます。ストリーミングルートは、通常のルートとは少し異なる方法で処理されます。

> **注:** ストリーミングレスポンスは、[`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) が `false` に設定されている場合にのみ利用できます。

#### 手動ヘッダーによるストリーミング

ルートの `stream()` メソッドを使用して、クライアントにレスポンスをストリーミングできます。これを行う場合、クライアントに何かを出力する前に、すべてのヘッダーを手動で設定する必要があります。これは、PHP の `header()` 関数または `Flight::response()->setRealHeader()` メソッドを使用して行います。

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// もちろん、パスをサニタイズするなどしてください。
	$fileNameSafe = basename($filename);

	// ルート実行後にここで追加のヘッダーを設定する場合、
	// 何かが出力される前に定義する必要があります。
	// それらはすべて header() 関数への生の呼び出し、または
	// Flight::response()->setRealHeader() への呼び出しである必要があります。
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// または
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// 必要に応じてコンテンツの長さを手動で設定する
	header('Content-Length: '.filesize($filePath));
	// または
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// ファイルを読み取りながらクライアントにストリーミングする
	readfile($filePath);

// ここが魔法の行です
})->stream();
```

#### ヘッダー付きストリーミング

`streamWithHeaders()` メソッドを使用して、ストリーミングを開始する前にヘッダーを設定することもできます。

```php
Flight::route('/stream-users', function() {

	// ここに追加のヘッダーを自由に追加できます
	// header() または Flight::response()->setRealHeader() を使用する必要があります

	// データの取得方法は何でも構いませんが、例として...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// これはデータをクライアントに送信するために必要です
		ob_flush();
	}
	echo '}';

// ストリーミングを開始する前にヘッダーを設定する方法は次のとおりです。
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// オプションのステータスコード、デフォルトは 200
	'status' => 200
]);
```

## 関連項目
- [ミドルウェア](/learn/middleware) - 認証、ログ記録などのためのルートでのミドルウェアの使用。
- [依存性注入](/learn/dependency-injection-container) - ルート内でのオブジェクト作成と管理の簡素化。
- [なぜフレームワークを使うのか？](/learn/why-frameworks) - Flight のようなフレームワークを使用する利点を理解する。
- [拡張](/learn/extending) - `notFound` メソッドを含む、独自の機能で Flight を拡張する方法。
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - 正規表現マッチングのための PHP 関数。

## トラブルシューティング
- ルートパラメータは名前ではなく順序でマッチングされます。コールバックのパラメータ順序がルート定義と一致していることを確認してください。
- `Flight::get()` はルートを定義しません。ルーティングには `Flight::route('GET /...')` を、グループ内では Router オブジェクトコンテキスト（例: `$router->get(...)`）を使用してください。
- executedRoute プロパティはルートが実行された後にのみ設定されます。実行前は NULL です。
- ストリーミングでは、従来の Flight 出力バッファリング機能を無効にする必要があります（`flight.v2.output_buffering = false`）。
- 依存性注入の場合、コンテナベースのインスタンス化をサポートするルート定義は限られています。

### 404 Not Found または予期しないルート動作

404 Not Found エラーが表示されている場合（しかし、それが本当に存在し、タイプミスではないと確信している場合）、これは実際にはルートエンドポイントで値をエコーせずに返していることが問題である可能性があります。この理由は意図的なものですが、一部の開発者には気づかれないかもしれません。

```php
Flight::route('/hello', function(){
	// これは 404 Not Found エラーを引き起こす可能性があります
	return 'Hello World';
});

// おそらく必要なのはこちら
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

この理由は、ルーターに組み込まれた特別なメカニズムによるもので、返された出力を「次のルートに進む」ための合図として処理します。この動作は、[ルーティング](/learn/routing#passing) セクションで文書化されています。

## 変更履歴
- v3: リソースルーティング、ルートエイリアス、ストリーミングサポート、ルートグループ、ミドルウェアサポートを追加。
- v1: 基本的な機能の大部分が利用可能。