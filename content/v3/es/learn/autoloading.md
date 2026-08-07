# Autocarga

## Resumen

La autocarga es un concepto en PHP donde especificas un directorio o directorios desde los cuales cargar clases. Es mucho más beneficioso que usar `require` o `include` para cargar clases. También es un requisito para usar paquetes de Composer.

Hacer bien la autocarga también es importante para el [desarrollo asistido por IA](/learn/ai): los agentes colocan archivos donde apunta el espacio de nombres. Si la **mayúscula/minúscula** de las carpetas y la del espacio de nombres no coinciden, aparecen errores de clase no encontrada en Linux incluso cuando las cosas "funcionaban" en un disco Mac que no distingue mayúsculas de minúsculas.

## Entendiendo

Por defecto, cualquier clase `Flight` se autocarga automáticamente gracias a Composer. Para las clases de **tu** aplicación tienes dos enfoques comunes:

1. **Composer PSR-4** (lo que usa el [esqueleto oficial](https://github.com/flightphp/skeleton)): mapea un prefijo de espacio de nombres a un directorio en `composer.json`, luego ejecuta `composer dump-autoload`.
2. **`Flight::path()`**: apunta el cargador de Flight a directorios (útil para aplicaciones simples o cuando no usas Composer para el código de la aplicación).

Usar un autocargador simplifica mucho tu código. En lugar de un muro de `include` / `require` al principio de cada archivo, las clases se cargan cuando las usas por primera vez.

### Sensibilidad a mayúsculas/minúsculas (lee esto dos veces)

**Los espacios de nombres deben coincidir con la estructura de directorios *y* con las mayúsculas/minúsculas de esos directorios.**

| Funciona | Se rompe en Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` con la carpeta `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Mezclar `App\` con `controllers` en minúsculas |

Los espacios de nombres de PHP no distinguen mayúsculas/minúsculas en algunos contextos, pero **Composer y el sistema de archivos no**. El esqueleto oficial estandariza en:

- Composer: `"App\\": "app/"`
- Carpetas: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), no `controllers` / `middlewares`

La documentación anterior y los ejemplos de la comunidad a veces usaban `app\controllers` en minúsculas. Eso sigue funcionando si tus carpetas están en minúsculas, pero **los nuevos proyectos de esqueleto usan `App\` + carpetas PascalCase**. Elige una convención por proyecto y mantente fiel a ella para que los humanos y las herramientas de IA no inventen una segunda estructura.

## Esqueleto (recomendado para nuevos proyectos)

Después de `composer create-project flightphp/skeleton`, el código de la aplicación se autocarga mediante Composer—no se requiere `Flight::path()` para las clases `App\`:

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
// app/config/routes.php — Dice resuelve App\Controller\… mediante el contenedor
$router->get('/', [HomeController::class, 'index']);
```

Consulta [Instalación](/install) para el árbol completo y [IA y experiencia de desarrollo](/learn/ai) para ver cómo `AGENTS.md` documenta esta estructura para asistentes de codificación.

## Uso básico (`Flight::path()`)

Supongamos que tenemos una estructura de directorios como la siguiente:

```text
# Ruta de ejemplo
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - contiene los controladores de este proyecto
│   ├── translations
│   ├── UTILS - contiene clases solo para esta aplicación (esto está todo en mayúsculas a propósito para un ejemplo más adelante)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Puede que hayas notado que esto es similar a un árbol de aplicación típico (el propio sitio de documentación usa una estructura organizada). La carpeta `controllers` en minúsculas aquí es una *elección* válida: simplemente no es el valor predeterminado actual del esqueleto.

Puedes especificar cada directorio desde el que cargar de esta manera:

```php

/**
 * public/index.php
 */

// Añade una ruta al autocargador
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// no se requiere espacio de nombres

// Se recomienda que todas las clases autocargadas utilicen PascalCase (cada palabra en mayúscula inicial, sin espacios)
class MyController {

	public function index() {
		// hacer algo
	}
}
```

## Espacios de nombres con `Flight::path()`

Si tienes espacios de nombres, en realidad se vuelve muy fácil implementar esto. Debes usar el método `Flight::path()` para especificar el directorio raíz (no la raíz del documento ni la carpeta `public/`) de tu aplicación.

```php

/**
 * public/index.php
 */

// Añade una ruta al autocargador
Flight::path(__DIR__.'/../');
```

Ahora así es como podría verse tu controlador. Mira el ejemplo a continuación, pero presta atención a los comentarios para obtener información importante.

```php
/**
 * app/controllers/MyController.php
 */

// los espacios de nombres son obligatorios
// los espacios de nombres son iguales a la estructura de directorios
// los espacios de nombres deben seguir las mismas mayúsculas/minúsculas que la estructura de directorios
// los espacios de nombres y los directorios no pueden tener guiones bajos (a menos que se establezca Loader::setV2ClassLoading(false))
namespace app\controllers;

// Se recomienda que todas las clases autocargadas utilicen PascalCase (cada palabra en mayúscula inicial, sin espacios)
// A partir de 3.7.2, puedes usar Pascal_Snake_Case para los nombres de tus clases ejecutando Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// hacer algo
	}
}
```

Y si quisieras autocargar una clase en tu directorio de utilidades, básicamente harías lo mismo:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// el espacio de nombres debe coincidir con la estructura de directorios y con las mayúsculas/minúsculas (nota que el directorio UTILS está todo en mayúsculas
//     como en el árbol de archivos anterior)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// hacer algo
	}
}
```

### Espacio de nombres estilo esqueleto (mismas reglas, diferente uso de mayúsculas)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

La regla no cambió—solo cambia el uso de mayúsculas/minúsculas elegido por el esqueleto para carpetas/espacios de nombres. **Cualquiera que sea el uso de mayúsculas/minúsculas de tus carpetas, tu línea `namespace` debe coincidir.**

## Guiones bajos en nombres de clases

A partir de 3.7.2, puedes usar Pascal_Snake_Case para los nombres de tus clases ejecutando `Loader::setV2ClassLoading(false);`. 
Esto te permitirá usar guiones bajos en los nombres de tus clases. 
No se recomienda, pero está disponible para quienes lo necesiten.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Añade una ruta al autocargador
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// no se requiere espacio de nombres

class My_Controller {

	public function index() {
		// hacer algo
	}
}
```

## Ver también

- [Instalación](/install) - Árbol del esqueleto y valores predeterminados `App\` para nuevos proyectos.
- [Enrutamiento](/learn/routing) - Cómo mapear rutas a controladores y renderizar vistas.
- [Inyección de dependencias](/learn/dependency-injection-container) - Cómo obtienen los controladores `Engine` y servicios.
- [IA y experiencia de desarrollo](/learn/ai) - Mantén a los agentes alineados con tu estructura mediante `AGENTS.md`.
- [¿Por qué un framework?](/learn/why-frameworks) - Comprende los beneficios de usar un framework como Flight.

## Solución de problemas

Si no logras entender por qué no se encuentran tus clases con espacios de nombres, recuerda: con `Flight::path()`, apunta a la **raíz del proyecto** (o la base correcta para tu espacio de nombres), no solo a una carpeta anidada que olvidaste reflejar en el espacio de nombres.

Con Composer PSR-4, ejecuta `composer dump-autoload` después de cambiar las asignaciones en `composer.json`.

En CI de Linux o producción, un uso incorrecto de mayúsculas/minúsculas en carpetas es una falla muy común de «funciona en mi máquina».

### Clase no encontrada (la autocarga no funciona)

Podría haber un par de razones para que esto no ocurra. A continuación se muestran algunos ejemplos.

#### Nombre de archivo incorrecto

La más común es que el nombre de la clase no coincida con el nombre del archivo.

Si tienes una clase llamada `MyClass`, entonces el archivo debe llamarse `MyClass.php`. Si tienes una clase llamada `MyClass` y el archivo se llama `myclass.php`, el autocargador no podrá encontrarla.

#### Espacio de nombres o mayúsculas/minúsculas de carpeta incorrectos

Si estás usando espacios de nombres, entonces el espacio de nombres debe coincidir con la estructura de directorios **incluyendo las mayúsculas/minúsculas**.

```php
// ...código...

// si tu MyController está en app/Controller (esqueleto) y tiene el espacio de nombres App\Controller
// esto no funcionará:
Flight::route('/hello', 'MyController->hello');

// Estilo esqueleto:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Estructura anterior en minúsculas (solo si tus carpetas son realmente app/controllers):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// o completamente calificado:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` no definido (código de aplicación sin Composer)

Si dependes de `Flight::path()` en lugar de Composer para las clases de la aplicación, define la ruta antes de las rutas que hagan referencia a esas clases (a menudo al inicio del arranque / `public/index.php`):

```php
// Añade una ruta al autocargador (raíz del proyecto para aplicaciones con espacios de nombres)
Flight::path(__DIR__.'/../');
```

El esqueleto oficial utiliza principalmente **Composer PSR-4** para `App\`, por lo que normalmente no necesitarás `Flight::path()` para controladores y modelos allí.

## Historial de cambios

- Documentación: documenta el esqueleto `App\` + carpetas PascalCase y las trampas de mayúsculas/minúsculas para humanos y herramientas de IA.
- v3.7.2 - Puedes usar Pascal_Snake_Case para los nombres de tus clases ejecutando `Loader::setV2ClassLoading(false);`
- v2.0 - Se añadió la funcionalidad de autocarga.