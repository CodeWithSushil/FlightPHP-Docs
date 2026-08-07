# Enrutamiento

## Descripción general
El enrutamiento en Flight PHP asigna patrones de URL a funciones de devolución de llamada o métodos de clases, lo que permite un manejo de solicitudes rápido y sencillo. Está diseñado para tener una sobrecarga mínima, ser amigable para principiantes y ser extensible sin dependencias externas.

## Comprendiendo el enrutamiento
El enrutamiento es el mecanismo central que conecta las solicitudes HTTP con la lógica de tu aplicación en Flight. Al definir rutas, especificas cómo diferentes URLs activan código específico, ya sea mediante funciones, métodos de clase o acciones de controladores. El sistema de enrutamiento de Flight es flexible, compatible con patrones básicos, parámetros nombrados, expresiones regulares y funciones avanzadas como inyección de dependencias y enrutamiento de recursos. Este enfoque mantiene tu código organizado y fácil de mantener, mientras sigue siendo rápido y simple para principiantes y extensible para usuarios avanzados.

> **Nota:** ¿Quieres entender más sobre el enrutamiento? Consulta la página ["¿por qué un framework?"](/learn/why-frameworks) para obtener una explicación más detallada.

## Uso básico

### Definiendo una ruta simple
El enrutamiento básico en Flight se realiza haciendo coincidir un patrón de URL con una función de devolución de llamada o un arreglo de una clase y un método.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Las rutas se comparan en el orden en que se definen. La primera ruta que coincida con una solicitud será invocada.

### Usando funciones como devoluciones de llamada
La devolución de llamada puede ser cualquier objeto que sea invocable. Entonces puedes usar una función regular:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Usando clases y métodos como controlador
También puedes usar un método (estático o no) de una clase:

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// o
Flight::route('/', [ GreetingController::class, 'hello' ]); // método preferido
// o
Flight::route('/', [ 'GreetingController::hello' ]);
// o 
Flight::route('/', [ 'GreetingController->hello' ]);
```

O creando un objeto primero y luego llamando al método:

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

> **Nota:** De forma predeterminada, cuando se llama a un controlador dentro del framework, la clase `flight\Engine` siempre se inyecta a menos que especifiques un [contenedor de inyección de dependencias](/learn/dependency-injection-container)

### Enrutamiento específico por método

De forma predeterminada, los patrones de ruta se comparan con todos los métodos de solicitud. Puedes responder
a métodos específicos colocando un identificador antes de la URL.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// No puedes usar Flight::get() para rutas, ya que es un método
//    para obtener variables, no para crear una ruta.
Flight::post('/', function() { /* código */ });
Flight::patch('/', function() { /* código */ });
Flight::put('/', function() { /* código */ });
Flight::delete('/', function() { /* código */ });
```

También puedes asignar múltiples métodos a una sola devolución de llamada usando un delimitador `|`:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### Manejo especial para solicitudes HEAD y OPTIONS

Flight proporciona manejo integrado para solicitudes HTTP `HEAD` y `OPTIONS`:

#### Solicitudes HEAD

- **Las solicitudes HEAD** se tratan igual que las solicitudes `GET`, pero Flight elimina automáticamente el cuerpo de la respuesta antes de enviarlo al cliente.
- Esto significa que puedes definir una ruta para `GET`, y las solicitudes HEAD a la misma URL devolverán solo los encabezados (sin contenido), como se espera según los estándares HTTP.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// Una solicitud HEAD a /info devolverá los mismos encabezados, pero sin cuerpo.
```

#### Solicitudes OPTIONS

Las solicitudes `OPTIONS` son manejadas automáticamente por Flight para cualquier ruta definida.
- Cuando se recibe una solicitud OPTIONS, Flight responde con un estado `204 No Content` y un encabezado `Allow` que enumera todos los métodos HTTP compatibles para esa ruta.
- No necesitas definir una ruta separada para OPTIONS.

```php
// Para una ruta definida como:
Flight::route('GET|POST /users', function() { /* ... */ });

// Una solicitud OPTIONS a /users responderá con:
//
// Estado: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Usando el objeto Router

Adicionalmente, puedes obtener el objeto Router que tiene algunos métodos auxiliares para tu uso:

```php

$router = Flight::router();

// mapea todos los métodos igual que Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// Solicitud GET
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* código */});
$router->put('/users/update/@id', 	function() { /* código */});
$router->delete('/users/@id', 		function() { /* código */});
$router->patch('/users/@id', 		function() { /* código */});
```

### Expresiones regulares (Regex)
Puedes usar expresiones regulares en tus rutas:

```php
Flight::route('/user/[0-9]+', function () {
  // Esto coincidirá con /user/1234
});
```

Aunque este método está disponible, se recomienda usar parámetros nombrados, o
parámetros nombrados con expresiones regulares, ya que son más legibles y fáciles de mantener.

### Parámetros nombrados
Puedes especificar parámetros nombrados en tus rutas que se pasarán a
tu función de devolución de llamada. **Esto es más por la legibilidad de la ruta que cualquier otra cosa. Consulta la sección a continuación sobre la advertencia importante.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

También puedes incluir expresiones regulares con tus parámetros nombrados usando
el delimitador `:`:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Esto coincidirá con /bob/123
  // Pero no coincidirá con /bob/12345
});
```

> **Nota:** No se admite la coincidencia de grupos de expresiones regulares `()` con parámetros posicionales. Ej.: `:'\(`

#### Advertencia importante

Aunque en el ejemplo anterior parece que `@name` está directamente vinculado a la variable `$name`, no es así. El orden de los parámetros en la función de devolución de llamada es lo que determina qué se le pasa. Si cambiaras el orden de los parámetros en la función de devolución de llamada, las variables también se intercambiarían. Aquí tienes un ejemplo:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

Y si fueras a la siguiente URL: `/bob/123`, la salida sería `hello, 123 (bob)!`. 
_Ten cuidado_ al configurar tus rutas y tus funciones de devolución de llamada.

### Parámetros opcionales
Puedes especificar parámetros nombrados que sean opcionales para la coincidencia envolviendo
los segmentos entre paréntesis.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Esto coincidirá con las siguientes URLs:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Cualquier parámetro opcional que no coincida se pasará como `NULL`.

### Enrutamiento con comodines
La coincidencia se realiza solo en segmentos individuales de la URL. Si deseas coincidir con múltiples
segmentos, puedes usar el comodín `*`.

```php
Flight::route('/blog/*', function () {
  // Esto coincidirá con /blog/2000/02/01
});
```

Para enrutar todas las solicitudes a una sola devolución de llamada, puedes hacer:

```php
Flight::route('*', function () {
  // Hacer algo
});
```

### Manejador de 404 No Encontrado

De forma predeterminada, si no se encuentra una URL, Flight enviará una respuesta `HTTP 404 Not Found` muy simple y plana.
Si deseas tener una respuesta 404 más personalizada, puedes [mapear](/learn/extending) tu propio método `notFound`:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// También podrías usar Flight::render() con una plantilla personalizada.
    $output = <<<HTML
		<h1>Mi 404 No Encontrado Personalizado</h1>
		<h3>La página que has solicitado {$url} no se pudo encontrar.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Manejador de Método No Encontrado

De forma predeterminada, si se encuentra una URL pero el método no está permitido, Flight enviará una respuesta `HTTP 405 Method Not Allowed` muy simple y plana (Ej.: Method Not Allowed. Allowed Methods are: GET, POST). También incluirá un encabezado `Allow` con los métodos permitidos para esa URL.

Si deseas tener una respuesta 405 más personalizada, puedes [mapear](/learn/extending) tu propio método `methodNotFound`:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// También podrías usar Flight::render() con una plantilla personalizada.
	$output = <<<HTML
		<h1>Mi 405 Método No Permitido Personalizado</h1>
		<h3>El método que has solicitado para {$url} no está permitido.</h3>
		<p>Los Métodos Permitidos son: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## Uso avanzado

### Inyección de dependencias en rutas
Si deseas usar inyección de dependencias mediante un contenedor (PSR-11, PHP-DI, Dice, etc.), el
único tipo de rutas donde esto está disponible es creando directamente el objeto tú mismo
y usando el contenedor para crear tu objeto, o puedes usar cadenas para definir la clase y el
método a llamar. Puedes ir a la página de [Inyección de Dependencias](/learn/dependency-injection-container) para
obtener más información.

Aquí tienes un ejemplo rápido:

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
		// hacer algo con $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// Configura el contenedor con los parámetros que necesites
// Consulta la página de Inyección de Dependencias para más información sobre PSR-11
$dice = new \Dice\Dice();

// ¡No olvides reasignar la variable con '$dice = '!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Registra el manejador del contenedor
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Rutas como siempre
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// o
Flight::route('/hello/@id', 'Greeting->hello');
// o
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Pasar la ejecución a la siguiente ruta
<span class="badge bg-warning">Obsoleto</span>
Puedes pasar la ejecución a la siguiente ruta que coincida devolviendo `true` desde
tu función de devolución de llamada.

```php
Flight::route('/user/@name', function (string $name) {
  // Verificar alguna condición
  if ($name !== "Bob") {
    // Continuar a la siguiente ruta
    return true;
  }
});

Flight::route('/user/*', function () {
  // Esto se llamará
});
```

Ahora se recomienda usar [middleware](/learn/middleware) para manejar casos de uso complejos como este.

### Alias de rutas
Al asignar un alias a una ruta, puedes llamar a ese alias más tarde en tu aplicación de forma dinámica para que se genere posteriormente en tu código (ej.: un enlace en una plantilla HTML, o generar una URL de redirección).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// o 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// más adelante en el código, en algún lugar
class UserController {
	public function update() {

		// código para guardar usuario...
		$id = $user['id']; // 5 por ejemplo

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // devolverá '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

Esto es especialmente útil si tu URL llega a cambiar. En el ejemplo anterior, digamos que los usuarios se movieron a `/admin/users/@id` en su lugar.
Con el alias establecido para la ruta, ya no necesitas buscar todas las URLs antiguas en tu código y cambiarlas porque el alias ahora devolverá `/admin/users/5` como en el ejemplo anterior.

El alias de rutas también funciona en grupos:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// o
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Inspeccionando información de la ruta
Si deseas inspeccionar la información de la ruta coincidente, hay 2 formas de hacerlo:

1. Puedes usar la propiedad `executedRoute` en el objeto `Flight::router()`.
2. Puedes solicitar que el objeto de ruta se pase a tu devolución de llamada pasando `true` como tercer parámetro en el método de ruta. El objeto de ruta siempre será el último parámetro pasado a tu función de devolución de llamada.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Hacer algo con $route
  // Arreglo de métodos HTTP comparados
  $route->methods;

  // Arreglo de parámetros nombrados
  $route->params;

  // Expresión regular coincidente
  $route->regex;

  // Contiene el contenido de cualquier '*' usado en el patrón de URL
  $route->splat;

  // Muestra la ruta de la URL... si realmente lo necesitas
  $route->pattern;

  // Muestra qué middleware está asignado a esta
  $route->middleware;

  // Muestra el alias asignado a esta ruta
  $route->alias;
});
```

> **Nota:** La propiedad `executedRoute` solo se establecerá después de que se haya ejecutado una ruta. Si intentas acceder a ella antes de que se haya ejecutado una ruta, será `NULL`. ¡También puedes usar executedRoute en [middleware](/learn/middleware)!

#### Pasar `true` en la definición de la ruta
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Arreglo de métodos HTTP comparados
  $route->methods;

  // Arreglo de parámetros nombrados
  $route->params;

  // Expresión regular coincidente
  $route->regex;

  // Contiene el contenido de cualquier '*' usado en el patrón de URL
  $route->splat;

  // Muestra la ruta de la URL... si realmente lo necesitas
  $route->pattern;

  // Muestra qué middleware está asignado a esta
  $route->middleware;

  // Muestra el alias asignado a esta ruta
  $route->alias;
}, true);// <-- Este parámetro true es lo que hace que eso suceda
```

### Agrupación de rutas y middleware
Puede haber ocasiones en las que desees agrupar rutas relacionadas (como `/api/v1`).
Puedes hacer esto usando el método `group`:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Coincide con /api/v1/users
  });

  Flight::route('/posts', function () {
	// Coincide con /api/v1/posts
  });
});
```

Incluso puedes anidar grupos de grupos:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() obtiene variables, no establece una ruta. Consulta el contexto del objeto a continuación.
	Flight::route('GET /users', function () {
	  // Coincide con GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Coincide con POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Coincide con PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() obtiene variables, no establece una ruta. Consulta el contexto del objeto a continuación.
	Flight::route('GET /users', function () {
	  // Coincide con GET /api/v2/users
	});
  });
});
```

#### Agrupación con contexto de objeto

Aún puedes usar la agrupación de rutas con el objeto `Engine` de la siguiente manera:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // usa la variable $router
  $router->get('/users', function () {
	// Coincide con GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Coincide con POST /api/v1/posts
  });
});
```

> **Nota:** Este es el método preferido para definir rutas y grupos con el objeto `$router`.

#### Agrupación con middleware

También puedes asignar middleware a un grupo de rutas:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Coincide con /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // o [ new MyAuthMiddleware() ] si deseas usar una instancia
```

Consulta más detalles en la página de [middleware de grupo](/learn/middleware#grouping-middleware).

### Enrutamiento de recursos
Puedes crear un conjunto de rutas para un recurso usando el método `resource`. Esto creará
un conjunto de rutas para un recurso que sigue las convenciones RESTful.

Para crear un recurso, haz lo siguiente:

```php
Flight::resource('/users', UsersController::class);
```

Y lo que sucederá en segundo plano es que creará las siguientes rutas:

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

Y tu controlador usará los siguientes métodos:

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

> **Nota**: Puedes ver las rutas recién agregadas con `runway` ejecutando `php runway routes`.

#### Personalizando rutas de recursos

Hay algunas opciones para configurar las rutas de recursos.

##### Base de alias

Puedes configurar `aliasBase`. De forma predeterminada, el alias es la última parte de la URL especificada.
Por ejemplo, `/users/` resultaría en un `aliasBase` de `users`. Cuando se crean estas rutas,
los alias son `users.index`, `users.create`, etc. Si deseas cambiar el alias, establece `aliasBase`
al valor que desees.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only y Except

También puedes especificar qué rutas deseas crear usando las opciones `only` y `except`.

```php
// Permitir solo estos métodos y bloquear el resto
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Bloquear solo estos métodos y permitir el resto
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

Básicamente, estas son opciones de lista blanca y lista negra para que puedas especificar qué rutas deseas crear.

##### Middleware

También puedes especificar middleware que se ejecute en cada una de las rutas creadas por el método `resource`.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Respuestas de transmisión (Streaming)

Ahora puedes transmitir respuestas al cliente usando `stream()` o `streamWithHeaders()`.
Esto es útil para enviar archivos grandes, procesos de larga duración o generar respuestas grandes.
Transmitir una ruta se maneja de manera un poco diferente a una ruta regular.

> **Nota:** Las respuestas de transmisión solo están disponibles si has establecido [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) en `false`.

#### Transmitir con encabezados manuales

Puedes transmitir una respuesta al cliente usando el método `stream()` en una ruta. Si
haces esto, debes establecer todos los encabezados manualmente antes de generar cualquier salida al cliente.
Esto se hace con la función `header()` de PHP o con el método `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// obviamente sanitizarías la ruta y todo eso.
	$fileNameSafe = basename($filename);

	// Si tienes encabezados adicionales que establecer aquí después de que la ruta se haya ejecutado,
	// debes definirlos antes de que se imprima cualquier cosa.
	// Todos deben ser una llamada directa a la función header()
	// o una llamada a Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// o
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// establece manualmente la longitud del contenido si lo deseas
	header('Content-Length: '.filesize($filePath));
	// o
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Transmite el archivo al cliente mientras se lee
	readfile($filePath);

// Esta es la línea mágica aquí
})->stream();
```

#### Transmitir con encabezados

También puedes usar el método `streamWithHeaders()` para establecer los encabezados antes de comenzar a transmitir.

```php
Flight::route('/stream-users', function() {

	// puedes agregar cualquier encabezado adicional aquí
	// solo debes usar header() o Flight::response()->setRealHeader()

	// sin importar cómo obtengas tus datos, solo como ejemplo...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Esto es necesario para enviar los datos al cliente
		ob_flush();
	}
	echo '}';

// Así es como estableces los encabezados antes de comenzar a transmitir.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// código de estado opcional, el valor predeterminado es 200
	'status' => 200
]);
```

## Ver también
- [Middleware](/learn/middleware) - Uso de middleware con rutas para autenticación, registro, etc.
- [Inyección de Dependencias](/learn/dependency-injection-container) - Simplificando la creación y gestión de objetos en rutas.
- [¿Por qué un Framework?](/learn/why-frameworks) - Comprendiendo los beneficios de usar un framework como Flight.
- [Extensión](/learn/extending) - Cómo extender Flight con tu propia funcionalidad, incluido el método `notFound`.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - Función de PHP para coincidencia de expresiones regulares.

## Solución de problemas
- Los parámetros de ruta se comparan por orden, no por nombre. Asegúrate de que el orden de los parámetros en la devolución de llamada coincida con la definición de la ruta.
- Usar `Flight::get()` no define una ruta; usa `Flight::route('GET /...')` para enrutar o el contexto del objeto Router en grupos (ej. `$router->get(...)`).
- La propiedad executedRoute solo se establece después de que una ruta se ejecuta; es NULL antes de la ejecución.
- La transmisión requiere que la funcionalidad de almacenamiento en búfer de salida heredada de Flight esté deshabilitada (`flight.v2.output_buffering = false`).
- Para la inyección de dependencias, solo ciertas definiciones de rutas admiten la instanciación basada en contenedor.

### 404 No Encontrado o Comportamiento Inesperado de Ruta

Si estás viendo un error 404 No Encontrado (pero juras por tu vida que realmente está ahí y no es un error tipográfico), esto en realidad podría ser un problema
con que estés devolviendo un valor en tu endpoint de ruta en lugar de solo imprimirlo. La razón de esto es intencional, pero podría sorprender a algunos desarrolladores.

```php
Flight::route('/hello', function(){
	// Esto podría causar un error 404 No Encontrado
	return 'Hello World';
});

// Lo que probablemente quieras
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

La razón de esto se debe a un mecanismo especial integrado en el enrutador que maneja la salida devuelta como una señal para "ir a la siguiente ruta".
Puedes ver el comportamiento documentado en la sección [Enrutamiento](/learn/routing#passing).

## Registro de cambios
- v3: Se agregó enrutamiento de recursos, alias de rutas y soporte de transmisión, grupos de rutas y soporte de middleware.
- v1: La gran mayoría de las características básicas están disponibles.