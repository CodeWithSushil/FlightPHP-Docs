# インストール手順

Flightをインストールする前に、いくつかの前提条件があります。具体的には以下が必要です：

1. [システムにPHPをインストール](#installing-php)
2. 最良の開発体験のために[Composerをインストール](https://getcomposer.org)

## 基本的なインストール

[Composer](https://getcomposer.org)を使用している場合は、次のコマンドを実行できます：

```bash
composer require flightphp/core
```

これにより、Flightのコアファイルだけがシステムにインストールされます。プロジェクト構造、[レイアウト](/learn/templates)、[依存関係](/learn/dependency-injection-container)、[設定](/learn/configuration)、[オートローディング](/learn/autoloading)などは自分で定義する必要があります。この方法では、Flight以外の依存関係はインストールされません。

[ファイルをダウンロード](https://github.com/flightphp/core/archive/master.zip)して、Webディレクトリに直接展開することもできます。

基本的なインストールは、学習、マイクロAPI、コピー＆ペーストの実験に最適です。人間と[AIコーディングツール](/learn/ai)が同じ方法で従える完全なアプリレイアウトが必要な場合は、以下の推奨スケルトンを使用してください。

## 推奨インストール

新しいプロジェクトには、[flightphp/skeleton](https://github.com/flightphp/skeleton)アプリから始めることを強くお勧めします。インストールは簡単です。

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# オプションのサンプルDB + 投稿デモ
php runway migrate
```

このステップにより、プロジェクト構造、Composer PSR-4オートローディング、設定、および[Tracy](/awesome-plugins/tracy)、[Tracy Extensions](/awesome-plugins/tracy-extensions)、[Runway](/awesome-plugins/runway)などのツールがセットアップされます。また、ルートの**`AGENTS.md`**（および`app/`配下のスコープ付きコピー）が同梱されているため、AIアシスタントはあなたと同じレイアウトを共有できます。[AIと開発者体験](/learn/ai)を参照してください。

### スケルトンが提供するもの

```text
project-root/
├── AGENTS.md              # AI / エージェントの情報源
├── SECURITY.md            # セキュリティの期待値
├── .env.example           # シークレット / デプロイオーバーレイ（.envにコピー）
├── public/index.php       # Webエントリのみ
├── app/
│   ├── config/            # bootstrap、routes、services、config_sample.php
│   ├── Controller/        # App\Controller\*（パスカルケースのフォルダ！）
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\*（ActiveRecord）
│   ├── Utils/             # Config、Env、DatabaseFactory
│   ├── commands/          # Runway CLIコマンド
│   ├── views/             # Twigテンプレート（*.twig）
│   ├── cache/
│   └── log/
├── migrations/            # SQLマイグレーション（.sql / .mysql.sql）
└── tests/                 # PHPUnit
```

**名前空間はフォルダーの大文字小文字に従います。** Composerは`"App\\": "app/"`をマッピングするため、次のようになります：

| ディスク上のパス | 名前空間 |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Linuxでは、`app/controller/`は`app/Controller/`と同じ**ではありません**。オートローディングは大文字小文字を区別します。スケルトンのパスカルケースフォルダーに合わせてください。詳細：[オートローディング](/learn/autoloading)。

**スタックのデフォルト（新規プロジェクト）：** Twigビュー、SimplePdo + ActiveRecord、`Engine`注入を使用したDice（アプリクラス内での`Flight::`の使用は避ける）、`php runway migrate`後のオプションのSQLite。

`create-project`は通常、`app/config/config_sample.php`を`config.php`に、`.env.example`を`.env`にコピーします（存在する場合）。ルートは`app/config/routes.php`に、サービスとDIは`app/config/services.php`にあります。

> **ドキュメント ↔ スケルトン：** これらのドキュメントはFlightの**API**を教えます（多くの場合、短い`Flight::`サンプルを使用）。スケルトンは**アプリケーションの形**を固定します。`app/`配下にコードを追加する場合は、スケルトンのツリーに従ってください。メソッド名、オプション、プラグインについてはドキュメントを使用してください。

## Webサーバーの設定

### PHPビルトイン開発サーバー

これは断然最も簡単な起動方法です。ビルトインサーバーを使用してアプリケーションを実行でき、データベースにSQLiteを使用することもできます（システムにsqlite3がインストールされていれば）。PHPがインストールされていれば、次のコマンドを実行するだけです：

```bash
php -S localhost:8000
# またはスケルトンアプリの場合
composer start
```

その後、ブラウザを開いて`http://localhost:8000`にアクセスします。

プロジェクトのドキュメントルートを別のディレクトリにしたい場合（例：プロジェクトが`~/myproject`で、ドキュメントルートが`~/myproject/public/`の場合）、`~/myproject`ディレクトリにいる状態で次のコマンドを実行できます：

```bash
php -S localhost:8000 -t public/
# スケルトンアプリでは、これは既に設定されています
composer start
```

その後、ブラウザを開いて`http://localhost:8000`にアクセスします。

### Apache

Apacheがシステムにインストールされていることを確認してください。インストールされていない場合は、お使いのシステムへのApacheのインストール方法をGoogleで検索してください。

Apacheの場合、`.htaccess`ファイルを次のように編集します：

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **注：** flightをサブディレクトリで使用する必要がある場合は、`RewriteEngine On`の直後に`RewriteBase /subdir/`の行を追加してください。

> **注：** DBやenvファイルなど、すべてのサーバーファイルを保護したい場合は、`.htaccess`ファイルに次のように記述します：

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Nginxがシステムにインストールされていることを確認してください。インストールされていない場合は、お使いのシステムへのNginxのインストール方法をGoogleで検索してください。

Nginxの場合、サーバー宣言に次を追加します：

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## `index.php`ファイルを作成する

基本的なインストールを行う場合は、開始するためのコードが必要です。

```php
<?php

// Composerを使用している場合は、オートローダーを読み込みます。
require 'vendor/autoload.php';
// Composerを使用しない場合は、フレームワークを直接読み込みます
// require 'flight/Flight.php';

// 次にルートを定義し、リクエストを処理する関数を割り当てます。
Flight::route('/', function () {
  echo 'hello world!';
});

// 最後に、フレームワークを起動します。
Flight::start();
```

スケルトンアプリでは、パブリックエントリはアプリを起動するだけです。ルートは`app/config/routes.php`で登録されます（通常は`[App\Controller\…::class, 'method']`の形式で、Diceが依存関係を注入できるようにします）。サービス、Twig、SimplePdo、コンテナは`app/config/services.php`で配線されます。この構造は、AIツールと人間が毎回同じ場所を編集するように意図されています。

## PHPのインストール

お使いのシステムに`php`が既にインストールされている場合は、これらの手順をスキップして[ダウンロードセクション](#download-the-files)に進んでください。

### **macOS**

#### **Homebrewを使用したPHPのインストール**

1. **Homebrewをインストール**（まだインストールされていない場合）：
   - ターミナルを開いて実行：
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **PHPをインストール**：
   - 最新バージョンをインストール：
     ```bash
     brew install php
     ```
   - 特定のバージョン（例：PHP 8.1）をインストールする場合：
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **PHPバージョンの切り替え**：
   - 現在のバージョンのリンクを解除し、希望するバージョンにリンク：
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - インストールされたバージョンを確認：
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **PHPの手動インストール**

1. **PHPをダウンロード**：
   - [PHP for Windows](https://windows.php.net/download/)にアクセスし、最新版または特定のバージョン（例：7.4、8.0）を非スレッドセーフ版のzipファイルとしてダウンロードします。

2. **PHPを展開**：
   - ダウンロードしたzipファイルを`C:\php`に展開します。

3. **PHPをシステムのPATHに追加**：
   - **システムのプロパティ** > **環境変数**に移動します。
   - **システム変数**の下で**Path**を見つけ、**編集**をクリックします。
   - パス`C:\php`（またはPHPを展開した場所）を追加します。
   - **OK**をクリックしてすべてのウィンドウを閉じます。

4. **PHPを設定**：
   - `php.ini-development`を`php.ini`にコピーします。
   - `php.ini`を編集して、必要に応じてPHPを設定します（例：`extension_dir`の設定、拡張機能の有効化）。

5. **PHPのインストールを確認**：
   - コマンドプロンプトを開いて実行：
     ```cmd
     php -v
     ```

#### **複数バージョンのPHPをインストール**

1. 各バージョンについて**上記の手順を繰り返し**、それぞれを別のディレクトリ（例：`C:\php7`、`C:\php8`）に配置します。

2. システムのPATH変数を目的のバージョンのディレクトリに調整して、**バージョンを切り替えます**。

### **Ubuntu（20.04、22.04など）**

#### **aptを使用したPHPのインストール**

1. **パッケージリストを更新**：
   - ターミナルを開いて実行：
     ```bash
     sudo apt update
     ```

2. **PHPをインストール**：
   - 最新のPHPバージョンをインストール：
     ```bash
     sudo apt install php
     ```
   - 特定のバージョン（例：PHP 8.1）をインストールする場合：
     ```bash
     sudo apt install php8.1
     ```

3. **追加モジュールをインストール**（オプション）：
   - 例えば、MySQLサポートをインストールする場合：
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **PHPバージョンの切り替え**：
   - `update-alternatives`を使用：
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **インストールされたバージョンを確認**：
   - 実行：
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **yum/dnfを使用したPHPのインストール**

1. **EPELリポジトリを有効化**：
   - ターミナルを開いて実行：
     ```bash
     sudo dnf install epel-release
     ```

2. **Remiリポジトリをインストール**：
   - 実行：
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **PHPをインストール**：
   - デフォルトバージョンをインストール：
     ```bash
     sudo dnf install php
     ```
   - 特定のバージョン（例：PHP 7.4）をインストールする場合：
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **PHPバージョンの切り替え**：
   - `dnf`モジュールコマンドを使用：
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **インストールされたバージョンを確認**：
   - 実行：
     ```bash
     php -v
     ```

### **一般的な注意事項**

- 開発環境では、プロジェクトの要件に応じてPHP設定を構成することが重要です。
- PHPバージョンを切り替える際は、使用予定の特定バージョンに対応するすべての関連PHP拡張機能がインストールされていることを確認してください。
- PHPバージョンの切り替えや設定の更新後は、Webサーバー（Apache、Nginxなど）を再起動して変更を適用してください。