# FlightMail

> **Сторонний плагин** - поддерживается [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), лицензия MIT). Не входит в ядро Flight - пожалуйста, сообщайте о проблемах в [его репозитории на GitHub](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) позволяет отправлять электронную почту из вашего приложения Flight без головной боли. Он оборачивает **Symfony Mailer** - самую проверенную в бою почтовую библиотеку в PHP - и делает так, будто это часть Flight. Одна строка для установки, одна fluent-цепочка для отправки:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('У вас получилось!')
    ->text('Ваше первое письмо уже в пути.')
    ->send();
```

## Возможности

- **Любой провайдер — по одной строке.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo и компания работают через простые DSN-строки.
- **Несколько провайдеров сразу.** Транзакционные письма через Postmark, рассылки через свой SMTP — выбирайте для каждого сообщения.
- **Шаблоны, если захотите.** Рендерьте тела писем через Twig или Latte. Не хотите шаблоны? Просто передайте строки и не устанавливайте ничего лишнего.
- **Лоск в момент отправки.** Опциональное встраивание CSS и автоматические текстовые части из вашего HTML — на библиотеках, которые ставятся только если вы ими пользуетесь.
- **Скучный в лучшем смысле.** Ленивые соединения, понятные ошибки вместо тихо проглоченных писем, и всё можно подменить, если нужно что-то своё.

## Требования

| Что            | Версия                                 |
| -------------- | -------------------------------------- |
| PHP            | 8.2 или новее                          |
| Flight PHP     | core ^3.15                             |
| Symfony Mailer | ^7.2 или ^8.0 (устанавливается автоматически) |

## Установка

```bash
composer require ryanstubbs/flightmail
```

Этого достаточно для отправки писем в простом тексте и HTML. Рендеринг шаблонов подключается по желанию — добавьте движок, только если будете им пользоваться:

```bash
composer require twig/twig      # для шаблонов .twig
composer require latte/latte    # для шаблонов .latte
```

Ещё две опциональные библиотеки обеспечивают улучшения в момент отправки, описанные [ниже](#styling-html-and-generating-text-parts):

```bash
composer require pelago/emogrifier         # для встраивания CSS ("inline_css")
composer require league/html-to-markdown   # для текстовых частей в Markdown ("text_from_html")
```

Все их можно ставить рядом; FlightMail выберет нужную исходя из вашей конфигурации.

## Ваше первое письмо

Добавьте это в bootstrap (туда же, где вы определяете маршруты):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Скажите FlightMail, откуда и через что отправлять почту.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Добро пожаловать на борт!')
        ->html('<h1>Добро пожаловать!</h1><p>Мы рады, что вы здесь.</p>')
        ->send();
});

Flight::start();
```

Пользуетесь [скелетом Flight PHP](https://github.com/flightphp/skeleton)? Зарегистрируйте в `app/config/services.php` в стиле экземпляра:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Оба стиля дают один и тот же мейлер: `Flight::mail()` и `$app->mail()` взаимозаменяемы.

> **Тестируете локально?** Если проект крутится в [DDEV](https://ddev.com), направьте DSN на `smtp://127.0.0.1:1025` и читайте каждое перехваченное письмо в Mailpit по адресу `http://<project>.ddev.site:8025`. Ничего не покидает вашу машину.

## Отправка электронной почты

### Простые строки (движок шаблонов не нужен)

`->text()` и `->html()` принимают обычные строки и больше ничего устанавливать не нужно:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Резервное копирование завершено')
    ->text('Ночное резервное копирование завершено за 42 минуты.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Счёт №123')
    ->html('<h1>Счёт №123</h1><p>К оплате: $42.00</p>')
    ->send();
```

### Шаблоны Twig

```php
// welcome.html.twig содержит: Привет, {{ name }}, спасибо за регистрацию!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Добро пожаловать!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Шаблоны Latte

Та же идея, расширение `.latte`:

```php
// welcome.latte содержит: Привет, {$name}, спасибо за регистрацию!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Добро пожаловать!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + простой текст вместе

Лучшая практика для доставляемости — дайте почтовым клиентам обе версии:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Добро пожаловать!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // богатая версия
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // запасная версия
    ->send();
```

Несколько вещей, которые стоит знать о шаблонах:

- Они рендерятся **лениво**, в момент отправки — составляйте сейчас, рендерьте потом.
- Движок выбирается по расширению: `.twig` → Twig, `.latte` → Latte, всё остальное → ваш настроенный по умолчанию (опция `renderer`).
- Явное тело `->html()` или `->text()` всегда побеждает шаблон, так что можно задать шаблон по умолчанию и переопределить его для конкретного сообщения.

## Стилизация HTML и генерация текстовых частей

Два опциональных улучшения в момент отправки, оба выключены по умолчанию и оба работают на библиотеках, которые вы ставите только если хотите:

| Функция                  | Установка                 | Ключ конфигурации |
| ------------------------ | ------------------------- | ----------------- |
| Встраивание CSS          | `pelago/emogrifier`       | `inline_css`      |
| Текстовая часть из HTML  | `league/html-to-markdown` | `text_from_html`  |

### Встраивание CSS в HTML-письмо

Gmail и большинство вебмейл-клиентов вырезают блоки `<style>` — атрибуты `style=""` внутри тегов — единственная стилизация, которую они надёжно уважают. Писать их вручную — мучение; пусть [Emogrifier](https://github.com/MyIntervals/emogrifier) сделает это в момент отправки:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Когда это включено, каждое HTML-тело получает встроенный CSS прямо перед отправкой — хоть из шаблона, хоть из `->html()`. Сообщение вроде `<style>p { color: red; }</style><p>Привет</p>` уходит как `<p style="color: red;">Привет</p>`.

Чтобы внедрить общие стили в каждое письмо (цвета бренда, сбросы) без повторения их в каждом шаблоне, передайте правила напрямую или укажите файл таблицы стилей:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// или
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Управление на уровне сообщения:

```php
$message->inlineCss();          // принудительно встроить CSS для этого сообщения
$message->withoutInlineCss();   // пропустить, даже если включено глобально
```

### Генерация текстовой части из HTML

Лучшая практика — отправлять HTML и простую текстовую версию вместе, но писать обе утомительно. FlightMail может вывести текстовую часть из итогового HTML автоматически — базовая конвертация не требует лишней зависимости, конвертер поставляется вместе с Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown когда возможно, иначе простой текст
]);
```

Режимы:

- `true` или `'auto'` — вывод Markdown, если установлен `league/html-to-markdown`, иначе простое срезание тегов.
- `'markdown'` — принудительно Markdown (`composer require league/html-to-markdown`; заголовки становятся `==`, ссылки `[text](url)`, жирный `**bold**`).
- `'plain'` — всегда срезать теги; работает без дополнительных пакетов.

Генерация запускается после рендеринга и встраивания CSS и только когда у сообщения есть HTML-тело, но нет текстового — явное `->text()` или `->textTemplate()` всегда побеждает. Переопределения на уровне сообщения зеркалят встраивание:

```php
$message->textFromHtml('plain');    // принудительно срезать теги для этого письма
$message->withoutTextFromHtml();    // письмо только с HTML
```

Включите режим, чья библиотека не установлена — получите понятную ошибку с точной командой `composer require`, которую нужно выполнить, без тихой деградации.

## Выбор провайдера

Провайдеры подключаются через DSN-строки. Установите пакет-мост, вставьте DSN в `dsns`, готово.

| Провайдер            | Установка                                    | Пример DSN                                   |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | встроенный                                   | `smtp://user:pass@host:587`                  |
| Sendmail             | встроенный                                   | `sendmail://default`                         |
| Dev/null (отбрасывать письма) | встроенный                          | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

Полный список живёт в [документации Symfony Mailer](https://symfony.com/doc/current/mailer.html) — всё, что там описано, работает здесь без изменений.

### Несколько провайдеров одновременно

Дайте имя каждому транспорту, затем выбирайте для сообщения:

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
// Нет вызова ->transport() = первый ключ в "dsns" (здесь "transactional").
Flight::mail()->compose()->to('...')->text('квитанция')->send();

// Явно выбрать другой маршрут.
Flight::mail()->compose()->to('...')->text('рассылка')->transport('bulk')->send();
```

## Справочник конфигурации

Всё опционально, кроме `dsns`.

```php
MailPlugin::install([
    // ОБЯЗАТЕЛЬНО - имя транспорта => Symfony DSN.
    // Первая запись используется, когда сообщение не указывает транспорт.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Транспорт, когда у сообщения нет явного ->transport() и
    // вы не хотите первый ключ. Должен существовать в "dsns".
    'default_transport' => 'default',

    // Глобальный отправитель. Строка, Symfony Address или ['email' => 'Name'].
    // Применяется только когда сообщение не задаёт свой ->from().
    'from' => ['no-reply@example.com' => 'Моё приложение'],

    // Движок шаблонов по умолчанию: 'twig', 'latte' или своё имя.
    // Используется только для шаблонов, чьё расширение не является зарегистрированным рендерером.
    'renderer' => 'twig',

    // Где живут шаблоны, поиск по порядку; плюс опциональный каталог кэша.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Дополнительные опции, которые передаются напрямую в Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Настройка движка Latte при загрузке: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Улучшения тела письма при отправке (см. «Стилизация HTML и генерация текстовых частей»).
    'inline_css' => true,           // или ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // или 'plain' / 'markdown'

    // Свои схемы DSN, свои рендереры, хуки перед отправкой (см. ниже).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Опциональная обвязка, передаваемая каждому транспорту.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Идём дальше

Всё ниже необязательно. Значения по умолчанию покрывают большинство приложений.

### Добавить свою схему DSN

Реализуйте `TransportFactoryInterface` из Symfony и зарегистрируйте его — тогда ваша схема будет работать точно как встроенная:

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
        // ... соберите транспорт, который общается с вашим оператором
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Добавить свой рендерер шаблонов

Подойдёт всё, что превращает имя шаблона плюс параметры в строку:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Шаблоны с окончанием .markdown теперь используют его автоматически:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Выполнить что-то прямо перед отправкой

Хуки получают готовое сообщение — после рендеринга, после значений по умолчанию, прямо перед отправкой в сеть:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### События и логирование

Передайте диспетчер событий Symfony и/или PSR-3 логгер — каждый транспорт будет их использовать:

```php
$plugin->eventDispatcher($dispatcher); // получает MessageEvent перед каждой отправкой
$plugin->logger($logger);              // логи на уровне транспорта
```

## Шпаргалка по API

```php
// Настройка
MailPlugin::install($config)             // регистрация в глобальном приложении Flight
MailPlugin::register($app, $config)      // регистрация в конкретном Engine
$mailer = Flight::mail();                // общий экземпляр Mailer

// Сборка сообщений
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // стандартные методы Symfony Mime
$message->text(string)                       // тело из простой строки
$message->html(string)                       // тело из HTML-строки
$message->template($name, $params)           // HTML-тело из шаблона
$message->htmlTemplate($name, $params)       // псевдоним template()
$message->textTemplate($name, $params)       // текстовое тело из шаблона
$message->inlineCss() / ->withoutInlineCss() // встраивание CSS для сообщения
$message->textFromHtml($mode)                // авто текстовая часть: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // письмо только с HTML
$message->transport($name)                   // маршрут через именованный DSN
$message->send(): ?SentMessage               // рендер + отправка

// На самом мейлере
$mailer->send($message): ?SentMessage        // явная альтернатива $message->send()
$mailer->render($template, $params): string  // рендер без отправки
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Поскольку `Message` расширяет `Symfony\Component\Mime\Email`, каждый метод Symfony, который вы уже знаете — `attach()`, `embed()`, `priority()`, `replyTo()` — работает из коробки.

## Устранение неисправностей

**"No mail DSNs configured"**
Вы вызвали `Flight::mail()` до регистрации плагина, или массив конфигурации не содержал `dsns`. Эта ошибка намеренная — FlightMail отказывается угадывать, куда должна идти почта, вместо того чтобы тихо её отбрасывать.

**"Unknown mail template renderer ..."**
Вы использовали шаблон, движок которого не установлен. Исправьте с помощью `composer require twig/twig` или `composer require latte/latte`, либо зарегистрируйте свой рендерер с именем расширения.

**"Unknown mail transport ..."**
Вызов `->transport('name')` (или `default_transport`) не совпадает ни с одним ключом в `dsns`. Проверьте написание — ошибка перечисляет настроенные имена.

**Письма не доходят**
Направьте `dsns` на `null://null`, чтобы убедиться, что остальной код работает, затем вернитесь к настоящему DSN. В DDEV используйте `smtp://127.0.0.1:1025` и смотрите сообщения в Mailpit на порту 8025.

---

Сообщения об ошибках, pull request'ы и полный исходный код — в [репозитории на GitHub](https://github.com/ryanstubbs/flightmail).
