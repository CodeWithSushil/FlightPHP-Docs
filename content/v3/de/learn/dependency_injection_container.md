# Dependency-Injection-Container

## Übersicht

Der Dependency-Injection-Container (DIC) ist eine leistungsstarke Erweiterung, mit der Sie die Abhängigkeiten Ihrer Anwendung verwalten können. Er ist auch einer der Hauptgründe, warum Flight gut mit [KI-Programmierwerkzeugen](/learn/ai) und Unit-Tests funktioniert: Controller nehmen im Konstruktor, was sie benötigen, anstatt auf globale Variablen zuzugreifen.

## Verständnis

Dependency Injection (DI) ist ein zentrales Konzept in modernen PHP-Frameworks und wird verwendet, um die Instanziierung und Konfiguration von Objekten zu verwalten. Einige Beispiele für DIC-Bibliotheken sind: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/), [PHP-DI](http://php-di.org/) und [league/container](https://container.thephpleague.com/).

Ein DIC ist eine ausgefallene Möglichkeit, Ihre Klassen an einem zentralen Ort zu erstellen und zu verwalten. Das ist nützlich, wenn Sie dasselbe Objekt an mehrere Klassen (Controller, Middleware, Befehle usw.) übergeben müssen.

Das offizielle [flightphp/skeleton](https://github.com/flightphp/skeleton) bindet **Dice** in `app/config/services.php` ein, ersetzt die gemeinsame `flight\Engine`-Instanz und löst Routenziele wie `[App\Controller\HomeController::class, 'index']` auf. Bevorzugen Sie dieses Muster für neue Projekte, damit Menschen und Agenten dieselben Stellen bearbeiten.

## Grundlegende Verwendung

Die alte Vorgehensweise könnte so aussehen:
```php

require 'vendor/autoload.php';

// Klasse zur Verwaltung von Benutzern aus der Datenbank
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// in Ihrer routes.php-Datei

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// weitere UserController-Routen...

Flight::start();
```

Sie können am obigen Code sehen, dass wir ein neues `PDO`-Objekt erstellen und es an unsere `UserController`-Klasse übergeben. Das ist für eine kleine Anwendung in Ordnung, aber wenn Ihre Anwendung wächst, werden Sie feststellen, dass Sie dasselbe `PDO`-Objekt an mehreren Stellen erstellen oder weiterreichen. Hier kommt ein DIC ins Spiel.

Hier ist dasselbe Beispiel mit einem DIC (unter Verwendung von Dice):
```php

require 'vendor/autoload.php';

// dieselbe Klasse wie oben. Nichts geändert
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// einen neuen Container erstellen
$container = new \Dice\Dice;

// eine Regel hinzufügen, die dem Container mitteilt, wie ein PDO-Objekt erstellt wird
// nicht vergessen, es wie unten gezeigt sich selbst zuzuweisen!
$container = $container->addRule('PDO', [
	// shared bedeutet, dass jedes Mal dasselbe Objekt zurückgegeben wird
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Dies registriert den Container-Handler, damit Flight weiß, dass er ihn verwenden soll.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// jetzt können wir den Container verwenden, um unseren UserController zu erstellen
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Ich wette, Sie denken vielleicht, dass dem Beispiel eine Menge zusätzlicher Code hinzugefügt wurde. Die Magie zeigt sich, wenn Sie einen weiteren Controller haben, der das `PDO`-Objekt benötigt.

```php

// Wenn alle Ihre Controller einen Konstruktor haben, der ein PDO-Objekt benötigt,
// wird es automatisch in jede der folgenden Routen injiziert!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

Ein zusätzlicher Vorteil der Nutzung eines DIC ist, dass Unit-Tests viel einfacher werden. Sie können ein Mock-Objekt erstellen und es an Ihre Klasse übergeben. Das ist ein großer Vorteil, wenn Sie Tests für Ihre Anwendung schreiben – und wenn ein KI-Assistent einen Controller generiert, gibt die Konstruktor-Injektion ihm ein klares, konsistentes Muster, dem er folgen kann ([Unit-Testing-Leitfaden](/guides/unit-testing)).

### Einen zentralen DIC-Handler erstellen

Sie können einen zentralen DIC-Handler in Ihrer Services-Datei erstellen, indem Sie [Ihre App erweitern](/learn/extending). Hier ist ein Beispiel:

```php
// services.php

// einen neuen Container erstellen
$container = new \Dice\Dice;
// nicht vergessen, es wie unten gezeigt sich selbst zuzuweisen!
$container = $container->addRule('PDO', [
	// shared bedeutet, dass jedes Mal dasselbe Objekt zurückgegeben wird
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// jetzt können wir eine mappbare Methode erstellen, um jedes Objekt zu erstellen.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Dies registriert den Container-Handler, damit Flight weiß, dass er ihn für Controller/Middleware verwenden soll
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// nehmen wir an, wir haben die folgende Beispielklasse, die ein PDO-Objekt im Konstruktor erhält
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// Code, der eine E-Mail sendet
	}
}

// Und schließlich können Sie Objekte mithilfe von Dependency Injection erstellen
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight hat ein Plugin, das einen einfachen PSR-11-konformen Container bereitstellt, den Sie für Ihre Dependency Injection verwenden können. Hier ist ein kurzes Beispiel zur Verwendung:

```php

// index.php zum Beispiel
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// wird dies korrekt ausgeben!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Erweiterte Verwendung von flightphp/container

Sie können Abhängigkeiten auch rekursiv auflösen. Hier ist ein Beispiel:

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Implementierung ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

Sie können auch Ihren eigenen DIC-Handler erstellen. Das ist nützlich, wenn Sie einen benutzerdefinierten Container verwenden möchten, der nicht PSR-11-konform ist (Dice). Siehe den Abschnitt [grundlegende Verwendung](#grundlegende-verwendung) für Details.

Zusätzlich gibt es einige hilfreiche Standardeinstellungen, die Ihnen die Arbeit mit Flight erleichtern.

#### Engine-Instanz (erforderlich für die `$app`-Injektion)

Wenn Sie `flight\Engine` in Controllern oder Middleware als Typ-Hinweis verwenden, **darf Dice keine neue Engine erstellen**. Ersetzen Sie sie durch dieselbe Instanz aus dem Bootstrap. Das macht das offizielle Skeleton, und es ist das Muster, das `AGENTS.md` für KI-generierte Controller erwartet:

```php
// Irgendwo in Ihrem Bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // oder $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Kritisch: die gebootstrappte Engine wiederverwenden – Dice nicht `new Engine()` ausführen lassen
		Engine::class => $app,
		// SimplePdo für neuen Code bevorzugen
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Optionaler Helfer für Nicht-Routen-Code
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (Skeleton-Layout – Ordner-Schreibweise entspricht dem Namespace)
namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// Keine Flight::-Fassade in der App-Ebene – einfacher zu testen und klarer für KI-Tools
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Wenn Sie die `Engine`-Substitution überspringen, könnte Dice eine zweite Engine erstellen und Ihr Controller wird keine Routen, Konfiguration oder das gemappte Twig-`render` aus dem Bootstrap teilen.

#### Weitere gemeinsame Services hinzufügen (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Nachdem Sie $db, $config, $twig in services.php erstellt haben:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

Dann können Controller `SimplePdo $db` (oder Ihren Konfigurationstyp) im Konstruktor entgegennehmen und müssen nie `Flight::db()` aufrufen. Das entspricht dem [Unit-Testing](/guides/unit-testing)-Leitfaden und dem Hausstil des Skeletons.

#### Weitere Klassen hinzufügen

Wenn Sie weitere Klassen zum Container hinzufügen möchten, ist das mit Dice einfach, da sie automatisch vom Container aufgelöst werden. Hier ist ein Beispiel:

```php

$container = new \Dice\Dice;
// Wenn Sie keine Abhängigkeiten in Ihre Klassen injizieren müssen,
// müssen Sie nichts definieren!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flight kann auch jeden PSR-11-konformen Container verwenden. Das bedeutet, dass Sie jeden Container verwenden können, der das PSR-11-Interface implementiert. Hier ist ein Beispiel mit dem PSR-11-Container von League:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// dieselbe UserController-Idee wie oben, mit SimplePdo als Typ-Hinweis statt rohem PDO

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

Das kann etwas ausführlicher sein als das vorherige Dice-Beispiel, erfüllt aber denselben Zweck mit denselben Vorteilen!

## Siehe auch
- [Installation](/install) – Skeleton-Layout und wo `services.php` liegt.
- [Autoloading](/learn/autoloading) – `App\`-Namespaces und Ordner-**Schreibweise**.
- [Flight erweitern](/learn/extending) – Erfahren Sie, wie Sie Dependency Injection zu Ihren eigenen Klassen hinzufügen können, indem Sie das Framework erweitern.
- [Konfiguration](/learn/configuration) – Erfahren Sie, wie Sie Flight für Ihre Anwendung konfigurieren.
- [Routing](/learn/routing) – Erfahren Sie, wie Sie Routen für Ihre Anwendung definieren und wie Dependency Injection mit Controllern funktioniert.
- [Middleware](/learn/middleware) – Erfahren Sie, wie Sie Middleware für Ihre Anwendung erstellen und wie Dependency Injection mit Middleware funktioniert.
- [Unit-Testing](/guides/unit-testing) – Warum Konstruktor-Injektion besser ist als `Flight::`-Globale.
- [KI & Entwicklererfahrung](/learn/ai) – Ein DI-Muster für Menschen und Agenten.
- [SimplePdo](/learn/simple-pdo) – Bevorzugter Datenbank-Helper für die Injektion.

## Fehlerbehebung
- Wenn Sie Probleme mit Ihrem Container haben, stellen Sie sicher, dass Sie die korrekten Klassennamen an den Container übergeben.
- Controller, die `Engine` als Typ-Hinweis verwenden, aber eine "leere" App erhalten: Fügen Sie die **Engine-Substitution** hinzu (siehe oben). Dice darf keine zweite Engine mit `new` erstellen.
- Klasse für `App\Controller\…` nicht gefunden: Überprüfen Sie die Ordner-Schreibweise unter `app/Controller/` – siehe [Autoloading](/learn/autoloading).
- Der Handler muss das erstellte Objekt von `registerContainerHandler` **zurückgeben** (rufen Sie `Flight::make()` nicht ohne `return` auf).

## Änderungsprotokoll
- Dokumentation – Skeleton-Dice + Engine-Substitutionen, SimplePdo und `App\Controller`-Layout für KI-freundliche Projekte dokumentiert.
- v3.7.0 – Möglichkeit hinzugefügt, einen DIC-Handler bei Flight zu registrieren.