# Dependency Injection Container

## Overview

The Dependency Injection Container (DIC) is a powerful enhancement that allows you to manage
your application's dependencies. It is also one of the biggest reasons Flight plays well with [AI coding tools](/learn/ai) and unit tests: controllers take what they need in the constructor instead of reaching for globals.

## Understanding

Dependency Injection (DI) is a key concept in modern PHP frameworks and is 
used to manage the instantiation and configuration of objects. Some examples of DIC 
libraries are: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/), 
[PHP-DI](http://php-di.org/), and [league/container](https://container.thephpleague.com/).

A DIC is a fancy way of creating and managing your classes in a
centralized location. That is useful when you need to pass the same object to 
multiple classes (controllers, middleware, commands, and so on).

The official [flightphp/skeleton](https://github.com/flightphp/skeleton) wires **Dice** in `app/config/services.php`, substitutes the shared `flight\Engine` instance, and resolves route targets like `[App\Controller\HomeController::class, 'index']`. Prefer that pattern for new projects so humans and agents edit the same places.

## Basic Usage

The old way of doing things might look like this:
```php

require 'vendor/autoload.php';

// class to manage users from the database
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

// in your routes.php file

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// other UserController routes...

Flight::start();
```

You can see from the above code that we are creating a new `PDO` object and passing it
to our `UserController` class. This is fine for a small application, but as your
application grows, you will find that you are creating or passing around the same `PDO` 
object in multiple places. This is where a DIC comes in handy.

Here is the same example using a DIC (using Dice):
```php

require 'vendor/autoload.php';

// same class as above. Nothing changed
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

// create a new container
$container = new \Dice\Dice;

// add a rule to tell the container how to create a PDO object
// don't forget to reassign it to itself like below!
$container = $container->addRule('PDO', [
	// shared means that the same object will be returned each time
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// This registers the container handler so Flight knows to use it.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// now we can use the container to create our UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

I bet you might be thinking that there was a lot of extra code added to the example.
The magic comes from when you have another controller that needs the `PDO` object. 

```php

// If all your controllers have a constructor that needs a PDO object
// each of the routes below will automatically have it injected!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

The added bonus of utilizing a DIC is that unit testing becomes much easier. You can
create a mock object and pass it to your class. This is a huge benefit when you are
writing tests for your application—and when an AI assistant generates a controller, constructor injection gives it a clear, consistent pattern to follow ([unit testing guide](/guides/unit-testing)).

### Creating a centralized DIC handler

You can create a centralized DIC handler in your services file by [extending](/learn/extending) your app. Here's an example:

```php
// services.php

// create a new container
$container = new \Dice\Dice;
// don't forget to reassign it to itself like below!
$container = $container->addRule('PDO', [
	// shared means that the same object will be returned each time
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// now we can create a mappable method to create any object. 
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// This registers the container handler so Flight knows to use it for controllers/middleware
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// lets say we have the following sample class that takes a PDO object in the constructor
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// code that sends an email
	}
}

// And finally you can create objects using dependency injection
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight has a plugin that provides a simple PSR-11 compliant container that you can use to handle
your dependency injection. Here's a quick example of how to use it:

```php

// index.php for example
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
	// will output this correctly!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Advanced Usage of flightphp/container

You can also resolve dependencies recursively. Here's an example:

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
    // Implementation ...
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

You can also create your own DIC handler. This is useful if you have a custom
container that you want to use that is not PSR-11 (Dice). See the 
[basic usage](#basic-usage) section for how to do this.

Additionally, there
are some helpful defaults that will make your life easier when using Flight.

#### Engine instance (required for `$app` injection)

If you type-hint `flight\Engine` on controllers or middleware, **Dice must not construct a new Engine**. Substitute the same instance from bootstrap. This is what the official skeleton does, and it is the pattern `AGENTS.md` expects for AI-generated controllers:

```php
// Somewhere in your bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // or $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Critical: reuse the bootstrapped Engine — do not let Dice `new Engine()`
		Engine::class => $app,
		// Prefer SimplePdo for new code
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Optional helper for non-route code
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (skeleton layout — folder case matches namespace)
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
		// No Flight:: facade in the app layer — easier to test and clearer for AI tools
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

If you skip the `Engine` substitution, Dice may build a second Engine and your controller will not share routes, config, or the mapped Twig `render` from bootstrap.

#### Adding other shared services (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// After you create $db, $config, $twig in services.php:
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

Then controllers can take `SimplePdo $db` (or your config type) in the constructor and never call `Flight::db()`. That matches the [unit testing](/guides/unit-testing) guidance and the skeleton house style.

#### Adding other classes

If you have other classes that you want to add to the container, with Dice it's easy as they will be automatically resolved by the container. Here is an example:

```php

$container = new \Dice\Dice;
// If you don't need to inject any dependencies into your classes
// you don't need to define anything!
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

Flight can also use any PSR-11 compliant container. This means that you can use any
container that implements the PSR-11 interface. Here is an example using League's
PSR-11 container:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// same UserController idea as above, type-hinting SimplePdo instead of raw PDO

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

This can be a little more verbose than the previous Dice example, it still
gets the job done with the same benefits!

## See Also
- [Installation](/install) - Skeleton layout and where `services.php` lives.
- [Autoloading](/learn/autoloading) - `App\` namespaces and folder **case**.
- [Extending Flight](/learn/extending) - Learn how you can add dependency injection to your own classes by extending the framework.
- [Configuration](/learn/configuration) - Learn how to configure Flight for your application.
- [Routing](/learn/routing) - Learn how to define routes for your application and how dependency injection works with controllers.
- [Middleware](/learn/middleware) - Learn how to create middleware for your application and how dependency injection works with middleware.
- [Unit Testing](/guides/unit-testing) - Why constructor injection beats `Flight::` globals.
- [AI & Developer Experience](/learn/ai) - One DI pattern for humans and agents.
- [SimplePdo](/learn/simple-pdo) - Preferred database helper for injection.

## Troubleshooting
- If you are having issues with your container, make sure that you are passing the correct class names to the container.
- Controllers that type-hint `Engine` but get a "blank" app: add the **Engine substitution** (see above). Dice must not `new` a second Engine.
- Class not found for `App\Controller\…`: check folder case under `app/Controller/` — see [Autoloading](/learn/autoloading).
- Handler must **return** the created object from `registerContainerHandler` (do not call `Flight::make()` without `return`).

## Changelog
- Docs – Document skeleton Dice + Engine substitutions, SimplePdo, and `App\Controller` layout for AI-friendly projects.
- v3.7.0 - Added ability to register a DIC handler to Flight.
