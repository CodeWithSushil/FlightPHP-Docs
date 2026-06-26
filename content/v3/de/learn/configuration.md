# Konfiguration

## Übersicht

Flight bietet eine einfache Möglichkeit, verschiedene Aspekte des Frameworks zu konfigurieren, um den Anforderungen Ihrer Anwendung gerecht zu werden. Einige sind standardmäßig festgelegt, können aber bei Bedarf überschrieben werden. Sie können auch eigene Variablen festlegen, die in Ihrer gesamten Anwendung verwendet werden können.

## Verständnis

Sie können bestimmte Verhaltensweisen von Flight anpassen, indem Sie Konfigurationswerte über die `set`-Methode festlegen.

```php
Flight::set('flight.log_errors', true);
```

In der Datei `app/config/config.php` sehen Sie alle verfügbaren Standard-Konfigurationsvariablen.

## Grundlegende Verwendung

### Flight-Konfigurationsoptionen

Die folgende Liste enthält alle verfügbaren Konfigurationseinstellungen:

- **flight.base_url** `?string` - Überschreiben Sie die Basis-URL der Anfrage, wenn Flight in einem Unterverzeichnis ausgeführt wird. (Standard: null)
- **flight.case_sensitive** `bool` - Groß-/Kleinschreibung beachten bei der URL-Übereinstimmung. (Standard: false)
- **flight.handle_errors** `bool` - Erlauben Sie Flight, alle Fehler intern zu behandeln. (Standard: true)
  - Wenn Flight die Fehler anstelle des Standardverhaltens von PHP behandeln soll, muss dies auf true gesetzt sein.
  - Wenn Sie [Tracy](/awesome-plugins/tracy) installiert haben, sollten Sie dies auf false setzen, damit Tracy die Fehler behandeln kann.
  - Wenn Sie das [APM](/awesome-plugins/apm)-Plugin installiert haben, sollten Sie dies auf true setzen, damit APM die Fehler protokollieren kann.
- **flight.log_errors** `bool` - Fehler in die Fehlerprotokolldatei des Webservers protokollieren. (Standard: false)
  - Wenn Sie [Tracy](/awesome-plugins/tracy) installiert haben, protokolliert Tracy Fehler basierend auf den Tracy-Konfigurationen, nicht auf dieser Konfiguration.
- **flight.debug** `bool` - Detaillierte Fehlerinformationen (Ausnahmemeldung, Code und Stack-Trace) im Browser ausgeben, wenn ein Fehler auftritt. (Standard: false)
  - **Niemals in der Produktion aktivieren** — dies gibt interne Anwendungsdetails preis. Verwenden Sie es nur für die lokale Entwicklung oder Staging.
  - Wenn `false`, wird stattdessen ein generischer `500 Internal Server Error` angezeigt. Kombinieren Sie dies mit `flight.log_errors`, um Fehler serverseitig zu erfassen.
- **flight.allow_method_override** `bool` - Erlauben Sie das Überschreiben der HTTP-Methode über den `X-HTTP-Method-Override`-Request-Header oder ein `_method`-Feld im POST-Body. (Standard: true)
  - **Das Setzen auf `false` wird empfohlen** für Anwendungen, die kein HTML-Formular-basiertes Methoden-Spoofing benötigen, da dies Clients daran hindert, `DELETE`- oder `PUT`-Anfragen über ein Standard-POST-Formular zu fälschen.
  - Siehe [Sicherheit](/learn/security#flight-configuration-hardening) für weitere Details.
- **flight.views.path** `string` - Verzeichnis, das View-Template-Dateien enthält. (Standard: ./views)
- **flight.views.extension** `string` - Dateierweiterung für View-Templates. (Standard: .php)
- **flight.content_length** `bool` - Setzen Sie den `Content-Length`-Header. (Standard: true)
  - Wenn Sie [Tracy](/awesome-plugins/tracy) verwenden, muss dies auf false gesetzt werden, damit Tracy korrekt gerendert werden kann.
- **flight.v2.output_buffering** `bool` - Legacy-Ausgabepufferung verwenden. Siehe [Migration zu v3](migrating-to-v3). (Standard: false)

### Loader-Konfiguration

Es gibt zusätzlich eine weitere Konfigurationseinstellung für den Loader. Dies ermöglicht es Ihnen, Klassen mit `_` im Klassennamen automatisch zu laden.

```php
// Aktivieren des Klassenladens mit Unterstrichen
// Standardmäßig auf true gesetzt
Loader::$v2ClassLoading = false;
```

### Variablen

Flight ermöglicht es Ihnen, Variablen zu speichern, damit sie überall in Ihrer Anwendung verwendet werden können.

```php
// Speichern Sie Ihre Variable
Flight::set('id', 123);

// An anderer Stelle in Ihrer Anwendung
$id = Flight::get('id');
```
Um zu überprüfen, ob eine Variable gesetzt wurde, können Sie Folgendes tun:

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

> **Hinweis:** Nur weil Sie eine Variable setzen können, bedeutet das nicht, dass Sie es tun sollten. Verwenden Sie diese Funktion sparsam. Der Grund dafür ist, dass alles, was hier gespeichert wird, zu einer globalen Variable wird. Globale Variablen sind schlecht, weil sie von überall in Ihrer Anwendung geändert werden können, was es schwierig macht, Fehler zu verfolgen. Zusätzlich kann dies Dinge wie [Unit-Tests](/guides/unit-testing) erschweren.

### Fehler und Ausnahmen

Alle Fehler und Ausnahmen werden von Flight abgefangen und an die `error`-Methode übergeben, wenn `flight.handle_errors` auf true gesetzt ist.

Das Standardverhalten besteht darin, eine generische `HTTP 500 Internal Server Error`-Antwort mit einigen Fehlerinformationen zu senden.

Sie können dieses Verhalten für Ihre eigenen Anforderungen [überschreiben](/learn/extending):

```php
Flight::map('error', function (Throwable $error) {
  // Fehler behandeln
  echo $error->getTraceAsString();
});
```

Standardmäßig werden Fehler nicht auf dem Webserver protokolliert. Sie können dies aktivieren, indem Sie die Konfiguration ändern:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Nicht gefunden

Wenn eine URL nicht gefunden werden kann, ruft Flight die `notFound`-Methode auf. Das Standardverhalten besteht darin, eine `HTTP 404 Not Found`-Antwort mit einer einfachen Nachricht zu senden.

Sie können dieses Verhalten für Ihre eigenen Anforderungen [überschreiben](/learn/extending):

```php
Flight::map('notFound', function () {
  // Nicht gefunden behandeln
});
```

## Siehe auch
- [Flight erweitern](/learn/extending) - Wie Sie die Kernfunktionalität von Flight erweitern und anpassen.
- [Unit-Tests](/guides/unit-testing) - Wie Sie Unit-Tests für Ihre Flight-Anwendung schreiben.
- [Tracy](/awesome-plugins/tracy) - Ein Plugin für erweiterte Fehlerbehandlung und Debugging.
- [Tracy-Erweiterungen](/awesome-plugins/tracy_extensions) - Erweiterungen zur Integration von Tracy mit Flight.
- [APM](/awesome-plugins/apm) - Ein Plugin für Anwendungsleistungsüberwachung und Fehlerverfolgung.

## Fehlerbehebung
- Wenn Sie Probleme haben, alle Werte Ihrer Konfiguration zu finden, können Sie `var_dump(Flight::get());` ausführen.

## Änderungsprotokoll
- v3.18.1 - `flight.debug` und `flight.allow_method_override` Konfigurationsoptionen hinzugefügt.
- v3.5.0 - Konfiguration für `flight.v2.output_buffering` hinzugefügt, um das Legacy-Ausgabepufferungsverhalten zu unterstützen.
- v2.0 - Kernkonfigurationen hinzugefügt.