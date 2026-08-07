# Conteneur d'Injection de Dépendances

## Aperçu

Le conteneur d'injection de dépendances (DIC) est une amélioration puissante qui vous permet de gérer
les dépendances de votre application. C'est également l'une des principales raisons pour lesquelles Flight fonctionne bien avec les [outils d'IA de codage](/learn/ai) et les tests unitaires : les contrôleurs prennent ce dont ils ont besoin dans le constructeur au lieu d'utiliser des variables globales.

## Compréhension

L'injection de dépendances (DI) est un concept clé dans les frameworks PHP modernes et est
utilisée pour gérer l'instanciation et la configuration des objets. Quelques exemples de bibliothèques
de DIC sont : [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/),
[PHP-DI](http://php-di.org/), et [league/container](https://container.thephpleague.com/).

Un DIC est une manière élégante de créer et de gérer vos classes dans un
emplacement centralisé. C'est utile lorsque vous devez passer le même objet à
plusieurs classes (contrôleurs, middlewares, commandes, etc.).

Le [flightphp/skeleton](https://github.com/flightphp/skeleton) officiel configure **Dice** dans `app/config/services.php`, substitue l'instance partagée de `flight\Engine`, et résout les cibles de routage comme `[App\Controller\HomeController::class, 'index']`. Préférez ce modèle pour les nouveaux projets afin que les humains et les agents modifient les mêmes endroits.

## Utilisation de base

L'ancienne façon de faire pourrait ressembler à ceci :
```php

require 'vendor/autoload.php';

// classe pour gérer les utilisateurs depuis la base de données
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

// dans votre fichier routes.php

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// autres routes de UserController...

Flight::start();
```

Vous pouvez voir dans le code ci-dessus que nous créons un nouvel objet `PDO` et le passons
à notre classe `UserController`. C'est acceptable pour une petite application, mais à mesure que votre
application grandit, vous constaterez que vous créez ou transmettez le même objet `PDO`
à plusieurs endroits. C'est là qu'un DIC devient pratique.

Voici le même exemple utilisant un DIC (avec Dice) :
```php

require 'vendor/autoload.php';

// même classe que ci-dessus. Rien n'a changé
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

// créer un nouveau conteneur
$container = new \Dice\Dice;

// ajouter une règle pour indiquer au conteneur comment créer un objet PDO
// n'oubliez pas de vous le réassigner comme ci-dessous !
$container = $container->addRule('PDO', [
	// shared signifie que le même objet sera retourné à chaque fois
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Ceci enregistre le gestionnaire de conteneur pour que Flight sache l'utiliser.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// maintenant nous pouvons utiliser le conteneur pour créer notre UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Je parie que vous pensez peut-être qu'il y a beaucoup de code supplémentaire ajouté à l'exemple.
La magie opère lorsque vous avez un autre contrôleur qui a besoin de l'objet `PDO`.

```php

// Si tous vos contrôleurs ont un constructeur qui nécessite un objet PDO
// chacune des routes ci-dessous l'aura automatiquement injecté !!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

L'avantage supplémentaire de l'utilisation d'un DIC est que les tests unitaires deviennent beaucoup plus faciles. Vous pouvez
créer un objet mock et le passer à votre classe. C'est un énorme avantage lorsque vous
écrivez des tests pour votre application—et lorsqu'un assistant IA génère un contrôleur, l'injection par constructeur lui donne un modèle clair et cohérent à suivre ([guide de tests unitaires](/guides/unit-testing)).

### Créer un gestionnaire DIC centralisé

Vous pouvez créer un gestionnaire DIC centralisé dans votre fichier de services en [étendant](/learn/extending) votre application. Voici un exemple :

```php
// services.php

// créer un nouveau conteneur
$container = new \Dice\Dice;
// n'oubliez pas de vous le réassigner comme ci-dessous !
$container = $container->addRule('PDO', [
	// shared signifie que le même objet sera retourné à chaque fois
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// maintenant nous pouvons créer une méthode mappable pour créer n'importe quel objet.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Ceci enregistre le gestionnaire de conteneur pour que Flight sache l'utiliser pour les contrôleurs/middlewares
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// disons que nous avons la classe d'exemple suivante qui prend un objet PDO dans le constructeur
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// code qui envoie un email
	}
}

// Et enfin vous pouvez créer des objets en utilisant l'injection de dépendances
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight dispose d'un plugin qui fournit un conteneur simple compatible PSR-11 que vous pouvez utiliser pour gérer
votre injection de dépendances. Voici un exemple rapide de son utilisation :

```php

// index.php par exemple
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
	// affichera cela correctement !
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Utilisation avancée de flightphp/container

Vous pouvez également résoudre les dépendances de manière récursive. Voici un exemple :

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
    // Implémentation ...
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

Vous pouvez également créer votre propre gestionnaire DIC. C'est utile si vous avez un
conteneur personnalisé que vous souhaitez utiliser et qui n'est pas PSR-11 (Dice). Consultez la
section [utilisation de base](#utilisation-de-base) pour savoir comment procéder.

De plus, il y a
quelques valeurs par défaut utiles qui vous faciliteront la vie lors de l'utilisation de Flight.

#### Instance Engine (requise pour l'injection de `$app`)

Si vous utilisez un indice de type `flight\Engine` sur des contrôleurs ou des middlewares, **Dice ne doit pas construire un nouveau Engine**. Substituez la même instance provenant du bootstrap. C'est ce que fait le skeleton officiel, et c'est le modèle que `AGENTS.md` attend pour les contrôleurs générés par IA :

```php
// Quelque part dans votre bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // ou $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Critique : réutiliser le Engine démarré — ne laissez pas Dice faire `new Engine()`
		Engine::class => $app,
		// Préférez SimplePdo pour le nouveau code
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Aide facultative pour le code hors routes
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (structure du skeleton — la casse du dossier correspond à l'espace de noms)
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
		// Pas de façade Flight:: dans la couche applicative — plus facile à tester et plus clair pour les outils IA
		$this->app->render('welcome', ['message' => 'Bonjour']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Si vous omettez la substitution de `Engine`, Dice peut construire un second Engine et votre contrôleur ne partagera ni les routes, ni la configuration, ni le rendu Twig mappé du bootstrap.

#### Ajouter d'autres services partagés (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Après avoir créé $db, $config, $twig dans services.php :
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

Ensuite, les contrôleurs peuvent accepter `SimplePdo $db` (ou votre type de configuration) dans le constructeur et ne jamais appeler `Flight::db()`. Cela correspond aux recommandations de [tests unitaires](/guides/unit-testing) et au style maison du skeleton.

#### Ajouter d'autres classes

Si vous avez d'autres classes que vous souhaitez ajouter au conteneur, avec Dice c'est facile car elles seront automatiquement résolues par le conteneur. Voici un exemple :

```php

$container = new \Dice\Dice;
// Si vous n'avez pas besoin d'injecter de dépendances dans vos classes
// vous n'avez rien à définir !
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

Flight peut également utiliser n'importe quel conteneur compatible PSR-11. Cela signifie que vous pouvez utiliser n'importe quel
conteneur qui implémente l'interface PSR-11. Voici un exemple utilisant le
conteneur PSR-11 de League :

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// même idée de UserController que ci-dessus, avec un indice de type SimplePdo au lieu de PDO brut

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

Cela peut être un peu plus verbeux que l'exemple précédent avec Dice, mais cela
fait le travail avec les mêmes avantages !

## Voir aussi
- [Installation](/install) - Structure du skeleton et où se trouve `services.php`.
- [Autoloading](/learn/autoloading) - Espaces de noms `App\` et **casse** des dossiers.
- [Étendre Flight](/learn/extending) - Apprenez comment ajouter l'injection de dépendances à vos propres classes en étendant le framework.
- [Configuration](/learn/configuration) - Apprenez comment configurer Flight pour votre application.
- [Routage](/learn/routing) - Apprenez comment définir des routes pour votre application et comment l'injection de dépendances fonctionne avec les contrôleurs.
- [Middleware](/learn/middleware) - Apprenez comment créer des middlewares pour votre application et comment l'injection de dépendances fonctionne avec les middlewares.
- [Tests unitaires](/guides/unit-testing) - Pourquoi l'injection par constructeur est préférable aux variables globales `Flight::`.
- [IA et expérience développeur](/learn/ai) - Un modèle DI unique pour les humains et les agents.
- [SimplePdo](/learn/simple-pdo) - Aide de base de données préféré pour l'injection.

## Dépannage
- Si vous rencontrez des problèmes avec votre conteneur, assurez-vous de passer les bons noms de classes au conteneur.
- Contrôleurs qui utilisent un indice de type `Engine` mais qui reçoivent une application « vide » : ajoutez la **substitution Engine** (voir ci-dessus). Dice ne doit pas faire `new` d'un second Engine.
- Classe introuvable pour `App\Controller\…` : vérifiez la casse du dossier sous `app/Controller/` — voir [Autoloading](/learn/autoloading).
- Le gestionnaire doit **retourner** l'objet créé depuis `registerContainerHandler` (n'appelez pas `Flight::make()` sans `return`).

## Journal des modifications
- Documentation – Documenter le skeleton Dice + les substitutions Engine, SimplePdo et la structure `App\Controller` pour des projets adaptés à l'IA.
- v3.7.0 - Ajout de la possibilité d'enregistrer un gestionnaire DIC dans Flight.