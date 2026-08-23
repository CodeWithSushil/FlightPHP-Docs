# 素晴らしいプラグイン

Flightは非常に拡張可能です。Flightアプリケーションに機能を追加するために使用できるプラグインがいくつかあります。一部はFlightチームによって公式にサポートされており、他のものは開始に役立つマイクロ/ライトライブラリです。

## AIツール

FlightはAIを活用したプラグインでさらにクールにすることができます。

- [Flight MCP](/awesome-plugins/mcp) - FlightとMCP (Model Control Protocol) を統合するためのプラグインで、シームレスなAIを活用した機能を実現します。主にドキュメントページに焦点を当てており、Flightプロジェクトに関する最新情報を提供することでトークンコストを抑えるのに役立ちます。

## APIドキュメント

APIドキュメントはどのAPIにとっても重要です。開発者がAPIとの対話方法や期待される結果を理解するのに役立ちます。FlightプロジェクトのAPIドキュメントを生成するためのツールがいくつか利用可能です。

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Daniel Schreiber氏による、FlightPHPでOpenAPI Specを使用したAPIファーストアプローチによるAPI構築方法についてのブログ記事です。
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UIはFlightプロジェクトのAPIドキュメントを生成するのに役立つ素晴らしいツールです。非常に使いやすく、ニーズに合わせてカスタマイズできます。これはSwaggerドキュメントを生成するためのPHPライブラリです。

## アプリケーションパフォーマンス監視 (APM)

アプリケーションパフォーマンス監視 (APM) はどのアプリケーションにとっても重要です。アプリケーションのパフォーマンスやボトルネックを理解するのに役立ちます。Flightで使用できるAPMツールがいくつかあります。
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APMはFlightアプリケーションを監視するために使用できるシンプルなAPMライブラリです。アプリケーションのパフォーマンスを監視し、ボトルネックを特定するのに役立ちます。

## 非同期処理

Flightはすでに高速なフレームワークですが、ターボエンジンを搭載することで、さらに楽しく（そして挑戦的になります）！

- [flightphp/async](/awesome-plugins/async) - 公式Flight Asyncライブラリ。このライブラリはアプリケーションに非同期処理を追加するシンプルな方法です。Swoole/Openswooleを使用して、シンプルで効果的な非同期タスク実行方法を提供します。

## 認可/権限

認可と権限は、誰が何にアクセスできるかを制御する必要があるアプリケーションにとって重要です。

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - 公式Flight Permissionsライブラリ。このライブラリは、ユーザーおよびアプリケーションレベルの権限をアプリケーションに追加するシンプルな方法です。

## 認証

認証は、ユーザーIDの検証やAPIエンドポイントのセキュリティ保護が必要なアプリケーションに不可欠です。

- [firebase/php-jwt](/awesome-plugins/jwt) - PHP用のJSON Web Token (JWT) ライブラリ。Flightアプリケーションでトークンベースの認証を実装するためのシンプルで安全な方法です。ステートレスAPI認証、ミドルウェアによるルートの保護、OAuthスタイルの認可フローの実装に最適です。

## キャッシュ

キャッシュはアプリケーションを高速化する素晴らしい方法です。Flightで使用できるキャッシュライブラリがいくつかあります。

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - 軽量でシンプル、スタンドアロンのPHPファイル内キャッシュクラス

## CLI

CLIアプリケーションはアプリケーションと対話する素晴らしい方法です。コントローラーの生成、すべてのルートの表示などに使用できます。

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - RunwayはFlightアプリケーションの管理を支援するCLIアプリケーションです。

## クッキー

クッキーはクライアント側に少量のデータを保存する素晴らしい方法です。ユーザー設定、アプリケーション設定などの保存に使用できます。

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookieはクッキーの管理を提供するシンプルで効果的なPHPライブラリです。

## デバッグ

ローカル環境での開発時には、デバッグが重要です。デバッグ体験を向上させるプラグインがいくつかあります。

- [tracy/tracy](/awesome-plugins/tracy) - Flightで使用できるフル機能のエラーハンドラです。アプリケーションのデバッグに役立つパネルが多数用意されており、非常に拡張しやすく、独自のパネルを追加することもできます。
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - [Tracy](/awesome-plugins/tracy) エラーハンドラと組み合わせて使用し、Flightプロジェクト特有のデバッグを支援する追加パネルを追加するプラグインです。

## データベース

データベースはほとんどのアプリケーションの中核です。データの保存と取得に使用されます。一部のデータベースライブラリはクエリを記述するための単なるラッパーであり、一部は本格的なORMです。

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - コアの一部である公式Flight PDOヘルパーです。`insert()`、`update()`、`delete()`、`transaction()`などの便利なヘルパーメソッドを備えたモダンなラッパーで、データベース操作を簡素化します。すべての結果はCollectionsとして返され、柔軟な配列/オブジェクトアクセスが可能です。ORMではなく、PDOで作業するためのより良い方法です。
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - コアの一部である公式Flight PDOラッパー（v3.18.0で非推奨）。代わりにSimplePdoを使用してください。
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - 公式Flight ActiveRecord ORM/Mapper。データベース内のデータの取得と保存を簡単に行うための優れた小さなライブラリです。
- [byjg/php-migration](/awesome-plugins/migrations) - プロジェクトのすべてのデータベース変更を追跡するためのプラグインです。
- [knifelemon/easy-query](/awesome-plugins/easy-query) - 準備済みステートメント用のSQLとパラメータを生成する軽量で流暢なSQLクエリビルダー。[SimplePdo](/learn/simple-pdo) と連携して動作します。

## 暗号化

暗号化は機密データを保存するアプリケーションにとって重要です。データの暗号化と復号化はそれほど難しくありませんが、暗号化キーの適切な保存は[難しい場合があります](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.)。[難しい場合があります](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/)。[難しい場合があります](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key)。最も重要なことは、暗号化キーを公開ディレクトリに保存したり、コードリポジトリにコミットしたりしないことです。

- [defuse/php-encryption](/awesome-plugins/php-encryption) - データの暗号化と復号化に使用できるライブラリです。データの暗号化と復号化を開始するのはかなり簡単です。

## メール

メール送信はほとんどの Web アプリケーションの中核的なニーズです。ウェルカムメッセージ、パスワードリセット、通知など。これらのライブラリは、配信品質をしっかり保ちながら、手間をかけずに送信できるようにします。

- [ryanstubbs/flightmail](/awesome-plugins/flightmail) - FlightMail は Symfony Mailer を、流暢で Flight に馴染む API でラップします。シンプルな DSN 文字列で SMTP や主要なプロバイダー経由で送信し、メッセージごとに異なるプロバイダーへルーティングし、Twig や Latte テンプレートで本文をレンダリングできます。これは Flight の非公式プラグインであり、Flight チームはメンテナンスしていません。

## ジョブキュー

ジョブキューはタスクを非同期で処理するのに非常に便利です。メールの送信、画像の処理、またはリアルタイムで行う必要のないタスクなどに使用できます。

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queueは非同期でジョブを処理するために使用できるライブラリです。beanstalkd、MySQL/MariaDB、SQLite、PostgreSQLで使用できます。

## セッション

セッションはAPIにはあまり役に立ちませんが、Webアプリケーションを構築する場合は、状態とログイン情報を維持するためにセッションが重要になります。

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - 公式Flight Sessionライブラリ。これはセッションデータの保存と取得に使用できるシンプルなセッションライブラリです。PHPの組み込みセッション処理を使用します。
- [Ghostff/Session](/awesome-plugins/ghost-session) - PHPセッションマネージャー（ノンブロッキング、フラッシュ、セグメント、セッション暗号化）。セッションデータのオプションの暗号化/復号化にPHP open_sslを使用します。

## テンプレート

テンプレートはUIを持つWebアプリケーションの中核です。Flightで使用できるテンプレートエンジンがいくつかあります。

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - コアの一部である非常に基本的なテンプレートエンジンです。プロジェクトに数ページ以上のページがある場合は使用しないことをお勧めします。
- [latte/latte](/awesome-plugins/latte) - Latteは非常に使いやすく、TwigやSmartyよりもPHPの構文に近いと感じるフル機能のテンプレートエンジンです。非常に拡張しやすく、独自のフィルタや関数を追加することもできます。
- [twig/twig](/awesome-plugins/twig) - Twigは柔軟で高速、セキュアなテンプレートエンジンです（Symfonyで使用されているものと同じ）。AIツールや多くのPHP開発者がよく知っており、デフォルトで出力を自動エスケープし、拡張機能の巨大なエコシステムがあります。
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplateは、アセットコンパイル、テンプレート継承、変数処理を備えた強力なPHPテンプレートエンジンです。自動CSS/JS縮小化、キャッシュ、Base64エンコード、オプションのFlight PHPフレームワーク統合機能を備えています。

## WordPress統合

WordPressプロジェクトでFlightを使用したいですか？それのための便利なプラグインがあります！

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - このWordPressプラグインは、FlightをWordPressと並行して実行できるようにします。Flightフレームワークを使用してカスタムAPI、マイクロサービス、または完全なアプリをWordPressサイトに追加するのに最適です。両方の世界の良いところを使いたい場合に非常に便利です！

## 貢献

共有したいプラグインがありますか？プルリクエストを送信してリストに追加してください！