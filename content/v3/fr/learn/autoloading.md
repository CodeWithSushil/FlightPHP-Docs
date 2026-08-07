# Autoloading

## Vue d'ensemble

L'autoloading est un concept en PHP qui consiste à spécifier un répertoire ou des répertoires à partir desquels charger les classes. C'est beaucoup plus avantageux que d'utiliser `require` ou `include` pour charger les classes. C'est également une exigence pour utiliser les paquets Composer.

Bien configurer l'autoloading est également important pour le [développement assisté par IA](/learn/ai) : les agents placent les fichiers là où le namespace pointe. Si la **casse** des dossiers et celle des namespaces ne correspondent pas, des erreurs de classe introuvable apparaissent sur Linux, même si tout « fonctionnait » sur un disque Mac insensible à la casse.

## Comprendre

Par défaut, toutes les classes `Flight` sont chargées automatiquement pour vous grâce à Composer. Pour les classes de **votre** application, vous avez deux approches courantes :

1. **PSR-4 de Composer** (ce que le [squelette officiel](https://github.com/flightphp/skeleton) utilise) : mappez un préfixe de namespace vers un répertoire dans `composer.json`, puis exécutez `composer dump-autoload`.
2. **`Flight::path()`** : indiquez des répertoires au chargeur de Flight (pratique pour les applications simples ou lorsque vous n'utilisez pas Composer pour le code applicatif).

Utiliser un autoloader simplifie énormément votre code. Au lieu d'un mur de `include` / `require` en haut de chaque fichier, les classes se chargent à leur première utilisation.

### Sensibilité à la casse (à lire deux fois)

**Les namespaces doivent correspondre à la structure des répertoires *et* à la casse de ces répertoires.**

| Fonctionne | Casse sur Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` avec le dossier `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Mélanger `App\` avec `controllers` en minuscules |

Les namespaces PHP sont insensibles à la casse dans certains contextes, mais **Composer et le système de fichiers ne le sont pas**. Le squelette officiel standardise sur :

- Composer : `"App\\": "app/"`
- Dossiers : **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), pas `controllers` / `middlewares`

Les anciennes documentations et exemples de la communauté utilisaient parfois `app\controllers` en minuscules. Cela fonctionne toujours si vos dossiers sont en minuscules – mais **les nouveaux projets de squelette utilisent `App\` + dossiers en PascalCase**. Choisissez une convention par projet et tenez-vous-y pour que les humains et les outils d'IA n'inventent pas une deuxième structure.

## Squelette (recommandé pour les nouveaux projets)

Après `composer create-project flightphp/skeleton`, le code applicatif est chargé automatiquement via Composer – aucun `Flight::path()` n'est requis pour les classes `App\` :

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
// app/config/routes.php — Découvre App\Controller\… via le conteneur
$router->get('/', [HomeController::class, 'index']);
```

Voir [Installation](/install) pour l'arborescence complète et [IA & expérience développeur](/learn/ai) pour comprendre comment `AGENTS.md` documente cette structure pour les assistants de codage.

## Utilisation de base (`Flight::path()`)

Supposons que nous ayons une arborescence de répertoires comme celle-ci :

```text
# Exemple de chemin
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - contient les contrôleurs de ce projet
│   ├── translations
│   ├── UTILS - contient des classes pour cette application uniquement (c'est en majuscules volontairement pour un exemple plus loin)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Vous avez peut-être remarqué que cela ressemble à une arborescence d'application typique (le site de documentation lui-même utilise une structure organisée). Le dossier `controllers` en minuscules est un *choix* valide – ce n'est simplement pas le défaut actuel du squelette.

Vous pouvez spécifier chaque répertoire à charger comme ceci :

```php

/**
 * public/index.php
 */

// Ajoute un chemin à l'autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// pas de namespace requis

// Toutes les classes autochargées devraient être en PascalCase (chaque mot commençant par une majuscule, sans espaces)
class MyController {

	public function index() {
		// faire quelque chose
	}
}
```

## Namespaces avec `Flight::path()`

Si vous utilisez des namespaces, cela devient en fait très simple à implémenter. Vous devez utiliser la méthode `Flight::path()` pour spécifier le répertoire racine (pas la racine du document ni le dossier `public/`) de votre application.

```php

/**
 * public/index.php
 */

// Ajoute un chemin à l'autoloader
Flight::path(__DIR__.'/../');
```

Voici maintenant à quoi pourrait ressembler votre contrôleur. Regardez l'exemple ci-dessous, mais faites attention aux commentaires pour des informations importantes.

```php
/**
 * app/controllers/MyController.php
 */

// les namespaces sont requis
// les namespaces correspondent à la structure des répertoires
// les namespaces doivent respecter la même casse que la structure des répertoires
// les namespaces et les répertoires ne peuvent pas contenir de underscores (sauf si Loader::setV2ClassLoading(false) est défini)
namespace app\controllers;

// Toutes les classes autochargées devraient être en PascalCase (chaque mot commençant par une majuscule, sans espaces)
// Depuis la version 3.7.2, vous pouvez utiliser Pascal_Snake_Case pour vos noms de classes en exécutant Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// faire quelque chose
	}
}
```

Et si vous vouliez charger automatiquement une classe dans votre répertoire utils, vous feriez essentiellement la même chose :

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// le namespace doit correspondre à la structure et à la casse du répertoire (notez que le répertoire UTILS est en majuscules
//     comme dans l'arborescence ci-dessus)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// faire quelque chose
	}
}
```

### Namespace style squelette (mêmes règles, casse différente)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

La règle n'a pas changé – seule la casse du dossier/namespace choisie par le squelette a changé. **Quelle que soit la casse de vos dossiers, votre ligne `namespace` doit correspondre.**

## Underscores dans les noms de classes

Depuis la version 3.7.2, vous pouvez utiliser Pascal_Snake_Case pour vos noms de classes en exécutant `Loader::setV2ClassLoading(false);`. 
Cela vous permet d'utiliser des underscores dans vos noms de classes. 
Ce n'est pas recommandé, mais c'est disponible pour ceux qui en ont besoin.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Ajoute un chemin à l'autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// pas de namespace requis

class My_Controller {

	public function index() {
		// faire quelque chose
	}
}
```

## Voir aussi
- [Installation](/install) - Arborescence du squelette et valeurs par défaut `App\` pour les nouveaux projets.
- [Routage](/learn/routing) - Comment mapper des routes vers des contrôleurs et rendre des vues.
- [Injection de dépendances](/learn/dependency-injection-container) - Comment les contrôleurs obtiennent `Engine` et les services.
- [IA & expérience développeur](/learn/ai) - Gardez les agents alignés avec votre structure via `AGENTS.md`.
- [Pourquoi un framework ?](/learn/why-frameworks) - Comprendre les avantages d'utiliser un framework comme Flight.

## Dépannage
- Si vous n'arrivez pas à comprendre pourquoi vos classes avec namespace ne sont pas trouvées, rappelez-vous : avec `Flight::path()`, pointez vers la **racine du projet** (ou la base correcte pour votre namespace), pas seulement un dossier imbriqué que vous avez oublié de refléter dans le namespace.
- Avec le PSR-4 de Composer, exécutez `composer dump-autoload` après avoir modifié les correspondances dans `composer.json`.
- Sur un CI Linux ou en production, une mauvaise casse de dossier est une cause très fréquente d'erreurs « ça marche sur ma machine ».

### Classe introuvable (autoloading ne fonctionne pas)

Il peut y avoir plusieurs raisons à cela. Voici quelques exemples.

#### Nom de fichier incorrect
Le plus courant est que le nom de la classe ne correspond pas au nom du fichier.

Si vous avez une classe nommée `MyClass`, le fichier doit être nommé `MyClass.php`. Si vous avez une classe nommée `MyClass` et que le fichier est nommé `myclass.php`, 
l'autoloader ne pourra pas la trouver.

#### Namespace ou casse de dossier incorrect
Si vous utilisez des namespaces, le namespace doit correspondre à la structure des répertoires **y compris la casse**.

```php
// ...code...

// si votre MyController est dans app/Controller (squelette) et a le namespace App\Controller
// cela ne fonctionnera pas :
Flight::route('/hello', 'MyController->hello');

// Style squelette :
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Ancienne structure en minuscules (seulement si vos dossiers sont réellement app/controllers) :
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// ou avec le nom complet :
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` non défini (code applicatif sans Composer)

Si vous comptez sur `Flight::path()` plutôt que sur Composer pour les classes de votre application, définissez le chemin avant les routes qui référencent ces classes (souvent tôt dans le bootstrap / `public/index.php`) :

```php
// Ajoute un chemin à l'autoloader (racine du projet pour les applications avec namespace)
Flight::path(__DIR__.'/../');
```

Le squelette officiel utilise principalement **le PSR-4 de Composer** pour `App\`, donc vous n'aurez généralement pas besoin de `Flight::path()` pour les contrôleurs et les modèles.

## Journal des modifications
- Docs – Documenter les dossiers `App\` + PascalCase du squelette et les pièges de sensibilité à la casse pour les humains et les outils d'IA.
- v3.7.2 - Vous pouvez utiliser Pascal_Snake_Case pour vos noms de classes en exécutant `Loader::setV2ClassLoading(false);`
- v2.0 - Fonctionnalité d'autoload ajoutée.