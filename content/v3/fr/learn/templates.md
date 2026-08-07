# Vues HTML et Templates

## Aperçu

Flight fournit par défaut des fonctionnalités de templating HTML de base. Le templating est un moyen très efficace de découpler votre logique applicative de votre couche de présentation. Un moteur dédié (Twig, Latte, etc.) offre également aux outils de codage IA une syntaxe familière et contrainte, réduisant ainsi le risque qu'ils n'injectent de la logique métier dans votre HTML.

## Compréhension

Lorsque vous créez une application, vous aurez probablement du HTML à renvoyer à l'utilisateur final. PHP est en soi un langage de templating, mais il est _très_ facile d'intégrer de la logique métier comme des appels de base de données, des appels API, etc., dans votre fichier HTML, ce qui rend les tests et le découplage très difficiles. En poussant les données dans un template et en laissant le template se générer lui-même, il devient beaucoup plus facile de découpler et de tester unitairement votre code. Vous nous remercierez si vous utilisez des templates !

## Utilisation de base

Flight vous permet de remplacer le moteur de vue par défaut simplement en mappant `render` (ou en enregistrant une classe de vue). Faites défiler pour Twig, Latte, Smarty, Blade et plus encore.

> **Défaut du squelette :** Le [flightphp/skeleton](https://github.com/flightphp/skeleton) officiel utilise **Twig uniquement** dans `app/views/` (`*.twig`). Les contrôleurs appellent `$this->app->render('welcome', $data)` (l'extension est facultative). C'est un choix d'application pour les nouveaux projets, et non une exigence du cœur de Flight. Latte et les autres moteurs restent entièrement pris en charge.

### Twig

<span class="badge bg-info">défaut du squelette</span>

[Twig](https://twig.symfony.com/) est un moteur de templates flexible, rapide et sécurisé utilisé par Symfony et de nombreux autres projets PHP. Les outils de codage IA connaissent particulièrement bien Twig, et il échappe automatiquement les sorties par défaut, ce qui aide à se protéger contre les XSS.

#### Installation

```bash
composer require twig/twig
```

(Déjà inclus lorsque vous exécutez `composer create-project flightphp/skeleton`.)

#### Configuration de base

Écrasez la méthode `render` pour utiliser Twig à la place du rendu PHP par défaut :

```php
// écrasez la méthode render pour utiliser Twig à la place du rendu PHP par défaut
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Emplacement où Twig stocke ses templates compilés
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Autorise "welcome" ou "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

Dans le squelette, ce câblage se trouve dans `app/config/services.php` (environnement Twig partagé, chemin de cache, globales comme `base_url` / nonce CSP). Privilégiez l'injection de `Engine` et l'appel à `$app->render()` depuis les contrôleurs pour que le code reste [compatible IA et tests](/learn/ai).

#### Utiliser Twig dans Flight

Maintenant que vous pouvez effectuer le rendu avec Twig, vous pouvez faire quelque chose comme ceci :

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Lorsque vous visitez `/Bob` dans votre navigateur, le résultat serait :

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Pour aller plus loin

Un exemple plus complet d'utilisation de Twig avec des mises en page (layouts) est présenté dans la section [awesome plugins](/awesome-plugins/twig) de cette documentation. Pour les métriques de temps de rendu dans la barre Tracy, consultez le [panneau Twig dans les extensions Tracy](/awesome-plugins/tracy-extensions#twig-panel-optional).

Vous pouvez en apprendre davantage sur toutes les capacités de Twig en lisant la [documentation officielle](https://twig.symfony.com/doc/3.x/).

### Latte

<span class="badge bg-secondary">excellente alternative</span>

[Latte](https://latte.nette.org/) est un moteur complet avec une syntaxe proche de PHP. C'est toujours un excellent choix pour les applications Flight ; le squelette standardise simplement sur Twig pour un défaut partagé (particulièrement utile lorsque les outils d'IA génèrent des templates).

#### Installation

```bash
composer require latte/latte
```

#### Configuration de base

L'idée principale est d'écraser la méthode `render` pour utiliser Latte à la place du rendu PHP par défaut.

```php
// écrasez la méthode render pour utiliser Latte à la place du rendu PHP par défaut
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Emplacement où Latte stocke spécifiquement son cache
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Utiliser Latte dans Flight

Maintenant que vous pouvez effectuer le rendu avec Latte, vous pouvez faire quelque chose comme ceci :

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Lorsque vous visitez `/Bob` dans votre navigateur, le résultat serait :

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Pour aller plus loin

Un exemple plus complexe d'utilisation de Latte avec des mises en page est présenté dans la section [awesome plugins](/awesome-plugins/latte) de cette documentation.

Vous pouvez en apprendre davantage sur toutes les capacités de Latte, y compris la traduction et les fonctionnalités linguistiques, en lisant la [documentation officielle](https://latte.nette.org/en/).

### Moteur de vue intégré

<span class="badge bg-warning">obsolète</span>

> **Remarque :** Bien qu'il s'agisse toujours de la fonctionnalité par défaut, elle fonctionne encore techniquement.

Pour afficher un template de vue, appelez la méthode `render` avec le nom du fichier template et éventuellement des données de template :

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Les données de template que vous transmettez sont automatiquement injectées dans le template et peuvent être référencées comme une variable locale. Les fichiers de template sont simplement des fichiers PHP. Si le contenu du fichier template `hello.php` est :

```php
Hello, <?= $name ?>!
```

Le résultat serait :

```text
Hello, Bob!
```

Vous pouvez également définir manuellement des variables de vue à l'aide de la méthode set :

```php
Flight::view()->set('name', 'Bob');
```

La variable `name` est désormais disponible dans toutes vos vues. Vous pouvez donc simplement faire :

```php
Flight::render('hello');
```

Notez que lorsque vous spécifiez le nom du template dans la méthode render, vous pouvez omettre l'extension `.php`.

Par défaut, Flight recherchera un répertoire `views` pour les fichiers de template. Vous pouvez définir un chemin alternatif pour vos templates en configurant ce qui suit :

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### Mises en page

Il est courant que les sites Web aient un seul fichier template de mise en page (layout) avec un contenu interchangeable. Pour rendre le contenu destiné à être utilisé dans une mise en page, vous pouvez passer un paramètre facultatif à la méthode `render`.

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

Votre vue aura alors des variables enregistrées appelées `headerContent` et `bodyContent`. Vous pouvez ensuite rendre votre mise en page en faisant :

```php
Flight::render('layout', ['title' => 'Home Page']);
```

Si les fichiers template ressemblent à ceci :

`header.php` :

```php
<h1><?= $heading ?></h1>
```

`body.php` :

```php
<div><?= $body ?></div>
```

`layout.php` :

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

Le résultat serait :
```html
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <div>World</div>
  </body>
</html>
```

### Smarty

Voici comment utiliser le moteur de templates [Smarty](http://www.smarty.net/) pour vos vues :

```php
// Charge la bibliothèque Smarty
require './Smarty/libs/Smarty.class.php';

// Enregistre Smarty comme classe de vue
// Passe également une fonction de rappel pour configurer Smarty au chargement
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Affecte les données de template
Flight::view()->assign('name', 'Bob');

// Affiche le template
Flight::view()->display('hello.tpl');
```

Pour être complet, vous devriez également écraser la méthode de rendu par défaut de Flight :

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

Voici comment utiliser le moteur de templates [Blade](https://laravel.com/docs/8.x/blade) pour vos vues :

Tout d'abord, vous devez installer la bibliothèque BladeOne via Composer :

```bash
composer require eftec/bladeone
```

Ensuite, vous pouvez configurer BladeOne comme classe de vue dans Flight :

```php
<?php
// Charge la bibliothèque BladeOne
use eftec\bladeone\BladeOne;

// Enregistre BladeOne comme classe de vue
// Passe également une fonction de rappel pour configurer BladeOne au chargement
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Affecte les données de template
Flight::view()->share('name', 'Bob');

// Affiche le template
echo Flight::view()->run('hello', []);
```

Pour être complet, vous devriez également écraser la méthode de rendu par défaut de Flight :

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

Dans cet exemple, le fichier template `hello.blade.php` pourrait ressembler à ceci :

```php
<?php
Hello, {{ $name }}!
```

Le résultat serait :

```
Hello, Bob!
```

## Voir aussi
- [Installation](/install) - Disposition du squelette (`app/views/*.twig`) pour les nouveaux projets.
- [Extending](/learn/extending) - Comment écraser la méthode `render` pour utiliser un autre moteur de templates.
- [Routing](/learn/routing) - Comment mapper des routes vers des contrôleurs et rendre des vues.
- [Responses](/learn/responses) - Comment personnaliser les réponses HTTP.
- [Security](/learn/security) - Échappement automatique et XSS.
- [AI & Developer Experience](/learn/ai) - Pourquoi un moteur de vue par défaut aide les agents de codage.
- [Why a Framework?](/learn/why-frameworks) - Comment les templates s'intègrent dans la vue d'ensemble.

## Dépannage
- Si vous avez une redirection dans votre middleware, mais que votre application ne semble pas rediriger, assurez-vous d'ajouter une instruction `exit;` dans votre middleware.
- Si Twig ne trouve pas un template, vérifiez `flight.views.path` et que le fichier existe sous ce chemin avec l'extension attendue (squelette : `app/views/`).

## Journal des modifications
- Docs – Twig documenté comme défaut officiel du squelette ; Latte reste une alternative de premier choix.
- v2.0 - Version initiale.