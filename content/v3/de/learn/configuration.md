# Konfiguration

## Überblick

Flight bietet eine einfache Möglichkeit, verschiedene Aspekte des Frameworks an die Bedürfnisse Ihrer Anwendung anzupassen. Einige sind standardmäßig eingestellt, aber Sie können sie nach Bedarf überschreiben. Sie können auch eigene Variablen festlegen, die in Ihrer gesamten Anwendung verwendet werden.

Klare, mehrschichtige Konfiguration (Datei-Standardwerte + Umgebungsgeheimnisse) hilft auch [KI-Codierungstools](/learn/ai): Agenten lernen einen Ort für Literale und einen Ort für Geheimnisse, anstatt `$_ENV`-Lesezugriffe in Controllern zu erfinden.

## Verständnis

Sie können bestimmte Verhaltensweisen von Flight anpassen, indem Sie Konfigurationswerte über die `set`-Methode festlegen.

```php
Flight::set('flight.log_errors', true);
```

In einer strukturierten Anwendung (einschließlich des [Skeletts](https://github.com/flightphp/skeleton)) laden Sie typischerweise Projekteinstellungen aus `app/config/config.php` und wenden dann relevante Schlüssel auf die Engine an (z. B. `flight.base_url`, `flight.views.path`). Sie können auch ein kleines Konfigurationsobjekt in Controller injizieren, anstatt überall Globale zu lesen – freundlicher für Tests und für Agenten, die `AGENTS.md` folgen.

## Grundlegende Verwendung

### Flight-Konfigurationsoptionen

Die folgende Liste enthält alle verfügbaren Konfigurationseinstellungen:

- **flight.base_url** `?string` - Überschreibt die Basis-URL der Anfrage, wenn Flight in einem Unterverzeichnis läuft. (Standard: null)
- **flight.case_sensitive** `bool` - Groß-/Kleinschreibung bei URL-Abgleich beachten. (Standard: false)
- **flight.handle_errors** `bool` - Flight erlaubt, alle Fehler intern zu behandeln. (Standard: true)
  - Wenn Sie möchten, dass Flight Fehler anstelle des Standard-PHP-Verhaltens behandelt, muss dies true sein.
  - Wenn Sie [Tracy](/awesome-plugins/tracy) installiert haben, setzen Sie dies auf false, damit Tracy Fehler behandeln kann.
  - Wenn Sie das [APM](/awesome-plugins/apm) Plugin installiert haben, setzen Sie dies auf true, damit das APM die Fehler protokollieren kann.
- **flight.log_errors** `bool` - Fehler in die Fehlerprotokolldatei des Webservers schreiben. (Standard: false)
  - Wenn Sie [Tracy](/awesome-plugins/tracy) installiert haben, protokolliert Tracy Fehler basierend auf den Tracy-Konfigurationen, nicht auf dieser Konfiguration.
- **flight.debug** `bool` - Detaillierte Fehlerinformationen (Ausnahmemeldung, Code und Stack-Trace) im Browser ausgeben, wenn ein Fehler auftritt. (Standard: false)
  - **Aktivieren Sie dies niemals in der Produktion** – es gibt interne Anwendungsdetails preis. Verwenden Sie es nur für lokale Entwicklung oder Staging.
  - Wenn `false`, wird stattdessen eine generische `500 Internal Server Error`-Antwort angezeigt. Kombinieren Sie dies mit `flight.log_errors`, um Fehler serverseitig zu erfassen.
- **flight.allow_method_override** `bool` - Ermöglicht das Überschreiben der HTTP-Methode über den `X-HTTP-Method-Override`-Anfrageheader oder ein `_method`-Feld im POST-Body. (Standard: true)
  - **Das Setzen auf `false` wird empfohlen** für Anwendungen, die kein HTML-Formular-basiertes Methoden-Spoofing benötigen, da es Clients verhindert, `DELETE`- oder `PUT`-Anfragen über ein Standard-POST-Formular zu fälschen.
  - Siehe [Sicherheit](/learn/security#flight-configuration-hardening) für weitere Details.
- **flight.views.path** `string` - Verzeichnis mit View-Template-Dateien. (Standard: ./views)
- **flight.views.extension** `string` - Dateiendung für View-Templates. (Standard: `.php`; das offizielle Skelett setzt dies auf `.twig`, wenn Twig verwendet wird)
- **flight.content_length** `bool` - Den `Content-Length`-Header setzen. (Standard: true)
  - Wenn Sie [Tracy](/awesome-plugins/tracy) verwenden, müssen Sie dies auf false setzen, damit Tracy korrekt rendern kann.
- **flight.v2.output_buffering** `bool` - Legacy-Ausgabepufferung verwenden. Siehe [Migration zu v3](migrating-to-v3). (Standard: false)

### Loader-Konfiguration

Zusätzlich gibt es eine weitere Konfigurationseinstellung für den Loader. Diese ermöglicht es Ihnen, Klassen mit `_` im Klassennamen automatisch zu laden.

```php
// Aktiviert das Laden von Klassen mit Unterstrichen
// Standardmäßig true
Loader::$v2ClassLoading = false;
```

Denken Sie daran, dass [Autoloading](/learn/autoloading) auch von der **Groß-/Kleinschreibung der Ordner** abhängt, die zu Ihren Namespaces passen muss – insbesondere beim Skelett-Layout mit `App\` + `app/Controller/`.

### Projektkonfiguration und `.env` (Skelett-Muster)

Der Flight-Kern erfordert keine `.env`-Dateien. Viele Anwendungen verwenden nur ein PHP-Konfigurationsarray. Das offizielle Skelett schichtet die Konfiguration, sodass Geheimnisse nicht in Git gelangen, während Runway weiterhin **literale** Konfiguration sicher umschreiben kann:

1. **`.env` / echte Umgebung** — Geheimnisse und Deployment-Überschreibungen (gitignoriert).
2. **`app/config/config.php`** — Literale PHP-Array-Standardwerte (kopiert aus `config_sample.php`). Bevorzugen Sie **keine** `$_ENV[...]`-Ausdrücke in dieser Datei: Tools wie `runway config:set` könnten sie als statische Werte umschreiben und Geheimnisse in die Datei backen.
3. **Zusammenführen beim Bootstrap** — Umgebung gewinnt für zugeordnete Schlüssel; Anwendungscode liest ein Konfigurationsobjekt oder `$app->get()`, nicht `$_ENV` in Controllern.

Beispielstruktur von `config_sample.php` / `config.php` (vereinfacht):

```php
<?php
// Nur Literale – Geheimnisse gehören für den Skelett-Workflow in .env
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // oder mysql, oder '' zum Deaktivieren
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env (Skelett)
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

Diese Trennung ist bewusst für [KI-freundliche Projekte](/learn/ai): Anweisungen können sagen: „Standardwerte in `config.php`, Geheimnisse in `.env`, Config / Engine injizieren – niemals Umgebungszugriff in einem Controller erfinden.“ Bestehende Anwendungen können `.env` vollständig ignorieren und eine einzige Konfigurationsdatei behalten.

### Variablen

Flight ermöglicht es Ihnen, Variablen zu speichern, sodass sie überall in Ihrer Anwendung verwendet werden können.

```php
// Speichern Sie Ihre Variable
Flight::set('id', 123);

// Anderswo in Ihrer Anwendung
$id = Flight::get('id');
```

Um zu prüfen, ob eine Variable gesetzt wurde, können Sie Folgendes tun:

```php
if (Flight::has('id')) {
  // Etwas tun
}
```

Sie können eine Variable löschen, indem Sie Folgendes tun:

```php
// Löscht die id-Variable
Flight::clear('id');

// Löscht alle Variablen
Flight::clear();
```

> **Hinweis:** Nur weil Sie eine Variable setzen können, bedeutet das nicht, dass Sie es tun sollten. Verwenden Sie diese Funktion sparsam. Der Grund dafür ist, dass alles, was hier gespeichert wird, zu einer globalen Variable wird. Globale Variablen sind schlecht, weil sie von überall in Ihrer Anwendung geändert werden können, was das Auffinden von Fehlern erschwert. Außerdem kann dies Dinge wie [Unit-Tests](/guides/unit-testing) verkomplizieren. Bevorzugen Sie Konstruktor-Injektion (wie im Skelett- und Dice-Setup) für Dienste und Konfiguration, die Controller benötigen.

### Fehler und Ausnahmen

Alle Fehler und Ausnahmen werden von Flight abgefangen und an die `error`-Methode übergeben, wenn `flight.handle_errors` auf true gesetzt ist.

Das Standardverhalten besteht darin, eine generische `HTTP 500 Internal Server Error`-Antwort mit einigen Fehlerinformationen zu senden.

Sie können dieses Verhalten für Ihre eigenen Bedürfnisse [überschreiben](/learn/extending):

```php
Flight::map('error', function (Throwable $error) {
  // Fehler behandeln
  echo $error->getTraceAsString();
});
```

Standardmäßig werden Fehler nicht an den Webserver protokolliert. Sie können dies aktivieren, indem Sie die Konfiguration ändern:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Nicht gefunden

Wenn eine URL nicht gefunden werden kann, ruft Flight die `notFound`-Methode auf. Das Standardverhalten besteht darin, eine `HTTP 404 Not Found`-Antwort mit einer einfachen Meldung zu senden.

Sie können dieses Verhalten für Ihre eigenen Bedürfnisse [überschreiben](/learn/extending):

```php
Flight::map('notFound', function () {
  // Nicht gefunden behandeln
});
```

## Siehe auch

- [Installation](/install) - Skelett-Konfiguration, `.env` und Bootstrap-Layout.
- [Autoloading](/learn/autoloading) - Namespaces und Groß-/Kleinschreibung von Ordnern.
- [Flight erweitern](/learn/extending) - So erweitern und passen Sie die Kernfunktionalität von Flight an.
- [Unit-Tests](/guides/unit-testing) - So schreiben Sie Unit-Tests für Ihre Flight-Anwendung.
- [KI & Entwicklererfahrung](/learn/ai) - `AGENTS.md` und konsistente Projektanweisungen.
- [Tracy](/awesome-plugins/tracy) - Ein Plugin für erweiterte Fehlerbehandlung und Debugging.
- [Tracy-Erweiterungen](/awesome-plugins/tracy_extensions) - Erweiterungen zur Integration von Tracy mit Flight.
- [APM](/awesome-plugins/apm) - Ein Plugin für Anwendungsleistungsüberwachung und Fehlerverfolgung.
- [Sicherheit](/learn/security) - Härtungsflags und Geheimnisbehandlung.

## Fehlerbehebung

- Wenn Sie Probleme haben, alle Werte Ihrer Konfiguration herauszufinden, können Sie `var_dump(Flight::get());` ausführen.
- Wenn Runway oder Deployment-Tools `config.php` umgeschrieben haben, bestätigen Sie, dass keine Geheimnisse committet wurden – bewahren Sie sie in `.env` oder in der echten Umgebung auf, wenn Sie das Skelett-Muster verwenden.

## Änderungsprotokoll

- Doku – Skelettartige Konfiguration / `.env`-Schichtung und Twig-View-Erweiterungsstandard für neue Projekte dokumentiert.
- v3.18.1 - Konfigurationsoptionen `flight.debug` und `flight.allow_method_override` hinzugefügt.
- v3.5.0 - Konfiguration für `flight.v2.output_buffering` hinzugefügt, um das Legacy-Ausgabepufferungsverhalten zu unterstützen.
- v2.0 - Kernkonfigurationen hinzugefügt.