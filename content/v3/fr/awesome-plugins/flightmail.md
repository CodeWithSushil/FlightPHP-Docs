# FlightMail

> **Plugin tiers** - maintenu par [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), licence MIT). Ne fait pas partie du cœur de Flight - merci de signaler les problèmes sur [son dépôt GitHub](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) vous permet d'envoyer des e-mails depuis votre app Flight sans les maux de tête. Il encapsule **Symfony Mailer** - la bibliothèque de mail la plus éprouvée en PHP - et le fait sentir comme faisant partie de Flight. Une ligne pour installer, une chaîne fluide pour envoyer :

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Vous l\'avez fait !')
    ->text('Votre premier e-mail est en route.')
    ->send();
```

## Fonctionnalités

- **N'importe quel fournisseur, une ligne chacun.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo et les autres fonctionnent tous via de simples chaînes DSN.
- **Utilisez plusieurs fournisseurs à la fois.** Mail transactionnel via Postmark, newsletters via votre propre SMTP - choisissez par message.
- **Des templates si vous en voulez.** Rendez les corps avec Twig ou Latte. Vous ne voulez pas de templates ? Passez simplement des chaînes et n'installez rien de plus.
- **Du polish à l'envoi.** Inlining CSS optionnel et parties texte brut automatiques dérivées de votre HTML, propulsés par des bibliothèques que vous n'installez que si vous les utilisez.
- **Ennuyeux, de la meilleure façon.** Connexions paresseuses, erreurs claires au lieu de mails avalés en silence, et tout est interchangeable si vous avez besoin de quelque chose de personnalisé.

## Prérequis

| Quoi           | Version                                      |
| -------------- | -------------------------------------------- |
| PHP            | 8.2 ou plus récent                           |
| Flight PHP     | core ^3.15                                   |
| Symfony Mailer | ^7.2 ou ^8.0 (installé automatiquement)      |

## Installation

```bash
composer require ryanstubbs/flightmail
```

C'est tout pour envoyer des e-mails en texte brut et HTML. Le rendu de templates est optionnel - ajoutez un moteur seulement si vous l'utiliserez :

```bash
composer require twig/twig      # pour les templates .twig
composer require latte/latte    # pour les templates .latte
```

Deux bibliothèques optionnelles de plus propulsent les améliorations à l'envoi couvertes [plus bas](#mise-en-forme-html-et-generation-des-parties-texte) :

```bash
composer require pelago/emogrifier         # pour l'inlining CSS ("inline_css")
composer require league/html-to-markdown   # pour les parties texte Markdown ("text_from_html")
```

Toutes peuvent être installées côte à côte ; FlightMail choisit la bonne selon ce que vous configurez.

## Votre premier e-mail

Ajoutez ceci à votre bootstrap (le même endroit où vous définissez les routes) :

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Indiquez à FlightMail d'où et par où envoyer le mail.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Bienvenue à bord !')
        ->html('<h1>Bienvenue !</h1><p>Nous sommes ravis que vous soyez là.</p>')
        ->send();
});

Flight::start();
```

Vous utilisez le [squelette Flight PHP](https://github.com/flightphp/skeleton) ? Enregistrez-le dans `app/config/services.php` avec le style instance à la place :

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Les deux styles exposent le même mailer : `Flight::mail()` et `$app->mail()` sont interchangeables.

> **Vous testez en local ?** Si votre projet tourne dans [DDEV](https://ddev.com), pointez le DSN vers `smtp://127.0.0.1:1025` et lisez chaque e-mail capturé dans Mailpit à `http://<project>.ddev.site:8025`. Rien ne quitte votre machine.

## Envoyer des e-mails

### Chaînes simples (aucun moteur de templates nécessaire)

`->text()` et `->html()` prennent des chaînes brutes et n'ont besoin de rien d'autre d'installé :

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Sauvegarde terminée')
    ->text('La sauvegarde nocturne s\'est terminée en 42 minutes.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Facture #123')
    ->html('<h1>Facture #123</h1><p>Total dû : $42.00</p>')
    ->send();
```

### Templates Twig

```php
// welcome.html.twig contient : Bonjour {{ name }}, merci de vous être inscrit !
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bienvenue !')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Templates Latte

Même idée, extension `.latte` :

```php
// welcome.latte contient : Bonjour {$name}, merci de vous être inscrit !
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bienvenue !')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + texte brut ensemble

Bonne pratique pour la délivrabilité - donnez aux clients mail les deux versions :

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bienvenue !')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // version enrichie
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // version de repli
    ->send();
```

Quelques points à connaître sur les templates :

- Ils sont rendus de façon **paresseuse**, au moment de l'envoi - composez maintenant, rendez plus tard.
- Le moteur est choisi par l'extension : `.twig` → Twig, `.latte` → Latte, tout le reste → votre défaut configuré (option `renderer`).
- Un corps explicite `->html()` ou `->text()` gagne toujours sur un template, donc vous pouvez définir un template par défaut et le surcharger par message.

## Mise en forme HTML et génération des parties texte

Deux améliorations optionnelles à l'envoi, toutes deux désactivées par défaut et toutes deux propulsées par des bibliothèques que vous n'installez que si vous les voulez :

| Fonctionnalité            | Installer                  | Clé de configuration |
| ------------------------- | -------------------------- | -------------------- |
| CSS en ligne              | `pelago/emogrifier`        | `inline_css`         |
| Partie texte depuis HTML  | `league/html-to-markdown`  | `text_from_html`     |

### Intégrer le CSS en ligne dans votre e-mail HTML

Gmail et la plupart des clients webmail suppriment les blocs `<style>` - les attributs `style=""` en ligne sont le seul style qu'ils honorent de façon fiable. Les écrire à la main est misérable ; laissez [Emogrifier](https://github.com/MyIntervals/emogrifier) le faire à l'envoi :

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Une fois activé, chaque corps HTML voit son CSS intégré en ligne juste avant l'envoi - qu'il vienne d'un template ou de `->html()`. Un message comme `<style>p { color: red; }</style><p>Salut</p>` part comme `<p style="color: red;">Salut</p>`.

Pour injecter des styles partagés dans chaque e-mail (couleurs de marque, resets) sans les répéter dans chaque template, passez les règles directement ou pointez vers un fichier de feuille de styles :

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// ou
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Contrôle par message :

```php
$message->inlineCss();          // forcer l'inlining pour ce message
$message->withoutInlineCss();   // le sauter même lorsqu'il est activé globalement
```

### Générer la partie texte à partir de votre HTML

La bonne pratique est d'envoyer une version HTML et une version texte brut ensemble, mais écrire les deux est fastidieux. FlightMail peut dériver la partie texte du HTML final automatiquement - la conversion de base n'a besoin d'aucune dépendance extra, puisque le convertisseur est fourni avec Symfony Mime :

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown quand c'est possible, brut sinon
]);
```

Modes :

- `true` ou `'auto'` - sortie Markdown si `league/html-to-markdown` est installé, sinon un simple strip des balises.
- `'markdown'` - forcer Markdown (`composer require league/html-to-markdown` ; les titres deviennent `==`, les liens `[text](url)`, le gras `**bold**`).
- `'plain'` - toujours stripper les balises ; fonctionne avec zéro paquet extra.

La génération s'exécute après le rendu et l'inlining CSS, et seulement lorsque le message a un corps HTML mais pas de corps texte - un `->text()` ou `->textTemplate()` explicite gagne toujours. Les surcharges par message reflètent l'inlining :

```php
$message->textFromHtml('plain');    // forcer le strip des balises pour celui-ci
$message->withoutTextFromHtml();    // e-mail HTML uniquement
```

Activez un mode dont la bibliothèque n'est pas installée et vous obtenez une erreur claire nommant le `composer require` exact à lancer - jamais de dégradation silencieuse.

## Choisir un fournisseur

Les fournisseurs se branchent via des chaînes DSN. Installez le paquet pont, collez le DSN dans `dsns`, terminé.

| Fournisseur          | Installer                                    | Exemple de DSN                               |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | intégré                                      | `smtp://user:pass@host:587`                  |
| Sendmail             | intégré                                      | `sendmail://default`                         |
| Dev/null (jeter le mail) | intégré                                  | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

La liste complète se trouve dans la [documentation de Symfony Mailer](https://symfony.com/doc/current/mailer.html) - tout ce qui y est documenté fonctionne ici sans changement.

### Plusieurs fournisseurs à la fois

Nommez chaque transport, puis choisissez par message :

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
// Pas d'appel ->transport() = première clé dans "dsns" ("transactional" ici).
Flight::mail()->compose()->to('...')->text('reçu')->send();

// Choisissez une autre route explicitement.
Flight::mail()->compose()->to('...')->text('newsletter')->transport('bulk')->send();
```

## Référence de configuration

Tout est optionnel sauf `dsns`.

```php
MailPlugin::install([
    // OBLIGATOIRE - nom de transport => Symfony DSN.
    // La première entrée est utilisée lorsqu'un message n'en nomme pas.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transport utilisé lorsqu'un message n'a pas de ->transport() explicite et
    // que vous ne voulez pas la première clé. Doit exister dans "dsns".
    'default_transport' => 'default',

    // Expéditeur global. String, Symfony Address, ou ['email' => 'Name'].
    // Appliqué seulement lorsqu'un message ne définit pas son propre ->from().
    'from' => ['no-reply@example.com' => 'Mon App'],

    // Moteur de templates par défaut : 'twig', 'latte', ou un nom personnalisé.
    // Consulté seulement pour les templates dont l'extension n'est pas un renderer enregistré.
    'renderer' => 'twig',

    // Où vivent les templates, cherchés dans l'ordre ; plus un dir de cache optionnel.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Options extra passées directement à Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Ajustez le moteur Latte au boot : fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Améliorations du corps à l'envoi (voir "Mise en forme HTML et génération des parties texte").
    'inline_css' => true,           // ou ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // ou 'plain' / 'markdown'

    // Schémas DSN personnalisés, renderers personnalisés, hooks pré-envoi (voir plus bas).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Plomberie optionnelle transmise à chaque transport.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Aller plus loin

Tout ce qui suit est optionnel. Les défauts couvrent la plupart des apps.

### Ajouter un schéma DSN personnalisé

Implémentez `TransportFactoryInterface` de Symfony et enregistrez-le - alors votre propre schéma fonctionne exactement comme un intégré :

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
        // ... construisez un transport qui parle à votre opérateur
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Ajouter un moteur de templates personnalisé

Tout ce qui transforme un nom de template plus des params en une chaîne convient :

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Les templates se terminant par .markdown l'utilisent maintenant automatiquement :
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Exécuter quelque chose juste avant l'envoi

Les hooks reçoivent le message terminé - après le rendu, après les défauts, juste avant le fil :

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Événements et journalisation

Passez un event dispatcher Symfony et/ou un logger PSR-3 et chaque transport les utilisera :

```php
$plugin->eventDispatcher($dispatcher); // reçoit MessageEvent avant chaque envoi
$plugin->logger($logger);              // logs au niveau transport
```

## Aide-mémoire de l'API

```php
// Configuration
MailPlugin::install($config)             // enregistrer sur l'app Flight globale
MailPlugin::register($app, $config)      // enregistrer sur un Engine spécifique
$mailer = Flight::mail();                // l'instance Mailer partagée

// Construire des messages
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // méthodes Symfony Mime standard
$message->text(string)                       // corps en chaîne texte brut
$message->html(string)                       // corps en chaîne HTML
$message->template($name, $params)           // corps HTML depuis un template
$message->htmlTemplate($name, $params)       // alias de template()
$message->textTemplate($name, $params)       // corps texte depuis un template
$message->inlineCss() / ->withoutInlineCss() // CSS en ligne par message
$message->textFromHtml($mode)                // partie texte auto : true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // e-mail HTML uniquement
$message->transport($name)                   // router via un DSN nommé
$message->send(): ?SentMessage               // rendu + envoi

// Sur le mailer lui-même
$mailer->send($message): ?SentMessage        // alternative explicite à $message->send()
$mailer->render($template, $params): string  // rendre sans envoyer
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Puisque `Message` étend `Symfony\Component\Mime\Email`, chaque méthode Symfony que vous connaissez déjà - `attach()`, `embed()`, `priority()`, `replyTo()` - fonctionne d'emblée.

## Dépannage

**"No mail DSNs configured"**
Vous avez appelé `Flight::mail()` avant d'enregistrer le plugin, ou le tableau de config n'incluait pas `dsns`. Cette erreur est délibérée - FlightMail refuse de deviner où votre mail doit aller plutôt que de le jeter en silence.

**"Unknown mail template renderer ..."**
Vous avez utilisé un template dont le moteur n'est pas installé. Corrigez avec `composer require twig/twig` ou `composer require latte/latte`, ou enregistrez un renderer personnalisé nommé d'après l'extension.

**"Unknown mail transport ..."**
Un `->transport('name')` (ou `default_transport`) ne correspond à aucune clé dans `dsns`. Vérifiez l'orthographe - l'erreur liste les noms configurés.

**Le mail n'arrive pas**
Pointez `dsns` vers `null://null` pour confirmer que le reste de votre code fonctionne, puis revenez au vrai DSN. Dans DDEV, utilisez `smtp://127.0.0.1:1025` et inspectez les messages dans Mailpit au port 8025.

---

Pour les rapports de bugs, les pull requests et le code source complet, visitez le [dépôt GitHub](https://github.com/ryanstubbs/flightmail).
