# Contenedor de Inyección de Dependencias

## Descripción General

El Contenedor de Inyección de Dependencias (DIC, por sus siglas en inglés) es una mejora potente que te permite gestionar
las dependencias de tu aplicación. También es una de las mayores razones por las que Flight se lleva bien con [herramientas de IA para codificación](/learn/ai) y pruebas unitarias: los controladores reciben lo que necesitan en el constructor en lugar de acceder a variables globales.

## Entendiendo

La Inyección de Dependencias (DI) es un concepto clave en los frameworks PHP modernos y se
utiliza para gestionar la instanciación y configuración de objetos. Algunos ejemplos de bibliotecas
DIC son: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/),
[PHP-DI](http://php-di.org/) y [league/container](https://container.thephpleague.com/).

Un DIC es una forma elegante de crear y gestionar tus clases en una
ubicación centralizada. Esto es útil cuando necesitas pasar el mismo objeto a
múltiples clases (controladores, middleware, comandos, etc.).

El [flightphp/skeleton](https://github.com/flightphp/skeleton) oficial conecta **Dice** en `app/config/services.php`, sustituye la instancia compartida de `flight\Engine` y resuelve destinos de rutas como `[App\Controller\HomeController::class, 'index']`. Prefiere ese patrón para proyectos nuevos para que humanos y agentes editen los mismos lugares.

## Uso Básico

La forma antigua de hacer las cosas podría verse así:
```php

require 'vendor/autoload.php';

// clase para gestionar usuarios desde la base de datos
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

// en tu archivo routes.php

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// otras rutas de UserController...

Flight::start();
```

Puedes ver en el código anterior que estamos creando un nuevo objeto `PDO` y pasándolo
a nuestra clase `UserController`. Esto está bien para una aplicación pequeña, pero a medida que tu
aplicación crezca, verás que estás creando o pasando el mismo objeto `PDO`
en múltiples lugares. Aquí es donde un DIC resulta útil.

Aquí está el mismo ejemplo usando un DIC (usando Dice):
```php

require 'vendor/autoload.php';

// misma clase que arriba. Nada ha cambiado
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

// crea un nuevo contenedor
$container = new \Dice\Dice;

// agrega una regla para indicar al contenedor cómo crear un objeto PDO
// ¡no olvides reasignarlo a sí mismo como abajo!
$container = $container->addRule('PDO', [
	// shared significa que se devolverá el mismo objeto cada vez
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Esto registra el manejador del contenedor para que Flight sepa usarlo.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// ahora podemos usar el contenedor para crear nuestro UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Apuesto a que podrías estar pensando que se agregó mucho código extra al ejemplo.
La magia viene cuando tienes otro controlador que necesita el objeto `PDO`.

```php

// Si todos tus controladores tienen un constructor que necesita un objeto PDO
// ¡cada una de las rutas siguientes lo tendrá inyectado automáticamente!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

El beneficio adicional de utilizar un DIC es que las pruebas unitarias se vuelven mucho más fáciles. Puedes
crear un objeto mock y pasarlo a tu clase. Este es un gran beneficio cuando estás
escribiendo pruebas para tu aplicación—y cuando un asistente de IA genera un controlador, la inyección por constructor le brinda un patrón claro y consistente a seguir ([guía de pruebas unitarias](/guides/unit-testing)).

### Creando un manejador DIC centralizado

Puedes crear un manejador DIC centralizado en tu archivo de servicios [extendiendo](/learn/extending) tu aplicación. Aquí tienes un ejemplo:

```php
// services.php

// crea un nuevo contenedor
$container = new \Dice\Dice;
// ¡no olvides reasignarlo a sí mismo como abajo!
$container = $container->addRule('PDO', [
	// shared significa que se devolverá el mismo objeto cada vez
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// ahora podemos crear un método mapeable para crear cualquier objeto.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Esto registra el manejador del contenedor para que Flight sepa usarlo para controladores/middleware
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// digamos que tenemos la siguiente clase de ejemplo que recibe un objeto PDO en el constructor
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// código que envía un correo electrónico
	}
}

// Y finalmente puedes crear objetos usando inyección de dependencias
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight tiene un plugin que proporciona un contenedor simple compatible con PSR-11 que puedes usar para manejar
tu inyección de dependencias. Aquí tienes un ejemplo rápido de cómo usarlo:

```php

// index.php por ejemplo
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
	// ¡esto se mostrará correctamente!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Uso Avanzado de flightphp/container

También puedes resolver dependencias de forma recursiva. Aquí tienes un ejemplo:

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
    // Implementación ...
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

También puedes crear tu propio manejador DIC. Esto es útil si tienes un
contenedor personalizado que quieras usar y que no sea PSR-11 (Dice). Consulta la
sección [uso básico](#uso-básico) para saber cómo hacerlo.

Además, hay
algunos valores predeterminados útiles que harán tu vida más fácil al usar Flight.

#### Instancia de Engine (requerida para la inyección de `$app`)

Si escribes la indicación de tipo `flight\Engine` en controladores o middleware, **Dice no debe construir un nuevo Engine**. Sustituye la misma instancia del bootstrap. Esto es lo que hace el skeleton oficial, y es el patrón que `AGENTS.md` espera para controladores generados por IA:

```php
// En algún lugar de tu bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // o $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Crítico: reutiliza el Engine de bootstrap—no dejes que Dice haga `new Engine()`
		Engine::class => $app,
		// Prefiere SimplePdo para código nuevo
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Helper opcional para código fuera de rutas
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (estructura del skeleton—la carpeta coincide con el namespace)
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
		// Sin fachada Flight:: en la capa de aplicación—más fácil de probar y más claro para herramientas de IA
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Si omites la sustitución de `Engine`, Dice puede construir un segundo Engine y tu controlador no compartirá rutas, configuración ni el `render` de Twig mapeado desde el bootstrap.

#### Agregando otros servicios compartidos (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Después de crear $db, $config, $twig en services.php:
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

Luego los controladores pueden recibir `SimplePdo $db` (o tu tipo de configuración) en el constructor y nunca llamar a `Flight::db()`. Eso coincide con la guía de [pruebas unitarias](/guides/unit-testing) y el estilo del skeleton.

#### Agregando otras clases

Si tienes otras clases que quieras agregar al contenedor, con Dice es fácil ya que serán resueltas automáticamente por el contenedor. Aquí tienes un ejemplo:

```php

$container = new \Dice\Dice;
// Si no necesitas inyectar dependencias en tus clases
// ¡no necesitas definir nada!
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

Flight también puede usar cualquier contenedor compatible con PSR-11. Esto significa que puedes usar cualquier
contenedor que implemente la interfaz PSR-11. Aquí tienes un ejemplo usando el contenedor
PSR-11 de League:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// misma idea de UserController que arriba, indicando SimplePdo en lugar de PDO crudo

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

Esto puede ser un poco más verboso que el ejemplo anterior con Dice, pero de todos modos
hace el trabajo con los mismos beneficios.

## Véase También
- [Instalación](/install) - Estructura del skeleton y dónde se encuentra `services.php`.
- [Autocarga](/learn/autoloading) - Namespaces `App\` y **mayúsculas/minúsculas** de las carpetas.
- [Extendiendo Flight](/learn/extending) - Aprende cómo puedes agregar inyección de dependencias a tus propias clases extendiendo el framework.
- [Configuración](/learn/configuration) - Aprende cómo configurar Flight para tu aplicación.
- [Enrutamiento](/learn/routing) - Aprende cómo definir rutas para tu aplicación y cómo funciona la inyección de dependencias con los controladores.
- [Middleware](/learn/middleware) - Aprende cómo crear middleware para tu aplicación y cómo funciona la inyección de dependencias con el middleware.
- [Pruebas Unitarias](/guides/unit-testing) - Por qué la inyección por constructor supera a las variables globales de `Flight::`.
- [IA y Experiencia de Desarrollo](/learn/ai) - Un patrón de DI para humanos y agentes.
- [SimplePdo](/learn/simple-pdo) - Helper de base de datos preferido para inyección.

## Solución de Problemas
- Si tienes problemas con tu contenedor, asegúrate de estar pasando los nombres de clase correctos al contenedor.
- Controladores que indican `Engine` pero reciben una aplicación "vacía": agrega la **sustitución de Engine** (ver arriba). Dice no debe hacer `new` de un segundo Engine.
- Clase no encontrada para `App\Controller\…`: verifica las mayúsculas/minúsculas de la carpeta bajo `app/Controller/`—consulta [Autocarga](/learn/autoloading).
- El manejador debe **devolver** el objeto creado desde `registerContainerHandler` (no llames a `Flight::make()` sin `return`).

## Registro de Cambios
- Documentación – Documentar el skeleton Dice + sustituciones de Engine, SimplePdo y la estructura `App\Controller` para proyectos amigables con IA.
- v3.7.0 - Se agregó la capacidad de registrar un manejador DIC en Flight.