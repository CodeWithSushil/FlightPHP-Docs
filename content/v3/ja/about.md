# Flight PHP Framework

Flightは、PHP向けの高速でシンプル、かつ拡張可能なフレームワークです。迅速に作業を完了したい開発者向けに構築されています。クラシックなWebアプリ、高速なAPI、またはAIコーディングアシスタントとの組み合わせなど、Flightの軽量なフットプリントとシンプルな設計は完璧に適合します。Flightは軽量であることを意図していますが、エンタープライズアーキテクチャ要件にも対応できます。

## なぜFlightを選ぶのか？

- **初心者フレンドリー:** Flightは、新しいPHP開発者にとって素晴らしいスタートポイントです。その明確な構造とシンプルな構文により、ボイラープレートに迷うことなくWeb開発を学ぶことができます。
- **プロフェッショナルに愛されている:** 経験豊富な開発者は、その柔軟性と制御性を理由にFlightを愛用しています。フレームワークを切り替えることなく、小さなプロトタイプから本格的なアプリまでスケールアップできます。
- **後方互換性:** 私たちはあなたの時間を大切にしています。Flight v3はv2の拡張であり、ほぼすべての同じAPIを保持しています。私たちは革命ではなく進化を信じています。メジャーバージョンが出るたびに「世界を破壊する」ことはありません。
- **依存関係ゼロ:** Flightのコアは完全に依存関係がありません。ポリフィルも外部パッケージも、PSRインターフェースすらありません。これは、より少ない攻撃対象領域、より小さなフットプリント、上流依存関係からの予期しない破壊的変更がないことを意味します。オプションのプラグインには依存関係が含まれる場合がありますが、コアは常に軽量で安全に保たれます。
- **AIフレンドリー:** Flightの小さなAPIサーフェスと[公式スケルトン](https://github.com/flightphp/skeleton)（1つのレイアウト、`AGENTS.md`、コンストラクタインジェクション）により、AIコーディングツールがパターンに従いやすくなっています。すべての行を入力する場合でもエージェントとペアリングする場合でも、同じコードベースを使用します。[FlightでのAIの使用について詳しく学ぶ](/learn/ai)。

## ビデオ概要

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">シンプルですよね？</span>
      <br>
      <a href="https://docs.flightphp.com/learn">詳細</a>については、ドキュメントでFlightについて学んでください！
    </div>
  </div>
</div>

## クイックスタート

高速なベアボーンインストールを行うには、Composerでインストールします：

```bash
composer require flightphp/core
```

または、リポジトリのzipを[こちら](https://github.com/flightphp/core)からダウンロードすることもできます。次に、以下のような基本的な`index.php`ファイルを作成します：

```php
<?php

// composerでインストールした場合
require 'vendor/autoload.php';
// またはzipファイルで手動インストールした場合
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

これだけです！基本的なFlightアプリケーションが完成しました。これで、`php -S localhost:8000`でこのファイルを実行し、ブラウザで`http://localhost:8000`にアクセスして出力を確認できます。

このような短い`Flight::`の例は、学習やマイクロアプリに最適です。人間とAIツールが共有する完全なプロジェクトレイアウトについては、以下のスケルトンを使用してください。

## スケルトン/ボイラープレートアプリ

新しいFlightプロジェクトを開始するための公式スターターがあります。構造、設定、Composerスクリプト、AIフレンドリーな指示を最初から設定します。

すぐに使えるプロジェクトについては[flightphp/skeleton](https://github.com/flightphp/skeleton)を確認するか、インスピレーションを得るために[examples](examples)ページを訪問してください。AIワークフローの詳細が必要ですか？[AIと開発者体験を探る](/learn/ai)。

（高レベルで）得られるもの：

- **PascalCaseフォルダー**を持つ`App\`名前空間（`app/Controller/`、`app/Middleware/`、`app/Model/`、…）—フォルダーの**大文字小文字**は名前空間と一致する必要があります（[Autoloading](/learn/autoloading)を参照）
- コントローラーをテスト可能に保つための**Dice + `Engine`インジェクション**（アプリコードでは`Flight::`よりも`$this->app`を優先）
- **Twig**ビュー、**SimplePdo** + ActiveRecordサンプル、Runway **migrate**
- アシスタントとセキュリティポリシーのためのルート**`AGENTS.md`**（およびスコープ付きコピー）と**`SECURITY.md`**

## スケルトンアプリのインストール

簡単です！

```bash
# 新しいプロジェクトを作成
composer create-project flightphp/skeleton my-project/
# 新しいプロジェクトディレクトリに入る
cd my-project/
# すぐに開始するためにローカル開発サーバーを起動！
composer start
```

プロジェクト構造を作成し、`config_sample.php` → `config.php`（および存在する場合は`.env.example` → `.env`）をコピーします。これで準備完了です。オプションのサンプルデータ：

```bash
php runway migrate
# その後 /posts と /api/posts にアクセス
```

## 高パフォーマンス

Flightは、現在存在する最も高速なPHPフレームワークの1つです。その軽量なコアは、オーバーヘッドを減らし、速度を向上させます。伝統的なアプリと現代のAI支援ワークフローの両方に最適です。[TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)で、すべてのベンチマークを確認できます。

他の人気のPHPフレームワークとのベンチマークを以下に示します。

| Framework | Plaintext Reqs/sec | JSON Reqs/sec |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## FlightとAI

コーディングLLMとのFlightの組み合わせに興味がありますか？`AGENTS.md`、Runway `ai:*`コマンド、スケルトンレイアウトがアシスタントを正しい軌道に保つ方法を[発見](/learn/ai)してください。

## 安定性と後方互換性

私たちはあなたの時間を大切にしています。私たちは皆、数年ごとに完全に自己再発明し、開発者に破損したコードと高価な移行を残すフレームワークを見てきました。Flightは違います。Flight v3はv2の拡張として設計されており、皆さんが知り愛用しているAPIが剥奪されていません。実際、ほとんどのv2プロジェクトはv3で変更なしに動作します。

Flightの安定性を保ち、フレームワークの修正ではなく、アプリの構築に集中できるようにすることにコミットしています。スケルトンは*新しい*プロジェクトに対しては意見を持っている場合がありますが、コアAPIは他のすべてのユーザーにとって馴染み深いままです。

# コミュニティ

Matrix Chatで活動しています

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

そしてDiscordでも

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# 貢献

Flightに貢献する方法は2つあります：

1. [コアリポジトリ](https://github.com/flightphp/core)を訪問して、コアフレームワークに貢献する。
2. ドキュメントの改善を手伝う！このドキュメントウェブサイトは[Github](https://github.com/flightphp/docs)でホストされています。エラーを見つけた場合や改善したいことがある場合は、プルリクエストを送信してください。更新や新しいアイデアを歓迎します。特にAIと新技術に関するものを歓迎します！

# 要件

FlightにはPHP 7.4以上が必要です。

**注意:** PHP 7.4がサポートされている理由は、執筆時点（2024年）で、PHP 7.4が一部のLTS Linuxディストリビューションのデフォルトバージョンであるためです。PHP >8への移行を強制すると、それらのユーザーに多くの問題を引き起こすことになります。フレームワークはPHP >8もサポートしています。

# ライセンス

Flightは[MIT](https://github.com/flightphp/core/blob/master/LICENSE)ライセンスの下でリリースされています。