# Extensiones del Panel Tracy de Flight

Este es un conjunto de extensiones para hacer el trabajo con Flight un poco más enriquecido.

- **Flight** - Analiza todas las variables de Flight.
- **Database** - Analiza todas las consultas que se han ejecutado en la página (si inicia correctamente la conexión a la base de datos)
- **Request** - Analiza todas las variables `$_SERVER` y examina todas las cargas globales (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Analiza todas las variables `$_SESSION` si las sesiones están activas.
- **Twig** *(opcional)* - Analiza el tiempo de renderizado de plantillas Twig, memoria y qué plantillas/bloques/macros se ejecutaron (requiere `twig/twig` y una configuración `twig_profile`)

Esto es especialmente útil con el [esqueleto oficial](https://github.com/flightphp/skeleton), que por defecto usa Twig: el mismo diseño que siguen las [herramientas de IA](/learn/ai) también aparece claramente en la barra de Tracy.

Este es el Panel

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

¡Y cada panel muestra información muy útil sobre su aplicación!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Haga clic [aquí](https://github.com/flightphp/tracy-extensions) para ver el código.

## Instalación

¡Ejecute `composer require flightphp/tracy-extensions --dev` y ya está en camino!

Twig **no** es una dependencia estricta del paquete. Instale `twig/twig` solo si desea el panel de Twig (el esqueleto ya lo hace para las vistas).

## Configuración

Hay muy poca configuración que necesite hacer para comenzar. Necesitará iniciar el depurador Tracy antes de usar esto [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// código de arranque
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Es posible que necesite especificar su entorno con Debugger::enable(Debugger::DEVELOPMENT)

// si usa conexiones de base de datos en su aplicación, hay un 
// envoltorio PDO requerido para usar SOLO EN DESARROLLO (¡no en producción!)
// Tiene los mismos parámetros que una conexión PDO normal
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// o si adjunta esto al framework Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// ahora cada vez que haga una consulta capturará el tiempo, la consulta y los parámetros

// Esto conecta los puntos
if(Debugger::$showBar === true) {
	// Esto necesita ser false o Tracy no puede renderizar realmente :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// más código

Flight::start();
```

## Configuración Adicional

### Datos de Sesión

Si tiene un manejador de sesión personalizado (como ghostff/session), puede pasar cualquier array de datos de sesión a Tracy y automáticamente lo mostrará. Lo pasa con la clave `session_data` en el segundo parámetro del constructor de `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// o use flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Esto necesita ser false o Tracy no puede renderizar realmente :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// rutas y otras cosas...

Flight::start();
```

### Panel de Twig (opcional)

Si su aplicación usa [Twig](/awesome-plugins/twig) (incluido el esqueleto oficial), puede mostrar métricas de plantillas en la barra de Tracy. Cree un `Profile` de Twig, adjunte `ProfilerExtension` a su entorno, luego pase ese perfil al cargador bajo la clave **`twig_profile`**. Adjunte el perfilado solo en desarrollo.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Opcional: expone helpers de dump de Tracy en las plantillas
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Mapea Flight::render() a Twig (ejemplo)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Lo que muestra el panel**

- Tiempo total de renderizado de Twig y memoria
- Conteo de llamadas a plantillas / bloques / macros
- Cada plantilla que se renderizó, con su propio tiempo y memoria

La pestaña de Twig está **oculta** cuando no se renderizaron plantillas para la solicitud, o cuando omite `twig_profile` (o no tiene Twig instalado): los otros paneles de Flight siguen funcionando.

En un `services.php` de estilo esqueleto, construya el mismo `$profile` / `ProfilerExtension` cuando la depuración esté activada, pase `twig_profile` a `TracyExtensionLoader`, y siga usando su entorno Twig compartido para `$app->render()`.

### Latte

_Se requiere PHP 8.1+ para esta sección._

Si tiene Latte instalado en su proyecto, Tracy tiene una integración nativa con Latte para analizar sus plantillas. Simplemente registre la extensión con su instancia de Latte (este es el propio puente de Tracy de Latte, no el panel de Twig anterior).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// otras configuraciones...

	// solo agrega la extensión si la Barra de Depuración de Tracy está habilitada
	if(Debugger::$showBar === true) {
		// aquí es donde agrega el Panel de Latte a Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Ver También

- [Tracy](/awesome-plugins/tracy) - Configuración base de Tracy para Flight
- [Twig](/awesome-plugins/twig) - Plantillas usadas por el esqueleto y el panel de Twig
- [Plantillas](/learn/templates) - Cómo Flight mapea `render` a Twig/Latte
- [Instalación](/install) - El esqueleto incluye tracy-extensions en dev