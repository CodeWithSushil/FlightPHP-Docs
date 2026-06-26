# セキュリティ

## 概要

ウェブアプリケーションにおいてセキュリティは非常に重要です。アプリケーションを安全に保ち、ユーザーデータを保護する必要があります。Flightは、ウェブアプリケーションを保護するためのさまざまな機能を提供しています。

## 理解

ウェブアプリケーションを構築する際には、注意すべき一般的なセキュリティ脅威がいくつかあります。最も一般的な脅威には以下が含まれます：
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates)は、デフォルトで出力をエスケープすることでXSSを防ぐのに役立ちます。[Sessions](/awesome-plugins/session)は、以下で説明するようにユーザーのセッションにCSRFトークンを保存することでCSRF対策に役立ちます。PDOでプリペアドステートメントを使用すると、SQLインジェクション攻撃を防ぐことができます（または[PdoWrapper](/learn/pdo-wrapper)クラスの便利なメソッドを使用します）。CORSは、`Flight::start()`が呼び出される前のシンプルなフックで処理できます。

これらの方法はすべて連携して、ウェブアプリケーションのセキュリティを維持します。セキュリティのベストプラクティスを学び、理解することは常に最優先事項であるべきです。

## 基本的な使用方法

### ヘッダー

HTTPヘッダーは、ウェブアプリケーションを安全に保つための最も簡単な方法の1つです。ヘッダーを使用してクリックジャッキング、XSS、その他の攻撃を防ぐことができます。これらのヘッダーをアプリケーションに追加する方法はいくつかあります。

ヘッダーのセキュリティを確認するための優れたウェブサイトとして、[securityheaders.com](https://securityheaders.com/)と[observatory.mozilla.org](https://observatory.mozilla.org/)があります。以下のコードを設定した後、これらの2つのウェブサイトでヘッダーが機能していることを簡単に確認できます。

#### 手動で追加

`Flight\Response`オブジェクトの`header`メソッドを使用して、これらのヘッダーを手動で追加できます。
```php
// Set the X-Frame-Options header to prevent clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Set the Content-Security-Policy header to prevent XSS
// Note: this header can get very complex, so you'll want
//  to consult examples on the internet for your application
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Set the X-XSS-Protection header to prevent XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Set the X-Content-Type-Options header to prevent MIME sniffing
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Set the Referrer-Policy header to control how much referrer information is sent
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Set the Strict-Transport-Security header to force HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Set the Permissions-Policy header to control what features and APIs can be used
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

これらは、`routes.php`または`index.php`ファイルの先頭に追加できます。

#### フィルターとして追加

以下のようにフィルター/フックで追加することもできます：

```php
// Add the headers in a filter
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### ミドルウェアとして追加

すべてのルートに適用するための最大の柔軟性を提供するミドルウェアクラスとして追加することもできます。一般的に、これらのヘッダーはすべてのHTMLおよびAPIレスポンスに適用する必要があります。

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php or wherever you have your routes
// FYI, this empty string group acts as a global middleware for
// all routes. Of course you could do the same thing and just add
// this only to specific routes.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF)は、悪意のあるウェブサイトがユーザーのブラウザにウェブサイトへのリクエストを送信させる攻撃の一種です。これを使用して、ユーザーの知らないうちにウェブサイト上でアクションを実行できます。Flightは組み込みのCSRF保護メカニズムを提供していませんが、ミドルウェアを使用して独自に簡単に実装できます。

#### セットアップ

まず、CSRFトークンを生成し、ユーザーのセッションに保存する必要があります。その後、このトークンをフォームで使用し、フォームが送信されたときにチェックできます。セッションの管理には[flightphp/session](/awesome-plugins/session)プラグインを使用します。

```php
// Generate a CSRF token and store it in the user's session
// (assuming you've created a session object at attached it to Flight)
// see the session documentation for more information
Flight::register('session', flight\Session::class);

// You only need to generate a single token per session (so it works 
// across multiple tabs and requests for the same user)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### デフォルトのPHP Flightテンプレートを使用する場合

```html
<!-- Use the CSRF token in your form -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- other form fields -->
</form>
```

##### Latteを使用する場合

LatteテンプレートでCSRFトークンを出力するカスタム関数を設定することもできます。

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// other configurations...

	// Set a custom function to output the CSRF token
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

これで、Latteテンプレートで`csrf()`関数を使用してCSRFトークンを出力できます。

```html
<form method="post">
	{csrf()}
	<!-- other form fields -->
</form>
```

#### CSRFトークンのチェック

CSRFトークンは、いくつかの方法でチェックできます。

##### ミドルウェア

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// index.php or wherever you have your routes
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ CsrfMiddleware::class ]);
```

##### イベントフィルター

```php
// This middleware checks if the request is a POST request and if it is, it checks if the CSRF token is valid
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// capture the csrf token from the form values
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// or for a JSON response
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS)は、悪意のあるフォーム入力がウェブサイトにコードを注入できる攻撃の一種です。これらの機会のほとんどは、エンドユーザーが入力するフォーム値から来ます。ユーザーの出力を**決して**信用してはいけません！彼らが皆、世界最高のハッカーであると常に想定してください。彼らは悪意のあるJavaScriptやHTMLをページに注入できます。このコードは、ユーザーの情報を盗んだり、ウェブサイト上でアクションを実行したりするために使用できます。Flightのビュークラスや[Latte](/awesome-plugins/latte)のようなテンプレートエンジンを使用すると、出力を簡単にエスケープしてXSS攻撃を防ぐことができます。

```php
// Let's assume the user is clever as tries to use this as their name
$name = '<script>alert("XSS")</script>';

// This will escape the output
Flight::view()->set('name', $name);
// This will output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// If you use something like Latte registered as your view class, it will also auto escape this.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injectionは、悪意のあるユーザーがデータベースにSQLコードを注入できる攻撃の一種です。これを使用して、データベースから情報を盗んだり、データベース上でアクションを実行したりできます。ここでも、ユーザーの入力を**決して**信用してはいけません！彼らは血を求めて来ていると常に想定してください。`PDO`オブジェクトでプリペアドステートメントを使用すると、SQLインジェクションを防ぐことができます。

```php
// Assuming you have Flight::db() registered as your PDO object
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// If you use the PdoWrapper class, this can easily be done in one line
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// You can do the same thing with a PDO object with ? placeholders
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### 安全でない例

以下は、SQLプリペアドステートメントを使用して無害な例から保護する理由です：

```php
// end user fills out a web form.
// for the value of the form, the hacker puts in something like this:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// After the query is build it looks like this
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// It looks strange, but it's a valid query that will work. In fact,
// it's a very common SQL injection attack that will return all users.

var_dump($users); // this will dump all users in the database, not just the one single username
```

### JSONPコールバックの検証

Flightの`Flight::jsonp()`メソッドを使用する場合、FlightはJSONPコールバックパラメータ名を厳格な許可リスト正規表現（`/^[A-Za-z_$][\w$.]{0,127}$/`）に対して検証することに注意してください。このパターンに一致しないコールバック名は、Flightが例外をスローし、悪意のあるコールバック値を通じて任意のJavaScriptが注入されるのを防ぎます。

この検証は組み込まれており、追加の設定は必要ありませんが、JSONPエンドポイントからの予期しないエラーをデバッグする際に知っておく価値があります。

### CORS

Cross-Origin Resource Sharing (CORS)は、ウェブページ上の多くのリソース（フォント、JavaScriptなど）が、リソースが発生したドメイン以外のドメインからリクエストされることを可能にするメカニズムです。Flightには組み込みの機能はありませんが、`Flight::start()`メソッドが呼び出される前に実行されるフックで簡単に処理できます。

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// customize your allowed hosts here.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php or wherever you have your routes
$CorsUtil = new CorsUtil();

// This needs to be run before start runs.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Flight設定の強化

Flightは、セキュリティに直接影響するいくつかのエンジン設定を公開しています。これらを正しく設定することは、アプリケーションを強化する最も簡単な方法の1つです。

#### `flight.allow_method_override`

デフォルトでは、Flightはクライアントが`X-HTTP-Method-Override`ヘッダーまたはPOSTボディの`_method`フィールドを使用して、リクエストのHTTPメソッドをオーバーライドすることを許可しています。これは`GET`/`POST`しか送信できないHTMLフォームには便利ですが、予期していない場合は危険です。攻撃者は通常のフォームを通じて`DELETE`または`PUT`リクエストを偽造できます。

アプリケーションがこの動作に依存していない場合（例：最新のクライアントや任意のHTTP動詞を送信できるJavaScriptフロントエンドが消費するAPIを構築している場合）、無効にする必要があります：

```php
// In your index.php or bootstrap file, before Flight::start()
Flight::set('flight.allow_method_override', false);
```

デフォルト値は後方互換性のために`true`ですが、この機能を明示的に必要としないアプリケーションには**`false`に設定することを強くお勧めします**。

#### `flight.debug`

Flightには、`flight.debug`設定があり、これによりハンドルされていない例外が発生したときに、ブラウザに詳細なエラー情報（例外メッセージ、コード、および完全なスタックトレース）がレンダリングされるかどうかを制御します。デフォルトは`false`で、これは一般的な`500 Internal Server Error`メッセージのみが表示されることを意味し、内部の詳細がクライアントに漏洩しません。

本番サーバーでは絶対に有効にしないでください。ローカルまたはステージング環境でのみ使用してください：

```php
// Safe for local development only — NEVER in production
Flight::set('flight.debug', true);
```

`flight.debug`が`false`（デフォルト）の場合、`flight.log_errors`を有効にすることでエラーをキャプチャできます：

```php
// Log errors server-side without exposing them to the client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### 推奨される本番設定

```php
// index.php or app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### エラーハンドリング
本番環境では、攻撃者に情報が漏洩するのを避けるために、機密性の高いエラーの詳細を非表示にします。本番環境では、`display_errors`を`0`に設定してエラーを表示する代わりにログに記録します。

```php
// In your bootstrap.php or index.php

// add this to your app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Disable error display
    ini_set('log_errors', 1);     // Log errors instead
    ini_set('error_log', '/path/to/error.log');
}

// In your routes or controllers
// Use Flight::halt() for controlled error responses
Flight::halt(403, 'Access denied');
```

### 入力のサニタイズ
ユーザーの入力を決して信用してはいけません。悪意のあるデータが紛れ込むのを防ぐために、処理する前に[filter_var](https://www.php.net/manual/en/function.filter-var.php)を使用してサニタイズします。

```php

// Lets assume a $_POST request with $_POST['input'] and $_POST['email']

// Sanitize a string input
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitize an email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### パスワードのハッシュ化
パスワードを安全に保存し、PHPの組み込み関数である[password_hash](https://www.php.net/manual/en/function.password-hash.php)と[password_verify](https://www.php.net/manual/en/function.password-verify.php)を使用して安全に検証します。パスワードは平文で保存したり、可逆的な方法で暗号化したりしてはいけません。ハッシュ化により、データベースが侵害された場合でも、実際のパスワードは保護されます。

```php
$password = Flight::request()->data->password;
// Hash a password when storing (e.g., during registration)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verify a password (e.g., during login)
if (password_verify($password, $stored_hash)) {
    // Password matches
}
```

### レート制限
キャッシュを使用してリクエストレートを制限することで、ブルートフォース攻撃やサービス拒否攻撃から保護します。

```php
// Assuming you have flightphp/cache installed and registered
// Using flightphp/cache in a filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Reset after 60 seconds
});
```

## 関連項目
- [Sessions](/awesome-plugins/session) - ユーザーセッションを安全に管理する方法。
- [Templates](/learn/templates) - テンプレートを使用して出力を自動的にエスケープし、XSSを防ぐ方法。
- [PDO Wrapper](/learn/pdo-wrapper) - プリペアドステートメントを使用した簡素化されたデータベース操作。
- [Middleware](/learn/middleware) - セキュリティヘッダーを追加するプロセスを簡素化するためのミドルウェアの使用方法。
- [Responses](/learn/responses) - 安全なヘッダーでHTTPレスポンスをカスタマイズする方法。
- [Requests](/learn/requests) - ユーザー入力を処理およびサニタイズする方法。
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - 入力サニタイズのためのPHP関数。
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - 安全なパスワードハッシュ化のためのPHP関数。
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - ハッシュ化されたパスワードを検証するためのPHP関数。

## トラブルシューティング
- Flight Frameworkのコンポーネントに関する問題のトラブルシューティング情報については、上記の「関連項目」セクションを参照してください。

## 変更履歴
- v3.18.1 - `flight.allow_method_override`、`flight.debug`、およびJSONPコールバック検証をカバーするFlight設定の強化セクションを追加。
- v3.1.0 - CORS、エラーハンドリング、入力のサニタイズ、パスワードのハッシュ化、およびレート制限に関するセクションを追加。
- v2.0 - XSSを防ぐためにデフォルトビューのエスケープを追加。