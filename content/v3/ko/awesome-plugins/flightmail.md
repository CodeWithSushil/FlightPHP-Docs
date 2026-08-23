# FlightMail

> **서드파티 플러그인** - [Ryan Stubbs](https://ryanstubbs.co.uk)가 유지보수합니다([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), MIT 라이선스). Flight 코어의 일부가 아닙니다. 문제는 [GitHub 저장소](https://github.com/ryanstubbs/flightmail/issues)에 보고해 주세요.

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail)을 사용하면 Flight 앱에서 골치 없이 이메일을 보낼 수 있습니다. PHP에서 가장 실전 검증된 메일 라이브러리인 **Symfony Mailer**를 래핑하여 Flight의 일부처럼 느껴지게 합니다. 설치는 한 줄, 전송은 유창한 체인 한 번입니다:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('해냈습니다!')
    ->text('첫 번째 이메일이 곧 도착합니다.')
    ->send();
```

## 기능

- **어떤 제공자든, 각각 한 줄.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo와 그 친구들은 간단한 DSN 문자열로 모두 동작합니다.
- **여러 제공자를 동시에 사용.** 트랜잭션 메일은 Postmark로, 뉴스레터는 자체 SMTP로 — 메시지마다 고릅니다.
- **템플릿은 원할 때만.** Twig 또는 Latte로 본문을 렌더링합니다. 템플릿이 필요 없다면 문자열만 넘기고 추가로 설치할 것은 없습니다.
- **전송 시점의 마무리.** 선택적 CSS 인라인화와 HTML에서 자동으로 파생되는 일반 텍스트 파트. 사용할 때만 설치하는 라이브러리로 동작합니다.
- **가장 좋은 의미에서 지루합니다.** 지연 연결, 메일을 조용히 삼키는 대신 명확한 오류, 커스텀이 필요하면 모든 것을 교체할 수 있습니다.

## 요구 사항

| 항목           | 버전                                   |
| -------------- | -------------------------------------- |
| PHP            | 8.2 이상                               |
| Flight PHP     | core ^3.15                             |
| Symfony Mailer | ^7.2 또는 ^8.0 (자동 설치)             |

## 설치

```bash
composer require ryanstubbs/flightmail
```

일반 텍스트와 HTML 이메일을 보내는 데는 이게 전부입니다. 템플릿 렌더링은 옵트인입니다. 사용할 경우에만 엔진을 추가하세요:

```bash
composer require twig/twig      # .twig 템플릿용
composer require latte/latte    # .latte 템플릿용
```

두 개의 추가 선택 라이브러리가 [아래](#styling-html-and-generating-text-parts)에서 다루는 전송 시점 향상 기능을 지원합니다:

```bash
composer require pelago/emogrifier         # CSS 인라인화용 ("inline_css")
composer require league/html-to-markdown   # Markdown 텍스트 파트용 ("text_from_html")
```

이 모두는 나란히 설치할 수 있습니다. FlightMail은 구성한 내용에 따라 알맞은 것을 고릅니다.

## 첫 번째 이메일

부트스트랩(라우트를 정의하는 같은 곳)에 이것을 추가하세요:

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// FlightMail에 메일을 어디서, 어떤 경로로 보낼지 알려 줍니다.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('가입을 환영합니다!')
        ->html('<h1>환영합니다!</h1><p>여기에 와 주셔서 기쁩니다.</p>')
        ->send();
});

Flight::start();
```

[Flight PHP skeleton](https://github.com/flightphp/skeleton)을 사용 중이신가요? 대신 인스턴스 방식으로 `app/config/services.php`에 등록하세요:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

두 스타일 모두 같은 메일러를 노출합니다. `Flight::mail()`과 `$app->mail()`은 서로 바꿔 쓸 수 있습니다.

> **로컬에서 테스트하나요?** 프로젝트가 [DDEV](https://ddev.com)에서 실행된다면 DSN을 `smtp://127.0.0.1:1025`로 향하게 하고, 캡처된 모든 이메일을 `http://<project>.ddev.site:8025`의 Mailpit에서 읽으세요. 아무것도 기기를 떠나지 않습니다.

## 이메일 보내기

### 일반 문자열 (템플릿 엔진 불필요)

`->text()`와 `->html()`은 원본 문자열을 받으며 다른 것은 설치할 필요가 없습니다:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('백업 완료')
    ->text('야간 백업이 42분 만에 완료되었습니다.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('청구서 #123')
    ->html('<h1>청구서 #123</h1><p>총 청구액: $42.00</p>')
    ->send();
```

### Twig 템플릿

```php
// welcome.html.twig 내용: Hello {{ name }}, 가입해 주셔서 감사합니다!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('환영합니다!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Latte 템플릿

같은 아이디어이며 확장자는 `.latte`입니다:

```php
// welcome.latte 내용: Hello {$name}, 가입해 주셔서 감사합니다!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('환영합니다!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + 일반 텍스트를 함께

전달성의 모범 사례 — 메일 클라이언트에 두 버전을 모두 제공합니다:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('환영합니다!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // 리치 버전
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // 폴백 버전
    ->send();
```

템플릿에 대해 알아 두면 좋은 점이 몇 가지 있습니다:

- **지연** 렌더링됩니다. 전송 시점에 실행됩니다. 지금은 compose하고 나중에 렌더링합니다.
- 엔진은 확장자로 선택됩니다. `.twig` → Twig, `.latte` → Latte, 그 외 → 구성한 기본값(`renderer` 옵션).
- 명시적인 `->html()` 또는 `->text()` 본문은 항상 템플릿보다 우선하므로, 기본 템플릿을 설정하고 메시지마다 덮어쓸 수 있습니다.

## HTML 스타일링과 텍스트 파트 생성

두 가지 선택적 전송 시점 향상 기능이 있으며, 둘 다 기본적으로 꺼져 있고 원할 때만 설치하는 라이브러리로 동작합니다:

| 기능                      | 설치                      | 설정 키          |
| ------------------------- | ------------------------- | ---------------- |
| CSS 인라인화              | `pelago/emogrifier`       | `inline_css`     |
| HTML에서 텍스트 파트 생성 | `league/html-to-markdown` | `text_from_html` |

### HTML 이메일에 CSS 인라인화하기

Gmail과 대부분의 웹메일 클라이언트는 `<style>` 블록을 벗겨 냅니다. 안정적으로 존중되는 스타일은 인라인 `style=""` 속성뿐입니다. 손으로 쓰는 것은 끔찍하니, 전송 시점에 [Emogrifier](https://github.com/MyIntervals/emogrifier)가 하도록 하세요:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

이것을 켜면 모든 HTML 본문은 전송 직전에 CSS가 인라인됩니다. 템플릿에서 왔든 `->html()`에서 왔든 마찬가지입니다. `<style>p { color: red; }</style><p>안녕하세요</p>` 같은 메시지는 `<p style="color: red;">안녕하세요</p>`로 나갑니다.

브랜드 색상이나 리셋처럼 공유 스타일을 각 템플릿에서 반복하지 않고 모든 이메일에 주입하려면, 규칙을 직접 넘기거나 스타일시트 파일을 가리키세요:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// 또는
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

메시지 단위 제어:

```php
$message->inlineCss();          // 이 메시지 하나에 인라인화를 강제
$message->withoutInlineCss();   // 전역에서 켜져 있어도 건너뜀
```

### HTML에서 텍스트 파트 생성하기

모범 사례는 HTML과 일반 텍스트 버전을 함께 보내는 것이지만, 둘 다 쓰는 것은 번거롭습니다. FlightMail은 최종 HTML에서 텍스트 파트를 자동으로 파생할 수 있습니다. 기본 변환에는 추가 의존성이 필요 없습니다. 변환기는 Symfony Mime과 함께 제공됩니다:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // 가능하면 Markdown, 아니면 일반 텍스트
]);
```

모드:

- `true` 또는 `'auto'` - `league/html-to-markdown`이 설치되어 있으면 Markdown 출력, 아니면 단순 태그 제거.
- `'markdown'` - Markdown 강제(`composer require league/html-to-markdown`. 제목은 `==`, 링크는 `[text](url)`, 굵게는 `**bold**`).
- `'plain'` - 항상 태그를 제거합니다. 추가 패키지 없이 동작합니다.

생성은 렌더링과 CSS 인라인화 이후에 실행되며, 메시지에 HTML 본문은 있고 텍스트 본문은 없을 때만입니다. 명시적인 `->text()` 또는 `->textTemplate()`은 항상 이깁니다. 메시지 단위 덮어쓰기는 인라인화와 같습니다:

```php
$message->textFromHtml('plain');    // 이 하나에 태그 제거를 강제
$message->withoutTextFromHtml();    // HTML 전용 이메일
```

라이브러리가 설치되지 않은 모드를 켜면 실행해야 할 정확한 `composer require`를 알려 주는 명확한 오류가 납니다. 조용히 저하되지 않습니다.

## 제공자 선택

제공자는 DSN 문자열로 연결됩니다. 브리지 패키지를 설치하고 DSN을 `dsns`에 붙여 넣으면 끝입니다.

| 제공자                   | 설치                                         | DSN 예제                                     |
| ------------------------ | -------------------------------------------- | -------------------------------------------- |
| SMTP                     | 내장                                         | `smtp://user:pass@host:587`                  |
| Sendmail                 | 내장                                         | `sendmail://default`                         |
| Dev/null (메일 폐기)     | 내장                                         | `null://null`                                |
| Postmark                 | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid                 | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun                  | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES               | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                    | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend               | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

전체 목록은 [Symfony Mailer 문서](https://symfony.com/doc/current/mailer.html)에 있습니다. 그곳에 문서화된 것은 여기서도 그대로 동작합니다.

### 여러 제공자를 동시에 사용

각 트랜스포트에 이름을 붙인 다음 메시지마다 고릅니다:

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
// ->transport() 호출 없음 = "dsns"의 첫 번째 키(여기서는 "transactional").
Flight::mail()->compose()->to('...')->text('영수증')->send();

// 다른 경로를 명시적으로 선택합니다.
Flight::mail()->compose()->to('...')->text('뉴스레터')->transport('bulk')->send();
```

## 구성 레퍼런스

`dsns`를 제외한 모든 것은 선택 사항입니다.

```php
MailPlugin::install([
    // 필수 - 트랜스포트 이름 => Symfony DSN.
    // 메시지가 지정하지 않으면 첫 번째 항목이 사용됩니다.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // 메시지에 명시적인 ->transport()가 없고 첫 번째 키를 쓰고 싶지 않을 때 사용하는 트랜스포트.
    // "dsns"에 존재해야 합니다.
    'default_transport' => 'default',

    // 전역 발신자. 문자열, Symfony Address, 또는 ['email' => 'Name'].
    // 메시지가 자체 ->from()을 설정하지 않은 경우에만 적용됩니다.
    'from' => ['no-reply@example.com' => '내 앱'],

    // 기본 템플릿 엔진: 'twig', 'latte', 또는 사용자 지정 이름.
    // 확장자가 등록된 렌더러가 아닌 템플릿에서만 조회됩니다.
    'renderer' => 'twig',

    // 템플릿이 있는 위치. 순서대로 검색됩니다. 선택적 캐시 디렉터리도 있습니다.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Twig\Environment에 그대로 전달되는 추가 옵션.
    'twig' => ['options' => ['strict_variables' => true]],

    // 부팅 시 Latte 엔진을 조정: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // 전송 시점 본문 향상 ("HTML 스타일링과 텍스트 파트 생성" 참조).
    'inline_css' => true,           // 또는 ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // 또는 'plain' / 'markdown'

    // 사용자 지정 DSN 스킴, 사용자 지정 렌더러, 전송 전 훅 (아래 참조).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // 모든 트랜스포트에 전달되는 선택적 배관.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## 더 나아가기

아래의 모든 것은 선택 사항입니다. 기본값으로 대부분의 앱을 커버합니다.

### 사용자 지정 DSN 스킴 추가

Symfony의 `TransportFactoryInterface`를 구현하고 등록하세요. 그러면 자체 스킴도 내장된 것과 똑같이 동작합니다:

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
        // ... 캐리어와 통신하는 트랜스포트를 구축합니다
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### 사용자 지정 템플릿 렌더러 추가

템플릿 이름과 파라미터를 문자열로 바꾸는 것이면 무엇이든 해당됩니다:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// .markdown으로 끝나는 템플릿은 이제 자동으로 이것을 사용합니다:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### 전송 직전에 무언가 실행하기

훅은 완성된 메시지를 받습니다. 렌더링 후, 기본값 적용 후, 회선에 오르기 직전입니다:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### 이벤트와 로깅

Symfony 이벤트 디스패처 및/또는 PSR-3 로거를 넘기면 모든 트랜스포트가 이를 사용합니다:

```php
$plugin->eventDispatcher($dispatcher); // 각 전송 전에 MessageEvent를 수신
$plugin->logger($logger);              // 트랜스포트 수준 로그
```

## API 치트시트

```php
// 설정
MailPlugin::install($config)             // 전역 Flight 앱에 등록
MailPlugin::register($app, $config)      // 특정 Engine에 등록
$mailer = Flight::mail();                // 공유 Mailer 인스턴스

// 메시지 구축
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // 표준 Symfony Mime 메서드
$message->text(string)                       // 일반 문자열 본문
$message->html(string)                       // HTML 문자열 본문
$message->template($name, $params)           // 템플릿에서 HTML 본문
$message->htmlTemplate($name, $params)       // template()의 별칭
$message->textTemplate($name, $params)       // 템플릿에서 텍스트 본문
$message->inlineCss() / ->withoutInlineCss() // 메시지 단위 CSS 인라인화
$message->textFromHtml($mode)                // 자동 텍스트 파트: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // HTML 전용 이메일
$message->transport($name)                   // 이름이 지정된 DSN을 통해 라우팅
$message->send(): ?SentMessage               // 렌더링 + 전송

// 메일러 자체에서
$mailer->send($message): ?SentMessage        // $message->send()의 명시적 대안
$mailer->render($template, $params): string  // 전송 없이 렌더링
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

`Message`가 `Symfony\Component\Mime\Email`을 확장하므로, 이미 알고 있는 모든 Symfony 메서드 — `attach()`, `embed()`, `priority()`, `replyTo()` — 는 바로 동작합니다.

## 문제 해결

**"No mail DSNs configured"**
플러그인을 등록하기 전에 `Flight::mail()`을 호출했거나, 구성 배열에 `dsns`가 포함되지 않았습니다. 이 오류는 의도적입니다. FlightMail은 메일이 어디로 가야 할지 추측해 조용히 버리는 대신 거부합니다.

**"Unknown mail template renderer ..."**
엔진이 설치되지 않은 템플릿을 사용했습니다. `composer require twig/twig` 또는 `composer require latte/latte`로 고치거나, 확장자 이름으로 사용자 지정 렌더러를 등록하세요.

**"Unknown mail transport ..."**
`->transport('name')`(또는 `default_transport`)가 `dsns`의 어떤 키와도 일치하지 않습니다. 철자를 확인하세요. 오류에 구성된 이름이 나열됩니다.

**메일이 도착하지 않음**
`dsns`를 `null://null`로 향하게 하여 나머지 코드가 동작하는지 확인한 다음 실제 DSN으로 되돌리세요. DDEV에서는 `smtp://127.0.0.1:1025`를 사용하고 포트 8025의 Mailpit에서 메시지를 검사하세요.

---

버그 리포트, 풀 리퀘스트, 전체 소스는 [GitHub 저장소](https://github.com/ryanstubbs/flightmail)를 방문하세요.
