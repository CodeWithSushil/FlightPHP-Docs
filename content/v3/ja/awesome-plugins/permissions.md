# FlightPHP/Permissions

これは、アプリに複数のロールがあり、各ロールに少し異なる機能がある場合にプロジェクトで使用できる権限モジュールです。このモジュールを使用すると、各ロールの権限を定義し、現在のユーザーが特定のページにアクセスしたり特定のアクションを実行したりする権限を持っているかどうかを確認できます。

GitHubのリポジトリは[こちら](https://github.com/flightphp/permissions)をご覧ください。

インストール
-------
`composer require flightphp/permissions` を実行すれば準備完了です！

使い方
-------
まず権限を設定し、次にアプリに権限の意味を伝えます。最終的には `$Permissions->has()`、`->can()`、または `is()` で権限を確認します。`has()` と `can()` は同じ機能を持ちますが、コードの可読性を高めるために異なる名前が付けられています。

## 基本的な例

アプリケーションにユーザーがログインしているかどうかを確認する機能があると仮定します。以下のように権限オブジェクトを作成できます：

```php
// index.php
require 'vendor/autoload.php';

// 何らかのコード

// 現在の役割が誰であるかを示すものがあるでしょう
// セッション変数から現在の役割を取得するものがあるでしょう
// ログイン後、そうでなければ「guest」または「public」の役割になります。
$current_role = 'admin';

// 権限の設定
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// このオブジェクトをFlightのどこかに保存したいでしょう
Flight::set('permission', $permission);
```

次に、コントローラー内のどこかで、以下のようなコードがあるでしょう。

```php
<?php

// 何らかのコントローラー
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// 何かを実行
		} else {
			// 他の何かを実行
		}
	}
}
```

これを使用して、アプリケーション内で何かを実行する権限があるかどうかを追跡することもできます。
たとえば、ソフトウェア上でユーザーが投稿を操作できる方法がある場合、特定のアクションを実行する権限があるかどうかを確認できます。

```php
$current_role = 'admin';

// 権限の設定
$permission = new \flight\Permission($current_role);
$permission->defineRule('post', function($current_role) {
	if($current_role === 'admin') {
		$permissions = ['create', 'read', 'update', 'delete'];
	} else if($current_role === 'editor') {
		$permissions = ['create', 'read', 'update'];
	} else if($current_role === 'author') {
		$permissions = ['create', 'read'];
	} else if($current_role === 'contributor') {
		$permissions = ['create'];
	} else {
		$permissions = [];
	}
	return $permissions;
});
Flight::set('permission', $permission);
```

次に、コントローラー内のどこかで...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// 何かを実行
		} else {
			// 他の何かを実行
		}
	}
}
```

## 依存性の注入
権限を定義するクロージャに依存性を注入できます。これは、トグル、ID、またはチェックしたい他のデータポイントがある場合に便利です。Class->Method タイプの呼び出しでも同じことが機能しますが、引数はメソッド内で定義します。

### クロージャ

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... コード
});

// コントローラーファイル内
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// 何かを実行
	} else {
		// 他の何かを実行
	}
}
```

### クラス

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... コード
	}
}
```

## クラスを使用した権限設定のショートカット
クラスを使用して権限を定義することもできます。これは、多数の権限があり、コードをクリーンに保ちたい場合に便利です。以下のようにできます：
```php
<?php

// ブートストラップコード
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// 事前に設定したと仮定
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // 誰でも注文を表示できます
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // マネージャーは注文を作成できます
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // ユーザーが特別なトグルを持っている場合、注文を更新できます
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // 管理者は注文を削除できます
		}
		return $allowed_permissions;
	}
}
```
素晴らしい点は、ショートカット（キャッシュも可能！）を使用できることです。権限クラスにクラス内のすべてのメソッドを権限にマッピングするよう指示するだけです。したがって、`order()` という名前のメソッドと `company()` という名前のメソッドがある場合、これらは自動的にマッピングされるため、`$Permissions->has('order.read')` または `$Permissions->has('company.read')` を実行するだけで機能します。これを定義するのは非常に難しいので、注意してください。以下のようにするだけです：

一緒にグループ化したい権限のクラスを作成します。
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// 権限を決定するコード
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// 権限を決定するコード
		return $permissions_array;
	}
}
```

次に、このライブラリを使用して権限を発見可能にします。

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

最後に、コードベースで権限を呼び出して、ユーザーが特定の権限を実行できるかどうかを確認します。

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('注文を作成できません。申し訳ありません！');
		}
	}
}
```

### キャッシュ

キャッシュを有効にするには、シンプルな[wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache) ライブラリを参照してください。有効にする例を以下に示します。
```php

// この $app はコードの一部である場合もあれば、
// null を渡すだけでコンストラクタで Flight::app() から取得できます
$app = Flight::app();

// 現在はファイルキャッシュとしてこれを受け入れます。他のものは簡単に
// 将来追加できます。
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 はこれをキャッシュする秒数です。キャッシュを使用しない場合はこれを省略してください
```

それでは始めましょう！