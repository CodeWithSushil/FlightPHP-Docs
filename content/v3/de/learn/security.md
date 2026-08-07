# Sicherheit

## Übersicht

Sicherheit ist ein großes Thema bei Webanwendungen. Du möchtest sicherstellen, dass deine Anwendung sicher ist und die Daten deiner Benutzer geschützt sind. Flight bietet eine Reihe von Funktionen, um deine Webanwendungen abzusichern.

Das offizielle [skeleton](https://github.com/flightphp/skeleton) enthält außerdem eine eigene **`SECURITY.md`** und eine Security-Header-Middleware, damit [KI-Codierungstools](/learn/ai) (und Menschen) einen zentralen Ort für Geheimnisse, Header und XSS/SQL-Regeln haben – getrennt vom allgemeinen Codierungsstil in `AGENTS.md`.

## Verständnis

Es gibt eine Reihe häufiger Sicherheitsbedrohungen, die du beim Erstellen von Webanwendungen kennen solltest. Zu den häufigsten Bedrohungen gehören:

- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates) helfen bei XSS, indem sie die Ausgabe standardmäßig escapen (Twig und Latte tun das; nutze diesen Vorteil). [Sessions](/awesome-plugins/session) können bei CSRF helfen, indem sie ein CSRF-Token in der Sitzung des Benutzers speichern, wie unten beschrieben. Die Verwendung von Prepared Statements mit PDO – oder Helfern auf [SimplePdo](/learn/simple-pdo) – hilft, SQL-Injection zu verhindern. CORS kann mit einem einfachen Hook vor dem Aufruf von `Flight::start()` behandelt werden.

Alle diese Methoden arbeiten zusammen, um deine Webanwendungen sicher zu halten. Es sollte dir immer bewusst sein, Sicherheits-Best-Practices zu lernen und zu verstehen. Bitte keinen KI-Assistenten darum, „CSP zu deaktivieren“ oder Header abzuschwächen, nur um eine Seite zu laden, ohne den Kompromiss zu verstehen.

## Grundlegende Verwendung

### Header

HTTP-Header sind eine der einfachsten Möglichkeiten, deine Webanwendungen abzusichern. Du kannst Header verwenden, um Clickjacking, XSS und andere Angriffe zu verhindern. Es gibt mehrere Möglichkeiten, diese Header zu deiner Anwendung hinzuzufügen.

Zwei großartige Websites, um die Sicherheit deiner Header zu überprüfen, sind [securityheaders.com](https://securityheaders.com/) und [observatory.mozilla.org](https://observatory.mozilla.org/). Nachdem du den folgenden Code eingerichtet hast, kannst du mit diesen beiden Websites leicht überprüfen, ob deine Header funktionieren.

Das Skeleton enthält `App\Middleware\SecurityHeadersMiddleware` (CSP mit einem Nonce pro Anfrage, Frame-Optionen, HSTS und mehr). Bevorzuge es, dies bewusst zu erweitern, anstatt Header abzuschalten.

#### Manuell hinzufügen

Du kannst diese Header manuell mit der Methode `header` des `Flight\Response`-Objekts hinzufügen.

```php
// Setze den X-Frame-Options-Header, um Clickjacking zu verhindern
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Setze den Content-Security-Policy-Header, um XSS zu verhindern
// Hinweis: Dieser Header kann sehr komplex werden, daher solltest du
//  Beispiele im Internet für deine Anwendung konsultieren
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Setze den X-XSS-Protection-Header, um XSS zu verhindern
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Setze den X-Content-Type-Options-Header, um MIME-Sniffing zu verhindern
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Setze den Referrer-Policy-Header, um zu steuern, wie viele Referrer-Informationen gesendet werden
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Setze den Strict-Transport-Security-Header, um HTTPS zu erzwingen
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Setze den Permissions-Policy-Header, um zu steuern, welche Funktionen und APIs verwendet werden können
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Diese können am Anfang deiner `routes.php`- oder `index.php`-Dateien hinzugefügt werden.

#### Als Filter hinzufügen

Du kannst sie auch in einem Filter/Hook wie folgt hinzufügen:

```php
// Füge die Header in einem Filter hinzu
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

Du kannst sie auch als Middleware-Klasse hinzufügen, die die größte Flexibilität bietet, auf welche Routen dies angewendet wird. Im Allgemeinen sollten diese Header auf alle HTML- und API-Antworten angewendet werden.

Skeleton-Stil Pfad und Namespace (**Ordner-Schreibweise entspricht `App\Middleware`**):

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

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
		// Bevorzuge ein CSP-Nonce aus dem Bootstrap, wenn du Inline-Skripte hast (Skeleton setzt csp_nonce)
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — leere String-Gruppe = globale Middleware für alle Routen
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// weitere Routen
}, [SecurityHeadersMiddleware::class]);
```

Ältere Projekte verwenden möglicherweise weiterhin `app/middlewares` und `app\middlewares`; das funktioniert, wenn die Ordner übereinstimmen. Neue Skeleton-Apps verwenden **`app/Middleware/`** und **`App\Middleware`**. Siehe [Autoloading](/learn/autoloading).

### Cross-Site-Request-Forgery (CSRF)

Cross-Site-Request-Forgery (CSRF) ist eine Angriffsart, bei der eine bösartige Website den Browser eines Benutzers dazu bringen kann, eine Anfrage an deine Website zu senden. Dies kann verwendet werden, um Aktionen auf deiner Website ohne das Wissen des Benutzers auszuführen. Flight bietet keinen eingebauten CSRF-Schutzmechanismus, aber du kannst deinen eigenen einfach mit Middleware implementieren.

#### Einrichtung

Zuerst musst du ein CSRF-Token generieren und es in der Sitzung des Benutzers speichern. Du kannst dieses Token dann in deinen Formularen verwenden und es beim Absenden des Formulars überprüfen. Wir verwenden das Plugin [flightphp/session](/awesome-plugins/session) zur Verwaltung von Sitzungen.

```php
// Generiere ein CSRF-Token und speichere es in der Sitzung des Benutzers
// (angenommen, du hast ein Sitzungsobjekt erstellt und an Flight angehängt)
// Weitere Informationen findest du in der Sitzungsdokumentation
Flight::register('session', flight\Session::class);

// Du musst nur ein einziges Token pro Sitzung generieren (damit es über mehrere Tabs und Anfragen für denselben Benutzer funktioniert)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Verwenden des standardmäßigen PHP-Flight-Templates

```html
<!-- Verwende das CSRF-Token in deinem Formular -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- andere Formularfelder -->
</form>
```

##### Verwenden von Twig (Skeleton-Standard)

Registriere eine Twig-Funktion oder übergib das Token an jede Formularansicht. Minimalbeispiel mit einem Global + Formularfeld:

```php
// Beim Konfigurieren von Twig (z. B. services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# andere Felder #}
</form>
```

##### Verwenden von Latte

Du kannst auch eine benutzerdefinierte Funktion einrichten, um das CSRF-Token in deinen Latte-Templates auszugeben.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// weitere Konfigurationen ...

	// Setze eine benutzerdefinierte Funktion, um das CSRF-Token auszugeben
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Und jetzt kannst du in deinen Latte-Templates die Funktion `csrf()` verwenden, um das CSRF-Token auszugeben.

```html
<form method="post">
	{csrf()}
	<!-- andere Formularfelder -->
</form>
```

#### CSRF-Token überprüfen

Du kannst das CSRF-Token mit mehreren Methoden überprüfen.

##### Middleware

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

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

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// weitere Routen
}, [CsrfMiddleware::class]);
```

##### Ereignisfilter

```php
// Diese Middleware prüft, ob die Anfrage eine POST-Anfrage ist, und wenn ja, prüft sie, ob das CSRF-Token gültig ist
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// Erfasse das CSRF-Token aus den Formularwerten
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// oder für eine JSON-Antwort
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross-Site-Scripting (XSS)

Cross-Site-Scripting (XSS) ist eine Angriffsart, bei der eine bösartige Formulareingabe Code in deine Website einschleusen kann. Die meisten dieser Möglichkeiten stammen aus Formularwerten, die deine Endbenutzer ausfüllen. Du solltest die Ausgabe deiner Benutzer **niemals** vertrauen! Gehe immer davon aus, dass alle die besten Hacker der Welt sind. Sie können bösartiges JavaScript oder HTML in deine Seite einschleusen. Dieser Code kann verwendet werden, um Informationen von deinen Benutzern zu stehlen oder Aktionen auf deiner Website auszuführen. Mit der View-Klasse von Flight oder einer Template-Engine wie [Twig](/awesome-plugins/twig) oder [Latte](/awesome-plugins/latte) kannst du die Ausgabe leicht escapen, um XSS-Angriffe zu verhindern.

```php
// Nehmen wir an, der Benutzer ist clever und versucht, dies als seinen Namen zu verwenden
$name = '<script>alert("XSS")</script>';

// Dies wird die Ausgabe escapen
Flight::view()->set('name', $name);
// Dies wird ausgegeben: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (Skeleton-Standard) und Latte escapen standardmäßig automatisch – bevorzuge sie gegenüber rohem PHP-echo
Flight::render('template', ['name' => $name]);
// Twig: {{ name }} → maskiert
// Vermeide |raw / unescaped Ausgabe, es sei denn, der Inhalt ist vollständig vertrauenswürdig
```

### SQL-Injection

SQL-Injection ist eine Angriffsart, bei der ein böswilliger Benutzer SQL-Code in deine Datenbank einschleusen kann. Dies kann verwendet werden, um Informationen aus deiner Datenbank zu stehlen oder Aktionen in deiner Datenbank auszuführen. Auch hier solltest du Eingaben von deinen Benutzern **niemals** vertrauen! Gehe immer davon aus, dass sie auf Blut aus sind. Verwende Prepared Statements – die Helfer von [SimplePdo](/learn/simple-pdo) machen dies zum Standardweg.

```php
// Angenommen, du hast Flight::db() als SimplePdo registriert (oder SimplePdo in den Controller injiziert)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (bevorzugt) — Einzeiler mit gebundenen Parametern
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Gleiche Idee mit ?-Platzhaltern
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

In Controllern im Skeleton-Stil solltest du die Konstruktor-Injektion von `SimplePdo` gegenüber `Flight::db()` bevorzugen, damit Tests und KI-generierter Code konsistent bleiben ([DIC](/learn/dependency-injection-container)).

#### Unsicheres Beispiel

Das Folgende zeigt, warum wir SQL-Prepared Statements verwenden, um vor harmlosen Beispielen wie dem folgenden zu schützen:

```php
// Endbenutzer füllt ein Webformular aus.
// Für den Wert des Formulars gibt der Hacker so etwas ein:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Nachdem die Abfrage erstellt wurde, sieht sie so aus
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Es sieht seltsam aus, aber es ist eine gültige Abfrage, die funktionieren wird. Tatsächlich
// ist es eine sehr häufige SQL-Injection-Angriffsmethode, die alle Benutzer zurückgibt.

var_dump($users); // dies wird alle Benutzer in der Datenbank ausgeben, nicht nur den einen einzelnen Benutzernamen
```

### Geheimnisse und Konfiguration

- Lege Geheimnisse in **`.env`** (oder in die echte Umgebung) ab, nicht in eingecheckte `config.php`-Beispiele.
- Skeleton-Regel: Literale Standardwerte in `config.php`; Env beim Bootstrap zusammenführen; **lies** `$_ENV` **nicht** in Controllern – injiziere stattdessen die Konfiguration. Siehe [Configuration](/learn/configuration).
- Commite niemals API-Schlüssel, Datenbankpasswörter oder Sitzungsverschlüsselungsschlüssel. Weise KI-Tools auf **`SECURITY.md`** hin, damit sie keine unsicheren Abkürzungen erfinden.

### JSONP-Callback-Validierung

Wenn du die Methode `Flight::jsonp()` von Flight verwendest, beachte, dass Flight den Namen des JSONP-Callback-Parameters gegen einen strengen Allowlist-Regex prüft (`/^[A-Za-z_$][\w$.]{0,127}$/`). Jeder Callback-Name, der diesem Muster nicht entspricht, führt dazu, dass Flight eine Ausnahme auslöst und so die Injektion von beliebigem JavaScript über einen böswilligen Callback-Wert verhindert.

Diese Validierung ist eingebaut und erfordert keine zusätzliche Konfiguration, aber es ist hilfreich, sie zu kennen, wenn unerwartete Fehler von JSONP-Endpunkten debuggt werden.

### CORS

Cross-Origin Resource Sharing (CORS) ist ein Mechanismus, der es ermöglicht, viele Ressourcen (z. B. Schriftarten, JavaScript usw.) auf einer Webseite von einer anderen Domain anzufordern, die außerhalb der Domain liegt, von der die Ressource stammt. Flight hat keine eingebaute Funktion, aber dies kann leicht mit einem Hook behandelt werden, der vor dem Aufruf der Methode `Flight::start()` ausgeführt wird.

```php
// app/Utils/CorsUtil.php (Skeleton: PascalCase-Ordner Utils → App\Utils)

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
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
		// Passe hier deine erlaubten Hosts an.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// Bootstrap / Routen — vor dem Start ausführen
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Härtung der Flight-Konfiguration

Flight legt mehrere Engine-Einstellungen offen, die direkte Sicherheitsauswirkungen haben. Diese richtig zu setzen, ist einer der einfachsten Wege, deine Anwendung zu härten.

#### `flight.allow_method_override`

Standardmäßig erlaubt Flight Clients, die HTTP-Methode einer Anfrage entweder über den `X-HTTP-Method-Override`-Header oder ein `_method`-Feld im POST-Body zu überschreiben. Das ist zwar praktisch für HTML-Formulare, die nur `GET`/`POST` senden können, kann aber gefährlich sein, wenn du es nicht erwartest – ein Angreifer könnte über ein normales Formular `DELETE`- oder `PUT`-Anfragen fälschen.

Wenn deine Anwendung sich nicht auf dieses Verhalten verlässt (z. B. wenn du eine API baust, die von modernen Clients oder JavaScript-Frontends konsumiert wird, die jedes HTTP-Verb senden können), solltest du es deaktivieren:

```php
// In deiner index.php- oder Bootstrap-Datei, vor Flight::start()
Flight::set('flight.allow_method_override', false);
```

Der Standardwert ist `true` für die Abwärtskompatibilität, aber **es wird dringend empfohlen, ihn auf `false` zu setzen** für jede Anwendung, die die Überschreibungsfunktion nicht explizit benötigt.

#### `flight.debug`

Flight hat eine Einstellung `flight.debug`, die steuert, ob detaillierte Fehlerinformationen (Ausnahmemeldung, Code und vollständiger Stack-Trace) im Browser angezeigt werden, wenn eine unbehandelte Ausnahme auftritt. Der Standardwert ist `false`, was bedeutet, dass nur eine allgemeine Meldung `500 Internal Server Error` angezeigt wird – keine internen Details werden an den Client weitergegeben.

Aktiviere dies niemals auf einem Produktionsserver. Verwende es nur lokal oder in einer Staging-Umgebung:

```php
// Nur für die lokale Entwicklung sicher — NIEMALS in der Produktion
Flight::set('flight.debug', true);
```

Wenn `flight.debug` `false` ist (Standard), kannst du Fehler dennoch erfassen, indem du `flight.log_errors` aktivierst:

```php
// Protokolliere Fehler serverseitig, ohne sie dem Client zu zeigen
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Empfohlene Produktionskonfiguration

```php
// index.php oder aus der App-Konfiguration / Bootstrap übernommen
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Fehlerbehandlung

Verstecke sensible Fehlerdetails in der Produktion, um zu vermeiden, dass Informationen an Angreifer gelangen. In der Produktion protokolliere Fehler, anstatt sie anzuzeigen, mit `display_errors` auf `0` gesetzt.

```php
// In deiner bootstrap.php oder index.php

// Füge dies zu deiner app/config/config.php hinzu
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Deaktiviere die Fehleranzeige
    ini_set('log_errors', 1);     // Protokolliere stattdessen Fehler
    ini_set('error_log', '/path/to/error.log');
}

// In deinen Routen oder Controllern
// Verwende Flight::halt() für kontrollierte Fehlerantworten
Flight::halt(403, 'Access denied');
```

### Eingabebereinigung

Vertraue niemals Benutzereingaben. Bereinige sie mit [filter_var](https://www.php.net/manual/en/function.filter-var.php), bevor du sie verarbeitest, um zu verhindern, dass bösartige Daten eingeschleust werden. Bevorzuge es, Eingaben über `$app->request()` (oder `Flight::request()`) zu lesen, anstatt rohe `$_GET` / `$_POST` im Anwendungscode zu verwenden.

```php

// Nehmen wir an, eine $_POST-Anfrage mit $_POST['input'] und $_POST['email']

// Bereinige eine Zeichenketteneingabe
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Bereinige eine E-Mail
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Passwort-Hashing

Speichere Passwörter sicher und verifiziere sie sicher mit den eingebauten PHP-Funktionen wie [password_hash](https://www.php.net/manual/en/function.password-hash.php) und [password_verify](https://www.php.net/manual/en/function.password-verify.php). Passwörter sollten niemals im Klartext gespeichert oder mit reversiblen Methoden verschlüsselt werden. Hashing stellt sicher, dass die tatsächlichen Passwörter geschützt bleiben, selbst wenn deine Datenbank kompromittiert wird.

```php
$password = Flight::request()->data->password;
// Hashe ein Passwort beim Speichern (z. B. bei der Registrierung)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verifiziere ein Passwort (z. B. beim Login)
if (password_verify($password, $stored_hash)) {
    // Passwort stimmt überein
}
```

### Ratenbegrenzung

Schütze vor Brute-Force-Angriffen oder Denial-of-Service-Angriffen, indem du die Anfrageraten mit einem Cache begrenzt.

```php
// Angenommen, du hast flightphp/cache installiert und registriert
// Verwendung von flightphp/cache in einem Filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Nach 60 Sekunden zurücksetzen
});
```

## Siehe auch

- [Sessions](/awesome-plugins/session) – So verwaltest du Benutzersitzungen sicher.
- [Templates](/learn/templates) – Twig/Latte automatisches Escaping und XSS.
- [SimplePdo](/learn/simple-pdo) – Datenbankhelfer mit Prepared Statements.
- [PdoWrapper](/learn/pdo-wrapper) – Veraltet; verwende SimplePdo für neuen Code.
- [Middleware](/learn/middleware) – So verwendest du Middleware, um das Hinzufügen von Sicherheits-Headern zu vereinfachen.
- [Configuration](/learn/configuration) – `.env` vs. literale Konfiguration, Produktions-Flags.
- [AI & Developer Experience](/learn/ai) – Halte die Sicherheitsrichtlinie in `SECURITY.md` für Agenten bereit.
- [Responses](/learn/responses) – So passt du HTTP-Antworten mit sicheren Headern an.
- [Requests](/learn/requests) – So verarbeitest und bereinigst du Benutzereingaben.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) – PHP-Funktion zur Eingabebereinigung.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) – PHP-Funktion für sicheres Passwort-Hashing.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) – PHP-Funktion zur Verifizierung von Passwort-Hashes.

## Fehlerbehebung

- Siehe den Abschnitt „Siehe auch“ oben für Informationen zur Fehlerbehebung bei Problemen mit Komponenten des Flight-Frameworks.
- Wenn CSP deine Skripte blockiert, füge ein Nonce hinzu (Skeleton-Muster) oder erlaube bestimmte Ursprünge in einer Allowlist – setze `script-src *` nicht ohne einen Plan.

## Changelog

- Docs – Skeleton `App\Middleware`, Twig CSRF/XSS-Hinweise, SimplePdo, Geheimnisse/`.env` und `SECURITY.md` für KI-freundliche Projekte.
- v3.18.1 – Abschnitt „Härtung der Flight-Konfiguration“ hinzugefügt, der `flight.allow_method_override`, `flight.debug` und JSONP-Callback-Validierung abdeckt.
- v3.1.0 – Abschnitte zu CORS, Fehlerbehandlung, Eingabebereinigung, Passwort-Hashing und Ratenbegrenzung hinzugefügt.
- v2.0 – Escaping für Standardansichten hinzugefügt, um XSS zu verhindern.