# Twig

[Twig](https://twig.symfony.com/) es un motor de plantillas flexible, rápido y seguro para PHP. Es el lenguaje de plantillas utilizado por Symfony y muchos otros proyectos, lo que significa que las herramientas de codificación de IA y la mayoría de los desarrolladores de PHP ya conocen bien su sintaxis. Twig compila las plantillas a PHP optimizado, escapa automáticamente la salida por defecto (excelente para la protección contra XSS), y es fácil de extender con filtros, funciones y extensiones.

## Instalación

Instalar con composer.

```bash
composer require twig/twig
```

## Configuración Básica

Hay algunas opciones de configuración básicas para comenzar. Puedes leer más sobre ellas en la [Documentación de Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Dónde Twig almacena sus plantillas compiladas
		'cache' => __DIR__ . '/../cache/twig',
		// Recompilar plantillas cuando la fuente cambia (útil en desarrollo)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Registrar Twig como la Clase de Vista

Si prefieres reutilizar un solo entorno de Twig (recomendado para producción), regístralo y apunta `render` a él:

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	],
]);

$app->map('render', function(string $template, array $data): void {
	echo Flight::view()->render($template, $data);
});
```

## Ejemplo Simple de Diseño

Aquí tienes un ejemplo simple de un archivo de diseño. Este es el archivo que se utilizará para envolver todas tus otras vistas.

```html
{# app/views/layout.twig #}
<!doctype html>
<html lang="en">
	<head>
		<title>{% if title %}{{ title }} - {% endif %}My App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				{# tus elementos de navegación aquí #}
			</nav>
		</header>
		<div id="content">
			{# Este es el truco aquí #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

Y ahora tenemos tu archivo que se va a renderizar dentro de ese bloque de contenido:

```html
{# app/views/home.twig #}
{# Esto le dice a Twig que este archivo está "dentro" del archivo layout.twig #}
{% extends 'layout.twig' %}

{# Este es el contenido que se renderizará dentro del diseño dentro del bloque de contenido #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Luego cuando vayas a renderizar esto dentro de tu función o controlador, harías algo como esto:

```php
// ruta simple
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// o si estás usando un controlador
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Home Page'
		]);
	}
}
```

¡Consulta la [Documentación de Twig](https://twig.symfony.com/doc/3.x/) para más información sobre cómo usar Twig a su máximo potencial!

## Depuración

Twig viene con una [Extensión de Depuración](https://twig.symfony.com/doc/3.x/functions/dump.html) que añade una función `dump()` que puedes usar dentro de las plantillas. Habilítala solo en desarrollo:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // requerido para la función dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Luego en una plantilla:

```html
{{ dump(user) }}
```

También puedes combinar Twig con [Tracy](/awesome-plugins/tracy) para depuración a nivel de PHP. Para métricas a nivel de plantilla (tiempo de renderizado, memoria, qué plantillas/bloques se ejecutaron), usa el **panel Twig** opcional en [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): pasa un `Twig\Profiler\Profile` como `twig_profile` a `TracyExtensionLoader`. La `TwigTracyExtension` opcional expone `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` en las plantillas cuando Tracy está activado.

## Nota de Seguridad

Twig escapa automáticamente la salida por defecto, lo que ayuda a proteger contra ataques XSS. Prefiere `{{ variable }}` para texto. Usa el filtro `|raw` solo cuando confíes intencionalmente en el contenido HTML (por ejemplo, markdown sanitizado que ya procesaste en el servidor).