# Runway

Runway es una aplicación CLI que te ayuda a gestionar tus aplicaciones Flight. Puede generar controladores, mostrar todas las rutas, ejecutar ayudantes de configuración de IA, migraciones (en el esqueleto) y más. Está basado en la excelente biblioteca [adhocore/php-cli](https://github.com/adhocore/php-cli).

Haz clic [aquí](https://github.com/flightphp/runway) para ver el código.

Los comandos de scaffolding están intencionalmente alineados con el [esqueleto oficial](https://github.com/flightphp/skeleton) para que las [herramientas de codificación de IA](/learn/ai) y los humanos obtengan las mismas rutas, espacios de nombres y estilo de inyección de constructor cada vez.

## Instalación

Instalar con composer.

```bash
composer require flightphp/runway
```

El esqueleto ya depende de Runway; usa `php runway` desde la raíz del proyecto.

## Configuración Básica

La primera vez que ejecutes Runway, intentará encontrar una configuración `runway` en `app/config/config.php` mediante la clave `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// opcional; el esqueleto también usa index_root para la entrada pública
		'index_root' => 'public/index.php',
    ],
];
```

> **NOTA** - A partir de **v1.2.0**, `.runway-config.json` está obsoleto en favor de `app/config/config.php`. Migra con `php runway config:migrate` al actualizar proyectos antiguos. El esqueleto puede seguir escribiendo un pequeño `.runway-config.json` al crear el proyecto para compatibilidad; prefiere la clave `runway` en `config.php` de ahora en adelante.

### Detección de Raíz del Proyecto

Runway es lo suficientemente inteligente para detectar la raíz de tu proyecto, incluso si lo ejecutas desde un subdirectorio. Busca indicadores como `composer.json`, `.git`, o `app/config/config.php` para determinar dónde está la raíz del proyecto. ¡Esto significa que puedes ejecutar comandos de Runway desde cualquier lugar de tu proyecto!

## Uso

Runway tiene una serie de comandos que puedes usar para gestionar tu aplicación Flight. Hay dos formas fáciles de usar Runway.

1. Si estás usando el proyecto esqueleto, puedes ejecutar `php runway [comando]` desde la raíz de tu proyecto.
1. Si estás usando Runway como un paquete instalado vía composer, puedes ejecutar `vendor/bin/runway [comando]` desde la raíz de tu proyecto.

### Lista de Comandos

Puedes ver una lista de todos los comandos disponibles ejecutando el comando `php runway`.

```bash
php runway
```

Solo confía en los comandos que realmente aparezcan en esa lista para tu instalación (comandos principales de Runway vs comandos específicos del proyecto como `migrate` del esqueleto).

### Ayuda de Comandos

Para cualquier comando, puedes pasar el flag `--help` para obtener más información sobre cómo usar el comando.

```bash
php runway routes --help
php runway make:controller --help
```

Aquí hay algunos ejemplos:

### Generar un Controlador

`make:controller` genera un controlador que coincide con el diseño del esqueleto oficial:

| | |
|--|--|
| **Ruta** | `app/Controller/{Nombre}.php` |
| **Espacio de nombres** | `App\Controller` |
| **Estilo** | Inyección de constructor de `flight\Engine` (sin `Flight::` en el cuerpo de la clase) |

```bash
php runway make:controller MiControlador
# → app/Controller/MiControlador.php
#   namespace App\Controller;
```

Ejemplo de la estructura que debes esperar (simplificada):

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MiControlador
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// ej. $this->app->render('…', […]);
	}
}
```

Regístralo con un callable de clase para que Dice pueda construir el controlador:

```php
// app/config/routes.php
use App\Controller\MiControlador;

$router->get('/mio', [MiControlador::class, 'index']);
```

**¿Por qué este diseño?** La **mayúscula** de la carpeta debe coincidir con el espacio de nombres (`Controller` no `controllers`) para Composer PSR-4 en Linux—ver [Autocarga](/learn/autoloading). La misma ruta es la que los archivos `AGENTS.md` raíz y con alcance indican a las herramientas de IA que usen, así que los controladores generados y escritos a mano permanecen idénticos.

> Documentación antigua y proyectos comunitarios a veces usaban `app/controllers/` y `app\controllers`. Eso sigue siendo válido si *tu* árbol aún usa carpetas en minúsculas. **Los nuevos proyectos esqueleto y la salida actual de `make:controller` usan `app/Controller/` + `App\Controller`.**

### Generar un Modelo Active Record

Primero asegúrate de haber instalado el plugin [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record usuarios
```

En el esqueleto oficial, los modelos viven bajo **`app/Model/`** con espacio de nombres **`App\Model`**, y la conexión a la base de datos es **[SimplePdo](/learn/simple-pdo)** (inyéctalo o pásalo al constructor de ActiveRecord). Los nombres de archivo/espacios de nombres generados siguen los valores predeterminados actuales de Runway y tu configuración `runway`—prefiere alinear los nuevos modelos con `App\Model` para que coincidan con [autocarga](/learn/autoloading) y `AGENTS.md`.

Ejemplo de un modelo consistente con la demostración de posts del esqueleto:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $título
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Si un generador más antiguo aún emite `app/records` / `app\records`, puedes mantener esa convención en aplicaciones heredadas o mover archivos a `app/Model/` y actualizar el espacio de nombres para que coincida con la mayúscula de la carpeta.

### Migraciones (esqueleto)

El esqueleto oficial incluye un comando de proyecto (descubierto desde `app/commands/`) como:

```bash
php runway migrate
```

Las migraciones son archivos SQL bajo `migrations/` (por ejemplo `YYYYMMDDHHMMSS_descripcion.sql` para SQLite y `…_descripcion.mysql.sql` para MySQL), seleccionados desde la configuración del controlador de base de datos / env. Los flags y comportamiento exactos están definidos por ese comando de proyecto—ejecuta `php runway migrate --help` en tu app.

### Ayudantes de IA

Runway expone comandos orientados a IA usados con [IA y experiencia de desarrollador](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Estos almacenan credenciales LLM y generan instrucciones de proyecto (principalmente **`AGENTS.md`**). En el esqueleto, trata `AGENTS.md` (y copias con alcance bajo `app/`) más **`SECURITY.md`** como la fuente de verdad para agentes.

### Mostrar Todas las Rutas

Esto mostrará todas las rutas que están actualmente registradas con Flight.

```bash
php runway routes
```

Si deseas ver solo rutas específicas, puedes pasar un flag para filtrar las rutas.

```bash
# Mostrar solo rutas GET
php runway routes --get

# Mostrar solo rutas POST
php runway routes --post

# etc.
```

## Agregar Comandos Personalizados a Runway

Si estás creando un paquete para Flight, o quieres agregar tus propios comandos personalizados a tu proyecto, puedes hacerlo creando un directorio `src/commands/`, `flight/commands/`, `app/commands/`, o `commands/` para tu proyecto/paquete. Si necesitas más personalización, consulta la sección de Configuración a continuación.

En el esqueleto, los comandos de proyecto viven en **`app/commands/`** con espacio de nombres **`App\Command`**. Runway los descubre por ruta; mantén esa carpeta sincronizada con el classmap/PSR-4 de Composer como ya hace tu proyecto.

Para crear un comando, simplemente extiende la clase `AbstractBaseCommand`, e implementa como mínimo un método `__construct` y un método `execute`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ComandoEjemplo extends AbstractBaseCommand
{
	/**
     * Construct
     *
     * @param array<string,mixed> $config Configuración desde app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Crear un ejemplo para la documentación', $config);
        $this->argument('<gif-divertido>', 'El nombre del gif divertido');
    }

	/**
     * Ejecuta la función
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Creando ejemplo...');

		// Hacer algo aquí

		$io->ok('¡Ejemplo creado!');
	}
}
```

¡Consulta la [Documentación de adhocore/php-cli](https://github.com/adhocore/php-cli) para más información sobre cómo construir tus propios comandos personalizados en tu aplicación Flight!

## Gestión de Configuración

Dado que la configuración se ha movido a `app/config/config.php` a partir de `v1.2.0`, hay algunos comandos de ayuda para gestionar la configuración.

> **Consejo del esqueleto:** Mantén `config.php` como valores PHP **literales**. Los secretos pertenecen a `.env`. Evita expresiones `$_ENV[...]` dentro de `config.php`—`config:set` reescribe ese archivo como datos estáticos y podría incrustar secretos en el archivo. Ver [Configuración](/learn/configuration).

### Migrar Configuración Antigua

Si tienes un archivo `.runway-config.json` antiguo, puedes migrarlo fácilmente a `app/config/config.php` con el siguiente comando:

```bash
php runway config:migrate
```

### Establecer Valor de Configuración

Puedes establecer un valor de configuración usando el comando `config:set`. Esto es útil si quieres actualizar un valor de configuración sin abrir el archivo.

```bash
php runway config:set app_root "app/"
```

### Obtener Valor de Configuración

Puedes obtener un valor de configuración usando el comando `config:get`.

```bash
php runway config:get app_root
```

## Todas las Configuraciones de Runway

Si necesitas personalizar la configuración para Runway, puedes establecer estos valores en `app/config/config.php`. A continuación hay algunas configuraciones adicionales que puedes establecer:

```php
<?php
// app/config/config.php
return [
    // ... otros valores de configuración ...

    'runway' => [
        // Aquí es donde está ubicado el directorio de tu aplicación
        'app_root' => 'app/',

        // Este es el directorio donde está ubicado tu archivo index raíz
        'index_root' => 'public/',

        // Estas son las rutas a las raíces de otros proyectos
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Las rutas base probablemente no necesiten configurarse, pero está aquí si lo quieres
        'base_paths' => [
            '/includes/libs/vendor', // si tienes una ruta realmente única para tu directorio vendor o algo similar
        ],

        // Las rutas finales son ubicaciones dentro de un proyecto para buscar los archivos de comando
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Si quieres agregar la ruta completa, adelante (absoluta o relativa a la raíz del proyecto)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Accediendo a la Configuración

Si necesitas acceder a los valores de configuración efectivamente, puedes acceder a ellos a través del método `__construct` o el método `app()`. También es importante notar que si tienes un archivo `app/config/services.php`, esos servicios también estarán disponibles para tu comando.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Acceder a la configuración
    $app_root = $this->config['runway']['app_root'];
    
    // Acceder a servicios como posiblemente una conexión a base de datos
    $database = $this->config['database']
    
    // ...
}
```

## Envoltorios de Ayudantes de IA

Runway tiene algunos envoltorios de ayudantes que facilitan que la IA genere comandos. Puedes usar `addOption` y `addArgument` de una manera que se sienta similar a Symfony Console. Esto es útil si estás usando herramientas de IA para generar tus comandos.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Crear un ejemplo para la documentación', $config);
    
    // El argumento mode es nullable y por defecto es completamente opcional
    $this->addOption('name', 'El nombre del ejemplo', null);
}
```

## Ver También

- [Instalación](/install) - Árbol del esqueleto y valores predeterminados de create-project
- [Autocarga](/learn/autoloading) - `App\` y mayúscula de carpeta
- [Inyección de Dependencias](/learn/dependency-injection-container) - Inyección Dice + Engine para controladores generados
- [IA y Experiencia de Desarrollador](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Modelos usados con `make:record` / `App\Model` del esqueleto
- [SimplePdo](/learn/simple-pdo) - Conexión a BD usada por migraciones y modelos del esqueleto