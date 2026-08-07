# HTML-Ansichten und Vorlagen

## Überblick

Flight bietet standardmäßig eine grundlegende HTML-Templating-Funktionalität. Templating ist eine sehr effektive Methode, um deine Anwendungslogik von der Darstellungsebene zu entkoppeln. Eine dedizierte Engine (Twig, Latte usw.) gibt [KI-Programmierwerkzeugen](/learn/ai) außerdem eine vertraute, eingeschränkte Syntax, sodass sie weniger wahrscheinlich Geschäftslogik in dein HTML einfügen.

## Verständnis

Wenn du eine Anwendung entwickelst, wirst du wahrscheinlich HTML haben, das du an den Endbenutzer ausliefern möchtest. PHP selbst ist eine Templating-Sprache, aber es ist _sehr_ einfach, Geschäftslogik wie Datenbankaufrufe, API-Aufrufe usw. in deine HTML-Datei zu packen und das Testen und Entkoppeln zu einem sehr schwierigen Prozess zu machen. Indem du Daten in eine Vorlage schiebst und die Vorlage sich selbst rendern lässt, wird es viel einfacher, deinen Code zu entkoppeln und Unit-Tests zu unterziehen. Du wirst uns danken, wenn du Vorlagen verwendest!

## Grundlegende Verwendung

Flight ermöglicht es dir, die Standard-View-Engine auszutauschen, indem du einfach `render` zuordnest (oder eine View-Klasse registrierst). Scrolle nach unten für Twig, Latte, Smarty, Blade und mehr.

> **Skeleton-Standard:** Das offizielle [flightphp/skeleton](https://github.com/flightphp/skeleton) verwendet **nur Twig** unter `app/views/` (`*.twig`). Controller rufen `$this->app->render('welcome', $data)` auf (Endung optional). Das ist eine Anwendungsentscheidung für neue Projekte – keine Anforderung des Flight-Kerns. Latte und andere Engines werden weiterhin vollständig unterstützt.

### Twig

<span class="badge bg-info">Skeleton-Standard</span>

[Twig](https://twig.symfony.com/) ist eine flexible, schnelle und sichere Template-Engine, die von Symfony und vielen anderen PHP-Projekten verwendet wird. KI-Programmierwerkzeuge kennen Twig tendenziell besonders gut, und es maskiert Ausgaben standardmäßig automatisch, was vor XSS schützt.

#### Installation

```bash
composer require twig/twig
```

(Bereits enthalten, wenn du `composer create-project flightphp/skeleton` ausführst.)

#### Grundlegende Konfiguration

Überschreibe die `render`-Methode, um Twig anstelle des standardmäßigen PHP-Renderers zu verwenden:

```php
// Überschreibe die render-Methode, um Twig anstelle des standardmäßigen PHP-Renderers zu verwenden
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Wo Twig seine kompilierten Vorlagen speichert
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Erlaubt "welcome" oder "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

Im Skeleton befindet sich diese Verdrahtung in `app/config/services.php` (gemeinsame Twig-Umgebung, Cache-Pfad, globale Variablen wie `base_url` / CSP-Nonce). Bevorzuge es, `Engine` zu injizieren und `$app->render()` aus Controllern aufzurufen, damit der Code [KI- und testfreundlich](/learn/ai) bleibt.

#### Twig in Flight verwenden

Jetzt, da du mit Twig rendern kannst, kannst du Folgendes tun:

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}Meine App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hallo, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Homepage',
		'name' => $name
	]);
});
```

Wenn du in deinem Browser `/Bob` aufrufst, wäre die Ausgabe:

```html
<html>
  <head>
	<title>Homepage - Meine App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hallo, Bob!</h1>
  </body>
</html>
```

#### Weiterführende Informationen

Ein vollständigeres Beispiel für die Verwendung von Twig mit Layouts findest du im Abschnitt [Klasse-Plugins](/awesome-plugins/twig) dieser Dokumentation. Für Renderzeit-Metriken auf der Tracy-Leiste siehe das [Twig-Panel in Tracy Extensions](/awesome-plugins/tracy-extensions#twig-panel-optional).

Du kannst mehr über die vollständigen Fähigkeiten von Twig erfahren, indem du die [offizielle Dokumentation](https://twig.symfony.com/doc/3.x/) liest.

### Latte

<span class="badge bg-secondary">gute Alternative</span>

[Latte](https://latte.nette.org/) ist eine voll ausgestattete Engine mit einer PHP-ähnlichen Syntax. Sie ist nach wie vor eine ausgezeichnete Wahl für Flight-Anwendungen; das Skeleton standardisiert lediglich auf Twig als gemeinsamen Standard (besonders hilfreich, wenn KI-Werkzeuge Vorlagen generieren).

#### Installation

```bash
composer require latte/latte
```

#### Grundlegende Konfiguration

Die Hauptidee ist, die `render`-Methode zu überschreiben, um Latte anstelle des standardmäßigen PHP-Renderers zu verwenden.

```php
// Überschreibe die render-Methode, um Latte anstelle des standardmäßigen PHP-Renderers zu verwenden
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Wo Latte seinen Cache speichert
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Latte in Flight verwenden

Jetzt, da du mit Latte rendern kannst, kannst du Folgendes tun:

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}Meine App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hallo, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Homepage',
		'name' => $name
	]);
});
```

Wenn du in deinem Browser `/Bob` aufrufst, wäre die Ausgabe:

```html
<html>
  <head>
	<title>Homepage - Meine App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hallo, Bob!</h1>
  </body>
</html>
```

#### Weiterführende Informationen

Ein komplexeres Beispiel für die Verwendung von Latte mit Layouts findest du im Abschnitt [Klasse-Plugins](/awesome-plugins/latte) dieser Dokumentation.

Du kannst mehr über die vollständigen Fähigkeiten von Latte einschließlich Übersetzungs- und Sprachfunktionen erfahren, indem du die [offizielle Dokumentation](https://latte.nette.org/en/) liest.

### Integrierte View-Engine

<span class="badge bg-warning">veraltet</span>

> **Hinweis:** Auch wenn dies weiterhin die Standardfunktionalität ist und technisch weiterhin funktioniert.

Um eine View-Vorlage anzuzeigen, rufe die `render`-Methode mit dem Namen der Vorlagendatei und optionalen Vorlagendaten auf:

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Die übergebenen Vorlagendaten werden automatisch in die Vorlage injiziert und können wie eine lokale Variable referenziert werden. Vorlagendateien sind einfach PHP-Dateien. Wenn der Inhalt der Vorlagendatei `hello.php` ist:

```php
Hallo, <?= $name ?>!
```

Die Ausgabe wäre:

```text
Hallo, Bob!
```

Du kannst Ansichtsvariablen auch manuell mit der `set`-Methode festlegen:

```php
Flight::view()->set('name', 'Bob');
```

Die Variable `name` ist nun in allen deinen Ansichten verfügbar. Du kannst also einfach Folgendes tun:

```php
Flight::render('hello');
```

Beachte, dass du bei der Angabe des Namens der Vorlage in der `render`-Methode die Erweiterung `.php` weglassen kannst.

Standardmäßig sucht Flight in einem `views`-Verzeichnis nach Vorlagendateien. Du kannst einen alternativen Pfad für deine Vorlagen festlegen, indem du die folgende Konfiguration setzt:

```php
Flight::set('flight.views.path', '/pfad/zu/views');
```

#### Layouts

Es ist üblich, dass Websites eine einzige Layout-Vorlagendatei mit wechselndem Inhalt haben. Um Inhalte zu rendern, die in einem Layout verwendet werden sollen, kannst du einen optionalen Parameter an die `render`-Methode übergeben.

```php
Flight::render('header', ['heading' => 'Hallo'], 'headerContent');
Flight::render('body', ['body' => 'Welt'], 'bodyContent');
```

Deine Ansicht enthält dann gespeicherte Variablen namens `headerContent` und `bodyContent`. Du kannst dann dein Layout rendern, indem du Folgendes tust:

```php
Flight::render('layout', ['title' => 'Homepage']);
```

Wenn die Vorlagendateien wie folgt aussehen:

`header.php`:

```php
<h1><?= $heading ?></h1>
```

`body.php`:

```php
<div><?= $body ?></div>
```

`layout.php`:

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

Die Ausgabe wäre:
```html
<html>
  <head>
    <title>Homepage</title>
  </head>
  <body>
    <h1>Hallo</h1>
    <div>Welt</div>
  </body>
</html>
```

### Smarty

So verwendest du die [Smarty](http://www.smarty.net/)-Template-Engine für deine Ansichten:

```php
// Lade die Smarty-Bibliothek
require './Smarty/libs/Smarty.class.php';

// Registriere Smarty als View-Klasse
// Übergebe außerdem eine Callback-Funktion, um Smarty beim Laden zu konfigurieren
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Weise Vorlagendaten zu
Flight::view()->assign('name', 'Bob');

// Zeige die Vorlage an
Flight::view()->display('hello.tpl');
```

Der Vollständigkeit halber solltest du auch die standardmäßige `render`-Methode von Flight überschreiben:

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

So verwendest du die [Blade](https://laravel.com/docs/8.x/blade)-Template-Engine für deine Ansichten:

Zuerst musst du die BladeOne-Bibliothek über Composer installieren:

```bash
composer require eftec/bladeone
```

Dann kannst du BladeOne als View-Klasse in Flight konfigurieren:

```php
<?php
// Lade die BladeOne-Bibliothek
use eftec\bladeone\BladeOne;

// Registriere BladeOne als View-Klasse
// Übergebe außerdem eine Callback-Funktion, um BladeOne beim Laden zu konfigurieren
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Weise Vorlagendaten zu
Flight::view()->share('name', 'Bob');

// Zeige die Vorlage an
echo Flight::view()->run('hello', []);
```

Der Vollständigkeit halber solltest du auch die standardmäßige `render`-Methode von Flight überschreiben:

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

In diesem Beispiel könnte die Vorlagendatei `hello.blade.php` wie folgt aussehen:

```php
<?php
Hallo, {{ $name }}!
```

Die Ausgabe wäre:

```
Hallo, Bob!
```

## Siehe auch
- [Installation](/install) – Skeleton-Layout (`app/views/*.twig`) für neue Projekte.
- [Erweitern](/learn/extending) – So überschreibst du die `render`-Methode, um eine andere Template-Engine zu verwenden.
- [Routing](/learn/routing) – So ordnest du Routen Controllern zu und renderst Ansichten.
- [Antworten](/learn/responses) – So passt du HTTP-Antworten an.
- [Sicherheit](/learn/security) – Automatisches Maskieren und XSS.
- [KI & Entwicklererfahrung](/learn/ai) – Warum eine Standard-View-Engine Programmieragenten hilft.
- [Warum ein Framework?](/learn/why-frameworks) – Wie Vorlagen in das Gesamtbild passen.

## Fehlerbehebung
- Wenn du eine Weiterleitung in deiner Middleware hast, aber deine App scheinbar nicht weiterleitet, stelle sicher, dass du eine `exit;`-Anweisung in deiner Middleware hinzufügst.
- Wenn Twig eine Vorlage nicht finden kann, überprüfe `flight.views.path` und dass die Datei unter diesem Pfad mit der erwarteten Erweiterung existiert (Skeleton: `app/views/`).

## Änderungsprotokoll
- Doku – Twig als offizieller Skeleton-Standard dokumentiert; Latte bleibt eine erstklassige Alternative.
- v2.0 – Erste Veröffentlichung.