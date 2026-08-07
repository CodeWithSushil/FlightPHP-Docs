# flightphp/cache

Leichte, einfache und eigenständige PHP-Datei-Caching-Klasse, abgeleitet von [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Vorteile**
- Leicht, eigenständig und einfach
- Gesamter Code in einer Datei - keine unnötigen Treiber.
- Sicher - jede generierte Cache-Datei hat einen PHP-Header mit die, was direkten Zugriff unmöglich macht, selbst wenn jemand den Pfad kennt und Ihr Server nicht richtig konfiguriert ist
- Gut dokumentiert und getestet
- Behandelt Parallelität korrekt über flock
- Unterstützt PHP 7.4+
- Kostenlos unter einer MIT-Lizenz

Diese Dokumentationsseite verwendet diese Bibliothek zum Caching jeder Seite!

Klicken Sie [hier](https://github.com/flightphp/cache), um den Code anzusehen.

## Installation

Über Composer installieren:

```bash
composer require flightphp/cache
```

## Verwendung

Die Verwendung ist ziemlich einfach. Dies speichert eine Cache-Datei im Cache-Verzeichnis.

```php
use flight\Cache;

$app = Flight::app();

// Sie übergeben das Verzeichnis, in dem der Cache gespeichert wird, an den Konstruktor
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Dies stellt sicher, dass der Cache nur im Produktionsmodus verwendet wird
	// ENVIRONMENT ist eine Konstante, die in Ihrer Bootstrap-Datei oder an anderer Stelle in Ihrer App gesetzt wird
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Einen Cache-Wert abrufen

Sie verwenden die Methode `get()`, um einen gecachten Wert abzurufen. Wenn Sie eine praktische Methode möchten, die den Cache aktualisiert, wenn er abgelaufen ist, können Sie `refreshIfExpired()` verwenden.

```php

// Cache-Instanz abrufen
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // Daten zurückgeben, die gecacht werden sollen
}, 10); // 10 Sekunden

// oder
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 Sekunden
}
```

### Einen Cache-Wert speichern

Sie verwenden die Methode `set()`, um einen Wert im Cache zu speichern.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 Sekunden
```

### Einen Cache-Wert löschen

Sie verwenden die Methode `delete()`, um einen Wert im Cache zu löschen.

```php
Flight::cache()->delete('simple-cache-test');
```

### Überprüfen, ob ein Cache-Wert existiert

Sie verwenden die Methode `exists()`, um zu prüfen, ob ein Wert im Cache existiert.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// etwas tun
}
```

### Cache leeren
Sie verwenden die Methode `flush()`, um den gesamten Cache zu leeren.

```php
Flight::cache()->flush();
```

### Metadaten mit Cache abrufen

Wenn Sie Zeitstempel und andere Metadaten zu einem Cache-Eintrag abrufen möchten, stellen Sie sicher, dass Sie `true` als korrekten Parameter übergeben.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // Daten zurückgeben, die gecacht werden sollen
}, 10, true); // true = mit Metadaten zurückgeben
// oder
$data = $cache->get("simple-cache-meta-test", true); // true = mit Metadaten zurückgeben

/*
Beispiel für ein gecachtes Element, das mit Metadaten abgerufen wurde:
{
    "time":1511667506, <-- Unix-Zeitstempel speichern
    "expire":10,       <-- Ablaufzeit in Sekunden
    "data":"04:38:26", <-- deserialisierte Daten
    "permanent":false
}

Mit Metadaten können wir beispielsweise berechnen, wann das Element gespeichert wurde oder wann es abläuft
Wir können auch über den Schlüssel "data" auf die Daten selbst zugreifen
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // Unix-Zeitstempel abrufen, wann die Daten ablaufen, und aktuellen Zeitstempel davon abziehen
$cacheddate = $data["data"]; // wir greifen über den Schlüssel "data" auf die Daten selbst zu

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## Quellcode

Besuchen Sie [https://github.com/flightphp/cache](https://github.com/flightphp/cache), um den Code anzusehen.