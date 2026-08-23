# FlightMail

> **Trešās puses spraudnis** - uztur [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), MIT licencēts). Nav Flight kodola daļa - lūdzu, ziņojiet par problēmām [tā GitHub krātuvē](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) ļauj sūtīt e-pastu no jūsu Flight lietotnes bez galvassāpēm. Tas ietver **Symfony Mailer** - visvairāk kaujā pārbaudīto pasta bibliotēku PHP - un liek tam justies kā Flight daļai. Viena rinda instalēšanai, viena plūstoša ķēde sūtīšanai:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Tev izdevās!')
    ->text('Jūsu pirmais e-pasts ir ceļā.')
    ->send();
```

## Funkcijas

- **Jebkurš pakalpojumu sniedzējs, katram viena rinda.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo un draugi visi darbojas caur vienkāršām DSN virknēm.
- **Izmantojiet vairākus pakalpojumu sniedzējus vienlaikus.** Transakciju pasts caur Postmark, biļeteni caur savu SMTP - izvēlieties katram ziņojumam.
- **Veidnes, ja tās vēlaties.** Renderējiet saturu ar Twig vai Latte. Nevēlaties veidnes? Vienkārši padodiet virknes un neinstalējiet neko papildu.
- **Noslīpējums sūtīšanas brīdī.** Neobligāta CSS iekļaušana un automātiskas vienkāršā teksta daļas, kas iegūtas no jūsu HTML, darbinātas ar bibliotēkām, kuras instalējat tikai tad, ja tās izmantojat.
- **Garlaicīgi vislabākajā veidā.** Slinkie savienojumi, skaidras kļūdas vietā klusībā norīta pasta, un viss ir aizstājams, ja jums vajag kaut ko pielāgotu.

## Prasības

| Kas            | Versija                                   |
| -------------- | ----------------------------------------- |
| PHP            | 8.2 vai jaunāka                           |
| Flight PHP     | core ^3.15                                |
| Symfony Mailer | ^7.2 vai ^8.0 (instalēts automātiski)     |

## Instalēšana

```bash
composer require ryanstubbs/flightmail
```

Tas ir viss, lai sūtītu vienkāršā teksta un HTML e-pastus. Veidņu renderēšana ir izvēles - pievienojiet dzinēju tikai tad, ja to izmantosiet:

```bash
composer require twig/twig      # .twig veidnēm
composer require latte/latte    # .latte veidnēm
```

Vēl divas neobligātas bibliotēkas darbina sūtīšanas brīža uzlabojumus, kas aprakstīti [zemāk](#html-stilizesana-un-teksta-dalu-generesana):

```bash
composer require pelago/emogrifier         # CSS iekļaušanai ("inline_css")
composer require league/html-to-markdown   # Markdown teksta daļām ("text_from_html")
```

Visas tās var instalēt līdzās; FlightMail izvēlas pareizo, pamatojoties uz to, ko konfigurējat.

## Jūsu pirmais e-pasts

Pievienojiet to savam bootstrap (tajā pašā vietā, kur definējat maršrutus):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Pastāstiet FlightMail, no kurienes un caur ko sūtīt pastu.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Laipni lūdzam uz klāja!')
        ->html('<h1>Laipni lūdzam!</h1><p>Mēs priecājamies, ka jūs esat šeit.</p>')
        ->send();
});

Flight::start();
```

Izmantojat [Flight PHP skeleton](https://github.com/flightphp/skeleton)? Reģistrējiet `app/config/services.php` ar instances stilu:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Abi stili piedāvā to pašu mailer: `Flight::mail()` un `$app->mail()` ir savstarpēji aizstājami.

> **Testējat lokāli?** Ja jūsu projekts darbojas [DDEV](https://ddev.com), vērsiet DSN uz `smtp://127.0.0.1:1025` un lasiet katru uztverto e-pastu Mailpit vietnē `http://<project>.ddev.site:8025`. Nekas neatstāj jūsu mašīnu.

## E-pasta sūtīšana

### Vienkāršas virknes (veidņu dzinējs nav nepieciešams)

`->text()` un `->html()` pieņem neapstrādātas virknes un neko citu instalētu nevajag:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Dublējums pabeigts')
    ->text('Nakts dublējums pabeigts 42 minūtēs.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Rēķins #123')
    ->html('<h1>Rēķins #123</h1><p>Kopējā summa: $42.00</p>')
    ->send();
```

### Twig veidnes

```php
// welcome.html.twig satur: Sveiki {{ name }}, paldies, ka reģistrējāties!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Laipni lūdzam!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Latte veidnes

Tā pati ideja, `.latte` paplašinājums:

```php
// welcome.latte satur: Sveiki {$name}, paldies, ka reģistrējāties!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Laipni lūdzam!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + vienkāršais teksts kopā

Labākā prakse piegādājamībai - dodiet pasta klientiem abas versijas:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Laipni lūdzam!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // bagātā versija
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // rezerves versija
    ->send();
```

Dažas lietas, ko vērts zināt par veidnēm:

- Tās tiek renderētas **slinki**, sūtīšanas brīdī - komponējiet tagad, renderējiet vēlāk.
- Dzinējs tiek izvēlēts pēc paplašinājuma: `.twig` → Twig, `.latte` → Latte, jebkas cits → jūsu konfigurētais noklusējums (`renderer` opcija).
- Eksplicīts `->html()` vai `->text()` saturs vienmēr uzvar pār veidni, tāpēc varat iestatīt noklusējuma veidni un pārrakstīt to katram ziņojumam.

## HTML stilizēšana un teksta daļu ģenerēšana

Divi neobligāti sūtīšanas brīža uzlabojumi, abi pēc noklusējuma izslēgti un abi darbināti ar bibliotēkām, kuras instalējat tikai tad, ja tās vēlaties:

| Funkcija                | Instalēšana               | Konfigurācijas atslēga |
| ----------------------- | ------------------------- | ---------------------- |
| CSS iekļaušana          | `pelago/emogrifier`       | `inline_css`           |
| Teksta daļa no HTML     | `league/html-to-markdown` | `text_from_html`       |

### CSS iekļaušana HTML e-pastā

Gmail un lielākā daļa tīmekļa pasta klientu noņem `<style>` blokus - iekļautie `style=""` atribūti ir vienīgais stils, ko tie uzticami ievēro. Rakstīt tos ar roku ir nožēlojami; ļaujiet [Emogrifier](https://github.com/MyIntervals/emogrifier) to izdarīt sūtīšanas brīdī:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Kad tas ir ieslēgts, katrs HTML saturs saņem savu CSS iekļautu tieši pirms sūtīšanas - neatkarīgi no tā, vai tas nāca no veidnes vai `->html()`. Ziņojums kā `<style>p { color: red; }</style><p>Sveiki</p>` iziet kā `<p style="color: red;">Sveiki</p>`.

Lai ievadītu kopīgus stilus katrā e-pastā (zīmola krāsas, atiestatījumi) bez to atkārtošanas katrā veidnē, padodiet kārtulas tieši vai norādiet uz stila lapas failu:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// vai
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Kontrole katram ziņojumam:

```php
$message->inlineCss();          // piespiest iekļaušanu šim vienam ziņojumam
$message->withoutInlineCss();   // izlaist to pat tad, kad globāli ieslēgts
```

### Teksta daļas ģenerēšana no HTML

Labākā prakse ir sūtīt HTML un vienkāršā teksta versiju kopā, bet rakstīt abas ir nogurdinoši. FlightMail var automātiski iegūt teksta daļu no galīgā HTML - pamata konversijai nav vajadzīga papildu atkarība, jo pārveidotājs nāk līdzi ar Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown, kad iespējams, citādi vienkāršs teksts
]);
```

Režīmi:

- `true` vai `'auto'` - Markdown izvade, ja `league/html-to-markdown` ir instalēts, citādi vienkārša tagu noņemšana.
- `'markdown'` - piespiest Markdown (`composer require league/html-to-markdown`; virsraksti kļūst par `==`, saites `[text](url)`, treknraksts `**bold**`).
- `'plain'` - vienmēr noņemt tagus; darbojas bez papildu paketēm.

Ģenerēšana notiek pēc renderēšanas un CSS iekļaušanas, un tikai tad, kad ziņojumam ir HTML saturs, bet nav teksta satura - eksplicīts `->text()` vai `->textTemplate()` vienmēr uzvar. Pārrakstījumi katram ziņojumam atspoguļo iekļaušanu:

```php
$message->textFromHtml('plain');    // piespiest tagu noņemšanu šim vienam
$message->withoutTextFromHtml();    // tikai HTML e-pasts
```

Ieslēdziet režīmu, kura bibliotēka nav instalēta, un saņemat skaidru kļūdu, kas nosauc precīzo `composer require`, kas jāizpilda - nekad klusa degradācija.

## Pakalpojumu sniedzēja izvēle

Pakalpojumu sniedzēji pievienojas caur DSN virknēm. Instalējiet tilta pakotni, ielīmējiet DSN `dsns`, gatavs.

| Pakalpojumu sniedzējs | Instalēšana                                  | DSN piemērs                                  |
| --------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                  | iebūvēts                                     | `smtp://user:pass@host:587`                  |
| Sendmail              | iebūvēts                                     | `sendmail://default`                         |
| Dev/null (atmest pastu) | iebūvēts                                   | `null://null`                                |
| Postmark              | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid              | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun               | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES            | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                 | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend            | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

Pilnais saraksts ir [Symfony Mailer dokumentācijā](https://symfony.com/doc/current/mailer.html) - jebkas, kas tur dokumentēts, šeit darbojas nemainīti.

### Vairāki pakalpojumu sniedzēji vienlaikus

Nosauciet katru transportu, tad izvēlieties katram ziņojumam:

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
// Nav ->transport() izsaukuma = pirmā atslēga "dsns" ("transactional" šeit).
Flight::mail()->compose()->to('...')->text('kvīts')->send();

// Eksplicīti izvēlieties citu maršrutu.
Flight::mail()->compose()->to('...')->text('biļetens')->transport('bulk')->send();
```

## Konfigurācijas atsauce

Viss ir neobligāts, izņemot `dsns`.

```php
MailPlugin::install([
    // OBLIGĀTS - transporta nosaukums => Symfony DSN.
    // Pirmais ieraksts tiek izmantots, kad ziņojums nenosauc nevienu.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transports, ko izmanto, kad ziņojumam nav eksplicīta ->transport() un
    // jūs nevēlaties pirmo atslēgu. Jābūt "dsns".
    'default_transport' => 'default',

    // Globālais sūtītājs. Virkne, Symfony Address vai ['email' => 'Name'].
    // Tiek piemērots tikai tad, kad ziņojums nenosaka savu ->from().
    'from' => ['no-reply@example.com' => 'Mana lietotne'],

    // Noklusējuma veidņu dzinējs: 'twig', 'latte' vai pielāgots nosaukums.
    // Konsultējas tikai veidnēm, kuru paplašinājums nav reģistrēts renderētājs.
    'renderer' => 'twig',

    // Kur dzīvo veidnes, meklētas pēc kārtas; plus neobligāts kešatmiņas katalogs.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Papildu opcijas, kas tiek padotas tieši Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Pielāgojiet Latte dzinēju palaišanas laikā: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Sūtīšanas brīža satura uzlabojumi (skatiet "HTML stilizēšana un teksta daļu ģenerēšana").
    'inline_css' => true,           // vai ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // vai 'plain' / 'markdown'

    // Pielāgotas DSN shēmas, pielāgoti renderētāji, pirms-sūtīšanas āķi (skatiet zemāk).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Neobligāta infrastruktūra, kas tiek nodota katram transportam.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Iet tālāk

Viss zemāk ir neobligāts. Noklusējumi nosedz lielāko daļu lietotņu.

### Pievienot pielāgotu DSN shēmu

Implementējiet Symfony `TransportFactoryInterface` un reģistrējiet to - tad jūsu pašu shēma darbojas tieši kā iebūvēta:

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
        // ... izveidojiet transportu, kas runā ar jūsu pārvadātāju
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Pievienot pielāgotu veidņu renderētāju

Jebkas, kas pārvērš veidnes nosaukumu plus parametrus virknē, kvalificējas:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Veidnes, kas beidzas ar .markdown, tagad to izmanto automātiski:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Palaist kaut ko tieši pirms sūtīšanas

Āķi saņem gatavo ziņojumu - pēc renderēšanas, pēc noklusējumiem, tieši pirms vada:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Notikumi un žurnalēšana

Nododiet Symfony notikumu dispečeru un/vai PSR-3 žurnalētāju, un katrs transports tos izmantos:

```php
$plugin->eventDispatcher($dispatcher); // saņem MessageEvent pirms katras sūtīšanas
$plugin->logger($logger);              // transporta līmeņa žurnāli
```

## API špikeris

```php
// Iestatīšana
MailPlugin::install($config)             // reģistrēt globālajā Flight lietotnē
MailPlugin::register($app, $config)      // reģistrēt konkrētā Engine
$mailer = Flight::mail();                // koplietotā Mailer instance

// Ziņojumu veidošana
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // standarta Symfony Mime metodes
$message->text(string)                       // vienkāršas virknes saturs
$message->html(string)                       // HTML virknes saturs
$message->template($name, $params)           // HTML saturs no veidnes
$message->htmlTemplate($name, $params)       // template() aizstājvārds
$message->textTemplate($name, $params)       // teksta saturs no veidnes
$message->inlineCss() / ->withoutInlineCss() // CSS iekļaušana katram ziņojumam
$message->textFromHtml($mode)                // automātiska teksta daļa: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // tikai HTML e-pasts
$message->transport($name)                   // maršrutēt caur nosauktu DSN
$message->send(): ?SentMessage               // renderēt + sūtīt

// Uz paša mailer
$mailer->send($message): ?SentMessage        // eksplicīta alternatīva $message->send()
$mailer->render($template, $params): string  // renderēt bez sūtīšanas
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Tā kā `Message` paplašina `Symfony\Component\Mime\Email`, katra Symfony metode, ko jau pazīstat - `attach()`, `embed()`, `priority()`, `replyTo()` - darbojas uzreiz.

## Problēmu novēršana

**"No mail DSNs configured"**
Jūs izsaucāt `Flight::mail()` pirms spraudņa reģistrēšanas, vai konfigurācijas masīvs neiekļāva `dsns`. Šī kļūda ir apzināta - FlightMail atsakās minēt, kur jūsu pastam jādodas, nevis to klusībā nomest.

**"Unknown mail template renderer ..."**
Jūs izmantojāt veidni, kuras dzinējs nav instalēts. Labojiet ar `composer require twig/twig` vai `composer require latte/latte`, vai reģistrējiet pielāgotu renderētāju, kas nosaukts pēc paplašinājuma.

**"Unknown mail transport ..."**
`->transport('name')` (vai `default_transport`) nesakrīt ar nevienu atslēgu `dsns`. Pārbaudiet pareizrakstību - kļūda uzskaita konfigurētos nosaukumus.

**E-pasts neienāk**
Vērsiet `dsns` uz `null://null`, lai apstiprinātu, ka pārējais kods darbojas, tad pārslēdzieties atpakaļ uz īsto DSN. DDEV izmantojiet `smtp://127.0.0.1:1025` un pārbaudiet ziņojumus Mailpit 8025 portā.

---

Kļūdu ziņojumiem, pull requestiem un pilnam avotam apmeklējiet [GitHub krātuvi](https://github.com/ryanstubbs/flightmail).
