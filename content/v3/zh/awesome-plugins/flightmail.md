# FlightMail

> **第三方插件** - 由 [Ryan Stubbs](https://ryanstubbs.co.uk) 维护（[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail)，MIT 许可）。不属于 Flight 核心——请在 [其 GitHub 仓库](https://github.com/ryanstubbs/flightmail/issues) 上报告问题。

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) 让您能从 Flight 应用发送邮件，告别那些头疼事。它封装了 **Symfony Mailer**——PHP 中久经考验的邮件库——用起来就像 Flight 的一部分。一行安装，一条流畅的链式调用就能发送：

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('搞定了！')
    ->text('您的第一封邮件正在路上。')
    ->send();
```

## 特性

- **任意服务商，各一行搞定。** SMTP、Postmark、Sendgrid、Mailgun、Amazon SES、Brevo 以及同类服务都可以通过简单的 DSN 字符串工作。
- **同时使用多个服务商。** 事务邮件走 Postmark，新闻通讯走您自己的 SMTP——按消息选择。
- **想用模板就用。** 用 Twig 或 Latte 渲染正文。不想用模板？直接传字符串，不用额外安装任何东西。
- **发送时的打磨。** 可选的 CSS 内联，以及从 HTML 自动派生纯文本部分，由您只有用到时才安装的库提供支持。
- **朴实可靠，恰到好处。** 惰性连接、清晰的错误而不是默默吞掉邮件，需要自定义时一切都可替换。

## 要求

| 项目           | 版本                                   |
| -------------- | -------------------------------------- |
| PHP            | 8.2 或更高                             |
| Flight PHP     | core ^3.15                             |
| Symfony Mailer | ^7.2 或 ^8.0（自动安装）               |

## 安装

```bash
composer require ryanstubbs/flightmail
```

发送纯文本和 HTML 邮件只需这些。模板渲染是可选的——只有在会用到时才添加引擎：

```bash
composer require twig/twig      # 用于 .twig 模板
composer require latte/latte    # 用于 .latte 模板
```

另外两个可选库为[下文](#styling-html-and-generating-text-parts)介绍的发送时增强功能提供支持：

```bash
composer require pelago/emogrifier         # 用于 CSS 内联（"inline_css"）
composer require league/html-to-markdown   # 用于 Markdown 文本部分（"text_from_html"）
```

这些都可以并排安装；FlightMail 会根据您的配置选择合适的那个。

## 您的第一封邮件

把它加到引导文件里（定义路由的同一个地方）：

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// 告诉 FlightMail 邮件从哪里发、经过哪里发。
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('欢迎加入！')
        ->html('<h1>欢迎！</h1><p>很高兴您来到这里。</p>')
        ->send();
});

Flight::start();
```

在用 [Flight PHP skeleton](https://github.com/flightphp/skeleton)？改用实例风格，在 `app/config/services.php` 中注册：

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

两种风格暴露的是同一个 Mailer：`Flight::mail()` 和 `$app->mail()` 可以互换。

> **在本地测试？** 如果项目运行在 [DDEV](https://ddev.com) 中，把 DSN 指向 `smtp://127.0.0.1:1025`，然后在 `http://<project>.ddev.site:8025` 的 Mailpit 里阅读每一封被捕获的邮件。什么都不会离开您的机器。

## 发送邮件

### 纯字符串（无需模板引擎）

`->text()` 和 `->html()` 接受原始字符串，不需要安装其他任何东西：

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('备份完成')
    ->text('夜间备份已在 42 分钟内完成。')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('发票 #123')
    ->html('<h1>发票 #123</h1><p>应付总额：$42.00</p>')
    ->send();
```

### Twig 模板

```php
// welcome.html.twig 包含：Hello {{ name }}，感谢注册！
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('欢迎！')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Latte 模板

同样的思路，`.latte` 扩展名：

```php
// welcome.latte 包含：Hello {$name}，感谢注册！
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('欢迎！')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + 纯文本一起发送

送达率的最佳实践——给邮件客户端同时提供两个版本：

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('欢迎！')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // 富文本版本
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // 回退版本
    ->send();
```

关于模板，有几点值得了解：

- 它们**惰性**渲染，在发送时才执行——现在组合，稍后渲染。
- 引擎按扩展名选择：`.twig` → Twig，`.latte` → Latte，其他任何扩展名 → 您配置的默认值（`renderer` 选项）。
- 显式的 `->html()` 或 `->text()` 正文始终优先于模板，因此您可以设置默认模板，再按消息覆盖。

## 为 HTML 设置样式并生成文本部分

两项可选的发送时增强功能，默认都关闭，并且都由您只有想用时才安装的库提供支持：

| 功能                 | 安装                      | 配置键           |
| -------------------- | ------------------------- | ---------------- |
| CSS 内联             | `pelago/emogrifier`       | `inline_css`     |
| 从 HTML 生成文本部分 | `league/html-to-markdown` | `text_from_html` |

### 将 CSS 内联到 HTML 邮件中

Gmail 和大多数 webmail 客户端会剥掉 `<style>` 块——内联的 `style=""` 属性是它们能可靠遵守的唯一样式方式。手写那些属性痛苦得很；让 [Emogrifier](https://github.com/MyIntervals/emogrifier) 在发送时来做：

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

开启后，每一封 HTML 正文都会在即将发送前把 CSS 内联进去——无论它来自模板还是 `->html()`。像 `<style>p { color: red; }</style><p>你好</p>` 这样的消息会发出去变成 `<p style="color: red;">你好</p>`。

要向每封邮件注入共享样式（品牌色、重置）而不在每个模板里重复，可以直接传入规则，或指向一个样式表文件：

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// 或者
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

按消息控制：

```php
$message->inlineCss();          // 强制为这一封消息内联
$message->withoutInlineCss();   // 即使全局已启用也跳过
```

### 从 HTML 生成文本部分

最佳实践是同时发送 HTML 和纯文本版本，但两个都写很繁琐。FlightMail 可以从最终的 HTML 自动派生文本部分——基础转换不需要额外依赖，因为转换器随 Symfony Mime 一起提供：

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // 尽可能用 Markdown，否则用纯文本
]);
```

模式：

- `true` 或 `'auto'` - 如果安装了 `league/html-to-markdown` 则输出 Markdown，否则做简单的标签剥离。
- `'markdown'` - 强制 Markdown（`composer require league/html-to-markdown`；标题变成 `==`，链接 `[text](url)`，粗体 `**bold**`）。
- `'plain'` - 始终剥离标签；不需要任何额外包。

生成发生在渲染和 CSS 内联之后，并且仅当消息有 HTML 正文但没有文本正文时——显式的 `->text()` 或 `->textTemplate()` 始终优先。按消息覆盖与内联类似：

```php
$message->textFromHtml('plain');    // 强制为这一封剥离标签
$message->withoutTextFromHtml();    // 仅 HTML 的邮件
```

启用一个其库尚未安装的模式时，您会得到一条清晰的错误，点明要运行的精确 `composer require`——绝不会静默降级。

## 选择服务商

服务商通过 DSN 字符串接入。安装桥接包，把 DSN 粘贴进 `dsns`，完成。

| 服务商               | 安装                                         | DSN 示例                                     |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | 内置                                         | `smtp://user:pass@host:587`                  |
| Sendmail             | 内置                                         | `sendmail://default`                         |
| Dev/null（丢弃邮件） | 内置                                         | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

完整列表见 [Symfony Mailer 文档](https://symfony.com/doc/current/mailer.html)——那里记录的任何内容在这里都能原样使用。

### 同时使用多个服务商

为每个传输命名，然后按消息选择：

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
// 不调用 ->transport() = 使用 "dsns" 中的第一个键（这里是 "transactional"）。
Flight::mail()->compose()->to('...')->text('收据')->send();

// 显式选择另一条路由。
Flight::mail()->compose()->to('...')->text('新闻通讯')->transport('bulk')->send();
```

## 配置参考

除了 `dsns` 之外，一切都是可选的。

```php
MailPlugin::install([
    // 必填 - 传输名称 => Symfony DSN。
    // 消息未指定时使用第一条。
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // 消息没有显式 ->transport()、且您不想用第一个键时使用的传输。
    // 必须存在于 "dsns" 中。
    'default_transport' => 'default',

    // 全局发件人。字符串、Symfony Address，或 ['email' => 'Name']。
    // 仅在消息没有设置自己的 ->from() 时应用。
    'from' => ['no-reply@example.com' => '我的应用'],

    // 默认模板引擎：'twig'、'latte'，或自定义名称。
    // 仅在模板扩展名不是已注册的渲染器时才会查询。
    'renderer' => 'twig',

    // 模板所在位置，按顺序搜索；外加一个可选的缓存目录。
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // 直接传给 Twig\Environment 的额外选项。
    'twig' => ['options' => ['strict_variables' => true]],

    // 启动时调整 Latte 引擎：fn(Latte\Engine $engine): void。
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // 发送时的正文增强（见“为 HTML 设置样式并生成文本部分”）。
    'inline_css' => true,           // 或 ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // 或 'plain' / 'markdown'

    // 自定义 DSN 方案、自定义渲染器、发送前钩子（见下文）。
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // 交给每个传输的可选基础设施。
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## 更进一步

下面的一切都是可选的。默认值已经覆盖大多数应用。

### 添加自定义 DSN 方案

实现 Symfony 的 `TransportFactoryInterface` 并注册它——然后您自己的方案就会像内置的一样工作：

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
        // ... 构建一个与您的运营商通信的传输
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### 添加自定义模板渲染器

任何能把模板名加参数变成字符串的东西都符合条件：

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// 以 .markdown 结尾的模板现在会自动使用它：
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### 在发送前立刻运行某些操作

钩子接收已完成的消息——在渲染之后、默认值应用之后、即将上线之前：

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### 事件与日志

交出一个 Symfony 事件分发器和/或 PSR-3 logger，每个传输都会使用它们：

```php
$plugin->eventDispatcher($dispatcher); // 每次发送前接收 MessageEvent
$plugin->logger($logger);              // 传输级日志
```

## API 速查表

```php
// 设置
MailPlugin::install($config)             // 在全局 Flight 应用上注册
MailPlugin::register($app, $config)      // 在特定 Engine 上注册
$mailer = Flight::mail();                // 共享的 Mailer 实例

// 构建消息
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // 标准 Symfony Mime 方法
$message->text(string)                       // 纯字符串正文
$message->html(string)                       // HTML 字符串正文
$message->template($name, $params)           // 来自模板的 HTML 正文
$message->htmlTemplate($name, $params)       // template() 的别名
$message->textTemplate($name, $params)       // 来自模板的文本正文
$message->inlineCss() / ->withoutInlineCss() // 按消息的 CSS 内联
$message->textFromHtml($mode)                // 自动文本部分：true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // 仅 HTML 的邮件
$message->transport($name)                   // 通过命名的 DSN 路由
$message->send(): ?SentMessage               // 渲染 + 发送

// 在 Mailer 本身上
$mailer->send($message): ?SentMessage        // $message->send() 的显式替代
$mailer->render($template, $params): string  // 只渲染不发送
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

因为 `Message` 扩展了 `Symfony\Component\Mime\Email`，您已经熟悉的每个 Symfony 方法——`attach()`、`embed()`、`priority()`、`replyTo()`——开箱即用。

## 故障排除

**"No mail DSNs configured"**
您在注册插件之前调用了 `Flight::mail()`，或者配置数组没有包含 `dsns`。这个错误是故意的——FlightMail 拒绝猜测邮件该发到哪里，而不是默默丢掉它。

**"Unknown mail template renderer ..."**
您使用了引擎尚未安装的模板。用 `composer require twig/twig` 或 `composer require latte/latte` 修复，或者注册一个以该扩展名命名的自定义渲染器。

**"Unknown mail transport ..."**
某个 `->transport('name')`（或 `default_transport`）与 `dsns` 中的任何键都不匹配。检查拼写——错误会列出已配置的名称。

**邮件没有到达**
把 `dsns` 指向 `null://null` 以确认其余代码能工作，然后再切回真正的 DSN。在 DDEV 中，使用 `smtp://127.0.0.1:1025`，并在 8025 端口的 Mailpit 中检查消息。

---

要提交错误报告、拉取请求，以及查看完整源码，请访问 [GitHub 仓库](https://github.com/ryanstubbs/flightmail)。
