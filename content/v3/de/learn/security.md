# Sicherheit

## Übersicht

Sicherheit ist ein wichtiges Thema bei Webanwendungen. Sie möchten sicherstellen, dass Ihre Anwendung sicher ist und die Daten Ihrer Benutzer 
geschützt sind. Flight bietet eine Reihe von Funktionen, um Ihnen bei der Absicherung Ihrer Webanwendungen zu helfen.

## Verständnis

Es gibt eine Reihe gängiger Sicherheitsbedrohungen, die Sie beim Erstellen von Webanwendungen kennen sollten. Zu den häufigsten Bedrohungen
gehören:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates) helfen bei XSS, indem sie die Ausgabe standardmäßig escapen, sodass Sie sich nicht daran erinnern müssen. [Sessions](/awesome-plugins/session) können bei CSRF helfen, indem sie ein CSRF-Token in der Sitzung des Benutzers speichern, wie unten beschrieben. Die Verwendung von Prepared Statements mit PDO kann SQL-Injection-Angriffe verhindern (oder praktische Methoden in der [PdoWrapper](/learn/pdo-wrapper)-Klasse verwenden). CORS kann mit einem einfachen Hook vor dem Aufruf von `Flight::start()` gehandhabt werden.

Alle diese Methoden arbeiten zusammen, um Ihre Webanwendungen sicher zu halten. Es sollte immer im Vordergrund Ihres Denkens stehen, Sicherheits-Best-Practices zu lernen und zu verstehen.

## Grundlegende Verwendung

### Header

HTTP-Header sind eine der einfachsten Möglichkeiten, Ihre Webanwendungen abzusichern. Sie können Header verwenden, um Clickjacking, XSS und andere Angriffe zu verhindern. 
Es gibt mehrere Möglichkeiten, diese Header zu Ihrer Anwendung hinzuzufügen.

Zwei großartige Websites, um die Sicherheit Ihrer Header zu überprüfen, sind [securityheaders.com](https://securityheaders.com/) und 
[observatory.mozilla.org](https://observatory.mozilla.org/). Nachdem Sie den untenstehenden Code eingerichtet haben, können Sie leicht überprüfen, ob Ihre Header mit diesen beiden Websites funktionieren.

#### Manuell hinzufügen

Sie können diese Header manuell hinzufügen, indem Sie die `header`-Methode des `Flight\Response`-Objekts verwenden.
```php
// Setzt den X-Frame-Options-Header, um Clickjacking zu verhindern
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Setzt den Content-Security-Policy-Header, um XSS zu verhindern
// Hinweis: Dieser Header kann sehr komplex werden, daher sollten Sie
//  Beispiele im Internet für Ihre Anwendung konsultieren
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Setzt den X-XSS-Protection-Header, um XSS zu verhindern
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Setzt den X-Content-Type-Options-Header, um MIME-Sniffing zu verhindern
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Setzt den Referrer-Policy-Header, um zu steuern, wie viele Referrer-Informationen gesendet werden
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Setzt den Strict-Transport-Security-Header, um HTTPS zu erzwingen
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Setzt den Permissions-Policy-Header, um zu steuern, welche Funktionen und APIs verwendet werden können
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Diese können oben in Ihren `routes.php`- oder `index.php`-Dateien hinzugefügt werden.

#### Als Filter hinzufügen

Sie können sie auch in einem Filter/Hook wie folgt hinzufügen: 

```php
// Fügt die Header in einem Filter hinzu
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### Als Middleware hinzufügen

Sie können sie auch als Middleware-Klasse hinzufügen, die die größte Flexibilität bietet, auf welche Routen dies angewendet werden soll. Im Allgemeinen sollten diese Header auf alle HTML- und API-Antworten angewendet werden.

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php oder wo auch immer Sie Ihre Routen haben
// FYI, diese leere String-Gruppe fungiert als globale Middleware für
// alle Routen. Natürlich könnten Sie dasselbe tun und dies nur zu
// bestimmten Routen hinzufügen.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// weitere Routen
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) ist eine Art von Angriff, bei dem eine bösartige Website den Browser eines Benutzers dazu bringen kann, eine Anfrage an Ihre Website zu senden. 
Dies kann verwendet werden, um Aktionen auf Ihrer Website ohne Wissen des Benutzers durchzuführen. Flight bietet keinen integrierten CSRF-Schutzmechanismus, 
aber Sie können Ihren eigenen einfach mit Middleware implementieren.

#### Einrichtung

Zuerst müssen Sie ein CSRF-Token generieren und es in der Sitzung des Benutzers speichern. Sie können dieses Token dann in Ihren Formularen verwenden und es überprüfen, wenn 
das Formular abgesendet wird. Wir verwenden das [flightphp/session](/awesome-plugins/session)-Plugin zur Verwaltung von Sitzungen.

```php
// Generiert ein CSRF-Token und speichert es in der Sitzung des Benutzers
// (angenommen, Sie haben ein Sitzungsobjekt erstellt und es an Flight angehängt)
// siehe die Sitzungsdokumentation für weitere Informationen
Flight::register('session', flight\Session::class);

// Sie müssen nur ein einziges Token pro Sitzung generieren (damit es 
// über mehrere Tabs und Anfragen für denselben Benutzer funktioniert)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Verwendung des standardmäßigen PHP Flight Template

```html
<!-- Verwendet das CSRF-Token in Ihrem Formular -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- andere Formularfelder -->
</form>
```

##### Verwendung von Latte

Sie können auch eine benutzerdefinierte Funktion festlegen, um das CSRF-Token in Ihren Latte-Vorlagen auszugeben.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// andere Konfigurationen...

	// Setzt eine benutzerdefinierte Funktion, um das CSRF-Token auszugeben
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Und jetzt können Sie in Ihren Latte-Vorlagen die `csrf()`-Funktion verwenden, um das CSRF-Token auszugeben.

```html
<form method="post">
	{csrf()}
	<!-- andere Formularfelder -->
</form>
```

#### Überprüfung des CSRF-Tokens

Sie können das CSRF-Token mit mehreren Methoden überprüfen.

##### Middleware

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// index.php oder wo auch immer Sie Ihre Routen haben
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// weitere Routen
}, [ CsrfMiddleware::class ]);
```

##### Ereignis-Filter

```php
// Diese Middleware prüft, ob die Anfrage eine POST-Anfrage ist und wenn ja, ob das CSRF-Token gültig ist
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// erfasst das csrf-Token aus den Formularwerten
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// oder für eine JSON-Antwort
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) ist eine Art von Angriff, bei dem eine bösartige Formulareingabe Code in Ihre Website einschleusen kann. Die meisten dieser Möglichkeiten kommen 
von Formularwerten, die Ihre Endbenutzer ausfüllen. Sie sollten **nie** der Ausgabe Ihrer Benutzer vertrauen! Gehen Sie immer davon aus, dass alle 
die besten Hacker der Welt sind. Sie können bösartiges JavaScript oder HTML in Ihre Seite einschleusen. Dieser Code kann verwendet werden, um Informationen von Ihren 
Benutzern zu stehlen oder Aktionen auf Ihrer Website durchzuführen. Mit Flights View-Klasse oder einer anderen Template-Engine wie [Latte](/awesome-plugins/latte) können Sie die Ausgabe leicht escapen, um XSS-Angriffe zu verhindern.

```php
// Nehmen wir an, der Benutzer ist clever und versucht, dies als seinen Namen zu verwenden
$name = '<script>alert("XSS")</script>';

// Dies wird die Ausgabe escapen
Flight::view()->set('name', $name);
// Dies gibt aus: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Wenn Sie etwas wie Latte verwenden, das als Ihre View-Klasse registriert ist, wird dies ebenfalls automatisch escapen.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection ist eine Art von Angriff, bei dem ein bösartiger Benutzer SQL-Code in Ihre Datenbank einschleusen kann. Dies kann verwendet werden, um Informationen 
aus Ihrer Datenbank zu stehlen oder Aktionen in Ihrer Datenbank durchzuführen. Auch hier sollten Sie **nie** der Eingabe Ihrer Benutzer vertrauen! Gehen Sie immer davon aus, dass sie 
es auf Blut abgesehen haben. Sie können Prepared Statements in Ihren `PDO`-Objekten verwenden, um SQL Injection zu verhindern.

```php
// Angenommen, Sie haben Flight::db() als Ihr PDO-Objekt registriert
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// Wenn Sie die PdoWrapper-Klasse verwenden, kann dies leicht in einer Zeile erledigt werden
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Sie können dasselbe mit einem PDO-Objekt mit ?-Platzhaltern tun
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### Unsicheres Beispiel

Das Folgende ist der Grund, warum wir SQL Prepared Statements verwenden, um vor unschuldigen Beispielen wie dem untenstehenden zu schützen:

```php
// Der Endbenutzer füllt ein Webformular aus.
// Für den Wert des Formulars gibt der Hacker etwas wie dies ein:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Nachdem die Abfrage erstellt wurde, sieht sie so aus
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Es sieht seltsam aus, aber es ist eine gültige Abfrage, die funktionieren wird. Tatsächlich
// ist es ein sehr häufiger SQL-Injection-Angriff, der alle Benutzer zurückgeben wird.

var_dump($users); // dies wird alle Benutzer in der Datenbank ausgeben, nicht nur den einen einzelnen Benutzernamen
```

### JSONP Callback-Validierung

Wenn Sie Flights `Flight::jsonp()`-Methode verwenden, beachten Sie, dass Flight den JSONP-Callback-Parameter-Namen gegen eine strenge Allowlist-Regex (`/^[A-Za-z_$][\w$.]{0,127}$/`) validiert. Jeder Callback-Name, der diesem Muster nicht entspricht, führt dazu, dass Flight eine Ausnahme wirft und die Einschleusung von beliebigem JavaScript durch einen bösartigen Callback-Wert verhindert.

Diese Validierung ist eingebaut und erfordert keine zusätzliche Konfiguration, aber es lohnt sich, darüber Bescheid zu wissen, wenn unerwartete Fehler von JSONP-Endpunkten debuggt werden.

### CORS

Cross-Origin Resource Sharing (CORS) ist ein Mechanismus, der es vielen Ressourcen (z.B. Schriftarten, JavaScript usw.) auf einer Webseite ermöglicht, 
von einer anderen Domain außerhalb der Domain angefordert zu werden, von der die Ressource stammt. Flight hat keine integrierte Funktionalität, 
aber dies kann leicht mit einem Hook gehandhabt werden, der vor der `Flight::start()`-Methode ausgeführt wird.

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// passen Sie hier Ihre erlaubten Hosts an.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php oder wo auch immer Sie Ihre Routen haben
$CorsUtil = new CorsUtil();

// Dies muss vor dem Start ausgeführt werden.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Flight-Konfigurationshärtung

Flight stellt mehrere Engine-Einstellungen zur Verfügung, die direkte Sicherheitsauswirkungen haben. Diese korrekt einzustellen ist eine der einfachsten Möglichkeiten, Ihre Anwendung zu härten.

#### `flight.allow_method_override`

Standardmäßig erlaubt Flight Clients, die HTTP-Methode einer Anfrage entweder mit dem `X-HTTP-Method-Override`-Header oder einem `_method`-Feld im POST-Body zu überschreiben. Während dies praktisch für HTML-Formulare ist, die nur `GET`/`POST` senden können, kann es gefährlich sein, wenn Sie es nicht erwarten — ein Angreifer könnte `DELETE`- oder `PUT`-Anfragen über ein normales Formular fälschen.

Wenn Ihre Anwendung nicht auf diesem Verhalten beruht (z.B. wenn Sie eine API erstellen, die von modernen Clients oder JavaScript-Frontends konsumiert wird, die jedes HTTP-Verb senden können), sollten Sie es deaktivieren:

```php
// In Ihrer index.php- oder Bootstrap-Datei, vor Flight::start()
Flight::set('flight.allow_method_override', false);
```

Der Standardwert ist `true` für die Abwärtskompatibilität, aber **das Setzen auf `false` wird für jede Anwendung dringend empfohlen**, die die Override-Funktion nicht explizit benötigt.

#### `flight.debug`

Flight hat eine `flight.debug`-Einstellung, die steuert, ob detaillierte Fehlerinformationen (Ausnahmemeldung, Code und vollständiger Stack-Trace) im Browser gerendert werden, wenn eine unbehandelte Ausnahme auftritt. Der Standardwert ist `false`, was bedeutet, dass nur eine generische `500 Internal Server Error`-Meldung angezeigt wird — keine internen Details werden an den Client weitergegeben.

Aktivieren Sie dies niemals auf einem Produktionsserver. Verwenden Sie es nur lokal oder in einer Staging-Umgebung:

```php
// Nur für die lokale Entwicklung sicher — NIEMALS in der Produktion
Flight::set('flight.debug', true);
```

Wenn `flight.debug` `false` ist (der Standard), können Sie Fehler trotzdem erfassen, indem Sie `flight.log_errors` aktivieren:

```php
// Protokolliert Fehler serverseitig, ohne sie dem Client anzuzeigen
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Empfohlene Produktionskonfiguration

```php
// index.php oder app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Fehlerbehandlung
Verbergen Sie sensible Fehlerdetails in der Produktion, um zu vermeiden, dass Informationen an Angreifer weitergegeben werden. In der Produktion protokollieren Sie Fehler stattdessen, anstatt sie mit `display_errors` auf `0` anzuzeigen.

```php
// In Ihrer bootstrap.php oder index.php

// fügen Sie dies zu Ihrer app/config/config.php hinzu
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Deaktiviert die Fehleranzeige
    ini_set('log_errors', 1);     // Protokolliert Fehler stattdessen
    ini_set('error_log', '/path/to/error.log');
}

// In Ihren Routen oder Controllern
// Verwendet Flight::halt() für kontrollierte Fehlerantworten
Flight::halt(403, 'Access denied');
```

### Eingabe-Sanitization
Vertrauen Sie niemals Benutzereingaben. Bereinigen Sie sie mit [filter_var](https://www.php.net/manual/en/function.filter-var.php), bevor Sie sie verarbeiten, um zu verhindern, dass bösartige Daten einschleichen.

```php

// Nehmen wir an, es gibt eine $_POST-Anfrage mit $_POST['input'] und $_POST['email']

// Bereinigt eine String-Eingabe
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Bereinigt eine E-Mail
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Passwort-Hashing
Speichern Sie Passwörter sicher und überprüfen Sie sie sicher mit PHPs integrierten Funktionen wie [password_hash](https://www.php.net/manual/en/function.password-hash.php) und [password_verify](https://www.php.net/manual/en/function.password-verify.php). Passwörter sollten niemals im Klartext gespeichert werden, noch sollten sie mit reversiblen Methoden verschlüsselt werden. Hashing stellt sicher, dass selbst wenn Ihre Datenbank kompromittiert wird, die tatsächlichen Passwörter geschützt bleiben.

```php
$password = Flight::request()->data->password;
// Hasht ein Passwort beim Speichern (z.B. während der Registrierung)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Überprüft ein Passwort (z.B. während der Anmeldung)
if (password_verify($password, $stored_hash)) {
    // Passwort stimmt überein
}
```

### Rate Limiting
Schützen Sie sich vor Brute-Force-Angriffen oder Denial-of-Service-Angriffen, indem Sie Anforderungsraten mit einem Cache begrenzen.

```php
// Angenommen, Sie haben flightphp/cache installiert und registriert
// Verwendung von flightphp/cache in einem Filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Zurücksetzen nach 60 Sekunden
});
```

## Siehe auch
- [Sessions](/awesome-plugins/session) - Wie man Benutzersitzungen sicher verwaltet.
- [Templates](/learn/templates) - Verwendung von Templates zum automatischen Escapen von Ausgaben und zur Verhinderung von XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Vereinfachte Datenbankinteraktionen mit Prepared Statements.
- [Middleware](/learn/middleware) - Wie man Middleware zur Vereinfachung des Hinzufügens von Sicherheits-Headern verwendet.
- [Responses](/learn/responses) - Wie man HTTP-Antworten mit sicheren Headern anpasst.
- [Requests](/learn/requests) - Wie man Benutzereingaben handhabt und bereinigt.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - PHP-Funktion zur Eingabe-Sanitization.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - PHP-Funktion für sicheres Passwort-Hashing.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - PHP-Funktion zur Überprüfung gehashter Passwörter.

## Fehlerbehebung
- Siehe den Abschnitt "Siehe auch" oben für Informationen zur Fehlerbehebung im Zusammenhang mit Komponenten des Flight-Frameworks.

## Changelog
- v3.18.1 - Abschnitt zur Flight-Konfigurationshärtung hinzugefügt, der `flight.allow_method_override`, `flight.debug` und JSONP-Callback-Validierung abdeckt.
- v3.1.0 - Abschnitte zu CORS, Fehlerbehandlung, Eingabe-Sanitization, Passwort-Hashing und Rate Limiting hinzugefügt.
- v2.0 - Escaping für Standard-Views hinzugefügt, um XSS zu verhindern.