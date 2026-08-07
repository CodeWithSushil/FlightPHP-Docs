# Flight PHP Framework

Flight ist ein schnelles, einfaches und erweiterbares Framework für PHP—entwickelt für Entwickler, die Dinge schnell und ohne Umstände erledigen möchten. Ob Sie eine klassische Webanwendung, eine blitzschnelle API oder eine Kombination mit KI-Coding-Assistenten erstellen, Flights geringe Größe und unkompliziertes Design machen es zur perfekten Wahl. Flight ist schlank ausgelegt, kann aber auch Enterprise-Architekturanforderungen erfüllen.

## Warum Flight wählen?

- **Anfängerfreundlich:** Flight ist ein großartiger Einstieg für neue PHP-Entwickler. Seine klare Struktur und einfache Syntax helfen Ihnen, Webentwicklung zu lernen, ohne in Boilerplate-Code zu versinken.
- **Von Profis geliebt:** Erfahrene Entwickler schätzen Flight für seine Flexibilität und Kontrolle. Sie können von einem kleinen Prototypen zu einer vollwertigen Anwendung skalieren, ohne Frameworks wechseln zu müssen.
- **Rückwärtskompatibel:** Wir schätzen Ihre Zeit. Flight v3 ist eine Erweiterung von v2 und behält fast die gesamte gleiche API bei. Wir glauben an Evolution, nicht an Revolution—kein "Weltzerbrechen" mehr bei jeder neuen Hauptversion.
- **Keine Abhängigkeiten:** Flights Kern ist vollständig frei von Abhängigkeiten—keine Polyfills, keine externen Pakete, nicht einmal PSR-Schnittstellen. Das bedeutet weniger Angriffsvektoren, einen kleineren Fußabdruck und keine überraschenden Breaking Changes durch Upstream-Abhängigkeiten. Optionale Plugins können Abhängigkeiten enthalten, aber der Kern bleibt immer schlank und sicher.
- **KI-freundlich:** Flights kleine API-Oberfläche und das [offizielle Skeleton](https://github.com/flightphp/skeleton) (ein Layout, `AGENTS.md`, Konstruktor-Injektion) erleichtern es KI-Coding-Tools, im Muster zu bleiben. Gleiche Codebasis, ob Sie jede Zeile selbst tippen oder mit einem Agenten zusammenarbeiten. [Mehr über die Verwendung von KI mit Flight erfahren](/learn/ai).

## Video-Übersicht

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">Ziemlich einfach, oder?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">Erfahren Sie mehr</a> über Flight in der Dokumentation!
    </div>
  </div>
</div>

## Schnellstart

Für eine schnelle Minimalinstallation installieren Sie es mit Composer:

```bash
composer require flightphp/core
```

Oder Sie können ein ZIP des Repositories [hier](https://github.com/flightphp/core) herunterladen. Dann haben Sie eine grundlegende `index.php`-Datei wie die folgende:

```php
<?php

// falls mit Composer installiert
require 'vendor/autoload.php';
// oder falls manuell per ZIP-Datei installiert
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

Das war's! Sie haben eine grundlegende Flight-Anwendung. Sie können diese Datei nun mit `php -S localhost:8000` ausführen und `http://localhost:8000` in Ihrem Browser aufrufen, um die Ausgabe zu sehen.

Kurze `Flight::`-Beispiele wie dieses sind großartig zum Lernen und für Micro-Apps. Für ein vollständiges Projektlayout, das Menschen und KI-Tools teilen, verwenden Sie das Skeleton unten.

## Skeleton/Boilerplate-App

Es gibt ein offizielles Starter-Template, um Ihnen den Einstieg in jedes neue Flight-Projekt zu erleichtern. Es richtet Struktur, Konfiguration, Composer-Skripte und KI-freundliche Anweisungen von Anfang an ein.

Schauen Sie sich [flightphp/skeleton](https://github.com/flightphp/skeleton) für ein sofort einsetzbares Projekt an oder besuchen Sie die [Beispiele](examples)-Seite für Inspiration. Möchten Sie Details zum KI-Workflow? [Entdecken Sie KI & Developer Experience](/learn/ai).

Was Sie erhalten (Übersicht):

- **`App\` Namespaces** mit PascalCase-Ordnern (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—die **Groß-/Kleinschreibung** des Ordners muss mit dem Namespace übereinstimmen (siehe [Autoloading](/learn/autoloading))
- **Dice + `Engine`-Injektion**, damit Controller testbar bleiben (bevorzugen Sie `$this->app` statt `Flight::` im Anwendungscode)
- **Twig**-Views, **SimplePdo** + ActiveRecord-Beispiel, Runway **migrate**
- Root **`AGENTS.md`** (plus bereichsspezifische Kopien) und **`SECURITY.md`** für Assistenten und Sicherheitsrichtlinien

## Installation der Skeleton-App

Ganz einfach!

```bash
# Erstellen Sie das neue Projekt
composer create-project flightphp/skeleton my-project/
# Wechseln Sie in das neue Projektverzeichnis
cd my-project/
# Starten Sie den lokalen Entwicklungsserver, um sofort loszulegen!
composer start
```

Es erstellt die Projektstruktur, kopiert `config_sample.php` → `config.php` (und `.env.example` → `.env`, falls vorhanden), und Sie können loslegen. Optionale Beispieldaten:

```bash
php runway migrate
# dann besuchen Sie /posts und /api/posts
```

## Hohe Performance

Flight ist eines der schnellsten PHP-Frameworks. Sein schlankes Kernsystem bedeutet weniger Overhead und mehr Geschwindigkeit—perfekt für traditionelle Anwendungen und moderne, KI-unterstützte Workflows. Alle Benchmarks finden Sie auf [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Sehen Sie den Benchmark unten mit einigen anderen beliebten PHP-Frameworks.

| Framework | Plaintext Reqs/sec | JSON Reqs/sec |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight und KI

Interessiert, wie Flight mit Coding-LLMs zusammenarbeitet? [Entdecken Sie](/learn/ai), wie `AGENTS.md`, Runway `ai:*`-Befehle und das Skeleton-Layout Assistenten auf Kurs halten.

## Stabilität und Rückwärtskompatibilität

Wir schätzen Ihre Zeit. Wir haben alle Frameworks gesehen, die sich alle paar Jahre komplett neu erfinden und Entwickler mit kaputtem Code und teuren Migrationen zurücklassen. Flight ist anders. Flight v3 wurde als Erweiterung von v2 konzipiert, was bedeutet, dass die API, die Sie kennen und schätzen, nicht entfernt wurde. Tatsächlich werden die meisten v2-Projekte ohne Änderungen in v3 funktionieren. 

Wir sind bestrebt, Flight stabil zu halten, damit Sie sich auf den Aufbau Ihrer Anwendung konzentrieren können, nicht auf die Reparatur Ihres Frameworks. Das Skeleton kann für *neue* Projekte meinungsstark sein; Kern-APIs bleiben für alle anderen vertraut.

# Community

Wir sind auf Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

Und Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Mitwirken

Es gibt zwei Möglichkeiten, wie Sie zu Flight beitragen können:

1. Tragen Sie zum Kern-Framework bei, indem Sie das [Core-Repository](https://github.com/flightphp/core) besuchen.
2. Helfen Sie mit, die Dokumentation zu verbessern! Diese Dokumentationswebsite wird auf [Github](https://github.com/flightphp/docs) gehostet. Wenn Sie einen Fehler entdecken oder etwas verbessern möchten, können Sie gerne einen Pull Request einreichen. Wir freuen uns über Updates und neue Ideen—besonders rund um KI und neue Technologien!

# Systemanforderungen

Flight erfordert PHP 7.4 oder höher.

**Hinweis:** PHP 7.4 wird unterstützt, weil zum Zeitpunkt der Erstellung dieser Dokumentation (2024) PHP 7.4 die Standardversion für einige LTS-Linux-Distributionen ist. Ein erzwungener Wechsel zu PHP >8 würde bei diesen Nutzern viel Frust verursachen. Das Framework unterstützt auch PHP >8.

# Lizenz

Flight wird unter der [MIT](https://github.com/flightphp/core/blob/master/LICENSE)-Lizenz veröffentlicht.