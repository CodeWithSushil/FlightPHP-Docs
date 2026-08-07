# FlightPHP APM Dokumentation

Willkommen bei FlightPHP APM – dem persönlichen Performance-Coach für Ihre App! Dieser Leitfaden ist Ihr Wegweiser zum Einrichten, Verwenden und Beherrschen von Application Performance Monitoring (APM) mit FlightPHP. Egal, ob Sie langsame Anfragen aufspüren oder einfach nur über Latenzdiagramme schwärmen möchten – wir haben alles für Sie. Lassen Sie uns Ihre App schneller machen, Ihre Benutzer glücklicher und Ihre Debugging-Sitzungen zum Kinderspiel!

Sehen Sie sich eine [Demo](https://flightphp-docs-apm.sky-9.com/apm/dashboard) des Dashboards für die Flight Docs Seite an.

![FlightPHP APM](/images/apm.png)

## Warum APM wichtig ist

Stellen Sie sich vor, Ihre App ist ein belebtes Restaurant. Ohne eine Möglichkeit zu verfolgen, wie lange Bestellungen dauern oder wo es in der Küche stockt, können Sie nur raten, warum Kunden verärgert gehen. APM ist Ihr Sous-Chef – er überwacht jeden Schritt, von eingehenden Anfragen bis hin zu Datenbankabfragen, und markiert alles, was Sie ausbremst. Langsame Seiten verlieren Nutzer (Studien zeigen, dass 53 % abspringen, wenn eine Website länger als 3 Sekunden zum Laden braucht!), und APM hilft Ihnen, diese Probleme *bevor* sie schmerzen zu erkennen. Es ist proaktive Seelenruhe – weniger „Warum funktioniert das nicht?“-Momente, mehr „Schaut mal, wie geschmeidig das läuft!“-Erfolge.

## Installation

Beginnen Sie mit Composer:

```bash
composer require flightphp/apm
```

Sie benötigen:
- **PHP 7.4+**: Hält uns kompatibel mit LTS-Linux-Distributionen und unterstützt modernes PHP.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: Das leichte Framework, das wir erweitern.

## Unterstützte Datenbanken

FlightPHP APM unterstützt derzeit die folgenden Datenbanken zur Speicherung von Metriken:

- **SQLite3**: Einfach, dateibasiert und ideal für die lokale Entwicklung oder kleine Apps. Standardoption in den meisten Setups.
- **MySQL/MariaDB**: Ideal für größere Projekte oder Produktionsumgebungen, in denen robuste, skalierbare Speicherung benötigt wird.

Sie können Ihren Datenbanktyp während des Konfigurationsschritts auswählen (siehe unten). Stellen Sie sicher, dass Ihre PHP-Umgebung die erforderlichen Erweiterungen installiert hat (z. B. `pdo_sqlite` oder `pdo_mysql`).

## Erste Schritte

Hier ist Ihre Schritt-für-Schritt-Anleitung zu APM-Höchstleistungen:

### 1. APM registrieren

Fügen Sie dies in Ihre `index.php` oder eine `services.php`-Datei ein, um mit der Nachverfolgung zu beginnen:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// Wenn Sie eine Datenbankverbindung hinzufügen
// Bevorzugen Sie SimplePdo (oder PdoQueryCapture von Tracy Extensions in der Entwicklung).
// Aktivieren Sie die APM-Abfrageverfolgung über das Options-Array (5. Argument).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // erforderlich, um Abfragen für das APM zu erfassen
]);
$Apm->addPdoConnection($pdo);
```

**Was passiert hier?**
- `LoggerFactory::create()` holt Ihre Konfiguration (mehr dazu gleich) und richtet einen Logger ein – standardmäßig SQLite.
- `Apm` ist der Star – er hört auf Flight-Events (Anfragen, Routen, Fehler usw.) und sammelt Metriken.
- `bindEventsToFlightInstance($app)` verknüpft alles mit Ihrer Flight-App.

**Pro-Tipp: Sampling**
Wenn Ihre App viel zu tun hat, kann das Protokollieren *jeder* Anfrage überlasten. Verwenden Sie eine Sampling-Rate (0.0 bis 1.0):

```php
$Apm = new Apm($ApmLogger, 0.1); // Protokolliert 10 % der Anfragen
```

Das hält die Leistung flott und liefert dennoch solide Daten.

### 2. Konfigurieren

Führen Sie dies aus, um Ihre `.runway-config.json` zu erstellen:

```bash
php vendor/bin/runway apm:init
```

**Was macht das?**
- Startet einen Assistenten, der fragt, woher die Rohmetriken kommen (Quelle) und wohin verarbeitete Daten gehen (Ziel).
- Standard ist SQLite – z. B. `sqlite:/tmp/apm_metrics.sqlite` für die Quelle, ein weiteres für das Ziel.
- Sie erhalten am Ende eine Konfiguration wie:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Dieser Vorgang fragt auch, ob Sie die Migrationen für dieses Setup ausführen möchten. Wenn Sie dies zum ersten Mal einrichten, lautet die Antwort ja.

**Warum zwei Standorte?**
Rohmetriken häufen sich schnell an (denken Sie an ungefilterte Protokolle). Der Worker verarbeitet sie in ein strukturiertes Ziel für das Dashboard. Hält alles übersichtlich!

### 3. Metriken mit dem Worker verarbeiten

Der Worker wandelt Rohmetriken in Dashboard-bereite Daten um. Führen Sie ihn einmal aus:

```bash
php vendor/bin/runway apm:worker
```

**Was macht er?**
- Liest aus Ihrer Quelle (z. B. `apm_metrics.sqlite`).
- Verarbeitet bis zu 100 Metriken (Standard-Batch-Größe) in Ihr Ziel.
- Stoppt, wenn fertig oder wenn keine Metriken mehr vorhanden sind.

**Dauerhaft laufen lassen**
Für Live-Apps möchten Sie eine kontinuierliche Verarbeitung. Hier sind Ihre Optionen:

- **Daemon-Modus**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Läuft für immer und verarbeitet Metriken bei Ankunft. Ideal für Entwicklung oder kleine Setups.

- **Crontab**:
  Fügen Sie dies zu Ihrer Crontab hinzu (`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  Wird jede Minute ausgeführt – perfekt für Produktion.

- **Tmux/Screen**:
  Starten Sie eine abtrennbare Sitzung:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Strg+B, dann D zum Abtrennen; `tmux attach -t apm-worker` zum Wiederverbinden
  ```
  Hält es auch nach dem Ausloggen am Laufen.

- **Benutzerdefinierte Anpassungen**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: Verarbeitet 50 Metriken gleichzeitig.
  - `--max_messages 1000`: Stoppt nach 1000 Metriken.
  - `--timeout 300`: Beendet nach 5 Minuten.

**Warum der Aufwand?**
Ohne den Worker ist Ihr Dashboard leer. Er ist die Brücke zwischen Rohprotokollen und umsetzbaren Erkenntnissen.

### 4. Dashboard starten

Sehen Sie sich die Vitalwerte Ihrer App an:

```bash
php vendor/bin/runway apm:dashboard
```

**Was macht das?**
- Startet einen PHP-Server unter `http://localhost:8001/apm/dashboard`.
- Zeigt Anfrageprotokolle, langsame Routen, Fehlerraten und mehr.

**Anpassen**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: Von jeder IP erreichbar (praktisch für Remote-Ansichten).
- `--port 8080`: Verwenden Sie einen anderen Port, wenn 8001 belegt ist.
- `--php-path`: Geben Sie PHP an, wenn es nicht in Ihrem PATH ist.

Rufen Sie die URL in Ihrem Browser auf und erkunden Sie!

#### Produktionsmodus

Für die Produktion müssen Sie möglicherweise einige Techniken ausprobieren, um das Dashboard zum Laufen zu bringen, da wahrscheinlich Firewalls und andere Sicherheitsmaßnahmen vorhanden sind. Hier sind einige Optionen:

- **Reverse-Proxy verwenden**: Richten Sie Nginx oder Apache ein, um Anfragen an das Dashboard weiterzuleiten.
- **SSH-Tunnel**: Wenn Sie per SSH auf den Server zugreifen können, verwenden Sie `ssh -L 8080:localhost:8001
youruser@yourserver`, um das Dashboard zu Ihrem lokalen Rechner zu tunneln.
- **VPN**: Wenn Ihr Server hinter einem VPN steht, verbinden Sie sich damit und greifen Sie direkt auf das Dashboard zu.
- **Firewall konfigurieren**: Öffnen Sie Port 8001 für Ihre IP oder das Netzwerk des Servers. (oder welchen Port Sie auch immer eingestellt haben).
- **Apache/Nginx konfigurieren**: Wenn Sie einen Webserver vor Ihrer Anwendung haben, können Sie ihn für eine Domain oder Subdomain konfigurieren. Wenn Sie dies tun, setzen Sie das Dokumenten-Root auf `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### Anderes Dashboard gewünscht?

Sie können Ihr eigenes Dashboard erstellen, wenn Sie möchten! Schauen Sie im Verzeichnis vendor/flightphp/apm/src/apm/presenter nach Ideen, wie Sie die Daten für Ihr eigenes Dashboard präsentieren!

## Dashboard-Funktionen

Das Dashboard ist Ihre APM-Zentrale – hier ist, was Sie sehen werden:

- **Anfrageprotokoll**: Jede Anfrage mit Zeitstempel, URL, Antwortcode und Gesamtzeit. Klicken Sie auf „Details“ für Middleware, Abfragen und Fehler.
- **Langsamste Anfragen**: Top 5 Anfragen, die Zeit verbrauchen (z. B. „/api/heavy“ mit 2,5s).
- **Langsamste Routen**: Top 5 Routen nach durchschnittlicher Zeit – ideal, um Muster zu erkennen.
- **Fehlerrate**: Prozentsatz fehlgeschlagener Anfragen (z. B. 2,3 % 500er).
- **Latenz-Quantile**: 95. (p95) und 99. (p99) Antwortzeiten – kennen Sie Ihre Worst-Case-Szenarien.
- **Antwortcode-Diagramm**: Visualisieren Sie 200er, 404er, 500er über die Zeit.
- **Lange Abfragen/Middleware**: Top 5 langsame Datenbankaufrufe und Middleware-Schichten.
- **Cache-Treffer/Fehlschlag**: Wie oft Ihr Cache den Tag rettet.

**Extras**:
- Nach „Letzte Stunde“, „Letzter Tag“ oder „Letzte Woche“ filtern.
- Dunkelmodus für nächtliche Sitzungen umschalten.

**Beispiel**:
Eine Anfrage an `/users` könnte zeigen:
- Gesamtzeit: 150ms
- Middleware: `AuthMiddleware->handle` (50ms)
- Abfrage: `SELECT * FROM users` (80ms)
- Cache: Treffer bei `user_list` (5ms)

## Benutzerdefinierte Events hinzufügen

Alles verfolgen – wie einen API-Aufruf oder Zahlungsvorgang:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**Wo wird es angezeigt?**
In den Anfragedetails des Dashboards unter „Benutzerdefinierte Events“ – erweiterbar mit hübscher JSON-Formatierung.

**Anwendungsfall**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
Jetzt sehen Sie, ob diese API Ihre App ausbremst!

## Datenbanküberwachung

PDO-Abfragen so verfolgen:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // erforderlich, um Abfragen für das APM zu erfassen
]);
$Apm->addPdoConnection($pdo);
```

**Was Sie erhalten**:
- Abfragetext (z. B. `SELECT * FROM users WHERE id = ?`)
- Ausführungszeit (z. B. 0.015s)
- Zeilenanzahl (z. B. 42)

**Achtung**:
- **Optional**: Überspringen Sie dies, wenn Sie keine DB-Verfolgung benötigen.
- **SimplePdo (bevorzugt)**: Verwenden Sie `SimplePdo` mit `trackApmQueries => true`. Der veraltete `PdoWrapper` funktioniert noch (5. Konstruktor-Argument `true`). Rohes Core-PDO ist noch nicht angebunden – bleiben Sie dran!
- **Leistungswarnung**: Das Protokollieren jeder Abfrage auf einer datenbanklastigen Site kann Dinge verlangsamen. Verwenden Sie Sampling (`$Apm = new Apm($ApmLogger, 0.1)`) zur Entlastung.

**Beispielausgabe**:
- Abfrage: `SELECT name FROM products WHERE price > 100`
- Zeit: 0.023s
- Zeilen: 15

## Worker-Optionen

Stellen Sie den Worker nach Ihren Wünschen ein:

- `--timeout 300`: Stoppt nach 5 Minuten – gut zum Testen.
- `--max_messages 500`: Begrenzt auf 500 Metriken – hält es endlich.
- `--batch_size 200`: Verarbeitet 200 auf einmal – balanciert Geschwindigkeit und Speicher.
- `--daemon`: Läuft ununterbrochen – ideal für Live-Monitoring.

**Beispiel**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Läuft eine Stunde lang, verarbeitet 100 Metriken gleichzeitig.

## Request-ID in der App

Jede Anfrage hat eine eindeutige Request-ID zur Nachverfolgung. Sie können diese ID in Ihrer App verwenden, um Protokolle und Metriken zu korrelieren. Sie können beispielsweise die Request-ID zu einer Fehlerseite hinzufügen:

```php
Flight::map('error', function($message) {
	// Die Request-ID aus dem Antwort-Header X-Flight-Request-Id abrufen
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Zusätzlich könnten Sie sie aus der Flight-Variablen abrufen
	// Diese Methode funktioniert nicht gut in Swoole oder anderen asynchronen Plattformen.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Fehler: $message (Request-ID: $requestId)";
});
```

## Upgrade

Wenn Sie auf eine neuere Version des APM upgraden, besteht die Möglichkeit, dass Datenbankmigrationen ausgeführt werden müssen. Sie können dies mit folgendem Befehl tun:

```bash
php vendor/bin/runway apm:migrate
```
Dies führt alle Migrationen aus, die benötigt werden, um das Datenbankschema auf die neueste Version zu aktualisieren.

**Hinweis:** Wenn Ihre APM-Datenbank groß ist, können diese Migrationen einige Zeit in Anspruch nehmen. Sie sollten diesen Befehl außerhalb der Stoßzeiten ausführen.

### Upgrade von 0.4.3 -> 0.5.0

Wenn Sie von 0.4.3 auf 0.5.0 upgraden, müssen Sie folgenden Befehl ausführen:

```bash
php vendor/bin/runway apm:config-migrate
```

Dies migriert Ihre Konfiguration vom alten Format mit der `.runway-config.json`-Datei zum neuen Format, das die Schlüssel/Werte in der `config.php`-Datei speichert.

## Alte Daten bereinigen

Um Ihre Datenbank sauber zu halten, können Sie alte Daten bereinigen. Dies ist besonders nützlich, wenn Sie eine stark frequentierte App betreiben und die Datenbankgröße überschaubar halten möchten.
Sie können dies mit folgendem Befehl tun:

```bash
php vendor/bin/runway apm:purge
```
Dies entfernt alle Daten, die älter als 30 Tage sind, aus der Datenbank. Sie können die Anzahl der Tage anpassen, indem Sie einen anderen Wert an die `--days`-Option übergeben:

```bash
php vendor/bin/runway apm:purge --days 7
```
Dies entfernt alle Daten, die älter als 7 Tage sind, aus der Datenbank.

## Fehlersuche

Stecken Sie fest? Versuchen Sie Folgendes:

- **Keine Dashboard-Daten?**
  - Läuft der Worker? Prüfen Sie `ps aux | grep apm:worker`.
  - Stimmen die Konfigurationspfade? Überprüfen Sie, ob die DSNs in `.runway-config.json` auf echte Dateien zeigen.
  - Führen Sie `php vendor/bin/runway apm:worker` manuell aus, um ausstehende Metriken zu verarbeiten.

- **Worker-Fehler?**
  - Werfen Sie einen Blick auf Ihre SQLite-Dateien (z. B. `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Prüfen Sie die PHP-Protokolle auf Stack-Traces.

- **Dashboard startet nicht?**
  - Port 8001 belegt? Verwenden Sie `--port 8080`.
  - PHP nicht gefunden? Verwenden Sie `--php-path /usr/bin/php`.
  - Firewall blockiert? Öffnen Sie den Port oder verwenden Sie `--host localhost`.

- **Zu langsam?**
  - Senken Sie die Sampling-Rate: `$Apm = new Apm($ApmLogger, 0.05)` (5 %).
  - Verringern Sie die Batch-Größe: `--batch_size 20`.

- **Verfolgt keine Ausnahmen/Fehler?**
  - Wenn Sie [Tracy](https://tracy.nette.org/) für Ihr Projekt aktiviert haben, überschreibt dies die Fehlerbehandlung von Flight. Sie müssen Tracy deaktivieren und sicherstellen, dass `Flight::set('flight.handle_errors', true);` gesetzt ist.

- **Verfolgt keine Datenbankabfragen?**
  - Bevorzugen Sie `SimplePdo` mit `['trackApmQueries' => true]` als 5. Konstruktor-Argument (Options-Array).
  - Wenn Sie noch den veralteten `PdoWrapper` verwenden, übergeben Sie `true` als 5. Argument.
  - Rufen Sie `$Apm->addPdoConnection($pdo)` nach dem Erstellen der Verbindung auf.