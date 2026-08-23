# FlightMail

> **サードパーティプラグイン** - [Ryan Stubbs](https://ryanstubbs.co.uk) がメンテナンスしています（[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail)、MIT ライセンス）。Flight コアの一部ではありません。問題は [GitHub リポジトリ](https://github.com/ryanstubbs/flightmail/issues) で報告してください。

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) を使えば、頭痛の種なしに Flight アプリからメールを送れます。PHP で最も実績のあるメールライブラリ **Symfony Mailer** をラップし、Flight の一部のように感じられるようにします。インストールは 1 行、送信は流暢なチェーン 1 本です：

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('できました！')
    ->text('最初のメールがまもなく届きます。')
    ->send();
```

## 機能

- **どのプロバイダーでも、それぞれ 1 行。** SMTP、Postmark、Sendgrid、Mailgun、Amazon SES、Brevo とその仲間は、シンプルな DSN 文字列ですべて動作します。
- **複数のプロバイダーを同時に使えます。** トランザクションメールは Postmark、ニュースレターは自前の SMTP——メッセージごとに選べます。
- **テンプレートは使いたいときだけ。** Twig または Latte で本文をレンダリングします。テンプレートが不要なら、文字列を渡すだけで追加インストールは不要です。
- **送信時の仕上げ。** 任意の CSS インライン化と、HTML から自動生成されるプレーンテキストパート。使うときだけインストールするライブラリで動きます。
- **最良の意味で地味です。** 遅延接続、黙ってメールを捨てるのではなく明確なエラー、カスタムが必要ならすべて差し替え可能です。

## 要件

| 項目           | バージョン                             |
| -------------- | -------------------------------------- |
| PHP            | 8.2 以降                               |
| Flight PHP     | core ^3.15                             |
| Symfony Mailer | ^7.2 または ^8.0（自動インストール）   |

## インストール

```bash
composer require ryanstubbs/flightmail
```

プレーンテキストと HTML メールの送信はこれだけです。テンプレートレンダリングはオプトインです。使う場合だけエンジンを追加してください：

```bash
composer require twig/twig      # .twig テンプレート用
composer require latte/latte    # .latte テンプレート用
```

さらに 2 つの任意ライブラリが、[下記](#styling-html-and-generating-text-parts)で扱う送信時の拡張機能を支えます：

```bash
composer require pelago/emogrifier         # CSS インライン化用（"inline_css"）
composer require league/html-to-markdown   # Markdown テキストパート用（"text_from_html"）
```

これらは並べてインストールできます。FlightMail は設定内容に基づいて適切なものを選びます。

## 最初のメール

これをブートストラップ（ルートを定義するのと同じ場所）に追加します：

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// FlightMail に、どこから・どの経路でメールを送るかを伝えます。
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('ようこそ！')
        ->html('<h1>ようこそ！</h1><p>ご参加いただき嬉しいです。</p>')
        ->send();
});

Flight::start();
```

[Flight PHP skeleton](https://github.com/flightphp/skeleton) を使っていますか？ 代わりにインスタンス方式で `app/config/services.php` に登録します：

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

どちらのスタイルも同じメーラーを公開します。`Flight::mail()` と `$app->mail()` は入れ替えて使えます。

> **ローカルでテストしますか？** プロジェクトが [DDEV](https://ddev.com) で動いているなら、DSN を `smtp://127.0.0.1:1025` に向け、キャプチャされたすべてのメールを `http://<project>.ddev.site:8025` の Mailpit で読んでください。何もマシンから外に出ません。

## メールの送信

### プレーンな文字列（テンプレートエンジン不要）

`->text()` と `->html()` は生の文字列を受け取り、他に何もインストールする必要はありません：

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('バックアップ完了')
    ->text('夜間バックアップが 42 分で完了しました。')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('請求書 #123')
    ->html('<h1>請求書 #123</h1><p>合計請求額: $42.00</p>')
    ->send();
```

### Twig テンプレート

```php
// welcome.html.twig の内容: Hello {{ name }}、サインアップありがとうございます！
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('ようこそ！')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Latte テンプレート

同じ考え方で、拡張子は `.latte` です：

```php
// welcome.latte の内容: Hello {$name}、サインアップありがとうございます！
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('ようこそ！')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + プレーンテキストを一緒に

到達性のベストプラクティス——メールクライアントに両方のバージョンを渡します：

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('ようこそ！')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // リッチ版
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // フォールバック版
    ->send();
```

テンプレートについて知っておくとよいことがいくつかあります：

- **遅延**レンダリングです。送信時に実行されます。今 compose し、あとでレンダリングします。
- エンジンは拡張子で選ばれます。`.twig` → Twig、`.latte` → Latte、それ以外 → 設定したデフォルト（`renderer` オプション）。
- 明示的な `->html()` または `->text()` の本文は常にテンプレートより優先されるので、デフォルトテンプレートを設定してメッセージごとに上書きできます。

## HTML のスタイル設定とテキストパートの生成

2 つの任意の送信時拡張機能があります。どちらもデフォルトではオフで、使いたいときだけインストールするライブラリで動きます：

| 機能                          | インストール              | 設定キー         |
| ----------------------------- | ------------------------- | ---------------- |
| CSS インライン化              | `pelago/emogrifier`       | `inline_css`     |
| HTML からテキストパートを生成 | `league/html-to-markdown` | `text_from_html` |

### HTML メールに CSS をインライン化する

Gmail とほとんどのウェブメールクライアントは `<style>` ブロックを取り除きます。確実に尊重されるスタイルはインラインの `style=""` 属性だけです。手書きは悲惨なので、送信時に [Emogrifier](https://github.com/MyIntervals/emogrifier) に任せましょう：

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

これをオンにすると、すべての HTML 本文は送信直前に CSS がインライン化されます。テンプレート由来でも `->html()` 由来でも同じです。`<style>p { color: red; }</style><p>こんにちは</p>` のようなメッセージは `<p style="color: red;">こんにちは</p>` として送出されます。

ブランドカラーやリセットなど、共有スタイルを各テンプレートで繰り返さずにすべてのメールへ注入するには、ルールを直接渡すか、スタイルシートファイルを指します：

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// または
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

メッセージ単位の制御：

```php
$message->inlineCss();          // この 1 通だけインライン化を強制
$message->withoutInlineCss();   // グローバルで有効でもスキップ
```

### HTML からテキストパートを生成する

ベストプラクティスは HTML とプレーンテキストを一緒に送ることですが、両方を書くのは面倒です。FlightMail は最終 HTML からテキストパートを自動派生できます。基本的な変換に追加依存は不要です。コンバーターは Symfony Mime に同梱されています：

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // 可能なら Markdown、そうでなければプレーン
]);
```

モード：

- `true` または `'auto'` - `league/html-to-markdown` がインストールされていれば Markdown 出力、そうでなければ単純なタグ除去。
- `'markdown'` - Markdown を強制（`composer require league/html-to-markdown`。見出しは `==`、リンクは `[text](url)`、太字は `**bold**`）。
- `'plain'` - 常にタグを除去します。追加パッケージは不要です。

生成はレンダリングと CSS インライン化の後に走り、メッセージに HTML 本文があってテキスト本文がないときだけです。明示的な `->text()` または `->textTemplate()` は常に優先されます。メッセージ単位の上書きはインライン化と同じです：

```php
$message->textFromHtml('plain');    // この 1 通だけタグ除去を強制
$message->withoutTextFromHtml();    // HTML のみのメール
```

ライブラリがインストールされていないモードを有効にすると、実行すべき正確な `composer require` を示す明確なエラーが出ます。静かに劣化することはありません。

## プロバイダーの選択

プロバイダーは DSN 文字列で接続します。ブリッジパッケージをインストールし、DSN を `dsns` に貼り付ければ完了です。

| プロバイダー             | インストール                                 | DSN の例                                     |
| ------------------------ | -------------------------------------------- | -------------------------------------------- |
| SMTP                     | 組み込み                                     | `smtp://user:pass@host:587`                  |
| Sendmail                 | 組み込み                                     | `sendmail://default`                         |
| Dev/null（メールを破棄） | 組み込み                                     | `null://null`                                |
| Postmark                 | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid                 | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun                  | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES               | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                    | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend               | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

完全な一覧は [Symfony Mailer のドキュメント](https://symfony.com/doc/current/mailer.html) にあります。そこに記載されているものは、ここでもそのまま動作します。

### 複数のプロバイダーを同時に使う

各トランスポートに名前を付け、メッセージごとに選びます：

```php
MailPlugin::install([
    'dsns' => [
        'transactional' => 'postmark+api://KEY@api.postmarkapp.com',
        'bulk'          => 'smtp://user:pass@bulk.example.com:587',
    ],
    'from' => 'no-reply@example.com',
]);
```

```php
// ->transport() を呼ばない = "dsns" の最初のキー（ここでは "transactional"）。
Flight::mail()->compose()->to('...')->text('領収書')->send();

// 別のルートを明示的に選びます。
Flight::mail()->compose()->to('...')->text('ニュースレター')->transport('bulk')->send();
```

## 設定リファレンス

`dsns` 以外はすべて任意です。

```php
MailPlugin::install([
    // 必須 - トランスポート名 => Symfony DSN。
    // メッセージが指定しないときは最初のエントリが使われます。
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // メッセージに明示的な ->transport() がなく、最初のキーを使いたくないときのトランスポート。
    // "dsns" に存在している必要があります。
    'default_transport' => 'default',

    // グローバル送信者。文字列、Symfony Address、または ['email' => 'Name']。
    // メッセージが独自の ->from() を設定していないときだけ適用されます。
    'from' => ['no-reply@example.com' => 'マイアプリ'],

    // デフォルトのテンプレートエンジン: 'twig'、'latte'、またはカスタム名。
    // 拡張子が登録済みレンダラーでないテンプレートのときだけ参照されます。
    'renderer' => 'twig',

    // テンプレートの場所。順番に検索されます。任意のキャッシュディレクトリも指定できます。
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Twig\Environment にそのまま渡す追加オプション。
    'twig' => ['options' => ['strict_variables' => true]],

    // 起動時に Latte エンジンを調整: fn(Latte\Engine $engine): void。
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // 送信時の本文拡張（「HTML のスタイル設定とテキストパートの生成」を参照）。
    'inline_css' => true,           // または ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // または 'plain' / 'markdown'

    // カスタム DSN スキーム、カスタムレンダラー、送信前フック（下記を参照）。
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // すべてのトランスポートに渡される任意の配管。
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## さらに進む

以下はすべて任意です。デフォルトでほとんどのアプリをカバーできます。

### カスタム DSN スキームを追加する

Symfony の `TransportFactoryInterface` を実装して登録します。そうすれば独自のスキームも組み込みのものと同じように動きます：

```php
use ryanstubbs\FlightMail\MailPlugin;
use Symfony\Component\Mailer\Transport\Dsn;
use Symfony\Component\Mailer\Transport\TransportFactoryInterface;
use Symfony\Component\Mailer\Transport\TransportInterface;

class MyCarrierFactory implements TransportFactoryInterface
{
    public function supports(Dsn $dsn): bool
    {
        return $dsn->getScheme() === 'mycarrier';
    }

    public function create(Dsn $dsn): TransportInterface
    {
        // ... キャリアと通信するトランスポートを構築します
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### カスタムテンプレートレンダラーを追加する

テンプレート名とパラメータを文字列に変換できるものなら何でも対象です：

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// .markdown で終わるテンプレートは自動的にこれを使います：
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### 送信直前に何かを実行する

フックは完成したメッセージを受け取ります。レンダリング後、デフォルト適用後、回線に乗る直前です：

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### イベントとログ

Symfony のイベントディスパッチャや PSR-3 ロガーを渡すと、すべてのトランスポートがそれらを使います：

```php
$plugin->eventDispatcher($dispatcher); // 各送信前に MessageEvent を受信
$plugin->logger($logger);              // トランスポートレベルのログ
```

## API チートシート

```php
// セットアップ
MailPlugin::install($config)             // グローバルな Flight アプリに登録
MailPlugin::register($app, $config)      // 特定の Engine に登録
$mailer = Flight::mail();                // 共有の Mailer インスタンス

// メッセージの構築
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // 標準の Symfony Mime メソッド
$message->text(string)                       // プレーン文字列の本文
$message->html(string)                       // HTML 文字列の本文
$message->template($name, $params)           // テンプレートからの HTML 本文
$message->htmlTemplate($name, $params)       // template() のエイリアス
$message->textTemplate($name, $params)       // テンプレートからのテキスト本文
$message->inlineCss() / ->withoutInlineCss() // メッセージ単位の CSS インライン化
$message->textFromHtml($mode)                // 自動テキストパート: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // HTML のみのメール
$message->transport($name)                   // 名前付き DSN 経由でルーティング
$message->send(): ?SentMessage               // レンダリング + 送信

// メーラー自身のメソッド
$mailer->send($message): ?SentMessage        // $message->send() の明示的な代替
$mailer->render($template, $params): string  // 送信せずにレンダリング
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

`Message` は `Symfony\Component\Mime\Email` を拡張しているので、すでに知っている Symfony のメソッド——`attach()`、`embed()`、`priority()`、`replyTo()`——はそのまま使えます。

## トラブルシューティング

**"No mail DSNs configured"**
プラグインを登録する前に `Flight::mail()` を呼んだか、設定配列に `dsns` が含まれていません。このエラーは意図的です。FlightMail はメールの行き先を推測して黙って捨てるのではなく、拒否します。

**"Unknown mail template renderer ..."**
エンジンがインストールされていないテンプレートを使いました。`composer require twig/twig` または `composer require latte/latte` で直すか、拡張子に合わせたカスタムレンダラーを登録してください。

**"Unknown mail transport ..."**
`->transport('name')`（または `default_transport`）が `dsns` のどのキーとも一致しません。スペルを確認してください。エラーには設定済みの名前が一覧されます。

**メールが届かない**
`dsns` を `null://null` に向けて残りのコードが動くことを確認してから、本番の DSN に戻してください。DDEV では `smtp://127.0.0.1:1025` を使い、ポート 8025 の Mailpit でメッセージを確認します。

---

バグ報告、プルリクエスト、完全なソースは [GitHub リポジトリ](https://github.com/ryanstubbs/flightmail) をご覧ください。
