# Twig

[Twig](https://twig.symfony.com/) is a flexible, fast, and secure template engine for PHP. It is the templating language used by Symfony and many other projects, which means AI coding tools and most PHP developers already know its syntax well. Twig compiles templates down to optimized PHP, auto-escapes output by default (great for XSS protection), and is easy to extend with filters, functions, and extensions.

## Installation

Install with composer.

```bash
composer require twig/twig
```

## Basic Configuration

There are some basic configuration options to get started. You can read more about them in the [Twig Documentation](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Where Twig stores its compiled templates
		'cache' => __DIR__ . '/../cache/twig',
		// Recompile templates when the source changes (handy in development)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Registering Twig as the View Class

If you prefer to reuse a single Twig environment (recommended for production), register it and point `render` at it:

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

## Simple Layout Example

Here's a simple example of a layout file. This is the file that will be used to wrap all of your other views.

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
				{# your nav elements here #}
			</nav>
		</header>
		<div id="content">
			{# This is the magic right here #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

And now we have your file that's going to render inside that content block:

```html
{# app/views/home.twig #}
{# This tells Twig that this file is "inside" the layout.twig file #}
{% extends 'layout.twig' %}

{# This is the content that will be rendered inside the layout inside the content block #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Then when you go to render this inside your function or controller, you would do something like this:

```php
// simple route
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// or if you're using a controller
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

See the [Twig Documentation](https://twig.symfony.com/doc/3.x/) for more information on how to use Twig to its fullest potential!

## Debugging

Twig ships with a [Debug Extension](https://twig.symfony.com/doc/3.x/functions/dump.html) that adds a `dump()` function you can use inside templates. Enable it only in development:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // required for the dump() function
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Then in a template:

```html
{{ dump(user) }}
```

You can also pair Twig with [Tracy](/awesome-plugins/tracy) for PHP-level debugging. For template-level metrics (render time, memory, which templates/blocks ran), use the optional **Twig panel** in [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): pass a `Twig\Profiler\Profile` as `twig_profile` to `TracyExtensionLoader`. Optional `TwigTracyExtension` exposes `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` in templates when Tracy is on.

## Security Note

Twig auto-escapes output by default, which helps protect against XSS attacks. Prefer `{{ variable }}` for text. Only use the `|raw` filter when you intentionally trust the HTML content (for example, sanitized markdown you already processed server-side).
