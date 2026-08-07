# オートローディング

## 概要

オートローディングとは、PHPにおいてクラスを読み込むディレクトリを指定する概念です。`require` や `include` を使ってクラスを読み込むよりもはるかに有益です。また、Composerパッケージを使用するための要件でもあります。

オートローディングを正しく設定することは、[AI支援開発](/learn/ai) にとっても重要です。エージェントは名前空間が示す場所にファイルを配置するからです。フォルダの**大文字小文字**と名前空間の大文字小文字が一致しない場合、大文字小文字を区別しないMacのディスク上では「動いていた」としても、Linuxではクラスが見つからないエラーが発生します。

## 理解

デフォルトでは、あらゆる `Flight` クラスはComposerのおかげで自動的にオートロードされます。**あなた自身の**アプリケーションクラスについては、一般的な2つの方法があります。

1. **Composer PSR-4**（[公式スケルトン](https://github.com/flightphp/skeleton) が使用）: `composer.json` で名前空間のプレフィックスをディレクトリにマッピングし、`composer dump-autoload` を実行します。
2. **`Flight::path()`**: Flightのローダーにディレクトリを指定します（シンプルなアプリや、アプリコードにComposerを使わない場合に便利です）。

オートローダーを使うとコードが大幅に簡素化されます。毎回ファイルの先頭に大量の `include` / `require` を並べる代わりに、クラスを最初に使用したときに読み込まれます。

### 大文字小文字の区別（2回読んでください）

**名前空間はディレクトリ構造と、そのディレクトリの大文字小文字の両方に一致している必要があります。**

| 動作する | Linuxで壊れる |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` でフォルダが `app/controllers/` の場合 |
| `app\controllers\MyController` → `app/controllers/MyController.php` | `App\` と小文字の `controllers` を混在させる |

PHPの名前空間は一部の文脈では大文字小文字を区別しませんが、**Composerとファイルシステムは区別します**。公式スケルトンは次のように統一しています。

- Composer: `"App\\": "app/"`
- フォルダ: **`Controller`**、**`Middleware`**、**`Model`**、**`Utils`**（パスカルケース）。`controllers` / `middlewares` ではありません

古いドキュメントやコミュニティの例では、小文字の `app\controllers` が使われることがありました。フォルダが小文字であればそれでも機能します。ただし**新しいスケルトンプロジェクトでは `App\` + パスカルケースのフォルダを使用します**。プロジェクトごとに1つの規約を選んでそれを守り、人間とAIツールが別のレイアウトを作り出さないようにしてください。

## スケルトン（新規プロジェクト向け推奨）

`composer create-project flightphp/skeleton` の後、アプリコードはComposer経由でオートロードされます。`App\` クラスに対して `Flight::path()` は不要です。

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Diceがコンテナ経由でApp\Controller\… を解決します
$router->get('/', [HomeController::class, 'index']);
```

完全なツリーは [インストール](/install) を、コーディング支援ツール向けにこのレイアウトを `AGENTS.md` がどう文書化するかは [AIと開発者体験](/learn/ai) を参照してください。

## 基本的な使用方法（`Flight::path()`）

次のようなディレクトリツリーがあると仮定します。

```text
# パスの例
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - このプロジェクトのコントローラを格納
│   ├── translations
│   ├── UTILS - このアプリケーション専用のクラスを格納（後で例を示すため意図的にすべて大文字）
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

これは典型的なアプリツリーに似ていることに気付いたかもしれません（ドキュメントサイト自体も構造化レイアウトを使用しています）。ここで小文字の `controllers` は有効な*選択肢*です。ただ、スケルトンの現在のデフォルトではありません。

各ディレクトリを次のように指定して読み込むことができます。

```php

/**
 * public/index.php
 */

// オートローダーにパスを追加
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// 名前空間は不要

// オートロードされるすべてのクラスはパスカルケース（各単語の先頭を大文字にし、スペースを入れない）を推奨
class MyController {

	public function index() {
		// 何かを行う
	}
}
```

## `Flight::path()` で名前空間を使う

名前空間を使う場合、実装は非常に簡単になります。`Flight::path()` メソッドで、アプリケーションのルートディレクトリ（ドキュメントルートや `public/` フォルダではなく）を指定する必要があります。

```php

/**
 * public/index.php
 */

// オートローダーにパスを追加
Flight::path(__DIR__.'/../');
```

これで、コントローラは次のようになります。下の例を見てください。ただし、重要な情報はコメントに注目してください。

```php
/**
 * app/controllers/MyController.php
 */

// 名前空間は必須
// 名前空間はディレクトリ構造と同じ
// 名前空間はディレクトリ構造と同じ大文字小文字に従う必要がある
// 名前空間とディレクトリにはアンダースコアを含めることはできない（Loader::setV2ClassLoading(false) が設定されている場合を除く）
namespace app\controllers;

// オートロードされるすべてのクラスはパスカルケース（各単語の先頭を大文字にし、スペースを入れない）を推奨
// 3.7.2以降、Loader::setV2ClassLoading(false); を実行することでクラス名にPascal_Snake_Caseを使用できます
class MyController {

	public function index() {
		// 何かを行う
	}
}
```

そして、utilsディレクトリ内のクラスをオートロードしたい場合は、基本的に同じことを行います。

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// 名前空間はディレクトリ構造と大文字小文字に一致させる必要がある（上記のファイルツリーのようにUTILSディレクトリがすべて大文字であることに注意）
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// 何かを行う
	}
}
```

### スケルトンスタイルの名前空間（同じルール、異なる大文字小文字）

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

ルールは変わりません。スケルトンが選んだフォルダ/名前空間の大文字小文字が変わるだけです。**フォルダがどの大文字小文字でも、`namespace` 行はそれに一致させる必要があります。**

## クラス名のアンダースコア

3.7.2以降、`Loader::setV2ClassLoading(false);` を実行することでクラス名にPascal_Snake_Caseを使用できます。これにより、クラス名にアンダースコアを使用できるようになります。推奨はされませんが、必要とする人のために利用可能です。

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// オートローダーにパスを追加
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// 名前空間は不要

class My_Controller {

	public function index() {
		// 何かを行う
	}
}
```

## 関連情報
- [インストール](/install) - スケルトンツリーと新規プロジェクト向けの `App\` デフォルト。
- [ルーティング](/learn/routing) - ルートをコントローラにマッピングしてビューをレンダリングする方法。
- [依存性注入](/learn/dependency-injection-container) - コントローラが `Engine` やサービスを取得する方法。
- [AIと開発者体験](/learn/ai) - `AGENTS.md` を使ってエージェントをレイアウトに合わせる方法。
- [なぜフレームワークを使うのか？](/learn/why-frameworks) - Flightのようなフレームワークを使う利点を理解する。

## トラブルシューティング
- 名前空間付きクラスが見つからない理由がどうしても分からない場合は、`Flight::path()` では**プロジェクトルート**（または名前空間の正しいベース）を指すようにしてください。名前空間に反映し忘れた入れ子フォルダだけを指すのは避けましょう。
- Composer PSR-4の場合、`composer.json` のマッピングを変更した後に `composer dump-autoload` を実行してください。
- LinuxのCIや本番環境では、フォルダの大文字小文字の誤りが「自分の環境では動くのに」という失敗の非常に一般的な原因です。

### クラスが見つからない（オートローディングが機能していない）

これが発生する理由はいくつか考えられます。以下に例を示します。

#### ファイル名の誤り
最も一般的なのは、クラス名がファイル名と一致しないことです。

`MyClass` というクラスがある場合、ファイル名は `MyClass.php` でなければなりません。`MyClass` というクラスなのにファイル名が `myclass.php` の場合、オートローダーはそれを見つけることができません。

#### 名前空間またはフォルダの大文字小文字の誤り
名前空間を使用している場合、名前空間はディレクトリ構造に**大文字小文字も含めて**一致する必要があります。

```php
// ...コード...

// MyControllerが app/Controller（スケルトン）にあり、名前空間が App\Controller の場合
// これは機能しません:
Flight::route('/hello', 'MyController->hello');

// スケルトンスタイル:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// 古い小文字レイアウト（フォルダが実際に app/controllers の場合のみ）:
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// または完全修飾名:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` が定義されていない（Composer以外のアプリコード）

アプリケーションクラスにComposerではなく `Flight::path()` を頼る場合、それらのクラスを参照するルートの前にパスを定義してください（多くの場合、ブートストラップの早い段階または `public/index.php` 内）。

```php
// オートローダーにパスを追加（名前空間付きアプリの場合はプロジェクトルート）
Flight::path(__DIR__.'/../');
```

公式スケルトンは主に `App\` に対して**Composer PSR-4**を使用するため、通常コントローラやモデルに `Flight::path()` は必要ありません。

## 変更履歴
- ドキュメント – スケルトンの `App\` + パスカルケースフォルダと、人間とAIツール向けの大文字小文字の落とし穴を文書化。
- v3.7.2 - `Loader::setV2ClassLoading(false);` を実行することでクラス名にPascal_Snake_Caseを使用できます。
- v2.0 - オートロード機能が追加されました。