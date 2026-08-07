# Twig

[Twig](https://twig.symfony.com/) est un moteur de templates flexible, rapide et sécurisé pour PHP. C'est le langage de templating utilisé par Symfony et de nombreux autres projets, ce qui signifie que les outils de codage IA et la plupart des développeurs PHP connaissent déjà bien sa syntaxe. Twig compile les templates en PHP optimisé, échappe automatiquement la sortie par défaut (excellent pour la protection XSS) et est facile à étendre avec des filtres, fonctions et extensions.

## Installation

Installez avec composer.

```bash
composer require twig/twig
```

## Configuration de base

Il existe quelques options de configuration de base pour commencer. Vous pouvez en savoir plus à leur sujet dans la [Documentation Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Où Twig stocke ses templates compilés
		'cache' => __DIR__ . '/../cache/twig',
		// Recompiler les templates lorsque la source change (pratique en développement)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Enregistrer Twig en tant que classe de vue

Si vous préférez réutiliser un seul environnement Twig (recommandé pour la production), enregistrez-le et pointez `render` dessus :

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

## Exemple de mise en page simple

Voici un exemple simple d'un fichier de mise en page. C'est le fichier qui sera utilisé pour envelopper toutes vos autres vues.

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
				{# vos éléments de navigation ici #}
			</nav>
		</header>
		<div id="content">
			{# C'est la magie ici #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

Et maintenant nous avons votre fichier qui va être rendu dans ce bloc de contenu :

```html
{# app/views/home.twig #}
{# Ceci indique à Twig que ce fichier est "à l'intérieur" du fichier layout.twig #}
{% extends 'layout.twig' %}

{# C'est le contenu qui sera rendu à l'intérieur de la mise en page dans le bloc content #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Ensuite, lorsque vous allez rendre ceci dans votre fonction ou contrôleur, vous feriez quelque chose comme ceci :

```php
// route simple
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// ou si vous utilisez un contrôleur
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

Consultez la [Documentation Twig](https://twig.symfony.com/doc/3.x/) pour plus d'informations sur comment utiliser Twig à son plein potentiel !

## Débogage

Twig est livré avec une [Extension de débogage](https://twig.symfony.com/doc/3.x/functions/dump.html) qui ajoute une fonction `dump()` que vous pouvez utiliser dans les templates. Activez-la uniquement en développement :

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // requis pour la fonction dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Puis dans un template :

```html
{{ dump(user) }}
```

Vous pouvez également associer Twig avec [Tracy](/awesome-plugins/tracy) pour le débogage au niveau PHP. Pour les métriques au niveau des templates (temps de rendu, mémoire, quels templates/blocs se sont exécutés), utilisez le **panneau Twig** optionnel dans [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) : passez un `Twig\Profiler\Profile` comme `twig_profile` à `TracyExtensionLoader`. L'extension optionnelle `TwigTracyExtension` expose `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` dans les templates lorsque Tracy est activé.

## Note de sécurité

Twig échappe automatiquement la sortie par défaut, ce qui aide à protéger contre les attaques XSS. Préférez `{{ variable }}` pour le texte. Utilisez uniquement le filtre `|raw` lorsque vous faites intentionnellement confiance au contenu HTML (par exemple, du markdown nettoyé que vous avez déjà traité côté serveur).