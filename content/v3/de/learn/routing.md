# Routing

## Überblick
Routing in Flight PHP ordnet URL-Muster Callback-Funktionen oder Klassenmethoden zu und ermöglicht so eine schnelle und einfache Anfrageverarbeitung. Es ist auf minimalen Overhead ausgelegt, anfängerfreundlich und ohne externe Abhängigkeiten erweiterbar.

## Verständnis
Routing ist der Kernmechanismus, der HTTP-Anfragen mit deiner Anwendungslogik in Flight verbindet. Durch die Definition von Routen legst du fest, wie verschiedene URLs bestimmten Code auslösen – sei es durch Funktionen, Klassenmethoden oder Controller-Aktionen. Das Routing-System von Flight ist flexibel und unterstützt grundlegende Muster, benannte Parameter, reguläre Ausdrücke sowie erweiterte Funktionen wie Dependency Injection und ressourcenorientiertes Routing. Dieser Ansatz hält deinen Code organisiert und wartungsfreundlich, bleibt dabei schnell und einfach für Anfänger und ist für fortgeschrittene Benutzer erweiterbar.

> **Hinweis:** Du möchtest mehr über Routing verstehen? Schau dir die Seite ["Warum ein Framework?"](/learn/why-frameworks) für eine ausführlichere Erklärung an.

## Grundlegende Verwendung

### Eine einfache Route definieren
Grundlegendes Routing in Flight erfolgt durch das Abgleichen eines URL-Musters mit einer Callback-Funktion oder einem Array aus Klasse und Methode.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Routen werden in der Reihenfolge abgeglichen, in der sie definiert sind. Die erste Route, die eine Anfrage matcht, wird aufgerufen.

### Funktionen als Callbacks verwenden
Der Callback kann jedes aufrufbare Objekt sein. Du kannst also eine normale Funktion verwenden:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Klassen und Methoden als Controller verwenden
Du kannst auch eine Methode (statisch oder nicht statisch) einer Klasse verwenden:

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// oder
Flight::route('/', [ GreetingController::class, 'hello' ]); // bevorzugte Methode
// oder
Flight::route('/', [ 'GreetingController::hello' ]);
// oder 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Oder indem du zuerst ein Objekt erstellst und dann die Methode aufrufst:

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **Hinweis:** Standardmäßig wird die Klasse `flight\Engine` immer injiziert, wenn ein Controller innerhalb des Frameworks aufgerufen wird, es sei denn, du legst dies über einen [Dependency-Injection-Container](/learn/dependency-injection-container) fest.

### Methodenspezifisches Routing

Standardmäßig werden Routenmuster gegen alle Anfragemethoden abgeglichen. Du kannst auf bestimmte Methoden reagieren, indem du einen Bezeichner vor die URL setzt.

```php
Flight::route('GET /', function () {
  echo 'Ich habe eine GET-Anfrage erhalten.';
});

Flight::route('POST /', function () {
  echo 'Ich habe eine POST-Anfrage erhalten.';
});

// Du kannst Flight::get() nicht für Routen verwenden, da dies eine Methode
//    zum Abrufen von Variablen ist, nicht zum Erstellen einer Route.
Flight::post('/', function() { /* Code */ });
Flight::patch('/', function() { /* Code */ });
Flight::put('/', function() { /* Code */ });
Flight::delete('/', function() { /* Code */ });
```

Du kannst auch mehrere Methoden auf einen einzelnen Callback abbilden, indem du ein `|`-Trennzeichen verwendest:

```php
Flight::route('GET|POST /', function () {
  echo 'Ich habe entweder eine GET- oder eine POST-Anfrage erhalten.';
});
```

### Spezielle Behandlung für HEAD- und OPTIONS-Anfragen

Flight bietet eine integrierte Behandlung für `HEAD`- und `OPTIONS`-HTTP-Anfragen:

#### HEAD-Anfragen

- **HEAD-Anfragen** werden wie `GET`-Anfragen behandelt, aber Flight entfernt automatisch den Antwort-Body, bevor er an den Client gesendet wird.
- Das bedeutet, du kannst eine Route für `GET` definieren, und HEAD-Anfragen an dieselbe URL geben nur Header zurück (keinen Inhalt), wie es HTTP-Standards erwarten.

```php
Flight::route('GET /info', function() {
    echo 'Dies ist eine Information!';
});
// Eine HEAD-Anfrage an /info gibt dieselben Header zurück, aber keinen Body.
```

#### OPTIONS-Anfragen

`OPTIONS`-Anfragen werden von Flight automatisch für jede definierte Route behandelt.
- Wenn eine OPTIONS-Anfrage empfangen wird, antwortet Flight mit einem `204 No Content`-Status und einem `Allow`-Header, der alle unterstützten HTTP-Methoden für diese Route auflistet.
- Du musst keine separate Route für OPTIONS definieren.

```php
// Für eine Route, die definiert ist als:
Flight::route('GET|POST /users', function() { /* ... */ });

// Eine OPTIONS-Anfrage an /users antwortet mit:
//
// Status: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Das Router-Objekt verwenden

Zusätzlich kannst du das Router-Objekt abrufen, das einige Hilfsmethoden für dich bereithält:

```php

$router = Flight::router();

// bildet alle Methoden ab, genau wie Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// GET-Anfrage
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* Code */});
$router->put('/users/update/@id', 	function() { /* Code */});
$router->delete('/users/@id', 		function() { /* Code */});
$router->patch('/users/@id', 		function() { /* Code */});
```

### Reguläre Ausdrücke (Regex)
Du kannst reguläre Ausdrücke in deinen Routen verwenden:

```php
Flight::route('/user/[0-9]+', function () {
  // Dies matcht /user/1234
});
```

Obwohl diese Methode verfügbar ist, wird empfohlen, benannte Parameter oder benannte Parameter mit regulären Ausdrücken zu verwenden, da sie lesbarer und leichter zu warten sind.

### Benannte Parameter
Du kannst benannte Parameter in deinen Routen angeben, die an deine Callback-Funktion übergeben werden. **Dies dient eher der Lesbarkeit der Route als allem anderen. Bitte beachte den wichtigen Hinweis im folgenden Abschnitt.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Du kannst auch reguläre Ausdrücke mit deinen benannten Parametern einbinden, indem du das `:`-Trennzeichen verwendest:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Dies matcht /bob/123
  // Aber nicht /bob/12345
});
```

> **Hinweis:** Das Abgleichen von Regex-Gruppen `()` mit Positionsparametern wird nicht unterstützt. Beispiel: `:'\(`

#### Wichtiger Hinweis

Obwohl es im obigen Beispiel so aussieht, als ob `@name` direkt mit der Variable `$name` verbunden ist, ist das nicht der Fall. Die Reihenfolge der Parameter in der Callback-Funktion bestimmt, was an sie übergeben wird. Wenn du die Reihenfolge der Parameter in der Callback-Funktion vertauschst, werden auch die Variablen vertauscht. Hier ist ein Beispiel:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

Und wenn du die folgende URL aufrufst: `/bob/123`, wäre die Ausgabe `hello, 123 (bob)!`. 
_Sei vorsichtig_, wenn du deine Routen und Callback-Funktionen einrichtest!

### Optionale Parameter
Du kannst benannte Parameter angeben, die für den Abgleich optional sind, indem du Segmente in Klammern setzt.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Dies matcht die folgenden URLs:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Alle optionalen Parameter, die nicht gematcht werden, werden als `NULL` übergeben.

### Wildcard-Routing
Der Abgleich erfolgt nur auf einzelnen URL-Segmenten. Wenn du mehrere Segmente abgleichen möchtest, kannst du den `*`-Platzhalter verwenden.

```php
Flight::route('/blog/*', function () {
  // Dies matcht /blog/2000/02/01
});
```

Um alle Anfragen an einen einzelnen Callback weiterzuleiten, kannst du Folgendes tun:

```php
Flight::route('*', function () {
  // Etwas tun
});
```

### 404 Not Found Handler

Standardmäßig sendet Flight eine sehr einfache und schlichte `HTTP 404 Not Found`-Antwort, wenn eine URL nicht gefunden werden kann.
Wenn du eine individuellere 404-Antwort wünschst, kannst du deine eigene `notFound`-Methode [mappen](/learn/extending):

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Du könntest auch Flight::render() mit einer eigenen Vorlage verwenden.
    $output = <<<HTML
		<h1>Mein benutzerdefinierter 404 Not Found</h1>
		<h3>Die von dir angeforderte Seite {$url} konnte nicht gefunden werden.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Method Not Found Handler

Standardmäßig sendet Flight eine sehr einfache und schlichte `HTTP 405 Method Not Allowed`-Antwort (z. B. Method Not Allowed. Zulässige Methoden sind: GET, POST), wenn eine URL gefunden wird, aber die Methode nicht erlaubt ist. Es wird auch ein `Allow`-Header mit den zulässigen Methoden für diese URL mitgesendet.

Wenn du eine individuellere 405-Antwort wünschst, kannst du deine eigene `methodNotFound`-Methode [mappen](/learn/extending):

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Du könntest auch Flight::render() mit einer eigenen Vorlage verwenden.
	$output = <<<HTML
		<h1>Mein benutzerdefinierter 405 Method Not Allowed</h1>
		<h3>Die von dir angeforderte Methode für {$url} ist nicht erlaubt.</h3>
		<p>Zulässige Methoden sind: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## Fortgeschrittene Verwendung

### Dependency Injection in Routen
Wenn du Dependency Injection über einen Container (PSR-11, PHP-DI, Dice, etc.) verwenden möchtest, sind die einzigen Routentypen, bei denen das verfügbar ist, entweder das direkte Erstellen des Objekts selbst und die Verwendung des Containers zum Erstellen deines Objekts, oder du kannst Strings verwenden, um die Klasse und Methode zu definieren, die aufgerufen werden sollen. Weitere Informationen findest du auf der Seite [Dependency Injection](/learn/dependency-injection-container).

Hier ist ein kurzes Beispiel:

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// etwas mit $this->db tun
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hallo, Welt! Mein Name ist {$name}!";
	}
}

// index.php

// Richte den Container mit den benötigten Parametern ein
// Weitere Informationen zu PSR-11 findest du auf der Dependency-Injection-Seite
$dice = new \Dice\Dice();

// Vergiss nicht, die Variable mit '$dice = ' neu zuzuweisen!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Registriere den Container-Handler
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Routen wie gewohnt
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// oder
Flight::route('/hello/@id', 'Greeting->hello');
// oder
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Ausführung an die nächste Route übergeben
<span class="badge bg-warning">Veraltet</span>
Du kannst die Ausführung an die nächste passende Route übergeben, indem du `true` aus deiner Callback-Funktion zurückgibst.

```php
Flight::route('/user/@name', function (string $name) {
  // Eine Bedingung prüfen
  if ($name !== "Bob") {
    // Mit der nächsten Route fortfahren
    return true;
  }
});

Flight::route('/user/*', function () {
  // Dies wird aufgerufen
});
```

Es wird nun empfohlen, [Middleware](/learn/middleware) zu verwenden, um komplexe Anwendungsfälle wie diesen zu behandeln.

### Route-Aliase
Durch die Vergabe eines Alias für eine Route kannst du diesen Alias später in deiner App dynamisch aufrufen, um ihn im Code generieren zu lassen (z. B. einen Link in einem HTML-Template oder die Erstellung einer Redirect-URL).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// oder 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// später irgendwo im Code
class UserController {
	public function update() {

		// Code zum Speichern des Benutzers...
		$id = $user['id']; // z. B. 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // gibt '/users/5' zurück
		Flight::redirect($redirectUrl);
	}
}

```

Dies ist besonders hilfreich, wenn sich deine URL ändert. Im obigen Beispiel nehmen wir an, dass users zu `/admin/users/@id` verschoben wurde.
Dank des Alias für die Route musst du nicht mehr alle alten URLs in deinem Code suchen und ändern, da der Alias nun `/admin/users/5` zurückgibt, wie im obigen Beispiel.

Route-Aliase funktionieren auch in Gruppen:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// oder
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Routeninformationen untersuchen
Wenn du die Informationen der passenden Route untersuchen möchtest, gibt es zwei Möglichkeiten:

1. Du kannst die Eigenschaft `executedRoute` auf dem Objekt `Flight::router()` verwenden.
2. Du kannst darum bitten, dass das Routenobjekt an deinen Callback übergeben wird, indem du `true` als dritten Parameter in der Routenmethode übergibst. Das Routenobjekt ist immer der letzte Parameter, der an deine Callback-Funktion übergeben wird.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Etwas mit $route tun
  // Array der abgeglichenen HTTP-Methoden
  $route->methods;

  // Array der benannten Parameter
  $route->params;

  // Passender regulärer Ausdruck
  $route->regex;

  // Enthält den Inhalt aller '*' im URL-Muster
  $route->splat;

  // Zeigt den URL-Pfad....falls du ihn wirklich brauchst
  $route->pattern;

  // Zeigt, welche Middleware dieser Route zugewiesen ist
  $route->middleware;

  // Zeigt den Alias, der dieser Route zugewiesen ist
  $route->alias;
});
```

> **Hinweis:** Die Eigenschaft `executedRoute` wird nur gesetzt, nachdem eine Route ausgeführt wurde. Wenn du versuchst, darauf vor der Ausführung einer Route zuzugreifen, ist sie `NULL`. Du kannst `executedRoute` auch in [Middleware](/learn/middleware) verwenden!

#### `true` an die Routendefinition übergeben
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Array der abgeglichenen HTTP-Methoden
  $route->methods;

  // Array der benannten Parameter
  $route->params;

  // Passender regulärer Ausdruck
  $route->regex;

  // Enthält den Inhalt aller '*' im URL-Muster
  $route->splat;

  // Zeigt den URL-Pfad....falls du ihn wirklich brauchst
  $route->pattern;

  // Zeigt, welche Middleware dieser Route zugewiesen ist
  $route->middleware;

  // Zeigt den Alias, der dieser Route zugewiesen ist
  $route->alias;
}, true);// <-- Dieser true-Parameter bewirkt das
```

### Routen-Gruppierung und Middleware
Es kann Zeiten geben, in denen du zusammengehörige Routen gruppieren möchtest (z. B. `/api/v1`).
Du kannst dies mit der `group`-Methode tun:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Matcht /api/v1/users
  });

  Flight::route('/posts', function () {
	// Matcht /api/v1/posts
  });
});
```

Du kannst sogar Gruppen von Gruppen verschachteln:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() ruft Variablen ab, es setzt keine Route! Siehe Objektkontext unten
	Flight::route('GET /users', function () {
	  // Matcht GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Matcht POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Matcht PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() ruft Variablen ab, es setzt keine Route! Siehe Objektkontext unten
	Flight::route('GET /users', function () {
	  // Matcht GET /api/v2/users
	});
  });
});
```

#### Gruppierung mit Objektkontext

Du kannst die Routengruppierung weiterhin mit dem `Engine`-Objekt auf folgende Weise verwenden:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // verwende die Variable $router
  $router->get('/users', function () {
	// Matcht GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Matcht POST /api/v1/posts
  });
});
```

> **Hinweis:** Dies ist die bevorzugte Methode zum Definieren von Routen und Gruppen mit dem `$router`-Objekt.

#### Gruppierung mit Middleware

Du kannst einer Gruppe von Routen auch Middleware zuweisen:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Matcht /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // oder [ new MyAuthMiddleware() ], wenn du eine Instanz verwenden möchtest
```

Weitere Details findest du auf der Seite [Gruppen-Middleware](/learn/middleware#grouping-middleware).

### Ressourcen-Routing
Du kannst mit der `resource`-Methode eine Reihe von Routen für eine Ressource erstellen. Dadurch wird eine Reihe von Routen für eine Ressource erstellt, die den RESTful-Konventionen folgt.

Um eine Ressource zu erstellen, gehe wie folgt vor:

```php
Flight::resource('/users', UsersController::class);
```

Im Hintergrund werden die folgenden Routen erstellt:

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

Und dein Controller verwendet die folgenden Methoden:

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **Hinweis:** Du kannst die neu hinzugefügten Routen mit `runway` anzeigen, indem du `php runway routes` ausführst.

#### Ressourcen-Routen anpassen

Es gibt einige Optionen, um die Ressourcen-Routen zu konfigurieren.

##### Alias-Basis

Du kannst die `aliasBase` konfigurieren. Standardmäßig ist der Alias der letzte Teil der angegebenen URL.
Zum Beispiel würde `/users/` zu einer `aliasBase` von `users` führen. Wenn diese Routen erstellt werden, sind die Aliase `users.index`, `users.create` usw. Wenn du den Alias ändern möchtest, setze die `aliasBase` auf den gewünschten Wert.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only und Except

Du kannst auch festlegen, welche Routen du erstellen möchtest, indem du die Optionen `only` und `except` verwendest.

```php
// Whitelist nur diese Methoden und Blacklist den Rest
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Blacklist nur diese Methoden und Whitelist den Rest
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

Dies sind im Grunde Whitelist- und Blacklist-Optionen, mit denen du festlegen kannst, welche Routen du erstellen möchtest.

##### Middleware

Du kannst auch Middleware angeben, die für jede der durch die `resource`-Methode erstellten Routen ausgeführt werden soll.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Streaming-Antworten

Du kannst jetzt Antworten an den Client mit `stream()` oder `streamWithHeaders()` streamen.
Dies ist nützlich für das Senden großer Dateien, langlaufender Prozesse oder das Erzeugen großer Antworten.
Das Streaming einer Route wird etwas anders behandelt als eine normale Route.

> **Hinweis:** Streaming-Antworten sind nur verfügbar, wenn [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) auf `false` gesetzt ist.

#### Streamen mit manuellen Headern

Du kannst eine Antwort an den Client streamen, indem du die `stream()`-Methode auf einer Route verwendest. Wenn du dies tust, musst du alle Header von Hand setzen, bevor du irgendetwas an den Client ausgibst.
Dies geschieht mit der PHP-Funktion `header()` oder der Methode `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// Offensichtlich würdest du den Pfad bereinigen und so weiter.
	$fileNameSafe = basename($filename);

	// Wenn du nach der Ausführung der Route zusätzliche Header setzen möchtest,
	// musst du sie definieren, bevor irgendetwas ausgegeben wird.
	// Sie müssen alle ein roher Aufruf der Funktion header() oder
	// ein Aufruf von Flight::response()->setRealHeader() sein.
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// oder
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'Datei nicht gefunden');
	}

	// setze die Content-Length manuell, wenn du möchtest
	header('Content-Length: '.filesize($filePath));
	// oder
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Stream die Datei an den Client, während sie gelesen wird
	readfile($filePath);

// Dies ist die magische Zeile hier
})->stream();
```

#### Streamen mit Headern

Du kannst auch die `streamWithHeaders()`-Methode verwenden, um die Header zu setzen, bevor du mit dem Streamen beginnst.

```php
Flight::route('/stream-users', function() {

	// Du kannst hier alle zusätzlichen Header hinzufügen, die du möchtest
	// Du musst nur header() oder Flight::response()->setRealHeader() verwenden

	// Wie auch immer du deine Daten abrufst, nur als Beispiel...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Dies ist erforderlich, um die Daten an den Client zu senden
		ob_flush();
	}
	echo '}';

// So setzt du die Header, bevor du mit dem Streamen beginnst.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// optionaler Statuscode, Standard ist 200
	'status' => 200
]);
```

## Siehe auch
- [Middleware](/learn/middleware) - Verwendung von Middleware mit Routen für Authentifizierung, Protokollierung usw.
- [Dependency Injection](/learn/dependency-injection-container) - Vereinfachung der Objekterstellung und -verwaltung in Routen.
- [Warum ein Framework?](/learn/why-frameworks) - Verständnis der Vorteile eines Frameworks wie Flight.
- [Erweiterung](/learn/extending) - Wie du Flight mit deiner eigenen Funktionalität erweiterst, einschließlich der `notFound`-Methode.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - PHP-Funktion für den Abgleich regulärer Ausdrücke.

## Fehlerbehebung
- Routenparameter werden nach Reihenfolge abgeglichen, nicht nach Namen. Stelle sicher, dass die Reihenfolge der Callback-Parameter der Routendefinition entspricht.
- Die Verwendung von `Flight::get()` definiert keine Route; verwende `Flight::route('GET /...')` für das Routing oder den Router-Objektkontext in Gruppen (z. B. `$router->get(...)`).
- Die Eigenschaft `executedRoute` wird nur gesetzt, nachdem eine Route ausgeführt wurde; sie ist vor der Ausführung `NULL`.
- Streaming erfordert, dass die Legacy-Output-Buffering-Funktion von Flight deaktiviert ist (`flight.v2.output_buffering = false`).
- Für Dependency Injection unterstützen nur bestimmte Routendefinitionen die containerbasierte Instanziierung.

### 404 Not Found oder unerwartetes Routenverhalten

Wenn du einen 404 Not Found-Fehler siehst (obwohl du dir sicher bist, dass die Route wirklich existiert und es kein Tippfehler ist), könnte das tatsächlich ein Problem damit sein, dass du in deinem Routen-Endpunkt einen Wert zurückgibst, anstatt ihn nur auszugeben. Der Grund dafür ist beabsichtigt, kann aber einige Entwickler überraschen.

```php
Flight::route('/hello', function(){
	// Dies könnte einen 404 Not Found-Fehler verursachen
	return 'Hello World';
});

// Was du wahrscheinlich willst
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

Der Grund dafür ist ein spezieller Mechanismus im Router, der den Rückgabewert als Signal zum "Weitergehen zur nächsten Route" behandelt.
Du kannst dieses Verhalten im Abschnitt [Routing](/learn/routing#passing) dokumentiert sehen.

## Änderungsprotokoll
- v3: Ressourcen-Routing, Route-Aliase, Streaming-Unterstützung, Routengruppen und Middleware-Unterstützung hinzugefügt.
- v1: Überwiegender Teil der grundlegenden Funktionen verfügbar.