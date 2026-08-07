# Extensions du panneau Tracy Flight

Il s'agit d'un ensemble d'extensions pour enrichir le travail avec Flight.

- **Flight** - Analyser toutes les variables Flight.
- **Database** - Analyser toutes les requêtes exécutées sur la page (si vous initialisez correctement la connexion à la base de données)
- **Request** - Analyser toutes les variables `$_SERVER` et examiner toutes les charges utiles globales (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Analyser toutes les variables `$_SESSION` si les sessions sont actives.
- **Twig** *(optionnel)* - Analyser le temps de rendu des templates Twig, la mémoire et quels templates/blocs/macros ont été exécutés (nécessite `twig/twig` et une configuration `twig_profile`)

Ceci est particulièrement utile avec le [squelette officiel](https://github.com/flightphp/skeleton), qui utilise Twig par défaut : la même disposition [Outils IA](/learn/ai) suit également apparaît clairement sur la barre Tracy.

C'est le panneau

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

Et chaque panneau affiche des informations très utiles sur votre application !

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Cliquez [ici](https://github.com/flightphp/tracy-extensions) pour voir le code.

## Installation

Exécutez `composer require flightphp/tracy-extensions --dev` et vous êtes prêt !

Twig n'est **pas** une dépendance obligatoire du package. Installez `twig/twig` uniquement si vous voulez le panneau Twig (le squelette le fait déjà pour les vues).

## Configuration

Il y a très peu de configuration nécessaire pour commencer. Vous devrez initialiser le débogueur Tracy avant d'utiliser ceci [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide) :

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// code de bootstrap
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Vous pouvez avoir besoin de spécifier votre environnement avec Debugger::enable(Debugger::DEVELOPMENT)

// si vous utilisez des connexions à la base de données dans votre application, il y a un 
// wrapper PDO requis à utiliser UNIQUEMENT EN DÉVELOPPEMENT (pas en production s'il vous plaît !)
// Il a les mêmes paramètres qu'une connexion PDO régulière
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// ou si vous attachez ceci au framework Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// maintenant chaque fois que vous effectuez une requête, le temps, la requête et les paramètres seront capturés

// Cela connecte les points
if(Debugger::$showBar === true) {
	// Cela doit être false ou Tracy ne peut pas réellement rendre :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// plus de code

Flight::start();
```

## Configuration supplémentaire

### Données de session

Si vous avez un gestionnaire de session personnalisé (comme ghostff/session), vous pouvez passer n'importe quel tableau de données de session à Tracy et il le sortira automatiquement pour vous. Vous le passez avec la clé `session_data` dans le deuxième paramètre du constructeur `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// ou utiliser flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Cela doit être false ou Tracy ne peut pas réellement rendre :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// routes et autres choses...

Flight::start();
```

### Panneau Twig (optionnel)

Si votre application utilise [Twig](/awesome-plugins/twig) (y compris le squelette officiel), vous pouvez afficher les métriques des templates sur la barre Tracy. Créez un `Profile` Twig, attachez `ProfilerExtension` à votre environnement, puis passez ce profil au loader sous la clé **`twig_profile`**. Attachez le profilage uniquement en développement.

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

// Optionnel : exposer les helpers de dump Tracy dans les templates
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

// Mapper Flight::render() à Twig (exemple)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Ce que le panneau affiche**

- Temps de rendu Twig total et mémoire
- Nombre d'appels de templates / blocs / macros
- Chaque template qui a été rendu, avec son propre temps et mémoire

L'onglet Twig est **caché** quand aucun template n'a été rendu pour la requête, ou quand vous omettez `twig_profile` (ou n'avez pas Twig installé)—les autres panneaux Flight continuent de fonctionner.

Dans un `services.php` de style squelette, construisez le même `$profile` / `ProfilerExtension` quand le debug est activé, passez `twig_profile` dans `TracyExtensionLoader`, et continuez à utiliser votre environnement Twig partagé pour `$app->render()`.

### Latte

_PHP 8.1+ est requis pour cette section._

Si vous avez Latte installé dans votre projet, Tracy a une intégration native avec Latte pour analyser vos templates. Vous enregistrez simplement l'extension avec votre instance Latte (c'est le pont Tracy de Latte lui-même, pas le panneau Twig ci-dessus).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// autres configurations...

	// ajouter l'extension uniquement si la barre de debug Tracy est activée
	if(Debugger::$showBar === true) {
		// c'est ici que vous ajoutez le panneau Latte à Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Voir aussi

- [Tracy](/awesome-plugins/tracy) - Configuration de base de Tracy pour Flight
- [Twig](/awesome-plugins/twig) - Templating utilisé par le squelette et le panneau Twig
- [Templates](/learn/templates) - Comment Flight mappe `render` à Twig/Latte
- [Installation](/install) - Le squelette inclut tracy-extensions en dev