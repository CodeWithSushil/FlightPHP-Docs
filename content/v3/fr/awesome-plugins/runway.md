# Runway

Runway est une application CLI qui vous aide à gérer vos applications Flight. Elle peut générer des contrôleurs, afficher toutes les routes, exécuter des assistants de configuration IA, des migrations (dans le squelette), et plus encore. Elle est basée sur l'excellente bibliothèque [adhocore/php-cli](https://github.com/adhocore/php-cli).

Cliquez [ici](https://github.com/flightphp/runway) pour voir le code.

Les commandes de scaffolding sont intentionnellement alignées avec le [squelette officiel](https://github.com/flightphp/skeleton) afin que les [outils de codage IA](/learn/ai) et les humains obtiennent les mêmes chemins, espaces de noms et style d'injection par constructeur à chaque fois.

## Installation

Installez avec composer.

```bash
composer require flightphp/runway
```

Le squelette dépend déjà de Runway ; utilisez `php runway` depuis la racine du projet.

## Configuration de base

La première fois que vous exécutez Runway, il essaiera de trouver une configuration `runway` dans `app/config/config.php` via la clé `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// optionnel ; le squelette utilise également index_root pour le point d'entrée public
		'index_root' => 'public/index.php',
    ],
];
```

> **NOTE** - À partir de **v1.2.0**, `.runway-config.json` est obsolète au profit de `app/config/config.php`. Migrez avec `php runway config:migrate` lors de la mise à niveau d'anciens projets. Le squelette peut encore écrire un petit `.runway-config.json` lors de create-project pour la compatibilité ; préférez la clé `runway` dans `config.php` pour la suite.

### Détection de la racine du projet

Runway est assez intelligent pour détecter la racine de votre projet, même si vous l'exécutez depuis un sous-répertoire. Il recherche des indicateurs comme `composer.json`, `.git`, ou `app/config/config.php` pour déterminer où se trouve la racine du projet. Cela signifie que vous pouvez exécuter les commandes Runway depuis n'importe où dans votre projet !

## Utilisation

Runway dispose d'un certain nombre de commandes que vous pouvez utiliser pour gérer votre application Flight. Il existe deux façons simples d'utiliser Runway.

1. Si vous utilisez le projet squelette, vous pouvez exécuter `php runway [commande]` depuis la racine de votre projet.
1. Si vous utilisez Runway comme package installé via composer, vous pouvez exécuter `vendor/bin/runway [commande]` depuis la racine de votre projet.

### Liste des commandes

Vous pouvez voir la liste de toutes les commandes disponibles en exécutant la commande `php runway`.

```bash
php runway
```

Ne comptez que sur les commandes qui apparaissent réellement dans cette liste pour votre installation (commandes Runway principales vs commandes spécifiques au projet comme `migrate` du squelette).

### Aide des commandes

Pour toute commande, vous pouvez passer le flag `--help` pour obtenir plus d'informations sur l'utilisation de la commande.

```bash
php runway routes --help
php runway make:controller --help
```

Voici quelques exemples :

### Générer un contrôleur

`make:controller` génère un contrôleur qui correspond à la mise en page du squelette officiel :

| | |
|--|--|
| **Chemin** | `app/Controller/{Nom}.php` |
| **Espace de noms** | `App\Controller` |
| **Style** | Injection par constructeur de `flight\Engine` (pas de `Flight::` dans le corps de la classe) |

```bash
php runway make:controller MonControleur
# → app/Controller/MonControleur.php
#   namespace App\Controller;
```

Exemple de la forme à laquelle vous devez vous attendre (simplifié) :

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MonControleur
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// ex. $this->app->render('…', […]);
	}
}
```

Enregistrez-le avec un callable de classe pour que Dice puisse construire le contrôleur :

```php
// app/config/routes.php
use App\Controller\MonControleur;

$router->get('/mine', [MonControleur::class, 'index']);
```

**Pourquoi cette mise en page ?** La **casse** du dossier doit correspondre à l'espace de noms (`Controller` pas `controllers`) pour Composer PSR-4 sur Linux—voir [Autoloading](/learn/autoloading). Le même chemin est ce que les fichiers `AGENTS.md` racine et scopés indiquent aux outils IA d'utiliser, donc les contrôleurs générés et écrits à la main restent identiques.

> Les anciennes documentations et projets communautaires utilisaient parfois `app/controllers/` et `app\controllers`. Cela reste valide si *votre* arbre utilise encore des dossiers en minuscules. **Les nouveaux projets squelette et la sortie actuelle de `make:controller` utilisent `app/Controller/` + `App\Controller`.**

### Générer un modèle Active Record

Assurez-vous d'abord d'avoir installé le plugin [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record utilisateurs
```

Dans le squelette officiel, les modèles vivent sous **`app/Model/`** avec l'espace de noms **`App\Model`**, et la connexion DB est **[SimplePdo](/learn/simple-pdo)** (injectez-la ou passez-la dans le constructeur ActiveRecord). Les noms/namespaces des fichiers générés suivent les valeurs par défaut actuelles de Runway et votre configuration `runway`—préférez aligner les nouveaux modèles avec `App\Model` afin qu'ils correspondent à [autoloading](/learn/autoloading) et `AGENTS.md`.

Exemple d'un modèle cohérent avec la démo posts du squelette :

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $titre
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

Si un ancien générateur émet encore `app/records` / `app\records`, vous pouvez conserver cette convention dans les applications legacy ou déplacer les fichiers dans `app/Model/` et mettre à jour l'espace de noms pour correspondre à la casse du dossier.

### Migrations (squelette)

Le squelette officiel fournit une commande de projet (découverte depuis `app/commands/`) telle que :

```bash
php runway migrate
```

Les migrations sont des fichiers SQL sous `migrations/` (par exemple `YYYYMMDDHHMMSS_description.sql` pour SQLite et `…_description.mysql.sql` pour MySQL), sélectionnés depuis votre configuration de pilote de base de données / env. Les flags et comportements exacts sont définis par cette commande de projet—exécutez `php runway migrate --help` dans votre application.

### Assistants IA

Runway expose des commandes orientées IA utilisées avec [IA et expérience développeur](/learn/ai) :

```bash
php runway ai:init
php runway ai:generate-instructions
```

Celles-ci stockent les identifiants LLM et génèrent les instructions de projet (principalement **`AGENTS.md`**). Sur le squelette, traitez `AGENTS.md` (et les copies scopées sous `app/`) plus **`SECURITY.md`** comme la source de vérité pour les agents.

### Afficher toutes les routes

Cela affichera toutes les routes actuellement enregistrées avec Flight.

```bash
php runway routes
```

Si vous souhaitez seulement voir des routes spécifiques, vous pouvez passer un flag pour filtrer les routes.

```bash
# Afficher seulement les routes GET
php runway routes --get

# Afficher seulement les routes POST
php runway routes --post

# etc.
```

## Ajouter des commandes personnalisées à Runway

Si vous créez un package pour Flight, ou souhaitez ajouter vos propres commandes personnalisées dans votre projet, vous pouvez le faire en créant un répertoire `src/commands/`, `flight/commands/`, `app/commands/`, ou `commands/` pour votre projet/package. Si vous avez besoin de plus de personnalisation, voir la section ci-dessous sur la Configuration.

Dans le squelette, les commandes de projet vivent dans **`app/commands/`** avec l'espace de noms **`App\Command`**. Runway les découvre par chemin ; gardez ce dossier synchronisé avec le classmap/PSR-4 de Composer comme le fait déjà votre projet.

Pour créer une commande, vous étendez simplement la classe `AbstractBaseCommand`, et implémentez au minimum une méthode `__construct` et une méthode `execute`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class CommandeExemple extends AbstractBaseCommand
{
	/**
     * Constructeur
     *
     * @param array<string,mixed> $config Config depuis app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Créer un exemple pour la documentation', $config);
        $this->argument('<gif-drole>', 'Le nom du gif drôle');
    }

	/**
     * Exécute la fonction
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Création de l\'exemple...');

		// Faites quelque chose ici

		$io->ok('Exemple créé !');
	}
}
```

Consultez la [Documentation adhocore/php-cli](https://github.com/adhocore/php-cli) pour plus d'informations sur la façon de construire vos propres commandes personnalisées dans votre application Flight !

## Gestion de la configuration

Puisque la configuration a été déplacée vers `app/config/config.php` à partir de `v1.2.0`, il existe quelques commandes d'aide pour gérer la configuration.

> **Astuce squelette :** Gardez `config.php` avec des valeurs PHP **littérales**. Les secrets appartiennent à `.env`. Évitez les expressions `$_ENV[...]` dans `config.php`—`config:set` réécrit ce fichier comme données statiques et pourrait intégrer des secrets dans le fichier. Voir [Configuration](/learn/configuration).

### Migrer l'ancienne configuration

Si vous avez un ancien fichier `.runway-config.json`, vous pouvez facilement le migrer vers `app/config/config.php` avec la commande suivante :

```bash
php runway config:migrate
```

### Définir une valeur de configuration

Vous pouvez définir une valeur de configuration en utilisant la commande `config:set`. C'est utile si vous souhaitez mettre à jour une valeur de configuration sans ouvrir le fichier.

```bash
php runway config:set app_root "app/"
```

### Obtenir une valeur de configuration

Vous pouvez obtenir une valeur de configuration en utilisant la commande `config:get`.

```bash
php runway config:get app_root
```

## Toutes les configurations Runway

Si vous avez besoin de personnaliser la configuration pour Runway, vous pouvez définir ces valeurs dans `app/config/config.php`. Voici quelques configurations supplémentaires que vous pouvez définir :

```php
<?php
// app/config/config.php
return [
    // ... autres valeurs de config ...

    'runway' => [
        // C'est là où se trouve le répertoire de votre application
        'app_root' => 'app/',

        // C'est le répertoire où se trouve votre fichier index racine
        'index_root' => 'public/',

        // Ce sont les chemins vers les racines d'autres projets
        'root_paths' => [
            '/home/user/projet-different',
            '/var/www/autre-projet'
        ],

        // Les chemins de base n'ont probablement pas besoin d'être configurés, mais c'est ici si vous en avez besoin
        'base_paths' => [
            '/includes/libs/vendor', // si vous avez un chemin vraiment unique pour votre répertoire vendor ou autre chose
        ],

        // Les chemins finaux sont des emplacements dans un projet pour rechercher les fichiers de commande
        'final_paths' => [
            'src/chemin-diff/commands',
            'app/module/admin/commands',
        ],

        // Si vous voulez simplement ajouter le chemin complet, allez-y (absolu ou relatif à la racine du projet)
        'paths' => [
            '/home/user/projet-different/src/chemin-diff/commands',
            '/var/www/autre-projet/app/module/admin/commands',
            'app/mes-commandes-uniques'
        ]
    ]
];
```

### Accès à la configuration

Si vous avez besoin d'accéder efficacement aux valeurs de configuration, vous pouvez y accéder via la méthode `__construct` ou la méthode `app()`. Il est également important de noter que si vous avez un fichier `app/config/services.php`, ces services seront également disponibles pour votre commande.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Accéder à la configuration
    $app_root = $this->config['runway']['app_root'];
    
    // Accéder aux services comme peut-être une connexion à la base de données
    $database = $this->config['database']
    
    // ...
}
```

## Wrappers d'aide IA

Runway dispose de quelques wrappers d'aide qui facilitent la génération de commandes par l'IA. Vous pouvez utiliser `addOption` et `addArgument` d'une manière similaire à Symfony Console. C'est utile si vous utilisez des outils IA pour générer vos commandes.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Créer un exemple pour la documentation', $config);
    
    // L'argument mode est nullable et par défaut complètement optionnel
    $this->addOption('nom', 'Le nom de l\'exemple', null);
}
```

## Voir aussi

- [Installation](/install) - Arborescence du squelette et valeurs par défaut de create-project
- [Autoloading](/learn/autoloading) - `App\` et casse des dossiers
- [Injection de dépendances](/learn/dependency-injection-container) - Injection Dice + Engine pour les contrôleurs générés
- [IA et expérience développeur](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Modèles utilisés avec `make:record` / squelette `App\Model`
- [SimplePdo](/learn/simple-pdo) - Connexion DB utilisée par les migrations et modèles du squelette