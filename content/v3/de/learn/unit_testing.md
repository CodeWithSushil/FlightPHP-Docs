# Unit-Tests

## Übersicht

Unit-Tests in Flight helfen Ihnen, sicherzustellen, dass Ihre Anwendung wie erwartet funktioniert, Fehler früh zu erkennen und Ihre Codebasis einfacher zu warten. Flight ist so konzipiert, dass es reibungslos mit [PHPUnit](https://phpunit.de/) funktioniert, dem beliebtesten PHP-Testframework.

## Verständnis

Unit-Tests prüfen das Verhalten kleiner Teile Ihrer Anwendung (wie Controller oder Services) isoliert. In Flight bedeutet dies, zu testen, wie Ihre Routen, Controller und Logik auf verschiedene Eingaben reagieren – ohne sich auf globalen Zustand oder echte externe Dienste zu verlassen.

Wichtige Grundsätze:
- **Verhalten testen, nicht Implementierung:** Konzentrieren Sie sich darauf, was Ihr Code tut, nicht darauf, wie er es tut.
- **Globalen Zustand vermeiden:** Verwenden Sie Dependency Injection anstelle von `Flight::set()` oder `Flight::get()`.
- **Externe Dienste mocken:** Ersetzen Sie Dinge wie Datenbanken oder Mailer durch Test-Doubles.
- **Tests schnell und fokussiert halten:** Unit-Tests sollten keine echten Datenbanken oder APIs verwenden.

## Grundlegende Verwendung

### PHPUnit einrichten

1. PHPUnit mit Composer installieren:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Erstellen Sie ein `tests`-Verzeichnis im Stammverzeichnis Ihres Projekts.
3. Fügen Sie ein Test-Skript zu Ihrer `composer.json` hinzu:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Erstellen Sie eine `phpunit.xml`-Datei:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Jetzt können Sie Ihre Tests mit `composer test` ausführen.

### Testen eines einfachen Routen-Handlers

Angenommen, Sie haben eine Route, die eine E-Mail validiert:

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
    protected $app;
    public function __construct(flight\Engine $app) {
        $this->app = $app;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Ein einfacher Test für diesen Controller:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Tipps:**
- Simulieren Sie POST-Daten mit `$app->request()->data`.
- Vermeiden Sie die Verwendung von statischen `Flight::`-Aufrufen in Ihren Tests – verwenden Sie die `$app`-Instanz.

### Verwenden von Dependency Injection für testbare Controller

Injizieren Sie Abhängigkeiten (wie Datenbank oder Mailer) in Ihre Controller, um sie in Tests einfach mocken zu können:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

Und ein Test mit Mocks:

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Erweiterte Verwendung

- **Mocking:** Verwenden Sie die integrierten Mocks von PHPUnit oder anonyme Klassen, um Abhängigkeiten zu ersetzen.
- **Controller direkt testen:** Instanziieren Sie Controller mit einer neuen `Engine` und mocken Sie Abhängigkeiten.
- **Übermäßiges Mocken vermeiden:** Lassen Sie echte Logik wo möglich laufen; mocken Sie nur externe Dienste.

## Siehe auch

- [Leitfaden für Unit-Tests](/guides/unit-testing) – Ein umfassender Leitfaden zu Best Practices für Unit-Tests.
- [Dependency-Injection-Container](/learn/dependency-injection-container) – So verwenden Sie DICs, um Abhängigkeiten zu verwalten und die Testbarkeit zu verbessern.
- [Erweitern](/learn/extending) – So fügen Sie eigene Helfer hinzu oder überschreiben Kernklassen.
- [SimplePdo](/learn/simple-pdo) – Vereinfacht Datenbankinteraktionen und ist in Tests einfacher zu mocken.
- [Anfragen](/learn/requests) – Verarbeitung von HTTP-Anfragen in Flight.
- [Antworten](/learn/responses) – Senden von Antworten an Benutzer.
- [Unit-Tests und SOLID-Prinzipien](/learn/unit-testing-and-solid-principles) – Erfahren Sie, wie SOLID-Prinzipien Ihre Unit-Tests verbessern können.

## Fehlerbehebung

- Vermeiden Sie die Verwendung von globalem Zustand (`Flight::set()`, `$_SESSION`, usw.) in Ihrem Code und Ihren Tests.
- Wenn Ihre Tests langsam sind, schreiben Sie möglicherweise Integrationstests – mocken Sie externe Dienste, um Unit-Tests schnell zu halten.
- Wenn die Testeinrichtung komplex ist, ziehen Sie in Betracht, Ihren Code zu refaktorieren, um Dependency Injection zu verwenden.

## Änderungsprotokoll

- v3.15.0 – Beispiele für Dependency Injection und Mocking hinzugefügt.