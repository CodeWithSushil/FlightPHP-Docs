# FlightMail

> **Drittanbieter-Plugin** - gepflegt von [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), MIT-lizenziert). Nicht Teil von Flight Core - bitte melden Sie Probleme im [GitHub-Repository](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) lässt Sie E-Mails aus Ihrer Flight-App versenden, ohne die Kopfschmerzen. Es umhüllt **Symfony Mailer** - die am härtesten erprobte Mail-Bibliothek in PHP - und lässt sie sich anfühlen wie ein Teil von Flight. Eine Zeile zum Installieren, eine fließende Kette zum Senden:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Geschafft!')
    ->text('Ihre erste E-Mail ist unterwegs.')
    ->send();
```

## Funktionen

- **Jeder Anbieter, jeweils eine Zeile.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo und Co. funktionieren alle über einfache DSN-Strings.
- **Mehrere Anbieter gleichzeitig nutzen.** Transaktionsmails über Postmark, Newsletter über Ihr eigenes SMTP - pro Nachricht wählen.
- **Templates, wenn Sie sie wollen.** Bodies mit Twig oder Latte rendern. Keine Templates? Einfach Strings übergeben und nichts Extra installieren.
- **Feinschliff beim Senden.** Optionales CSS-Inlining und automatische Klartext-Teile, die aus Ihrem HTML abgeleitet werden, angetrieben von Bibliotheken, die Sie nur installieren, wenn Sie sie nutzen.
- **Langweilig auf die beste Art.** Lazy Connections, klare Fehler statt stillschweigend verschluckter Mails, und alles ist austauschbar, wenn Sie etwas Eigenes brauchen.

## Anforderungen

| Was            | Version                                      |
| -------------- | -------------------------------------------- |
| PHP            | 8.2 oder neuer                               |
| Flight PHP     | core ^3.15                                   |
| Symfony Mailer | ^7.2 oder ^8.0 (wird automatisch installiert) |

## Installation

```bash
composer require ryanstubbs/flightmail
```

Das war's für das Senden von Klartext- und HTML-E-Mails. Template-Rendering ist optional - fügen Sie eine Engine nur hinzu, wenn Sie sie nutzen:

```bash
composer require twig/twig      # für .twig-Templates
composer require latte/latte    # für .latte-Templates
```

Zwei weitere optionale Bibliotheken treiben die Verbesserungen beim Senden an, die [weiter unten](#html-stylen-und-textteile-erzeugen) beschrieben werden:

```bash
composer require pelago/emogrifier         # für CSS-Inlining ("inline_css")
composer require league/html-to-markdown   # für Markdown-Textteile ("text_from_html")
```

Alle davon können nebeneinander installiert werden; FlightMail wählt anhand Ihrer Konfiguration die richtige aus.

## Ihre erste E-Mail

Fügen Sie das in Ihren Bootstrap ein (dieselbe Stelle, an der Sie Routen definieren):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// FlightMail sagen, von wo und worüber Mail gesendet werden soll.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Willkommen an Bord!')
        ->html('<h1>Willkommen!</h1><p>Wir freuen uns, dass Sie da sind.</p>')
        ->send();
});

Flight::start();
```

Nutzen Sie das [Flight PHP Skeleton](https://github.com/flightphp/skeleton)? Registrieren Sie es in `app/config/services.php` im Instanz-Stil:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Beide Stile stellen denselben Mailer bereit: `Flight::mail()` und `$app->mail()` sind austauschbar.

> **Lokal testen?** Wenn Ihr Projekt in [DDEV](https://ddev.com) läuft, zeigen Sie den DSN auf `smtp://127.0.0.1:1025` und lesen Sie jede erfasste E-Mail in Mailpit unter `http://<project>.ddev.site:8025`. Nichts verlässt Ihren Rechner.

## E-Mails senden

### Einfache Strings (keine Template-Engine nötig)

`->text()` und `->html()` nehmen Roh-Strings und brauchen sonst nichts Installiertes:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Backup abgeschlossen')
    ->text('Nächtliches Backup in 42 Minuten abgeschlossen.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Rechnung #123')
    ->html('<h1>Rechnung #123</h1><p>Gesamtbetrag: $42.00</p>')
    ->send();
```

### Twig-Templates

```php
// welcome.html.twig enthält: Hallo {{ name }}, danke für die Anmeldung!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Willkommen!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Latte-Templates

Dieselbe Idee, `.latte`-Erweiterung:

```php
// welcome.latte enthält: Hallo {$name}, danke für die Anmeldung!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Willkommen!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + Klartext zusammen

Best Practice für die Zustellbarkeit - geben Sie Mail-Clients beide Versionen:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Willkommen!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // reichhaltige Version
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // Fallback-Version
    ->send();
```

Ein paar Dinge, die sich bei Templates zu wissen lohnen:

- Sie werden **lazy** gerendert, zum Sendezeitpunkt - jetzt zusammenbauen, später rendern.
- Die Engine wird anhand der Erweiterung gewählt: `.twig` → Twig, `.latte` → Latte, alles andere → Ihr konfigurierter Standard (`renderer`-Option).
- Ein expliziter `->html()`- oder `->text()`-Body gewinnt immer gegenüber einem Template, sodass Sie ein Standard-Template setzen und es pro Nachricht überschreiben können.

## HTML stylen und Textteile erzeugen

Zwei optionale Verbesserungen beim Senden, beide standardmäßig aus und beide angetrieben von Bibliotheken, die Sie nur installieren, wenn Sie sie wollen:

| Funktion            | Installation              | Config-Schlüssel |
| ------------------- | ------------------------- | ---------------- |
| CSS-Inlining        | `pelago/emogrifier`       | `inline_css`     |
| Textteil aus HTML   | `league/html-to-markdown` | `text_from_html` |

### CSS in HTML-E-Mails inlinen

Gmail und die meisten Webmail-Clients entfernen `<style>`-Blöcke - inline `style=""`-Attribute sind das einzige Styling, das sie zuverlässig ehren. Die von Hand zu schreiben ist miserabel; lassen Sie [Emogrifier](https://github.com/MyIntervals/emogrifier) das zum Sendezeitpunkt erledigen:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Wenn das an ist, bekommt jeder HTML-Body sein CSS direkt vor dem Senden inline gesetzt - egal ob er aus einem Template oder `->html()` kam. Eine Nachricht wie `<style>p { color: red; }</style><p>Hallo</p>` geht als `<p style="color: red;">Hallo</p>` raus.

Um gemeinsame Styles in jede E-Mail einzuspritzen (Markenfarben, Resets), ohne sie in jedem Template zu wiederholen, übergeben Sie Regeln direkt oder zeigen Sie auf eine Stylesheet-Datei:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// oder
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Steuerung pro Nachricht:

```php
$message->inlineCss();          // Inlining für diese eine Nachricht erzwingen
$message->withoutInlineCss();   // überspringen, selbst wenn global aktiviert
```

### Den Textteil aus Ihrem HTML erzeugen

Best Practice ist, eine HTML- und eine Klartext-Version zusammen zu senden, aber beides zu schreiben ist mühsam. FlightMail kann den Textteil automatisch aus dem finalen HTML ableiten - die Basis-Konvertierung braucht keine Extra-Abhängigkeit, da der Converter mit Symfony Mime mitgeliefert wird:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown wenn möglich, sonst Klartext
]);
```

Modi:

- `true` oder `'auto'` - Markdown-Ausgabe, wenn `league/html-to-markdown` installiert ist, sonst einfaches Entfernen der Tags.
- `'markdown'` - Markdown erzwingen (`composer require league/html-to-markdown`; Überschriften werden zu `==`, Links `[text](url)`, fett `**bold**`).
- `'plain'` - immer Tags entfernen; funktioniert ohne Extra-Pakete.

Die Erzeugung läuft nach dem Rendern und CSS-Inlining und nur, wenn die Nachricht einen HTML-Body, aber keinen Text-Body hat - ein explizites `->text()` oder `->textTemplate()` gewinnt immer. Überschreibungen pro Nachricht spiegeln das Inlining:

```php
$message->textFromHtml('plain');    // Tag-Stripping für diese eine erzwingen
$message->withoutTextFromHtml();    // nur-HTML-E-Mail
```

Aktivieren Sie einen Modus, dessen Bibliothek nicht installiert ist, und Sie bekommen einen klaren Fehler, der das genaue `composer require` nennt, das Sie ausführen müssen - niemals stiller Qualitätsverlust.

## Einen Anbieter wählen

Anbieter werden über DSN-Strings eingebunden. Installieren Sie das Bridge-Paket, fügen Sie den DSN in `dsns` ein, fertig.

| Anbieter             | Installation                                 | DSN-Beispiel                                 |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | eingebaut                                    | `smtp://user:pass@host:587`                  |
| Sendmail             | eingebaut                                    | `sendmail://default`                         |
| Dev/null (Mails verwerfen) | eingebaut                              | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

Die vollständige Liste steht in den [Symfony Mailer-Docs](https://symfony.com/doc/current/mailer.html) - alles, was dort dokumentiert ist, funktioniert hier unverändert.

### Mehrere Anbieter gleichzeitig

Benennen Sie jeden Transport und wählen Sie dann pro Nachricht:

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
// Kein ->transport()-Aufruf = erster Schlüssel in "dsns" ("transactional" hier).
Flight::mail()->compose()->to('...')->text('Beleg')->send();

// Explizit eine andere Route wählen.
Flight::mail()->compose()->to('...')->text('Newsletter')->transport('bulk')->send();
```

## Konfigurationsreferenz

Alles ist optional außer `dsns`.

```php
MailPlugin::install([
    // ERFORDERLICH - Transportname => Symfony DSN.
    // Der erste Eintrag wird verwendet, wenn eine Nachricht keinen nennt.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transport, der verwendet wird, wenn eine Nachricht kein explizites
    // ->transport() hat und Sie nicht den ersten Schlüssel wollen. Muss in "dsns" existieren.
    'default_transport' => 'default',

    // Globaler Absender. String, Symfony Address oder ['email' => 'Name'].
    // Wird nur angewendet, wenn eine Nachricht kein eigenes ->from() setzt.
    'from' => ['no-reply@example.com' => 'Meine App'],

    // Standard-Template-Engine: 'twig', 'latte' oder ein eigener Name.
    // Wird nur für Templates konsultiert, deren Erweiterung kein registrierter Renderer ist.
    'renderer' => 'twig',

    // Wo Templates liegen, der Reihe nach durchsucht; plus ein optionales Cache-Verzeichnis.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Extra-Optionen, die direkt an Twig\Environment übergeben werden.
    'twig' => ['options' => ['strict_variables' => true]],

    // Die Latte-Engine beim Boot anpassen: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Body-Verbesserungen beim Senden (siehe "HTML stylen und Textteile erzeugen").
    'inline_css' => true,           // oder ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // oder 'plain' / 'markdown'

    // Eigene DSN-Schemata, eigene Renderer, Pre-Send-Hooks (siehe unten).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Optionale Infrastruktur, die jedem Transport übergeben wird.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Noch weiter gehen

Alles darunter ist optional. Die Defaults decken die meisten Apps ab.

### Ein eigenes DSN-Schema hinzufügen

Implementieren Sie Symfonys `TransportFactoryInterface` und registrieren Sie es - dann funktioniert Ihr eigenes Schema genau wie ein eingebautes:

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
        // ... einen Transport bauen, der mit Ihrem Carrier spricht
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Einen eigenen Template-Renderer hinzufügen

Alles, das einen Template-Namen plus Parameter in einen String verwandelt, qualifiziert sich:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Templates, die auf .markdown enden, nutzen ihn jetzt automatisch:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Etwas direkt vor dem Senden ausführen

Hooks erhalten die fertige Nachricht - nach dem Rendern, nach den Defaults, direkt vor dem Draht:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Events und Logging

Übergeben Sie einen Symfony Event Dispatcher und/oder PSR-3 Logger und jeder Transport wird sie nutzen:

```php
$plugin->eventDispatcher($dispatcher); // empfängt MessageEvent vor jedem Senden
$plugin->logger($logger);              // Logs auf Transport-Ebene
```

## API-Spickzettel

```php
// Einrichtung
MailPlugin::install($config)             // auf der globalen Flight-App registrieren
MailPlugin::register($app, $config)      // auf einer bestimmten Engine registrieren
$mailer = Flight::mail();                // die gemeinsame Mailer-Instanz

// Nachrichten bauen
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // Standard-Symfony-Mime-Methoden
$message->text(string)                       // Klartext-String-Body
$message->html(string)                       // HTML-String-Body
$message->template($name, $params)           // HTML-Body aus einem Template
$message->htmlTemplate($name, $params)       // Alias von template()
$message->textTemplate($name, $params)       // Text-Body aus einem Template
$message->inlineCss() / ->withoutInlineCss() // CSS-Inlining pro Nachricht
$message->textFromHtml($mode)                // Auto-Textteil: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // nur-HTML-E-Mail
$message->transport($name)                   // über einen benannten DSN leiten
$message->send(): ?SentMessage               // rendern + senden

// Am Mailer selbst
$mailer->send($message): ?SentMessage        // explizite Alternative zu $message->send()
$mailer->render($template, $params): string  // rendern ohne zu senden
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Da `Message` `Symfony\Component\Mime\Email` erweitert, funktioniert jede Symfony-Methode, die Sie schon kennen - `attach()`, `embed()`, `priority()`, `replyTo()` - sofort.

## Fehlerbehebung

**"No mail DSNs configured"**
Sie haben `Flight::mail()` aufgerufen, bevor das Plugin registriert war, oder das Config-Array enthielt kein `dsns`. Dieser Fehler ist Absicht - FlightMail weigert sich zu raten, wohin Ihre Mail soll, statt sie stillschweigend zu verwerfen.

**"Unknown mail template renderer ..."**
Sie haben ein Template verwendet, dessen Engine nicht installiert ist. Beheben Sie das mit `composer require twig/twig` oder `composer require latte/latte`, oder registrieren Sie einen eigenen Renderer, der nach der Erweiterung benannt ist.

**"Unknown mail transport ..."**
Ein `->transport('name')` (oder `default_transport`) passt zu keinem Schlüssel in `dsns`. Prüfen Sie die Schreibweise - der Fehler listet die konfigurierten Namen.

**E-Mail kommt nicht an**
Zeigen Sie `dsns` auf `null://null`, um zu bestätigen, dass der Rest Ihres Codes funktioniert, und wechseln Sie dann zurück zum echten DSN. In DDEV nutzen Sie `smtp://127.0.0.1:1025` und prüfen Sie Nachrichten in Mailpit auf Port 8025.

---

Für Fehlerberichte, Pull Requests und den vollständigen Quellcode besuchen Sie das [GitHub-Repository](https://github.com/ryanstubbs/flightmail).
