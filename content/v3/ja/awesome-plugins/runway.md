# ランナウェイ

ランナウェイは、Flightアプリケーションを管理するためのCLIアプリケーションです。コントローラーの生成、すべてのルートの表示、AIセットアップヘルパー、マイグレーション（スケルトン内）などを実行できます。優れた[adocore/php-cli](https://github.com/adhocore/php-cli)ライブラリを基にしています。

コードを表示するには[こちら](https://github.com/flightphp/runway)をクリックしてください。

スキャフォールディングコマンドは[official skeleton](https://github.com/flightphp/skeleton)と意図的に連携されており、[AI coding tools](/learn/ai)と人間が毎回同じパス、名前空間、コンストラクタインジェクションスタイルを取得できるようにしています。

## インストール

composerでインストールします。

```bash
composer require flightphp/runway
```

スケルトンはすでにランナウェイに依存しているため、プロジェクトルートから`php runway`を使用します。

## 基本設定

ランナウェイを初めて実行すると、`app/config/config.php`の`'runway'`キーを介して`runway`設定を見つけようとします。

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// オプション: スケルトンはパブリックエントリのindex_rootも使用します
		'index_root' => 'public/index.php',
    ],
];
```

> **注意** - **v1.2.0**以降、`.runway-config.json`は`app/config/config.php`に置き換えられ、非推奨となりました。古いプロジェクトをアップグレードする場合は`php runway config:migrate`で移行してください。スケルトンは互換性のため、create-project時に小さな`.runway-config.json`を書き込む場合がありますが、今後は`config.php`の`runway`キーを優先してください。

### プロジェクトルート検出

ランナウェイはプロジェクトのルートを検出する機能が十分にあり、サブディレクトリから実行しても検出できます。`composer.json`、`.git`、`app/config/config.php`などのインジケータを探して、プロジェクトルートを判断します。つまり、プロジェクト内のどこからでもランナウェイコマンドを実行できるということです！

## 使用方法

ランナウェイには、Flightアプリケーションを管理するために使用できるいくつかのコマンドがあります。ランナウェイを使用するには、2つの簡単な方法があります。

1. スケルトンプロジェクトを使用している場合は、プロジェクトのルートから`php runway [command]`を実行できます。
1. composer経由でインストールされたパッケージとしてランナウェイを使用している場合は、プロジェクトのルートから`vendor/bin/runway [command]`を実行できます。

### コマンドリスト

`php runway`コマンドを実行すると、利用可能なすべてのコマンドのリストを表示できます。

```bash
php runway
```

インストールに実際に表示されるコマンドのみに依存してください（コアランナウェイコマンドと、スケルトンの`migrate`のようなプロジェクト固有のコマンド）。

### コマンドヘルプ

任意のコマンドで`--help`フラグを渡すと、コマンドの使用方法に関する詳細情報を取得できます。

```bash
php runway routes --help
php runway make:controller --help
```

いくつかの例を以下に示します：

### コントローラーの生成

`make:controller`は公式スケルトンレイアウトに一致するコントローラーをスキャフォールドします：

| | |
|--|--|
| **パス** | `app/Controller/{Name}.php` |
| **名前空間** | `App\Controller` |
| **スタイル** | `flight\Engine`のコンストラクタインジェクション（クラス本体に`Flight::`なし） |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

期待される形状の例（簡略化）：

```php
<?php

declare(strict_types=1);

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
		// 例: $this->app->render('…', […]);
	}
}
```

Diceがコントローラーを構築できるようにクラス呼び出し可能で登録します：

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**このレイアウトの理由は？** フォルダーの**大文字小文字**は、LinuxでのComposer PSR-4のために名前空間と一致する必要があります（`controllers`ではなく`Controller`） - [Autoloading](/learn/autoloading)を参照してください。ルートとスコープ付き`AGENTS.md`ファイルがAIツールに使用するよう指示するパスも同じであり、生成されたコントローラーと手書きのコントローラーが同一に保たれます。

> 古いドキュメントやコミュニティプロジェクトでは、`app/controllers/`と`app\controllers`が使用されることがありました。*あなたの*ツリーがまだ小文字のフォルダーを使用している場合は、そのまま有効です。**新しいスケルトンプロジェクトと現在の`make:controller`出力は、`app/Controller/` + `App\Controller`を使用します。**

### アクティブレコードモデルの生成

まず、[Active Record](/awesome-plugins/active-record)プラグインをインストールしていることを確認してください。

```bash
php runway make:record users
```

公式スケルトンでは、モデルは名前空間**`App\Model`**で**`app/Model/`**の下に配置され、DB接続は**[SimplePdo](/learn/simple-pdo)**（ActiveRecordコンストラクタに注入または渡す）です。生成されるファイル名と名前空間はランナウェイの現在のデフォルトと`runway`設定に従います。新しいモデルを`App\Model`に合わせることで、[autoloading](/learn/autoloading)と`AGENTS.md`に一致するようにしてください。

スケルトンの投稿デモと一致するモデルの例：

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

古いジェネレータがまだ`app/records` / `app\records`を出力する場合は、レガシーアプリでその規約を維持するか、ファイルを`app/Model/`に移動して名前空間をフォルダーケースに合わせて更新できます。

### マイグレーション（スケルトン）

公式スケルトンには、`app/commands/`から検出されたプロジェクトコマンド（以下のようなもの）が付属しています：

```bash
php runway migrate
```

マイグレーションは`migrations/`の下にあるSQLファイル（SQLiteの場合は`YYYYMMDDHHMMSS_description.sql`、MySQLの場合は`…_description.mysql.sql`など）で、データベースドライバ設定/環境から選択されます。正確なフラグと動作はそのプロジェクトコマンドによって定義されています - アプリで`php runway migrate --help`を実行してください。

### AIヘルパー

ランナウェイは[AI & developer experience](/learn/ai)で使用されるAI指向のコマンドを公開しています：

```bash
php runway ai:init
php runway ai:generate-instructions
```

これらはLLM認証情報を保存し、プロジェクトの指示（主に**`AGENTS.md`**）を生成します。スケルトンでは、`AGENTS.md`（および`app/`の下のスコープ付きコピー）と**`SECURITY.md`**をエージェントの真実のソースとして扱ってください。

### すべてのルートの表示

現在Flightに登録されているすべてのルートを表示します。

```bash
php runway routes
```

特定のルートのみを表示したい場合は、フラグを渡してルートをフィルタリングできます。

```bash
# GETルートのみを表示
php runway routes --get

# POSTルートのみを表示
php runway routes --post

# など
```

## ランナウェイへのカスタムコマンドの追加

Flight用のパッケージを作成する場合、またはプロジェクトに独自のカスタムコマンドを追加したい場合は、プロジェクト/パッケージの`src/commands/`、`flight/commands/`、`app/commands/`、または`commands/`ディレクトリを作成することで実行できます。さらなるカスタマイズが必要な場合は、以下の設定セクションを参照してください。

スケルトンでは、プロジェクトコマンドは名前空間**`App\Command`**で**`app/commands/`**に配置されます。ランナウェイはパスでそれらを検出します。そのフォルダーは、プロジェクトがすでにComposer classmap/PSR-4と同期しているようにしてください。

コマンドを作成するには、`AbstractBaseCommand`クラスを拡張し、最低限`__construct`メソッドと`execute`メソッドを実装します。

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * コンストラクタ
     *
     * @param array<string,mixed> $config app/config/config.phpからの設定
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'ドキュメントの例を作成します', $config);
        $this->argument('<funny-gif>', '面白いgifの名前');
    }

	/**
     * 関数を実行します
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('例を作成中...');

		// ここで何かを実行

		$io->ok('例が作成されました！');
	}
}
```

Flightアプリケーションに独自のカスタムコマンドを構築する方法の詳細については、[adhocore/php-cli Documentation](https://github.com/adhocore/php-cli)を参照してください！

## 設定管理

設定は`v1.2.0`以降、`app/config/config.php`に移動したため、設定を管理するためのヘルパーコマンドがいくつかあります。

> **スケルトンのヒント:** `config.php`を**リテラル**なPHP値として保持してください。シークレットは`.env`に属します。`config.php`内に`$_ENV[...]`式を使用しないでください - `config:set`はファイルを静的データとして書き換えるため、シークレットがファイルに焼き付けられる可能性があります。[Configuration](/learn/configuration)を参照してください。

### 古い設定の移行

古い`.runway-config.json`ファイルがある場合は、次のコマンドで`app/config/config.php`に簡単に移行できます：

```bash
php runway config:migrate
```

### 設定値の設定

`config:set`コマンドを使用して設定値を設定できます。これはファイルを開かずに設定値を更新したい場合に便利です。

```bash
php runway config:set app_root "app/"
```

### 設定値の取得

`config:get`コマンドを使用して設定値を取得できます。

```bash
php runway config:get app_root
```

## すべてのランナウェイ設定

ランナウェイの設定をカスタマイズする必要がある場合は、`app/config/config.php`にこれらの値を設定できます。以下に設定できる追加の設定をいくつか示します：

```php
<?php
// app/config/config.php
return [
    // ... 他の設定値 ...

    'runway' => [
        // アプリケーションのディレクトリが配置されている場所
        'app_root' => 'app/',

        // ルートインデックスファイルが配置されているディレクトリ
        'index_root' => 'public/',

        // 他のプロジェクトのルートへのパス
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // ベースパスはほとんどの場合設定する必要はありませんが、必要であればここにあります
        'base_paths' => [
            '/includes/libs/vendor', // ベンダーディレクトリや何かに対して本当にユニークなパスがある場合
        ],

        // ファイナルパスはコマンドファイルを検索するためのプロジェクト内の場所です
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // フルパスを追加したい場合は、すぐに追加してください（プロジェクトルートからの絶対パスまたは相対パス）
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### 設定へのアクセス

設定値に効果的にアクセスする必要がある場合は、`__construct`メソッドまたは`app()`メソッドを通じてアクセスできます。`app/config/services.php`ファイルがある場合、それらのサービスもコマンドで利用できることも重要です。

```php
public function execute()
{
    $io = $this->app()->io();
    
    // 設定へのアクセス
    $app_root = $this->config['runway']['app_root'];
    
    // データベース接続などのサービスへのアクセス
    $database = $this->config['database']
    
    // ...
}
```

## AIヘルパーラッパー

ランナウェイには、AIがコマンドを生成しやすくするためのヘルパーラッパーがいくつかあります。Symfony Consoleに似た方法で`addOption`と`addArgument`を使用できます。これはAIツールを使用してコマンドを生成する場合に役立ちます。

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'ドキュメントの例を作成します', $config);
    
    // モード引数はnull可能で、完全にオプションがデフォルトです
    $this->addOption('name', '例の名前', null);
}
```

## 関連項目

- [Installation](/install) - スケルトンツリーとcreate-projectのデフォルト
- [Autoloading](/learn/autoloading) - `App\`とフォルダーケース
- [Dependency Injection](/learn/dependency-injection-container) - 生成されたコントローラーのDice + Engineインジェクション
- [AI & Developer Experience](/learn/ai) - `ai:init`、`ai:generate-instructions`、`AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - `make:record` / スケルトン`App\Model`で使用されるモデル
- [SimplePdo](/learn/simple-pdo) - スケルトンのマイグレーションとモデルで使用されるDB接続