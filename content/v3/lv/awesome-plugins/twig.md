# Twig

[Twig](https://twig.symfony.com/) ir elastīgs, ātrs un drošs PHP veidņu dzinējs. Tā ir veidņu valoda, ko izmanto Symfony un daudzi citi projekti, kas nozīmē, ka mākslīgā intelekta kodēšanas rīki un lielākā daļa PHP izstrādātāju jau labi zina tās sintaksi. Twig kompilē veidnes uz optimizētu PHP, automātiski aizsargā izvadi pēc noklusējuma (lieliski XSS aizsardzībai) un ir viegli paplašināms ar filtriem, funkcijām un paplašinājumiem.

## Instalācija

Instalējiet ar composer.

```bash
composer require twig/twig
```

## Pamata konfigurācija

Ir dažas pamata konfigurācijas opcijas, lai sāktu darbu. Vairāk par tām var lasīt [Twig dokumentācijā](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Kur Twig glabā kompilētās veidnes
		'cache' => __DIR__ . '/../cache/twig',
		// Pārkompilēt veidnes, kad mainās avots (ērti izstrādes laikā)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Twig reģistrēšana kā skata klasi

Ja vēlaties atkārtoti izmantot vienu Twig vidi (ieteicams ražošanai), reģistrējiet to un norādiet `render` uz to:

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

## Vienkāršs izkārtojuma piemērs

Šeit ir vienkāršs izkārtojuma faila piemērs. Šis ir fails, kas tiks izmantots, lai ietītu visas jūsu citas skatnes.

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
				{# jūsu navigācijas elementi šeit #}
			</nav>
		</header>
		<div id="content">
			{# Šeit ir burvība #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

Un tagad mums ir jūsu fails, kas tiks renderēts šajā satura blokā:

```html
{# app/views/home.twig #}
{# Tas saka Twig, ka šis fails ir "iekšpusē" layout.twig failā #}
{% extends 'layout.twig' %}

{# Tas ir saturs, kas tiks renderēts izkārtojumā satura blokā #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Tad, kad renderējat to savā funkcijā vai kontrolierī, jūs darītu kaut ko līdzīgu šim:

```php
// vienkāršs maršruts
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// vai ja izmantojat kontrolieri
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

Skatiet [Twig dokumentāciju](https://twig.symfony.com/doc/3.x/) lai iegūtu vairāk informācijas par to, kā izmantot Twig pilnā potenciālā!

## Atkļūdošana

Twig nāk ar [Atkļūdošanas paplašinājumu](https://twig.symfony.com/doc/3.x/functions/dump.html), kas pievieno `dump()` funkciju, ko varat izmantot veidnēs. Iespējojiet to tikai izstrādes laikā:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // nepieciešams dump() funkcijai
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Tad veidnē:

```html
{{ dump(user) }}
```

Jūs varat arī savienot Twig ar [Tracy](/awesome-plugins/tracy) PHP līmeņa atkļūdošanai. Veidņu līmeņa metrikām (renderēšanas laiks, atmiņa, kuras veidnes/bloki darbināti), izmantojiet izvēles **Twig paneli** [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): nododiet `Twig\Profiler\Profile` kā `twig_profile` uz `TracyExtensionLoader`. Izvēles `TwigTracyExtension` atklāj `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` veidnēs, kad Tracy ir ieslēgts.

## Drošības piezīme

Twig automātiski aizsargā izvadi pēc noklusējuma, kas palīdz aizsargāt pret XSS uzbrukumiem. Dodiet priekšroku `{{ variable }}` tekstam. Izmantojiet tikai `|raw` filtru, kad jūs apzināti uzticaties HTML saturam (piemēram, sanitizēts markdown, ko jūs jau apstrādājāt servera pusē).