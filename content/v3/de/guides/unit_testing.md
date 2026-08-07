# Unit-Tests in Flight PHP mit PHPUnit

Diese Anleitung führt in Unit-Tests in Flight PHP mit [PHPUnit](https://phpunit.de/) ein und richtet sich an Anfänger, die verstehen möchten, *warum* Unit-Tests wichtig sind und wie man sie praktisch anwendet. Wir konzentrieren uns darauf, *Verhalten* zu testen – sicherzustellen, dass Ihre Anwendung das tut, was Sie erwarten, wie das Senden einer E-Mail oder das Speichern eines Datensatzes – und nicht auf triviale Berechnungen. Wir beginnen mit einem einfachen [Route-Handler](/learn/routing) und gehen zu einem komplexeren [Controller](/learn/routing) über, einschließlich [Abhängigkeitsinjektion](/learn/dependency-injection-container) (DI) und dem Mocken von Diensten von Drittanbietern.

## Warum Unit-Tests?

Unit-Tests stellen sicher, dass Ihr Code sich wie erwartet verhält, und fangen Fehler, bevor sie in die Produktion gelangen. Sie sind besonders wertvoll in Flight, wo leichtgewichtiges Routing und Flexibilität zu komplexen Interaktionen führen können. Für Einzelentwickler oder Teams dienen Unit-Tests als Sicherheitsnetz, dokumentieren erwartetes Verhalten und verhindern Regressionen, wenn Sie Code später erneut aufrufen. Sie verbessern auch das Design: Schwer zu testender Code deutet oft auf übermäßig komplexe oder eng gekoppelte Klassen hin.

Im Gegensatz zu vereinfachten Beispielen (z. B. Testen von `x * y = z`) konzentrieren wir uns auf reale Verhaltensweisen wie das Validieren von Eingaben, das Speichern von Daten oder das Auslösen von Aktionen wie E-Mails. Unser Ziel ist es, Tests zugänglich und aussagekräftig zu machen.

## Allgemeine Leitprinzipien

1. **Verhalten testen, nicht Implementierung**: Konzentrieren Sie sich auf Ergebnisse (z. B. „E-Mail gesendet“ oder „Datensatz gespeichert“) statt auf interne Details. Das macht Tests robust gegenüber Refactoring.
2. **Hören Sie auf, `Flight::` zu verwenden**: Die statischen Methoden von Flight sind sehr praktisch, erschweren aber das Testen. Sie sollten sich daran gewöhnen, die Variable `$app` aus `$app = Flight::app();` zu verwenden. `$app` hat alle Methoden, die auch `Flight::` hat. Sie können weiterhin `$app->route()` oder `$this->app->json()` in Ihrem Controller usw. verwenden. Sie sollten auch den echten Flight-Router verwenden mit `$router = $app->router()` und dann `$router->get()`, `$router->post()`, `$router->group()` usw. Siehe [Routing](/learn/routing).
3. **Halten Sie Tests schnell**: Schnelle Tests fördern die häufige Ausführung. Vermeiden Sie langsame Operationen wie Datenbankaufrufe in Unit-Tests. Wenn ein Test langsam ist, ist das ein Zeichen dafür, dass Sie einen Integrationstest schreiben, keinen Unit-Test. Integrationstests sind Tests, bei denen echte Datenbanken, echte HTTP-Aufrufe, echtes E-Mail-Versenden usw. beteiligt sind. Sie haben ihre Berechtigung, aber sie sind langsam und können unbeständig sein, das heißt, sie schlagen manchmal aus unbekannten Gründen fehl.
4. **Verwenden Sie aussagekräftige Namen**: Testnamen sollten das zu testende Verhalten klar beschreiben. Das verbessert Lesbarkeit und Wartbarkeit.
5. **Vermeiden Sie Globale wie die Pest**: Minimieren Sie die Verwendung von `$app->set()` und `$app->get()`, da sie wie globaler Zustand wirken und in jedem Test Mocks erfordern. Bevorzugen Sie DI oder einen DI-Container (siehe [Abhängigkeitsinjektions-Container](/learn/dependency-injection-container)). Auch die Verwendung der Methode `$app->map()` ist technisch gesehen ein „Global“ und sollte zugunsten von DI vermieden werden. Verwenden Sie eine Session-Bibliothek wie [flightphp/session](https://github.com/flightphp/session), damit Sie das Session-Objekt in Ihren Tests mocken können. **Rufen Sie** [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) **nicht** direkt in Ihrem Code auf, da dies eine globale Variable in Ihren Code einbringt und das Testen erschwert.
6. **Verwenden Sie Abhängigkeitsinjektion**: Injizieren Sie Abhängigkeiten (z. B. [`PDO`](https://www.php.net/manual/en/class.pdo.php), Mailer) in Controller, um Logik zu isolieren und Mocken zu vereinfachen. Wenn eine Klasse zu viele Abhängigkeiten hat, sollten Sie erwägen, sie in kleinere Klassen zu refaktorisieren, die jeweils gemäß den [SOLID-Prinzipien](https://en.wikipedia.org/wiki/SOLID) eine einzige Verantwortung haben.
7. **Mocken Sie Dienste von Drittanbietern**: Mocken Sie Datenbanken, HTTP-Clients (cURL) oder E-Mail-Dienste, um externe Aufrufe zu vermeiden. Testen Sie ein oder zwei Ebenen tief, aber lassen Sie Ihre Kernlogik laufen. Wenn Ihre App zum Beispiel eine Textnachricht sendet, möchten Sie **NICHT** wirklich jedes Mal eine Textnachricht senden, wenn Sie Ihre Tests ausführen, da sich diese Kosten summieren (und es langsamer wird). Stattdessen mocken Sie den Textnachrichtendienst und stellen nur sicher, dass Ihr Code den Textnachrichtendienst mit den richtigen Parametern aufgerufen hat.
8. **Zielen Sie auf hohe Abdeckung, nicht auf Perfektion**: 100% Zeilenabdeckung ist gut, aber es bedeutet nicht wirklich, dass alles in Ihrem Code so getestet ist, wie es sein sollte (recherchieren Sie ruhig [Zweig-/Pfadabdeckung in PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Priorisieren Sie kritisches Verhalten (z. B. Benutzerregistrierung, API-Antworten und das Erfassen fehlgeschlagener Antworten).
9. **Verwenden Sie Controller für Routen**: Verwenden Sie in Ihren Routendefinitionen Controller und keine Closures. Das `flight\Engine $app` wird standardmäßig über den Konstruktor in jeden Controller injiziert. Verwenden Sie in Tests `$app = new Flight\Engine()`, um Flight innerhalb eines Tests zu instanziieren, injizieren Sie es in Ihren Controller und rufen Sie Methoden direkt auf (z. B. `$controller->register()`). Siehe [Flight erweitern](/learn/extending) und [Routing](/learn/routing).
10. **Wählen Sie einen Mocking-Stil und bleiben Sie dabei**: PHPUnit unterstützt mehrere Mocking-Stile (z. B. Prophecy, integrierte Mocks), oder Sie können anonyme Klassen verwenden, die ihre eigenen Vorteile haben, wie Codevervollständigung, Fehler bei geänderter Methodendefinition usw. Seien Sie einfach konsistent in Ihren Tests. Siehe [PHPUnit-Mock-Objekte](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Verwenden Sie `protected`-Sichtbarkeit für Methoden/Eigenschaften, die Sie in Unterklassen testen möchten**: Dies ermöglicht es Ihnen, sie in Testunterklassen zu überschreiben, ohne sie öffentlich zu machen. Das ist besonders nützlich für anonyme Klassen-Mocks.

## Einrichten von PHPUnit

Richten Sie zuerst [PHPUnit](https://phpunit.de/) in Ihrem Flight-PHP-Projekt mit Composer ein, um einfaches Testen zu ermöglichen. Weitere Details finden Sie im [PHPUnit-Getting-Started-Leitfaden](https://phpunit.readthedocs.io/en/12.3/installation.html).

1. Führen Sie in Ihrem Projektverzeichnis aus:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Dies installiert die neueste PHPUnit-Version als Entwicklungsabhängigkeit.

2. Erstellen Sie ein `tests`-Verzeichnis im Projektstamm für Testdateien.

3. Fügen Sie `composer.json` ein Testskript hinzu, der Einfachheit halber:
   ```json
   // weiterer Inhalt von composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Erstellen Sie eine `phpunit.xml`-Datei im Stammverzeichnis:
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

Wenn Ihre Tests erstellt sind, können Sie `composer test` ausführen, um die Tests auszuführen.

## Testen eines einfachen Route-Handlers

Beginnen wir mit einer einfachen [Route](/learn/routing), die die E-Mail-Eingabe eines Benutzers validiert. Wir testen ihr Verhalten: Für gültige E-Mails wird eine Erfolgsmeldung zurückgegeben, für ungültige ein Fehler. Für die E-Mail-Validierung verwenden wir [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

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
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

Um dies zu testen, erstellen Sie eine Testdatei. Siehe [Unit-Tests und SOLID-Prinzipien](/learn/unit-testing-and-solid-principles) für weitere Informationen zur Strukturierung von Tests:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simuliere POST-Daten
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Simuliere POST-Daten
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Wichtige Punkte**:
- Wir simulieren POST-Daten mit der Request-Klasse. Verwenden Sie keine Globals wie `$_POST`, `$_GET` usw., da dies das Testen komplizierter macht (Sie müssen diese Werte immer zurücksetzen, sonst könnten andere Tests fehlschlagen).
- Alle Controller erhalten standardmäßig die `flight\Engine`-Instanz injiziert, auch ohne dass ein DIC-Container eingerichtet wurde. Das macht es viel einfacher, Controller direkt zu testen.
- Es gibt überhaupt keine Verwendung von `Flight::`, was den Code leichter testbar macht.
- Tests verifizieren das Verhalten: korrekter Status und Meldung für gültige/ungültige E-Mails.

Führen Sie `composer test` aus, um zu überprüfen, ob die Route wie erwartet funktioniert. Weitere Informationen zu [Requests](/learn/requests) und [Responses](/learn/responses) in Flight finden Sie in der entsprechenden Dokumentation.

## Verwenden von Abhängigkeitsinjektion für testbare Controller

Für komplexere Szenarien verwenden Sie [Abhängigkeitsinjektion](/learn/dependency-injection-container) (DI), um Controller testbar zu machen. Vermeiden Sie die Globals von Flight (z. B. `Flight::set()`, `Flight::map()`, `Flight::register()`), da sie wie globaler Zustand wirken und für jeden Test Mocks erfordern. Verwenden Sie stattdessen den DI-Container von Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) oder manuelle DI.

Verwenden wir [`flight\database\SimplePdo`](/learn/simple-pdo) anstelle von rohem PDO. Dieser Helfer ist viel einfacher zu mocken und zu testen (und wird gegenüber dem veralteten `PdoWrapper` bevorzugt).

Hier ist ein Controller, der einen Benutzer in einer Datenbank speichert und eine Willkommens-E-Mail sendet:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// das Hinzufügen des return hier hilft beim Unit-Testen, die Ausführung zu stoppen
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Wichtige Punkte**:
- Der Controller hängt von einer [`SimplePdo`](/learn/simple-pdo)-Instanz und einem `MailerInterface` (einem angenommenen E-Mail-Dienst von Drittanbietern) ab.
- Abhängigkeiten werden über den Konstruktor injiziert, sodass Globals vermieden werden.

### Testen des Controllers mit Mocks

Jetzt testen wir das Verhalten des `UserController`: Validieren von E-Mails, Speichern in der Datenbank und Senden von E-Mails. Wir mocken die Datenbank und den Mailer, um den Controller zu isolieren.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Manchmal ist das Mischen von Mocking-Stilen notwendig
		// Hier verwenden wir den eingebauten Mock von PHPUnit für PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Verwenden einer anonymen Klasse zum Mocken von SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Wenn wir es auf diese Weise mocken, führen wir keinen echten Datenbankaufruf durch.
			// Wir können dies weiter einrichten, um den PDOStatement-Mock zu ändern, um Fehler zu simulieren usw.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Ein leerer Konstruktor umgeht den Elternkonstruktor
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// jsonHalt muss gemappt werden, um das Beenden zu vermeiden
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**Wichtige Punkte**:
- Wir mocken `SimplePdo` und `MailerInterface`, um echte Datenbank- oder E-Mail-Aufrufe zu vermeiden.
- Tests verifizieren das Verhalten: Gültige E-Mails lösen Datenbankeinfügungen und E-Mail-Versand aus; ungültige E-Mails überspringen beides.
- Mocken Sie Abhängigkeiten von Drittanbietern (z. B. `SimplePdo`, `MailerInterface`) und lassen Sie die Logik des Controllers laufen.

### Zu viel mocken

Seien Sie vorsichtig, nicht zu viel von Ihrem Code zu mocken. Ich gebe Ihnen unten ein Beispiel, warum das schlecht sein kann, unter Verwendung unseres `UserController`. Wir ändern diese Prüfung in eine Methode namens `isEmailValid` (unter Verwendung von `filter_var`) und die anderen neuen Ergänzungen in eine separate Methode namens `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// das Hinzufügen des return hier hilft beim Unit-Testen, die Ausführung zu stoppen
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

Und nun der übermäßig gemockte Unit-Test, der eigentlich nichts testet:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// wir überspringen die zusätzliche Abhängigkeitsinjektion hier, weil es „einfach“ ist
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Abhängigkeiten im Konstruktor umgehen
			public function __construct($app) {
				$this->app = $app;
			}

			// Wir erzwingen einfach, dass dies gültig ist.
			protected function isEmailValid($email) {
				return true; // Immer true zurückgeben, echte Validierung umgehen
			}

			// Die tatsächlichen DB- und Mailer-Aufrufe umgehen
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

Hurra, wir haben Unit-Tests und sie bestehen! Aber Moment, was, wenn ich tatsächlich die internen Abläufe von `isEmailValid` oder `registerUser` ändere? Meine Tests bestehen weiterhin, weil ich die gesamte Funktionalität gemockt habe. Lassen Sie mich zeigen, was ich meine.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... andere Methoden ...

	protected function isEmailValid($email) {
		// Geänderte Logik
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Jetzt sollte es nur eine bestimmte Domain haben
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Wenn ich meine obigen Unit-Tests ausführe, bestehen sie immer noch! Aber weil ich nicht auf Verhalten getestet habe (also einen Teil des Codes tatsächlich habe laufen lassen), habe ich möglicherweise einen Fehler programmiert, der in der Produktion nur darauf wartet, aufzutreten. Der Test sollte geändert werden, um das neue Verhalten zu berücksichtigen, und auch das Gegenteil, wenn das Verhalten nicht unseren Erwartungen entspricht.

## Vollständiges Beispiel

Ein vollständiges Beispiel eines Flight-PHP-Projekts mit Unit-Tests finden Sie auf GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide).
Für ein tieferes Verständnis siehe [Unit-Tests und SOLID-Prinzipien](/learn/unit-testing-and-solid-principles).

## Häufige Fallstricke

- **Übermäßiges Mocken**: Mocken Sie nicht jede Abhängigkeit; lassen Sie etwas Logik (z. B. Controller-Validierung) laufen, um echtes Verhalten zu testen. Siehe [Unit-Tests und SOLID-Prinzipien](/learn/unit-testing-and-solid-principles).
- **Globaler Zustand**: Die starke Verwendung globaler PHP-Variablen (z. B. [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) macht Tests brüchig. Gleiches gilt für `Flight::`. Refaktorisieren Sie, um Abhängigkeiten explizit zu übergeben.
- **Komplexe Einrichtung**: Wenn die Testeinrichtung umständlich ist, hat Ihre Klasse möglicherweise zu viele Abhängigkeiten oder Verantwortlichkeiten, die gegen die [SOLID-Prinzipien](/learn/unit-testing-and-solid-principles) verstoßen.

## Skalierung mit Unit-Tests

Unit-Tests glänzen in größeren Projekten oder wenn Sie Code nach Monaten erneut aufrufen. Sie dokumentieren Verhalten und fangen Regressionen ab, sodass Sie Ihre App nicht neu lernen müssen. Für Einzelentwickler: Testen Sie kritische Pfade (z. B. Benutzeranmeldung, Zahlungsabwicklung). Für Teams: Tests stellen konsistentes Verhalten über Beiträge hinweg sicher. Siehe [Warum Frameworks?](/learn/why-frameworks) für weitere Vorteile von Frameworks und Tests.

Tragen Sie Ihre eigenen Testtipps zum Flight-PHP-Dokumentations-Repository bei!

_Geschrieben von [n0nag0n](https://github.com/n0nag0n) 2025_