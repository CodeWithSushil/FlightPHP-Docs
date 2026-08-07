# Autoloading

## Übersicht

Autoloading ist ein Konzept in PHP, bei dem Sie ein Verzeichnis oder mehrere Verzeichnisse angeben, aus denen Klassen geladen werden. Dies ist viel vorteilhafter als `require` oder `include` zum Laden von Klassen zu verwenden. Es ist auch eine Voraussetzung für die Verwendung von Composer-Paketen.

Ein korrektes Autoloading ist auch für KI-gestützte Entwicklung wichtig: Agents legen Dateien dort ab, wohin der Namespace zeigt. Wenn die Groß-/Kleinschreibung von Ordnern und Namespaces nicht übereinstimmt, treten unter Linux Fehler wie „Klasse nicht gefunden" auf, selbst wenn die Dinge auf einer Mac-Festplatte ohne Beachtung der Groß-/Kleinschreibung funktioniert haben.

## Verständnis

Standardmäßig wird jede `Flight`-Klasse dank Composer automatisch für Sie geladen. Für **Ihre** Anwendungsklassen gibt es zwei gängige Ansätze:

1. **Composer PSR-4** (was das [offizielle Grundgerüst](https://github.com/flightphp/skeleton) verwendet): Ordnen Sie ein Namespace-Präfix einem Verzeichnis in `composer.json` zu und führen Sie dann `composer dump-autoload` aus.
2. **`Flight::path()`**: Weisen Sie den Loader von Flight auf Verzeichnisse hin (praktisch für einfache Apps oder wenn Sie Composer nicht für Anwendungscode verwenden).

Die Verwendung eines Autoloaders vereinfacht Ihren Code erheblich. Anstatt einer Wand von `include` / `require` am Anfang jeder Datei werden Klassen geladen, wenn Sie sie zum ersten Mal verwenden.

### Groß-/Kleinschreibung (lesen Sie dies zweimal)

**Namespaces müssen mit der Verzeichnisstruktur *und* der Groß-/Kleinschreibung dieser Verzeichnisse übereinstimmen.**

| Funktioniert | Schlägt unter Linux fehl |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` mit Ordner `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Mischung aus `App\` mit kleingeschriebenem `controllers` |

PHP-Namespaces sind in manchen Kontexten case-insensitiv (Groß-/Kleinschreibung wird ignoriert), aber **Composer und das Dateisystem sind es nicht**. Das offizielle Grundgerüst standardisiert auf:

- Composer: `"App\\": "app/"`
- Ordner: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), nicht `controllers` / `middlewares`

Ältere Dokumentationen und Beispiele aus der Community verwendeten manchmal kleingeschriebenes `app\controllers`. Das funktioniert weiterhin, wenn Ihre Ordner kleingeschrieben sind – aber **neue Skeleton-Projekte verwenden `App\` + PascalCase-Ordner**. Wählen Sie eine Konvention pro Projekt und bleiben Sie dabei, damit Menschen und KI-Tools kein zweites Layout erfinden.

## Skeleton (empfohlen für neue Projekte)

Nach `composer create-project flightphp/skeleton` wird Anwendungscode über Composer automatisch geladen – für `App\`-Klassen ist kein `Flight::path()` erforderlich:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php – Dice löst App\Controller\… über den Container auf
$router->get('/', [HomeController::class, 'index']);
```

Siehe [Installation](/install) für den vollständigen Verzeichnisbaum und [KI & Entwicklererfahrung](/learn/ai) für die Dokumentation dieses Layouts für Programmierassistenten in `AGENTS.md`.

## Grundlegende Verwendung (`Flight::path()`)

Nehmen wir an, wir haben einen Verzeichnisbaum wie den folgenden:

```text
# Beispielpfad
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers – enthält die Controller für dieses Projekt
│   ├── translations
│   ├── UTILS – enthält Klassen nur für diese Anwendung (dies ist absichtlich in Großbuchstaben für ein späteres Beispiel)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Ihnen ist vielleicht aufgefallen, dass dies einem typischen App-Verzeichnisbaum ähnelt (die Dokumentationsseite selbst verwendet eine strukturierte Darstellung). Kleingeschriebenes `controllers` ist hier eine *bewusste Wahl* – es ist nur nicht die aktuelle Standardeinstellung des Skeletons.

Sie können jedes Verzeichnis zum Laden wie folgt angeben:

```php

/**
 * public/index.php
 */

// Fügen Sie einen Pfad zum Autoloader hinzu
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// Keine Namespaces erforderlich

// Alle automatisch geladenen Klassen sollten in Pascal Case sein (jedes Wort großgeschrieben, keine Leerzeichen)
class MyController {

	public function index() {
		// etwas tun
	}
}
```

## Namespaces mit `Flight::path()`

Wenn Sie Namespaces verwenden, wird die Implementierung tatsächlich sehr einfach. Sie sollten die Methode `Flight::path()` verwenden, um das Wurzelverzeichnis (nicht das Dokumentenwurzelverzeichnis oder den `public/`-Ordner) Ihrer Anwendung anzugeben.

```php

/**
 * public/index.php
 */

// Fügen Sie einen Pfad zum Autoloader hinzu
Flight::path(__DIR__.'/../');
```

So könnte Ihr Controller nun aussehen. Schauen Sie sich das folgende Beispiel an, aber achten Sie auf die Kommentare mit wichtigen Informationen.

```php
/**
 * app/controllers/MyController.php
 */

// Namespaces sind erforderlich
// Namespaces entsprechen der Verzeichnisstruktur
// Namespaces müssen die gleiche Groß-/Kleinschreibung wie die Verzeichnisstruktur verwenden
// Namespaces und Verzeichnisse dürfen keine Unterstriche enthalten (außer wenn Loader::setV2ClassLoading(false) gesetzt ist)
namespace app\controllers;

// Alle automatisch geladenen Klassen sollten in Pascal Case sein (jedes Wort großgeschrieben, keine Leerzeichen)
// Ab 3.7.2 können Sie Pascal_Snake_Case für Ihre Klassennamen verwenden, indem Sie Loader::setV2ClassLoading(false); ausführen.
class MyController {

	public function index() {
		// etwas tun
	}
}
```

Und wenn Sie eine Klasse in Ihrem utils-Verzeichnis automatisch laden möchten, gehen Sie im Grunde genauso vor:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// Namespace muss mit der Verzeichnisstruktur und der Groß-/Kleinschreibung übereinstimmen (beachten Sie, dass das UTILS-Verzeichnis komplett groß geschrieben ist
//     wie im obigen Verzeichnisbaum)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// etwas tun
	}
}
```

### Skeleton-Stil-Namespace (gleiche Regeln, andere Groß-/Kleinschreibung)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

Die Regel hat sich nicht geändert – nur die vom Skeleton gewählte Groß-/Kleinschreibung von Ordnern und Namespaces. **Egal welche Groß-/Kleinschreibung Ihre Ordner verwenden, Ihre `namespace`-Zeile muss übereinstimmen.**

## Unterstriche in Klassennamen

Ab 3.7.2 können Sie Pascal_Snake_Case für Ihre Klassennamen verwenden, indem Sie `Loader::setV2ClassLoading(false);` ausführen. Dadurch können Sie Unterstriche in Ihren Klassennamen verwenden. Dies wird nicht empfohlen, ist aber für diejenigen verfügbar, die es benötigen.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Fügen Sie einen Pfad zum Autoloader hinzu
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// Keine Namespaces erforderlich

class My_Controller {

	public function index() {
		// etwas tun
	}
}
```

## Siehe auch

- [Installation](/install) – Skeleton-Verzeichnisbaum und `App\`-Standardwerte für neue Projekte.
- [Routing](/learn/routing) – Wie Sie Routen auf Controller abbilden und Ansichten rendern.
- [Dependency Injection](/learn/dependency-injection-container) – Wie Controller `Engine` und Dienste erhalten.
- [KI & Entwicklererfahrung](/learn/ai) – Halten Sie Agents mit Ihrem Layout durch `AGENTS.md` im Einklang.
- [Warum ein Framework?](/learn/why-frameworks) – Vorteile der Verwendung eines Frameworks wie Flight verstehen.

## Fehlerbehebung

Wenn Sie nicht herausfinden können, warum Ihre Namespace-Klassen nicht gefunden werden, denken Sie daran: Verwenden Sie mit `Flight::path()` das **Projektwurzelverzeichnis** (oder die korrekte Basis für Ihren Namespace), nicht nur einen verschachtelten Ordner, den Sie im Namespace vergessen haben zu spiegeln.

Führen Sie bei Composer PSR-4 nach Änderungen an den `composer.json`-Zuordnungen `composer dump-autoload` aus.

Bei Linux-CI oder Produktion ist eine falsche Groß-/Kleinschreibung von Ordnern ein sehr häufiger Fall von „Funktioniert auf meinem Rechner"-Fehlern.

### Klasse nicht gefunden (Autoloading funktioniert nicht)

Es kann mehrere Gründe geben, warum dies nicht funktioniert. Im Folgenden finden Sie einige Beispiele.

#### Falscher Dateiname

Der häufigste Grund ist, dass der Klassenname nicht mit dem Dateinamen übereinstimmt.

Wenn Sie eine Klasse mit dem Namen `MyClass` haben, sollte die Datei `MyClass.php` heißen. Wenn Sie eine Klasse mit dem Namen `MyClass` haben und die Datei `myclass.php` heißt, kann der Autoloader sie nicht finden.

#### Falscher Namespace oder falsche Groß-/Kleinschreibung des Ordners

Wenn Sie Namespaces verwenden, sollte der Namespace der Verzeichnisstruktur entsprechen, **einschließlich der Groß-/Kleinschreibung**.

```php
// ...code...

// wenn sich Ihr MyController in app/Controller (Skeleton) befindet und App\Controller als Namespace hat
// das funktioniert nicht:
Flight::route('/hello', 'MyController->hello');

// Skeleton-Stil:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Älteres Layout mit Kleinbuchstaben (nur wenn Ihre Ordner tatsächlich app/controllers heißen):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// oder voll qualifiziert:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` nicht definiert (Anwendungscode ohne Composer)

Wenn Sie für Anwendungsklassen auf `Flight::path()` anstelle von Composer setzen, definieren Sie den Pfad vor Routen, die diese Klassen referenzieren (oft früh im Bootstrap / in `public/index.php`):

```php
// Einen Pfad zum Autoloader hinzufügen (Projektwurzelverzeichnis für Apps mit Namespaces)
Flight::path(__DIR__.'/../');
```

Das offizielle Skeleton verwendet hauptsächlich **Composer PSR-4** für `App\`, daher benötigen Sie dort normalerweise kein `Flight::path()` für Controller und Modelle.

## Änderungsprotokoll

- Docs – Dokumentieren von Skeleton `App\` + PascalCase-Ordnern und Fallstricken zur Groß-/Kleinschreibung für Menschen und KI-Tools.
- v3.7.2 – Sie können Pascal_Snake_Case für Ihre Klassennamen verwenden, indem Sie `Loader::setV2ClassLoading(false);` ausführen.
- v2.0 – Autoload-Funktionalität hinzugefügt.