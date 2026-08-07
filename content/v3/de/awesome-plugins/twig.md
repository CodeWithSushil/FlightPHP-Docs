# Twig

[Twig](https://twig.symfony.com/) ist eine flexible, schnelle und sichere Template-Engine für PHP. Es ist die Template-Sprache, die von Symfony und vielen anderen Projekten verwendet wird, was bedeutet, dass KI-Codierungstools und die meisten PHP-Entwickler ihre Syntax bereits gut kennen. Twig kompiliert Templates in optimiertes PHP, maskiert die Ausgabe standardmäßig automatisch (ideal für XSS-Schutz) und lässt sich leicht mit Filtern, Funktionen und Erweiterungen erweitern.

## Installation

Mit Composer installieren.

```bash
composer require twig/twig
```

## Grundkonfiguration

Es gibt einige grundlegende Konfigurationsoptionen, um zu beginnen. Sie können mehr darüber in der [Twig-Dokumentation](https://twig.symfony.com/doc/3.x/) lesen.

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Wo Twig seine kompilierten Templates speichert
		'cache' => __DIR__ . '/../cache/twig',
		// Templates neu kompilieren, wenn sich die Quelle ändert (praktisch in der Entwicklung)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Twig als View-Klasse registrieren

Wenn Sie eine einzelne Twig-Umgebung wiederverwenden möchten (für die Produktion empfohlen), registrieren Sie diese und verweisen Sie `render` darauf:

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

## Einfaches Layout-Beispiel

Hier ist ein einfaches Beispiel für eine Layout-Datei. Dies ist die Datei, die verwendet wird, um alle anderen Views einzubetten.

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
				{# Ihre Navigationselemente hier #}
			</nav>
		</header>
		<div id="content">
			{# Hier passiert die Magie #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

Und nun haben wir Ihre Datei, die im Content-Block gerendert wird:

```html
{# app/views/home.twig #}
{# Dies teilt Twig mit, dass diese Datei "innerhalb" der layout.twig-Datei ist #}
{% extends 'layout.twig' %}

{# Dies ist der Inhalt, der im Layout innerhalb des Content-Blocks gerendert wird #}
{% block content %}
	<h1>Startseite</h1>
	<p>Willkommen in meiner App!</p>
{% endblock %}
```

Wenn Sie dies dann in Ihrer Funktion oder Ihrem Controller rendern möchten, würden Sie so etwas tun:

```php
// einfache Route
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Startseite'
	]);
});

// oder wenn Sie einen Controller verwenden
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Startseite'
		]);
	}
}
```

Weitere Informationen zur vollen Nutzung von Twig finden Sie in der [Twig-Dokumentation](https://twig.symfony.com/doc/3.x/)!

## Debugging

Twig enthält eine [Debug-Erweiterung](https://twig.symfony.com/doc/3.x/functions/dump.html), die eine `dump()`-Funktion hinzufügt, die Sie in Templates verwenden können. Aktivieren Sie sie nur in der Entwicklung:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // erforderlich für die dump()-Funktion
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Dann in einem Template:

```html
{{ dump(user) }}
```

Sie können Twig auch mit [Tracy](/awesome-plugins/tracy) für das PHP-Level-Debugging kombinieren. Für Template-Level-Metriken (Renderzeit, Speicher, welche Templates/Blöcke ausgeführt wurden), verwenden Sie das optionale **Twig-Panel** in [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): übergeben Sie ein `Twig\Profiler\Profile` als `twig_profile` an `TracyExtensionLoader`. Die optionale `TwigTracyExtension` stellt `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` in Templates zur Verfügung, wenn Tracy aktiviert ist.

## Sicherheitshinweis

Twig maskiert die Ausgabe standardmäßig automatisch, was vor XSS-Angriffen schützt. Verwenden Sie bevorzugt `{{ variable }}` für Text. Verwenden Sie den Filter `|raw` nur, wenn Sie dem HTML-Inhalt bewusst vertrauen (z. B. bereinigtes Markdown, das Sie bereits serverseitig verarbeitet haben).