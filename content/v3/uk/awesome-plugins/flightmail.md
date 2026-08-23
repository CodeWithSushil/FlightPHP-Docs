# FlightMail

> **Сторонній плагін** - підтримується [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), ліцензія MIT). Не є частиною ядра Flight - будь ласка, повідомляйте про проблеми в [його репозиторії на GitHub](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) дозволяє надсилати електронну пошту з вашого додатка Flight без головного болю. Він обгортає **Symfony Mailer** - найбільш перевірену в бою поштову бібліотеку в PHP - і робить так, ніби це частина Flight. Один рядок для встановлення, один fluent-ланцюжок для надсилання:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('У вас вийшло!')
    ->text('Ваш перший лист уже в дорозі.')
    ->send();
```

## Можливості

- **Будь-який провайдер — по одному рядку.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo і компанія працюють через прості DSN-рядки.
- **Кілька провайдерів одразу.** Транзакційні листи через Postmark, розсилки через власний SMTP — обирайте для кожного повідомлення.
- **Шаблони, якщо захочете.** Рендеріть тіла листів через Twig або Latte. Не хочете шаблони? Просто передайте рядки й не встановлюйте нічого зайвого.
- **Глянець у момент надсилання.** Необов'язкове вбудовування CSS і автоматичні текстові частини з вашого HTML — на бібліотеках, які ставляться лише якщо ви ними користуєтесь.
- **Нудний у найкращому сенсі.** Ліниві з'єднання, зрозумілі помилки замість тихо проковтнутих листів, і все можна підмінити, якщо потрібно щось своє.

## Вимоги

| Що             | Версія                                 |
| -------------- | -------------------------------------- |
| PHP            | 8.2 або новіша                         |
| Flight PHP     | core ^3.15                             |
| Symfony Mailer | ^7.2 або ^8.0 (встановлюється автоматично) |

## Встановлення

```bash
composer require ryanstubbs/flightmail
```

Цього достатньо для надсилання листів у простому тексті та HTML. Рендеринг шаблонів підключається за бажанням — додайте шаблонізатор, лише якщо ним користуватиметесь:

```bash
composer require twig/twig      # для шаблонів .twig
composer require latte/latte    # для шаблонів .latte
```

Ще дві необов'язкові бібліотеки забезпечують покращення в момент надсилання, описані [нижче](#styling-html-and-generating-text-parts):

```bash
composer require pelago/emogrifier         # для вбудовування CSS ("inline_css")
composer require league/html-to-markdown   # для текстових частин у Markdown ("text_from_html")
```

Усі їх можна ставити поряд; FlightMail обере потрібну, спираючись на вашу конфігурацію.

## Ваш перший лист

Додайте це до bootstrap (туди ж, де ви визначаєте маршрути):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Скажіть FlightMail, звідки і через що надсилати пошту.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Ласкаво просимо на борт!')
        ->html('<h1>Ласкаво просимо!</h1><p>Ми раді, що ви тут.</p>')
        ->send();
});

Flight::start();
```

Користуєтесь [скелетом Flight PHP](https://github.com/flightphp/skeleton)? Зареєструйте в `app/config/services.php` у стилі екземпляра:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Обидва стилі дають той самий мейлер: `Flight::mail()` і `$app->mail()` взаємозамінні.

> **Тестуєте локально?** Якщо проєкт крутиться в [DDEV](https://ddev.com), спрямуйте DSN на `smtp://127.0.0.1:1025` і читайте кожен перехоплений лист у Mailpit за адресою `http://<project>.ddev.site:8025`. Нічого не покидає вашу машину.

## Надсилання електронної пошти

### Прості рядки (шаблонізатор не потрібен)

`->text()` і `->html()` приймають звичайні рядки, і більше нічого встановлювати не потрібно:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Резервне копіювання завершено')
    ->text('Нічне резервне копіювання завершено за 42 хвилини.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Рахунок №123')
    ->html('<h1>Рахунок №123</h1><p>До сплати: $42.00</p>')
    ->send();
```

### Шаблони Twig

```php
// welcome.html.twig містить: Привіт, {{ name }}, дякуємо за реєстрацію!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Ласкаво просимо!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Шаблони Latte

Та сама ідея, розширення `.latte`:

```php
// welcome.latte містить: Привіт, {$name}, дякуємо за реєстрацію!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Ласкаво просимо!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + простий текст разом

Найкраща практика для доставлюваності — дайте поштовим клієнтам обидві версії:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Ласкаво просимо!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // багата версія
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // запасна версія
    ->send();
```

Кілька речей, які варто знати про шаблони:

- Вони рендеряться **ліниво**, у момент надсилання — складайте зараз, рендеріть потім.
- Шаблонізатор обирається за розширенням: `.twig` → Twig, `.latte` → Latte, усе інше → ваш налаштований за замовчуванням (опція `renderer`).
- Явне тіло `->html()` або `->text()` завжди перемагає шаблон, тож можна задати шаблон за замовчуванням і перевизначити його для конкретного повідомлення.

## Стилізація HTML і генерація текстових частин

Два необов'язкові покращення в момент надсилання, обидва вимкнені за замовчуванням і обидва працюють на бібліотеках, які ви ставите лише якщо хочете:

| Функція                   | Встановлення              | Ключ конфігурації |
| ------------------------- | ------------------------- | ----------------- |
| Вбудовування CSS          | `pelago/emogrifier`       | `inline_css`      |
| Текстова частина з HTML   | `league/html-to-markdown` | `text_from_html`  |

### Вбудовування CSS у HTML-лист

Gmail і більшість клієнтів вебпошти вирізають блоки `<style>` — атрибути `style=""` усередині тегів — єдина стилізація, яку вони надійно шанують. Писати їх вручну — мука; нехай [Emogrifier](https://github.com/MyIntervals/emogrifier) зробить це в момент надсилання:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Коли це увімкнено, кожне HTML-тіло отримує вбудований CSS безпосередньо перед надсиланням — чи то з шаблону, чи з `->html()`. Повідомлення на кшталт `<style>p { color: red; }</style><p>Привіт</p>` йде як `<p style="color: red;">Привіт</p>`.

Щоб упровадити спільні стилі в кожен лист (кольори бренду, скидання стилів) без повторення їх у кожному шаблоні, передайте правила напряму або вкажіть файл таблиці стилів:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// або
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Керування на рівні повідомлення:

```php
$message->inlineCss();          // примусово вбудувати CSS для цього повідомлення
$message->withoutInlineCss();   // пропустити, навіть якщо увімкнено глобально
```

### Генерація текстової частини з HTML

Найкраща практика — надсилати HTML і просту текстову версію разом, але писати обидві стомлює. FlightMail може вивести текстову частину з підсумкового HTML автоматично — базова конвертація не потребує зайвої залежності, конвертер постачається разом із Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown коли можливо, інакше простий текст
]);
```

Режими:

- `true` або `'auto'` — вивід Markdown, якщо встановлено `league/html-to-markdown`, інакше просте зрізання тегів.
- `'markdown'` — примусово Markdown (`composer require league/html-to-markdown`; заголовки стають `==`, посилання `[text](url)`, жирний `**bold**`).
- `'plain'` — завжди зрізати теги; працює без додаткових пакетів.

Генерація запускається після рендерингу та вбудовування CSS і лише коли в повідомлення є HTML-тіло, але немає текстового — явне `->text()` або `->textTemplate()` завжди перемагає. Перевизначення на рівні повідомлення дзеркалять вбудовування:

```php
$message->textFromHtml('plain');    // примусово зрізати теги для цього листа
$message->withoutTextFromHtml();    // лист лише з HTML
```

Увімкніть режим, чия бібліотека не встановлена — отримаєте зрозумілу помилку з точною командою `composer require`, яку потрібно виконати, без тихої деградації.

## Вибір провайдера

Провайдери підключаються через DSN-рядки. Встановіть пакет-міст, вставте DSN у `dsns`, готово.

| Провайдер            | Встановлення                                 | Приклад DSN                                  |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | вбудований                                   | `smtp://user:pass@host:587`                  |
| Sendmail             | вбудований                                   | `sendmail://default`                         |
| Dev/null (відкидати листи) | вбудований                             | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

Повний список живе в [документації Symfony Mailer](https://symfony.com/doc/current/mailer.html) — усе, що там описано, працює тут без змін.

### Кілька провайдерів одночасно

Дайте ім'я кожному транспорту, потім обирайте для повідомлення:

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
// Немає виклику ->transport() = перший ключ у "dsns" (тут "transactional").
Flight::mail()->compose()->to('...')->text('квитанція')->send();

// Явно обрати інший маршрут.
Flight::mail()->compose()->to('...')->text('розсилка')->transport('bulk')->send();
```

## Довідник конфігурації

Усе необов'язкове, крім `dsns`.

```php
MailPlugin::install([
    // ОБОВ'ЯЗКОВО - ім'я транспорту => Symfony DSN.
    // Перший запис використовується, коли повідомлення не вказує транспорт.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Транспорт, коли в повідомлення немає явного ->transport() і
    // ви не хочете перший ключ. Має існувати в "dsns".
    'default_transport' => 'default',

    // Глобальний відправник. Рядок, Symfony Address або ['email' => 'Name'].
    // Застосовується лише коли повідомлення не задає свій ->from().
    'from' => ['no-reply@example.com' => 'Мій додаток'],

    // Шаблонізатор за замовчуванням: 'twig', 'latte' або власне ім'я.
    // Використовується лише для шаблонів, чиє розширення не є зареєстрованим рендерером.
    'renderer' => 'twig',

    // Де живуть шаблони, пошук за порядком; плюс необов'язковий каталог кешу.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Додаткові опції, які передаються напряму в Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Налаштування рушія Latte під час завантаження: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Покращення тіла листа під час надсилання (див. «Стилізація HTML і генерація текстових частин»).
    'inline_css' => true,           // або ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // або 'plain' / 'markdown'

    // Власні схеми DSN, власні рендерери, хуки перед надсиланням (див. нижче).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Необов'язкова обв'язка, що передається кожному транспорту.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Йдемо далі

Усе нижче необов'язкове. Значення за замовчуванням покривають більшість додатків.

### Додати власну схему DSN

Реалізуйте `TransportFactoryInterface` із Symfony і зареєструйте його — тоді ваша схема працюватиме точно як вбудована:

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
        // ... зберіть транспорт, який спілкується з вашим оператором
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Додати власний рендерер шаблонів

Підійде все, що перетворює ім'я шаблону плюс параметри на рядок:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Шаблони із закінченням .markdown тепер використовують його автоматично:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Виконати щось безпосередньо перед надсиланням

Хуки отримують готове повідомлення — після рендерингу, після значень за замовчуванням, безпосередньо перед відправкою в мережу:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Події та логування

Передайте диспетчер подій Symfony та/або PSR-3 логер — кожен транспорт їх використовуватиме:

```php
$plugin->eventDispatcher($dispatcher); // отримує MessageEvent перед кожним надсиланням
$plugin->logger($logger);              // логи на рівні транспорту
```

## Шпаргалка з API

```php
// Налаштування
MailPlugin::install($config)             // реєстрація в глобальному додатку Flight
MailPlugin::register($app, $config)      // реєстрація в конкретному Engine
$mailer = Flight::mail();                // спільний екземпляр Mailer

// Збирання повідомлень
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // стандартні методи Symfony Mime
$message->text(string)                       // тіло зі звичайного рядка
$message->html(string)                       // тіло з HTML-рядка
$message->template($name, $params)           // HTML-тіло з шаблону
$message->htmlTemplate($name, $params)       // псевдонім template()
$message->textTemplate($name, $params)       // текстове тіло з шаблону
$message->inlineCss() / ->withoutInlineCss() // вбудовування CSS для повідомлення
$message->textFromHtml($mode)                // авто текстова частина: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // лист лише з HTML
$message->transport($name)                   // маршрут через іменований DSN
$message->send(): ?SentMessage               // рендер + надсилання

// На самому мейлері
$mailer->send($message): ?SentMessage        // явна альтернатива $message->send()
$mailer->render($template, $params): string  // рендер без надсилання
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Оскільки `Message` розширює `Symfony\Component\Mime\Email`, кожен метод Symfony, який ви вже знаєте — `attach()`, `embed()`, `priority()`, `replyTo()` — працює з коробки.

## Вирішення проблем

**"No mail DSNs configured"**
Ви викликали `Flight::mail()` до реєстрації плагіна, або масив конфігурації не містив `dsns`. Ця помилка навмисна — FlightMail відмовляється вгадувати, куди має йти пошта, замість того щоб тихо її відкидати.

**"Unknown mail template renderer ..."**
Ви використали шаблон, шаблонізатор якого не встановлено. Виправте за допомогою `composer require twig/twig` або `composer require latte/latte`, або зареєструйте власний рендерер з іменем розширення.

**"Unknown mail transport ..."**
Виклик `->transport('name')` (або `default_transport`) не збігається з жодним ключем у `dsns`. Перевірте написання — помилка перелічує налаштовані імена.

**Листи не доходять**
Спрямуйте `dsns` на `null://null`, щоб переконатися, що решта коду працює, потім поверніться до справжнього DSN. У DDEV використовуйте `smtp://127.0.0.1:1025` і переглядайте повідомлення в Mailpit на порту 8025.

---

Звіти про помилки, pull request'и та повний вихідний код — у [репозиторії на GitHub](https://github.com/ryanstubbs/flightmail).
