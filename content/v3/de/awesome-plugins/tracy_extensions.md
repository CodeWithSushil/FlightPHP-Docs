# Tracy Flight Panel-Erweiterungen

Dies ist ein Satz von Erweiterungen, um die Arbeit mit Flight etwas reichhaltiger zu gestalten.

- **Flight** - Alle Flight-Variablen analysieren.
- **Datenbank** - Alle Abfragen analysieren, die auf der Seite ausgeführt wurden (wenn Sie die Datenbankverbindung korrekt initiieren)
- **Anfrage** - Alle `$_SERVER`-Variablen analysieren und alle globalen Nutzlasten (`$_GET`, `$_POST`, `$_FILES`) untersuchen
- **Session** - Alle `$_SESSION`-Variablen analysieren, falls Sessions aktiv sind.
- **Twig** *(optional)* - Twig-Vorlagen-Renderzeit, Speicher und welche Vorlagen/Blöcke/Makros ausgeführt wurden analysieren (erfordert `twig/twig` und eine `twig_profile`-Konfiguration)

Dies ist besonders praktisch mit dem [offiziellen Skeleton](https://github.com/flightphp/skeleton), das standardmäßig Twig verwendet: das gleiche Layout [KI-Tools](/learn/ai) folgen, wird auch deutlich auf der Tracy-Leiste angezeigt.

Dies ist das Panel

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

Und jedes Panel zeigt sehr hilfreiche Informationen über Ihre Anwendung an!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Klicken Sie [hier](https://github.com/flightphp/tracy-extensions), um den Code anzusehen.

## Installation

Führen Sie `composer require flightphp/tracy-extensions --dev` aus und schon geht es los!

Twig ist **keine** harte Abhängigkeit des Pakets. Installieren Sie `twig/twig` nur, wenn Sie das Twig-Panel möchten (das Skeleton macht dies bereits für Views).

## Konfiguration

Es gibt sehr wenig Konfiguration, die Sie vornehmen müssen, um dies zu starten. Sie müssen den Tracy-Debugger vor der Verwendung initiieren [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// Bootstrap-Code
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Sie müssen möglicherweise Ihre Umgebung mit Debugger::enable(Debugger::DEVELOPMENT) angeben

// wenn Sie Datenbankverbindungen in Ihrer App verwenden, gibt es einen 
// erforderlichen PDO-Wrapper, der NUR IN DER ENTWICKLUNG verwendet werden sollte (nicht in der Produktion bitte!)
// Er hat die gleichen Parameter wie eine reguläre PDO-Verbindung
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// oder wenn Sie dies an das Flight-Framework anhängen
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// jetzt wird jedes Mal, wenn Sie eine Abfrage ausführen, die Zeit, die Abfrage und die Parameter erfasst

// Dies verbindet die Punkte
if(Debugger::$showBar === true) {
	// Dies muss false sein, oder Tracy kann nicht wirklich rendern :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// mehr Code

Flight::start();
```

## Zusätzliche Konfiguration

### Session-Daten

Wenn Sie einen benutzerdefinierten Session-Handler haben (wie ghostff/session), können Sie jedes Array von Session-Daten an Tracy übergeben und es wird automatisch für Sie ausgegeben. Sie übergeben es mit dem `session_data`-Schlüssel im zweiten Parameter des `TracyExtensionLoader`-Konstruktors.

```php

use Ghostff\Session\Session;
// oder flight\Session verwenden;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Dies muss false sein, oder Tracy kann nicht wirklich rendern :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// Routen und andere Dinge...

Flight::start();
```

### Twig-Panel (optional)

Wenn Ihre App [Twig](/awesome-plugins/twig) verwendet (einschließlich des offiziellen Skeletons), können Sie Vorlagenmetriken auf der Tracy-Leiste anzeigen. Erstellen Sie ein Twig `Profile`, hängen Sie `ProfilerExtension` an Ihre Umgebung an, dann übergeben Sie dieses Profil an den Loader unter dem **`twig_profile`**-Schlüssel. Hängen Sie Profiling nur in der Entwicklung an.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Optional: Tracy-Dump-Helfer in Vorlagen verfügbar machen
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Flight::render() zu Twig abbilden (Beispiel)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Was das Panel anzeigt**

- Gesamte Twig-Renderzeit und Speicher
- Vorlagen-/Block-/Makro-Aufrufzahlen
- Jede Vorlage, die gerendert wurde, mit ihrer eigenen Zeit und Speicher

Der Twig-Tab ist **ausgeblendet**, wenn für die Anfrage keine Vorlagen gerendert wurden, oder wenn Sie `twig_profile` weglassen (oder Twig nicht installiert haben) - andere Flight-Panels funktionieren weiterhin.

In einer skeleton-ähnlichen `services.php`, bauen Sie das gleiche `$profile` / `ProfilerExtension` auf, wenn Debug eingeschaltet ist, übergeben Sie `twig_profile` an `TracyExtensionLoader`, und verwenden Sie weiterhin Ihre gemeinsame Twig-Umgebung für `$app->render()`.

### Latte

_PHP 8.1+ ist für diesen Abschnitt erforderlich._

Wenn Sie Latte in Ihrem Projekt installiert haben, hat Tracy eine native Integration mit Latte zur Analyse Ihrer Vorlagen. Sie registrieren einfach die Erweiterung mit Ihrer Latte-Instanz (dies ist Latte's eigene Tracy-Bridge, nicht das Twig-Panel oben).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// andere Konfigurationen...

	// die Erweiterung nur hinzufügen, wenn die Tracy Debug Bar aktiviert ist
	if(Debugger::$showBar === true) {
		// hier fügen Sie das Latte-Panel zu Tracy hinzu
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Siehe auch

- [Tracy](/awesome-plugins/tracy) - Basis Tracy-Setup für Flight
- [Twig](/awesome-plugins/twig) - Von dem Skeleton und dem Twig-Panel verwendetes Templating
- [Vorlagen](/learn/templates) - Wie Flight `render` zu Twig/Latte abbildet
- [Installation](/install) - Skeleton enthält tracy-extensions in dev