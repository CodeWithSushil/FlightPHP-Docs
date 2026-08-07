# Pruebas Unitarias en Flight PHP con PHPUnit

Esta guía introduce las pruebas unitarias en Flight PHP usando [PHPUnit](https://phpunit.de/), pensada para principiantes que quieren entender *por qué* las pruebas unitarias importan y cómo aplicarlas de manera práctica. Nos centraremos en probar el *comportamiento*—asegurando que tu aplicación hace lo que esperas, como enviar un correo electrónico o guardar un registro—en lugar de cálculos triviales. Comenzaremos con un [manejador de rutas](/learn/routing) simple y avanzaremos hacia un [controlador](/learn/routing) más complejo, incorporando [inyección de dependencias](/learn/dependency-injection-container) (DI) y simulando (mock) servicios de terceros.

## ¿Por qué realizar pruebas unitarias?

Las pruebas unitarias aseguran que tu código se comporte como se espera, detectando errores antes de que lleguen a producción. Son especialmente valiosas en Flight, donde el enrutamiento ligero y la flexibilidad pueden llevar a interacciones complejas. Para desarrolladores en solitario o equipos, las pruebas unitarias actúan como una red de seguridad, documentando el comportamiento esperado y previniendo regresiones cuando revisas el código más tarde. También mejoran el diseño: el código difícil de probar a menudo señala clases demasiado complejas o fuertemente acopladas.

A diferencia de ejemplos simplistas (p. ej., probar `x * y = z`), nos centraremos en comportamientos del mundo real, como validar entrada, guardar datos o desencadenar acciones como correos electrónicos. Nuestro objetivo es hacer que las pruebas sean accesibles y significativas.

## Principios Generales de Guía

1. **Prueba el comportamiento, no la implementación**: Concéntrate en los resultados (p. ej., "correo enviado" o "registro guardado") en lugar de los detalles internos. Esto hace que las pruebas sean robustas frente a la refactorización.
2. **Deja de usar `Flight::`**: Los métodos estáticos de Flight son terriblemente convenientes, pero dificultan las pruebas. Debes acostumbrarte a usar la variable `$app` de `$app = Flight::app();`. `$app` tiene todos los mismos métodos que `Flight::`. Aún podrás usar `$app->route()` o `$this->app->json()` en tu controlador, etc. También debes usar el enrutador real de Flight con `$router = $app->router()` y luego puedes usar `$router->get()`, `$router->post()`, `$router->group()`, etc. Consulta [Enrutamiento](/learn/routing).
3. **Mantén las pruebas rápidas**: Las pruebas rápidas fomentan la ejecución frecuente. Evita operaciones lentas como llamadas a bases de datos en pruebas unitarias. Si tienes una prueba lenta, es una señal de que estás escribiendo una prueba de integración, no una prueba unitaria. Las pruebas de integración son cuando realmente involucras bases de datos reales, llamadas HTTP reales, envío de correos reales, etc. Tienen su lugar, pero son lentas y pueden ser inestables, lo que significa que a veces fallan por una razón desconocida.
4. **Usa nombres descriptivos**: Los nombres de las pruebas deben describir claramente el comportamiento que se está probando. Esto mejora la legibilidad y el mantenimiento.
5. **Evita las variables globales como la peste**: Minimiza el uso de `$app->set()` y `$app->get()`, ya que actúan como estado global, requiriendo mocks en cada prueba. Prefiere DI o un contenedor de DI (consulta [Contenedor de Inyección de Dependencias](/learn/dependency-injection-container)). Incluso usar el método `$app->map()` es técnicamente un "global" y debe evitarse en favor de DI. Usa una librería de sesión como [flightphp/session](https://github.com/flightphp/session) para que puedas simular el objeto de sesión en tus pruebas. **No** llames a [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) directamente en tu código, ya que eso inyecta una variable global en tu código, dificultando las pruebas.
6. **Usa inyección de dependencias**: Inyecta dependencias (p. ej., [`PDO`](https://www.php.net/manual/en/class.pdo.php), mailers) en los controladores para aislar la lógica y simplificar la simulación (mock). Si tienes una clase con demasiadas dependencias, considera refactorizarla en clases más pequeñas que cada una tenga una única responsabilidad siguiendo los [principios SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Simula servicios de terceros**: Simula bases de datos, clientes HTTP (cURL) o servicios de correo para evitar llamadas externas. Prueba una o dos capas de profundidad, pero deja que tu lógica central se ejecute. Por ejemplo, si tu aplicación envía un mensaje de texto, **NO** quieres enviar un mensaje de texto real cada vez que ejecutas tus pruebas porque esos cargos se acumularán (y será más lento). En su lugar, simula el servicio de mensajes de texto y solo verifica que tu código llamó al servicio de mensajes de texto con los parámetros correctos.
8. **Apunta a una alta cobertura, no a la perfección**: Una cobertura de línea del 100% es buena, pero en realidad no significa que todo en tu código esté probado como debería (investiga sobre [cobertura de ramas/rutas en PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Prioriza los comportamientos críticos (p. ej., registro de usuarios, respuestas de API y captura de respuestas fallidas).
9. **Usa controladores para las rutas**: En tus definiciones de rutas, usa controladores en lugar de closures. El `flight\Engine $app` se inyecta en cada controlador a través del constructor por defecto. En las pruebas, usa `$app = new Flight\Engine()` para instanciar Flight dentro de una prueba, inyéctalo en tu controlador y llama métodos directamente (p. ej., `$controller->register()`). Consulta [Extendiendo Flight](/learn/extending) y [Enrutamiento](/learn/routing).
10. **Elige un estilo de simulación (mock) y mantente consistente**: PHPUnit soporta varios estilos de simulación (p. ej., profecía, mocks incorporados), o puedes usar clases anónimas que tienen sus propios beneficios como el autocompletado de código, romperse si cambias la definición del método, etc. Solo sé consistente en todas tus pruebas. Consulta [Objetos simulados de PHPUnit](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Usa visibilidad `protected` para métodos/propiedades que quieras probar en subclases**: Esto te permite sobrescribirlos en subclases de prueba sin hacerlos públicos, lo cual es especialmente útil para mocks de clases anónimas.

## Configuración de PHPUnit

Primero, configura [PHPUnit](https://phpunit.de/) en tu proyecto Flight PHP usando Composer para facilitar las pruebas. Consulta la [guía de inicio de PHPUnit](https://phpunit.readthedocs.io/en/12.3/installation.html) para más detalles.

1. En el directorio de tu proyecto, ejecuta:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Esto instala la última versión de PHPUnit como una dependencia de desarrollo.

2. Crea un directorio `tests` en la raíz de tu proyecto para los archivos de prueba.

3. Agrega un script de prueba a `composer.json` por conveniencia:
   ```json
   // otro contenido de composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Crea un archivo `phpunit.xml` en la raíz:
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

Ahora, cuando tus pruebas estén creadas, puedes ejecutar `composer test` para ejecutar las pruebas.

## Probando un Manejador de Rutas Simple

Comencemos con una [ruta](/learn/routing) básica que valida la entrada de correo electrónico de un usuario. Probaremos su comportamiento: devolver un mensaje de éxito para correos válidos y un error para los inválidos. Para la validación de correo, usamos [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

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

Para probar esto, crea un archivo de prueba. Consulta [Pruebas Unitarias y Principios SOLID](/learn/unit-testing-and-solid-principles) para más información sobre cómo estructurar pruebas:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simular datos POST
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
		$request->data->email = 'invalid-email'; // Simular datos POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Puntos clave**:
- Simulamos los datos POST usando la clase de solicitud. No uses variables globales como `$_POST`, `$_GET`, etc., ya que esto hace que las pruebas sean más complicadas (tienes que restablecer siempre esos valores o otras pruebas podrían fallar).
- Todos los controladores, por defecto, tendrán la instancia de `flight\Engine` inyectada en ellos incluso sin tener un contenedor DIC configurado. Esto hace que sea mucho más fácil probar los controladores directamente.
- No hay uso de `Flight::` en absoluto, lo que hace que el código sea más fácil de probar.
- Las pruebas verifican el comportamiento: estado y mensaje correctos para correos válidos/inválidos.

Ejecuta `composer test` para verificar que la ruta se comporte como se espera. Para más información sobre [solicitudes](/learn/requests) y [respuestas](/learn/responses) en Flight, consulta la documentación relevante.

## Uso de la Inyección de Dependencias para Controladores Comprobables

Para escenarios más complejos, usa [inyección de dependencias](/learn/dependency-injection-container) (DI) para que los controladores sean comprobables. Evita las globales de Flight (p. ej., `Flight::set()`, `Flight::map()`, `Flight::register()`) ya que actúan como estado global, requiriendo mocks para cada prueba. En su lugar, usa el contenedor DI de Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) o DI manual.

Usemos [`flight\database\SimplePdo`](/learn/simple-pdo) en lugar de PDO crudo. Este helper es mucho más fácil de simular y probar unitariamente (y se prefiere sobre el obsoleto `PdoWrapper`).

Aquí hay un controlador que guarda un usuario en una base de datos y envía un correo de bienvenida:

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
			// añadir el return aquí ayuda a que las pruebas unitarias detengan la ejecución
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Puntos clave**:
- El controlador depende de una instancia de [`SimplePdo`](/learn/simple-pdo) y de una `MailerInterface` (un servicio de correo simulado de terceros).
- Las dependencias se inyectan a través del constructor, evitando variables globales.

### Probando el Controlador con Mocks (Simulaciones)

Ahora, probemos el comportamiento de `UserController`: validar correos, guardar en la base de datos y enviar correos. Simularemos la base de datos y el mailer para aislar el controlador.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// A veces es necesario mezclar estilos de simulación
		// Aquí usamos el mock incorporado de PHPUnit para PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Usando una clase anónima para simular SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Cuando lo simulamos de esta manera, no estamos haciendo realmente una llamada a la base de datos.
			// Podemos configurar esto además para alterar el mock de PDOStatement y simular fallos, etc.
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
			// Un constructor vacío omite el constructor padre
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

		// Necesitamos mapear jsonHalt para evitar la salida
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

**Puntos clave**:
- Simulamos `SimplePdo` y `MailerInterface` para evitar llamadas reales a la base de datos o al correo.
- Las pruebas verifican el comportamiento: los correos válidos desencadenan inserciones en la base de datos y envíos de correo; los correos inválidos omiten ambos.
- Simula dependencias de terceros (p. ej., `SimplePdo`, `MailerInterface`), dejando que la lógica del controlador se ejecute.

### Simulando demasiado

Ten cuidado de no simular demasiado tu código. Te doy un ejemplo a continuación de por qué esto podría ser malo usando nuestro `UserController`. Cambiaremos esa verificación a un método llamado `isEmailValid` (usando `filter_var`) y las otras nuevas adiciones a un método separado llamado `registerUser`.

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
			// añadir el return aquí ayuda a que las pruebas unitarias detengan la ejecución
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

Y ahora la prueba unitaria sobresimulada que en realidad no prueba nada:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// estamos omitiendo la inyección de dependencias extra aquí porque es "fácil"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Omitimos las dependencias en el constructor
			public function __construct($app) {
				$this->app = $app;
			}

			// Simplemente forzaremos que esto sea válido.
			protected function isEmailValid($email) {
				return true; // Siempre devuelve true, omitiendo la validación real
			}

			// Omitimos las llamadas reales a la base de datos y al correo
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

¡Hurra, tenemos pruebas unitarias y están pasando! Pero espera, ¿y si realmente cambio el funcionamiento interno de `isEmailValid` o `registerUser`? Mis pruebas seguirán pasando porque he simulado toda la funcionalidad. Déjame mostrarte lo que quiero decir.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... otros métodos ...

	protected function isEmailValid($email) {
		// Lógica cambiada
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Ahora debería tener solo un dominio específico
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Si ejecutara mis pruebas unitarias anteriores, ¡aún pasarían! Pero debido a que no estaba probando el comportamiento (dejando que parte del código se ejecute realmente), potencialmente he codificado un error a punto de ocurrir en producción. La prueba debe modificarse para tener en cuenta el nuevo comportamiento, y también lo contrario cuando el comportamiento no es el que esperamos.

## Ejemplo Completo

Puedes encontrar un ejemplo completo de un proyecto Flight PHP con pruebas unitarias en GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide). Para una comprensión más profunda, consulta [Pruebas Unitarias y Principios SOLID](/learn/unit-testing-and-solid-principles).

## Errores Comunes

- **Sobresimulación (Over-Mocking)**: No simules cada dependencia; deja que algo de lógica (p. ej., la validación del controlador) se ejecute para probar el comportamiento real. Consulta [Pruebas Unitarias y Principios SOLID](/learn/unit-testing-and-solid-principles).
- **Estado global**: Usar variables globales de PHP (p. ej., [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) en gran medida hace que las pruebas sean frágiles. Lo mismo ocurre con `Flight::`. Refactoriza para pasar las dependencias explícitamente.
- **Configuración compleja**: Si la configuración de la prueba es engorrosa, tu clase puede tener demasiadas dependencias o responsabilidades, violando los [principios SOLID](/learn/unit-testing-and-solid-principles).

## Escalando con Pruebas Unitarias

Las pruebas unitarias brillan en proyectos más grandes o al revisar código después de meses. Documentan el comportamiento y detectan regresiones, ahorrándote tener que reaprender tu aplicación. Para desarrolladores en solitario, prueba las rutas críticas (p. ej., registro de usuarios, procesamiento de pagos). Para equipos, las pruebas aseguran un comportamiento consistente en todas las contribuciones. Consulta [¿Por qué frameworks?](/learn/why-frameworks) para más información sobre los beneficios de usar frameworks y pruebas.

¡Contribuye con tus propios consejos de pruebas al repositorio de documentación de Flight PHP!

_Escrito por [n0nag0n](https://github.com/n0nag0n) 2025_