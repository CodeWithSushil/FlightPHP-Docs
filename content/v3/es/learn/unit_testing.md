# Pruebas Unitarias

## Resumen

Las pruebas unitarias en Flight te ayudan a garantizar que tu aplicación se comporte como se espera, detectar errores a tiempo y hacer que tu base de código sea más fácil de mantener. Flight está diseñado para funcionar sin problemas con [PHPUnit](https://phpunit.de/), el framework de pruebas más popular de PHP.

## Comprensión

Las pruebas unitarias verifican el comportamiento de pequeñas partes de tu aplicación (como controladores o servicios) de forma aislada. En Flight, esto significa probar cómo tus rutas, controladores y lógica responden a diferentes entradas, sin depender del estado global o de servicios externos reales.

Principios clave:
- **Prueba el comportamiento, no la implementación:** Céntrate en lo que hace tu código, no en cómo lo hace.
- **Evita el estado global:** Usa inyección de dependencias en lugar de `Flight::set()` o `Flight::get()`.
- **Simula servicios externos:** Reemplaza cosas como bases de datos o servicios de correo con dobles de prueba.
- **Mantén las pruebas rápidas y enfocadas:** Las pruebas unitarias no deben acceder a bases de datos o APIs reales.

## Uso Básico

### Configuración de PHPUnit

1. Instala PHPUnit con Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Crea un directorio `tests` en la raíz de tu proyecto.
3. Agrega un script de prueba a tu `composer.json`:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Crea un archivo `phpunit.xml`:
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

Ahora puedes ejecutar tus pruebas con `composer test`.

### Probando un Manejador de Ruta Simple

Supón que tienes una ruta que valida un correo electrónico:

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
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Una prueba simple para este controlador:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Consejos:**
- Simula datos POST usando `$app->request()->data`.
- Evita usar estáticos de `Flight::` en tus pruebas: usa la instancia de `$app`.

### Usando Inyección de Dependencias para Controladores Comprobables

Inyecta dependencias (como la base de datos o el servicio de correo) en tus controladores para que sean fáciles de simular en las pruebas:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

Y una prueba con simulacros (mocks):

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Uso Avanzado

- **Simulación (Mocks):** Usa los simulacros integrados de PHPUnit o clases anónimas para reemplazar dependencias.
- **Prueba de controladores directamente:** Instancia controladores con un nuevo `Engine` y simula las dependencias.
- **Evita simular en exceso:** Deja que la lógica real se ejecute cuando sea posible; solo simula servicios externos.

## Ver También

- [Guía de Pruebas Unitarias](/guides/unit-testing) - Una guía completa sobre las mejores prácticas de pruebas unitarias.
- [Contenedor de Inyección de Dependencias](/learn/dependency-injection-container) - Cómo usar DICs para gestionar dependencias y mejorar la comprobabilidad.
- [Extensión](/learn/extending) - Cómo agregar tus propios ayudantes o sobrescribir clases principales.
- [SimplePdo](/learn/simple-pdo) - Simplifica las interacciones con la base de datos y es más fácil de simular en pruebas.
- [Solicitudes](/learn/requests) - Manejo de solicitudes HTTP en Flight.
- [Respuestas](/learn/responses) - Envío de respuestas a los usuarios.
- [Pruebas Unitarias y Principios SOLID](/learn/unit-testing-and-solid-principles) - Aprende cómo los principios SOLID pueden mejorar tus pruebas unitarias.

## Solución de Problemas

- Evita usar estado global (`Flight::set()`, `$_SESSION`, etc.) en tu código y en tus pruebas.
- Si tus pruebas son lentas, es posible que estés escribiendo pruebas de integración: simula servicios externos para mantener las pruebas unitarias rápidas.
- Si la configuración de las pruebas es compleja, considera refactorizar tu código para usar inyección de dependencias.

## Registro de Cambios

- v3.15.0 - Se agregaron ejemplos de inyección de dependencias y simulacros (mocks).