# セキュリティ

## 概要

Webアプリケーションにおいて、セキュリティは非常に重要です。アプリケーションを安全に保ち、ユーザーのデータを保護する必要があります。Flightは、Webアプリケーションを保護するための多くの機能を提供します。

公式の[skeleton](https://github.com/flightphp/skeleton)には、専用の**`SECURITY.md`**とセキュリティヘッダーミドルウェアも同梱されており、[AIコーディングツール](/learn/ai)（および人間）が、`AGENTS.md`の一般的なコーディングスタイルとは別に、シークレット、ヘッダー、XSS/SQLルールを意図的に配置できる場所を提供します。

## 理解

Webアプリケーションを構築する際に注意すべき一般的なセキュリティ脅威がいくつかあります。最も一般的な脅威には次のようなものがあります。
- クロスサイトリクエストフォージェリ（CSRF）
- クロスサイトスクリプティング（XSS）
- SQLインジェクション
- クロスオリジンリソースシェアリング（CORS）

[テンプレート](/learn/templates)は、出力をデフォルトでエスケープすることでXSSを防ぐのに役立ちます（TwigとLatteはこれを自動で行います。この利点を活かしてください）。[セッション](/awesome-plugins/session)は、以下に説明するようにCSRFトークンをユーザーのセッションに保存することでCSRFを防ぐことができます。PDOでのプリペアドステートメント、または[SimplePdo](/learn/simple-pdo)のヘルパーを使用すると、SQLインジェクションを防ぐことができます。CORSは、`Flight::start()`が呼び出される前のシンプルなフックで処理できます。

これらの方法はすべて連携して、Webアプリケーションのセキュリティを維持します。セキュリティのベストプラクティスを学び、理解することを常に最優先にしてください。トレードオフを理解せずにページを読み込むためだけに、AIアシスタントに「CSPを無効にする」またはヘッダーを弱めるように依頼してはいけません。

## 基本的な使い方

### ヘッダー

HTTPヘッダーは、Webアプリケーションを保護する最も簡単な方法のひとつです。クリックジャッキング、XSS、その他の攻撃を防ぐためにヘッダーを使用できます。アプリケーションにこれらのヘッダーを追加する方法はいくつかあります。

ヘッダーのセキュリティを確認するのに役立つ優れたWebサイトは、[securityheaders.com](https://securityheaders.com/)と[observatory.mozilla.org](https://observatory.mozilla.org/)です。以下のコードを設定したら、これらのWebサイトでヘッダーが機能していることを簡単に確認できます。

skeletonには、`App\Middleware\SecurityHeadersMiddleware`（リクエストごとのnonce、フレームオプション、HSTSなどを備えたCSP）が含まれています。ヘッダーを無効にするよりも、意図的に拡張することを推奨します。

#### 手動で追加する

`Flight\Response`オブジェクトの`header`メソッドを使用して、これらのヘッダーを手動で追加できます。
```php
// クリックジャッキングを防ぐためにX-Frame-Optionsヘッダーを設定
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// XSSを防ぐためにContent-Security-Policyヘッダーを設定
// 注：このヘッダーは非常に複雑になるため、
//  アプリケーションに合わせてインターネット上の例を参照してください
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// XSSを防ぐためにX-XSS-Protectionヘッダーを設定
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// MIMEスニッフィングを防ぐためにX-Content-Type-Optionsヘッダーを設定
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// 送信されるリファラー情報の量を制御するためにReferrer-Policyヘッダーを設定
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// HTTPSを強制するためにStrict-Transport-Securityヘッダーを設定
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// 使用できる機能とAPIを制御するためにPermissions-Policyヘッダーを設定
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

これらは、`routes.php`または`index.php`ファイルの先頭に追加できます。

#### フィルターとして追加する

次のように、フィルター/フックで追加することもできます。

```php
// フィルターでヘッダーを追加
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

#### ミドルウェアとして追加する

適用するルートを柔軟に指定できるミドルウェアクラスとして追加することもできます。一般に、これらのヘッダーはすべてのHTMLおよびAPIレスポンスに適用する必要があります。

skeletonスタイルのパスと名前空間（**フォルダーの大文字小文字は`App\Middleware`と一致**）：

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

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
		// インラインスクリプトがある場合は、ブートストラップのCSP nonceを優先します（skeletonはcsp_nonceを設定します）
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — 空の文字列グループ = すべてのルートに適用されるグローバルミドルウェア
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// その他のルート
}, [SecurityHeadersMiddleware::class]);
```

古いプロジェクトでは`app/middlewares`と`app\middlewares`がまだ使用されている場合があります。フォルダーが一致していれば機能します。新しいskeletonアプリでは**`app/Middleware/`**と**`App\Middleware`**を使用します。[オートローディング](/learn/autoloading)を参照してください。

### クロスサイトリクエストフォージェリ（CSRF）

クロスサイトリクエストフォージェリ（CSRF）は、悪意のあるWebサイトがユーザーのブラウザーにあなたのWebサイトへのリクエストを送信させる攻撃の一種です。これは、ユーザーの知らないうちにあなたのWebサイトでアクションを実行するために使用される可能性があります。Flightには組み込みのCSRF保護メカニズムはありませんが、ミドルウェアを使用して簡単に独自のものを実装できます。

#### セットアップ

まず、CSRFトークンを生成してユーザーのセッションに保存する必要があります。次に、このトークンをフォームで使用し、フォームが送信されたときにチェックできます。セッションを管理するには、[flightphp/session](/awesome-plugins/session)プラグインを使用します。

```php
// CSRFトークンを生成し、ユーザーのセッションに保存します
// （セッションオブジェクトを作成してFlightにアタッチしていると仮定します）
// 詳細はセッションドキュメントを参照してください
Flight::register('session', flight\Session::class);

// セッションごとにトークンを1つ生成するだけで済みます（同じユーザーの
// 複数のタブやリクエストにわたって機能します）
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### デフォルトのPHP Flightテンプレートを使用する場合

```html
<!-- フォームでCSRFトークンを使用 -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- その他のフォームフィールド -->
</form>
```

##### Twigを使用する場合（skeletonのデフォルト）

Twig関数を登録するか、トークンをすべてのフォームビューに渡します。グローバルとフォームフィールドを使用した最小限の例：

```php
// Twigを設定するとき（例：services.php）
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# その他のフィールド #}
</form>
```

##### Latteを使用する場合

LatteテンプレートでCSRFトークンを出力するカスタム関数を設定することもできます。

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// その他の設定...

	// CSRFトークンを出力するカスタム関数を設定
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

そして、Latteテンプレートで`csrf()`関数を使用してCSRFトークンを出力できます。

```html
<form method="post">
	{csrf()}
	<!-- その他のフォームフィールド -->
</form>
```

#### CSRFトークンのチェック

CSRFトークンは、いくつかの方法でチェックできます。

##### ミドルウェア

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

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

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// その他のルート
}, [CsrfMiddleware::class]);
```

##### イベントフィルター

```php
// このミドルウェアは、リクエストがPOSTかどうかを確認し、POSTの場合はCSRFトークンが有効かどうかをチェックします
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// フォームの値からCSRFトークンを取得
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// またはJSONレスポンスの場合
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### クロスサイトスクリプティング（XSS）

クロスサイトスクリプティング（XSS）は、悪意のあるフォーム入力をあなたのWebサイトにコードを注入できる攻撃の一種です。これらの機会のほとんどは、エンドユーザーが記入するフォームの値から発生します。ユーザーからの出力を**決して**信頼してはいけません！すべてのユーザーが世界最高のハッカーであると常に想定してください。彼らは悪意のあるJavaScriptやHTMLをあなたのページに注入する可能性があります。このコードは、ユーザーから情報を盗んだり、あなたのWebサイトでアクションを実行したりするために使用される可能性があります。Flightのビュークラスまたは[Twig](/awesome-plugins/twig)や[Latte](/awesome-plugins/latte)のようなテンプレートエンジンを使用すると、XSS攻撃を防ぐために出力を簡単にエスケープできます。

```php
// ユーザーが賢くて、これを自分の名前として使用しようとしていると仮定します
$name = '<script>alert("XSS")</script>';

// これにより出力がエスケープされます
Flight::view()->set('name', $name);
// これにより出力されます: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig（skeletonのデフォルト）とLatteはデフォルトで自動エスケープします — 生のPHP echoよりもこれらを優先してください
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → エスケープされます
// コンテンツが完全に信頼できる場合を除き、|raw やエスケープされていない出力は避けてください
```

### SQLインジェクション

SQLインジェクションは、悪意のあるユーザーがSQLコードをデータベースに注入できる攻撃の一種です。これは、データベースから情報を盗んだり、データベースでアクションを実行したりするために使用される可能性があります。繰り返しますが、ユーザーからの入力を**決して**信頼してはいけません！常に彼らは血を求めていると想定してください。プリペアドステートメントを使用してください。[SimplePdo](/learn/simple-pdo)ヘルパーはこれをデフォルトのパスにします。

```php
// Flight::db() が SimplePdo として登録されていると仮定します（またはコントローラーに SimplePdo を注入します）
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo（推奨）— バインドされたパラメーターを使用したワンライナー
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// ？プレースホルダーでも同様
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

skeletonスタイルのコントローラーでは、テストとAI生成コードの一貫性を保つために、`Flight::db()`よりも`SimplePdo`のコンストラクター注入を推奨します（[DIC](/learn/dependency-injection-container)）。

#### 安全でない例

以下は、SQLプリペアドステートメントを使用して、以下のような無害な例から保護する理由です。

```php
// エンドユーザーがWebフォームに記入します。
// フォームの値として、ハッカーは次のようなものを入力します：
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// クエリが構築された後、次のようになります
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// 奇妙に見えますが、これは機能する有効なクエリです。実際、
// これはすべてのユーザーを返す非常に一般的なSQLインジェクション攻撃です。

var_dump($users); // これにより、データベース内のすべてのユーザーがダンプされます（単一のユーザー名だけでなく）
```

### シークレットと設定

- シークレットは**`.env`**（または実際の環境）に置き、コミットされた`config.php`サンプルには置かないでください。
- skeletonのルール：`config.php`にはリテラルのデフォルト値を設定し、ブートストラップでenvをマージします。コントローラー内で`$_ENV`を読み取らず、代わりに設定を注入してください。[設定](/learn/configuration)を参照してください。
- APIキー、DBパスワード、セッション暗号化キーをコミットしないでください。AIツールに**`SECURITY.md`**を指定して、安全でないショートカットを作成しないようにしてください。

### JSONPコールバックの検証

Flightの`Flight::jsonp()`メソッドを使用する場合、FlightはJSONPコールバックパラメーター名を厳格な許可リストregex（`/^[A-Za-z_$][\w$.]{0,127}$/`）に対して検証することに注意してください。このパターンに一致しないコールバック名があると、Flightは例外をスローし、悪意のあるコールバック値を介した任意のJavaScriptの注入を防ぎます。

この検証は組み込まれており、追加の設定は不要ですが、JSONPエンドポイントから予期しないエラーをデバッグするときに知っておくと便利です。

### CORS

クロスオリジンリソースシェアリング（CORS）は、Webページ上の多くのリソース（フォント、JavaScriptなど）を、そのリソースが発信されたドメインの外部の別のドメインからリクエストできるようにするメカニズムです。Flightには組み込みの機能はありませんが、`Flight::start()`メソッドが呼び出される前に実行されるフックで簡単に処理できます。

```php
// app/Utils/CorsUtil.php  (skeleton: PascalCaseのUtilsフォルダー → App\Utils)

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
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
		// 許可するホストをここでカスタマイズします。
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// ブートストラップ / ルート — startの前に実行
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Flight設定の堅牢化

Flightには、セキュリティに直接影響するいくつかのエンジン設定があります。これらを正しく設定することは、アプリケーションを堅牢化する最も簡単な方法のひとつです。

#### `flight.allow_method_override`

デフォルトでは、Flightはクライアントが`X-HTTP-Method-Override`ヘッダーまたはPOST本文の`_method`フィールドを使用してリクエストのHTTPメソッドを上書きできるようにします。これは`GET`/`POST`のみを送信できるHTMLフォームには便利ですが、予期していない場合は危険です。攻撃者が通常のフォームを介して`DELETE`または`PUT`リクエストを偽造する可能性があります。

アプリケーションがこの動作に依存していない場合（たとえば、任意のHTTP動詞を送信できる最新のクライアントやJavaScriptフロントエンドが使用するAPIを構築している場合）、これを無効にする必要があります。

```php
// index.phpまたはブートストラップファイル内、Flight::start()の前
Flight::set('flight.allow_method_override', false);
```

デフォルト値は後方互換性のため`true`ですが、オーバーライド機能を明示的に必要としないアプリケーションでは**`false`に設定することを強くお勧めします**。

#### `flight.debug`

Flightには、未処理の例外が発生したときに、詳細なエラー情報（例外メッセージ、コード、完全なスタックトレース）をブラウザーに表示するかどうかを制御する`flight.debug`設定があります。デフォルトは`false`で、クライアントには内部詳細が漏洩せず、一般的な`500 Internal Server Error`メッセージのみが表示されます。

本番サーバーでこれを有効にしないでください。ローカルまたはステージング環境でのみ使用してください。

```php
// ローカル開発のみで安全 — 本番では絶対に使用しないでください
Flight::set('flight.debug', true);
```

`flight.debug`が`false`（デフォルト）の場合でも、`flight.log_errors`を有効にすることでエラーをキャプチャできます。

```php
// クライアントに公開せずにサーバー側でエラーを記録
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### 推奨される本番設定

```php
// index.phpまたはアプリ設定/ブートストラップから適用
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### エラーハンドリング

本番環境では、攻撃者に情報を漏洩しないように、機密性の高いエラー詳細を非表示にします。本番環境では、`display_errors`を`0`に設定してエラーを表示するのではなく、ログに記録します。

```php
// bootstrap.php または index.php 内

// これを app/config/config.php に追加します
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // エラー表示を無効化
    ini_set('log_errors', 1);     // 代わりにエラーをログに記録
    ini_set('error_log', '/path/to/error.log');
}

// ルートまたはコントローラー内
// 制御されたエラーレスポンスには Flight::halt() を使用
Flight::halt(403, 'Access denied');
```

### 入力のサニタイズ

ユーザー入力を信頼しないでください。悪意のあるデータが混入するのを防ぐために、処理前に[filter_var](https://www.php.net/manual/en/function.filter-var.php)を使用してサニタイズします。アプリコードでは、生の`$_GET` / `$_POST`ではなく、`$app->request()`（または`Flight::request()`）を介して入力を読み取ることを推奨します。

```php

// $_POST['input'] と $_POST['email'] を持つ $_POST リクエストを想定します

// 文字列入力をサニタイズ
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// メールアドレスをサニタイズ
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### パスワードのハッシュ化

PHPの組み込み関数である[password_hash](https://www.php.net/manual/en/function.password-hash.php)や[password_verify](https://www.php.net/manual/en/function.password-verify.php)を使用して、パスワードを安全に保存および検証します。パスワードは平文で保存してはいけません。また、可逆的な方法で暗号化してもいけません。ハッシュ化により、データベースが侵害された場合でも、実際のパスワードは保護されたままになります。

```php
$password = Flight::request()->data->password;
// 保存時にパスワードをハッシュ化（例：登録時）
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// パスワードを検証（例：ログイン時）
if (password_verify($password, $stored_hash)) {
    // パスワードが一致
}
```

### レート制限

キャッシュを使用してリクエストレートを制限し、ブルートフォース攻撃やサービス拒否攻撃から保護します。

```php
// flightphp/cache がインストールおよび登録されていると仮定します
// フィルターで flightphp/cache を使用
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // 60秒後にリセット
});
```

## 関連情報
- [セッション](/awesome-plugins/session) - ユーザーセッションを安全に管理する方法。
- [テンプレート](/learn/templates) - Twig/Latteの自動エスケープとXSS。
- [SimplePdo](/learn/simple-pdo) - プリペアドステートメントを使用したデータベースヘルパー。
- [PdoWrapper](/learn/pdo-wrapper) - 非推奨です。新しいコードにはSimplePdoを使用してください。
- [ミドルウェア](/learn/middleware) - セキュリティヘッダーを追加するプロセスを簡素化するためのミドルウェアの使用方法。
- [設定](/learn/configuration) - `.env`とリテラル設定、本番フラグ。
- [AIと開発者エクスペリエンス](/learn/ai) - エージェント向けに`SECURITY.md`にセキュリティポリシーを保持します。
- [レスポンス](/learn/responses) - セキュリティヘッダーを使用してHTTPレスポンスをカスタマイズする方法。
- [リクエスト](/learn/requests) - ユーザー入力を処理およびサニタイズする方法。
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - 入力サニタイズ用のPHP関数。
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - 安全なパスワードハッシュ化のためのPHP関数。
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - ハッシュ化されたパスワードを検証するためのPHP関数。

## トラブルシューティング
- Flight Frameworkのコンポーネントに関する問題のトラブルシューティング情報については、上記の「関連情報」セクションを参照してください。
- CSPがスクリプトをブロックする場合は、nonce（skeletonパターン）を追加するか、特定のオリジンを許可リストに追加してください。計画なしに`script-src *`を設定しないでください。

## 変更履歴
- ドキュメント - skeleton `App\Middleware`、Twig CSRF/XSSメモ、SimplePdo、シークレット/`.env`、AIフレンドリーなプロジェクト向け`SECURITY.md`。
- v3.18.1 - `flight.allow_method_override`、`flight.debug`、JSONPコールバック検証をカバーするFlight設定の堅牢化セクションを追加。
- v3.1.0 - CORS、エラーハンドリング、入力サニタイズ、パスワードハッシュ化、レート制限に関するセクションを追加。
- v2.0 - XSSを防ぐためにデフォルトビューのエスケープを追加。