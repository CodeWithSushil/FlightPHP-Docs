# Tracy

Tracy ist ein erstaunlicher Fehlerhandler, der mit Flight verwendet werden kann. Er verfügt über eine Reihe von Panels, die Ihnen beim Debuggen Ihrer Anwendung helfen können. Er ist auch sehr einfach zu erweitern und eigene Panels hinzuzufügen. Das Flight Team hat einige Panels speziell für Flight-Projekte mit dem Plugin [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) erstellt (Flight-Variablen, DB-Abfragen, Request, Session und ein optionales **Twig**-Panel, wenn Sie ein Profiler-Profil übergeben – siehe [Tracy Extensions](/awesome-plugins/tracy-extensions)).

## Installation

Mit Composer installieren. Und Sie möchten dies tatsächlich ohne die Dev-Version installieren, da Tracy mit einer Produktions-Fehlerbehandlungskomponente kommt.

```bash
composer require tracy/tracy
```

## Grundkonfiguration

Es gibt einige grundlegende Konfigurationsoptionen, um zu beginnen. Sie können mehr darüber in der [Tracy-Dokumentation](https://tracy.nette.org/en/configuring) lesen.

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Tracy aktivieren
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // manchmal müssen Sie explizit sein (auch Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // Sie können auch ein Array von IP-Adressen angeben

// Hier werden Fehler und Ausnahmen protokolliert. Stellen Sie sicher, dass dieses Verzeichnis existiert und beschreibbar ist.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // alle Fehler anzeigen
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // alle Fehler außer veralteten Hinweisen
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // wenn die Debugger-Leiste sichtbar ist, kann die Content-Length nicht von Flight gesetzt werden

	// Dies ist spezifisch für die Tracy-Erweiterung für Flight, wenn Sie diese eingebunden haben
	// andernfalls kommentieren Sie dies aus.
	new TracyExtensionLoader($app);
}
```

## Hilfreiche Tipps

Wenn Sie Ihren Code debuggen, gibt es einige sehr hilfreiche Funktionen, um Daten für Sie auszugeben.

- `bdump($var)` - Dies wird die Variable in die Tracy-Leiste in einem separaten Panel ausgeben.
- `dumpe($var)` - Dies wird die Variable ausgeben und dann sofort beenden.