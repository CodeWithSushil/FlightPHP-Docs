# Configuration

## Aperçu

Flight propose un moyen simple de configurer différents aspects du framework pour répondre aux besoins de votre application. Certains sont définis par défaut, mais vous pouvez les remplacer selon vos besoins. Vous pouvez également définir vos propres variables à utiliser dans toute votre application.

Une configuration claire et en couches (valeurs par défaut des fichiers + secrets d'environnement) aide également les [outils de codage IA](/learn/ai) : les agents apprennent un seul endroit pour les littéraux et un seul endroit pour les secrets, plutôt que d'inventer des lectures `$_ENV` dans les contrôleurs.

## Compréhension

Vous pouvez personnaliser certains comportements de Flight en définissant des valeurs de configuration
via la méthode `set`.

```php
Flight::set('flight.log_errors', true);
```

Dans une application structurée (y compris le [skeleton](https://github.com/flightphp/skeleton)), vous chargez généralement les paramètres du projet depuis `app/config/config.php`, puis vous appliquez les clés pertinentes sur le moteur (par exemple `flight.base_url`, `flight.views.path`). Vous pouvez également injecter un petit objet de configuration dans les contrôleurs au lieu de lire les globales partout — plus convivial pour les tests et pour les agents qui suivent `AGENTS.md`.

## Utilisation de base

### Options de configuration de Flight

Voici une liste de tous les paramètres de configuration disponibles :

- **flight.base_url** `?string` - Remplace l'URL de base de la requête si Flight s'exécute dans un sous-répertoire. (défaut : null)
- **flight.case_sensitive** `bool` - Correspondance sensible à la casse pour les URL. (défaut : false)
- **flight.handle_errors** `bool` - Permet à Flight de gérer toutes les erreurs en interne. (défaut : true)
  - Si vous souhaitez que Flight gère les erreurs plutôt que le comportement PHP par défaut, ce paramètre doit être à true.
  - Si vous avez [Tracy](/awesome-plugins/tracy) installé, vous devez le définir sur false afin que Tracy puisse gérer les erreurs.
  - Si vous avez le plugin [APM](/awesome-plugins/apm) installé, vous devez le définir sur true afin que l'APM puisse journaliser les erreurs.
- **flight.log_errors** `bool` - Journalise les erreurs dans le fichier journal d'erreurs du serveur web. (défaut : false)
  - Si vous avez [Tracy](/awesome-plugins/tracy) installé, Tracy journalisera les erreurs selon ses propres configurations, pas selon celle-ci.
- **flight.debug** `bool` - Affiche des informations d'erreur détaillées (message d'exception, code et trace de la pile) dans le navigateur lorsqu'une erreur se produit. (défaut : false)
  - **Ne l'activez jamais en production** — cela divulgue des détails internes de l'application. Utilisez-le uniquement pour le développement local ou la préproduction.
  - Lorsque la valeur est `false`, une réponse générique `500 Internal Server Error` est affichée à la place. À combiner avec `flight.log_errors` pour capturer les erreurs côté serveur.
- **flight.allow_method_override** `bool` - Permet de remplacer la méthode HTTP via l'en-tête de requête `X-HTTP-Method-Override` ou un champ `_method` dans le corps POST. (défaut : true)
  - **Il est recommandé de définir ce paramètre sur `false`** pour les applications qui n'ont pas besoin de l'imitation de méthode basée sur des formulaires HTML, car cela empêche les clients de forger des requêtes `DELETE` ou `PUT` via un formulaire POST standard.
  - Voir [Sécurité](/learn/security) pour plus de détails.
- **flight.views.path** `string` - Répertoire contenant les fichiers de modèles de vues. (défaut : ./views)
- **flight.views.extension** `string` - Extension de fichier des modèles de vues. (défaut : `.php` ; le skeleton officiel définit `.twig` lors de l'utilisation de Twig)
- **flight.content_length** `bool` - Définit l'en-tête `Content-Length`. (défaut : true)
  - Si vous utilisez [Tracy](/awesome-plugins/tracy), ce paramètre doit être défini sur false pour que Tracy puisse s'afficher correctement.
- **flight.v2.output_buffering** `bool` - Utilise la mise en mémoire tampon de sortie héritée. Voir [migration vers v3](migrating-to-v3). (défaut : false)

### Configuration du chargeur

Il existe également un autre paramètre de configuration pour le chargeur. Il vous permet
de charger automatiquement les classes avec `_` dans leur nom.

```php
// Active le chargement des classes avec des underscores
// Valeur par défaut : true
Loader::$v2ClassLoading = false;
```

Rappelez-vous que [l'autoloading](/learn/autoloading) dépend aussi de la **casse des dossiers** qui doit correspondre à vos espaces de noms — en particulier avec la structure `App\` + `app/Controller/` du skeleton.

### Configuration du projet et `.env` (modèle skeleton)

Le cœur de Flight n'exige pas de fichiers `.env`. De nombreuses applications utilisent uniquement un tableau de configuration PHP. Le skeleton officiel superpose la configuration afin que les secrets restent hors de Git tout en permettant à Runway de réécrire en toute sécurité la configuration **littérale** :

1. **`.env` / environnement réel** — secrets et remplacements de déploiement (ignorés par Git).
2. **`app/config/config.php`** — valeurs par défaut littérales d'un tableau PHP (copiées depuis `config_sample.php`). Préférez **ne pas** mettre d'expressions `$_ENV[...]` dans ce fichier : des outils comme `runway config:set` peuvent le réécrire avec des valeurs statiques et risquent d'y intégrer des secrets.
3. **Fusion au démarrage** — l'environnement gagne pour les clés mappées ; le code applicatif lit un objet de configuration ou `$app->get()`, et non `$_ENV` dans les contrôleurs.

Exemple de forme de `config_sample.php` / `config.php` (simplifié) :

```php
<?php
// Uniquement des littéraux — les secrets appartiennent à .env pour le workflow skeleton
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // ou mysql, ou '' pour désactiver
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env (skeleton)
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

Cette séparation est délibérée pour les [projets adaptés à l'IA](/learn/ai) : les instructions peuvent indiquer « valeurs par défaut dans `config.php`, secrets dans `.env`, injectez Config / Engine — n'inventez jamais un accès à l'environnement dans un contrôleur ». Les applications existantes peuvent ignorer `.env` entièrement et conserver un seul fichier de configuration.

### Variables

Flight vous permet d'enregistrer des variables afin de pouvoir les utiliser n'importe où dans votre application.

```php
// Enregistrez votre variable
Flight::set('id', 123);

// Ailleurs dans votre application
$id = Flight::get('id');
```

Pour vérifier si une variable a été définie, vous pouvez faire :

```php
if (Flight::has('id')) {
  // Faire quelque chose
}
```

Vous pouvez effacer une variable en faisant :

```php
// Efface la variable id
Flight::clear('id');

// Efface toutes les variables
Flight::clear();
```

> **Remarque :** Ce n'est pas parce que vous pouvez définir une variable que vous devez le faire. Utilisez cette fonctionnalité avec parcimonie. La raison est que tout ce qui est stocké ici devient une variable globale. Les variables globales sont mauvaises car elles peuvent être modifiées depuis n'importe où dans votre application, ce qui rend la recherche de bogues difficile. De plus, cela peut compliquer des choses comme les [tests unitaires](/guides/unit-testing). Préférez l'injection par constructeur (comme dans le skeleton avec la configuration Dice) pour les services et la configuration dont les contrôleurs ont besoin.

### Erreurs et exceptions

Toutes les erreurs et exceptions sont interceptées par Flight et transmises à la méthode `error`
si `flight.handle_errors` est défini sur true.

Le comportement par défaut consiste à envoyer une réponse générique `HTTP 500 Internal Server Error`
avec quelques informations sur l'erreur.

Vous pouvez [remplacer](/learn/extending) ce comportement selon vos besoins :

```php
Flight::map('error', function (Throwable $error) {
  // Gérer l'erreur
  echo $error->getTraceAsString();
});
```

Par défaut, les erreurs ne sont pas journalisées sur le serveur web. Vous pouvez activer cette fonctionnalité en
modifiant la configuration :

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

Lorsqu'une URL ne peut pas être trouvée, Flight appelle la méthode `notFound`. Le comportement
par défaut consiste à envoyer une réponse `HTTP 404 Not Found` avec un message simple.

Vous pouvez [remplacer](/learn/extending) ce comportement selon vos besoins :

```php
Flight::map('notFound', function () {
  // Gérer l'introuvable
});
```

## Voir aussi
- [Installation](/install) - Configuration du skeleton, `.env` et structure de démarrage.
- [Autoloading](/learn/autoloading) - Espaces de noms et casse des dossiers.
- [Étendre Flight](/learn/extending) - Comment étendre et personnaliser les fonctionnalités principales de Flight.
- [Tests unitaires](/guides/unit-testing) - Comment écrire des tests unitaires pour votre application Flight.
- [IA et expérience développeur](/learn/ai) - `AGENTS.md` et instructions de projet cohérentes.
- [Tracy](/awesome-plugins/tracy) - Un plugin pour la gestion avancée des erreurs et le débogage.
- [Extensions Tracy](/awesome-plugins/tracy_extensions) - Extensions pour intégrer Tracy avec Flight.
- [APM](/awesome-plugins/apm) - Un plugin pour la supervision des performances applicatives et le suivi des erreurs.
- [Sécurité](/learn/security) - Indicateurs de durcissement et gestion des secrets.

## Dépannage
- Si vous avez du mal à trouver toutes les valeurs de votre configuration, vous pouvez faire `var_dump(Flight::get());`
- Si Runway ou un outil de déploiement a réécrit `config.php`, vérifiez que les secrets n'ont pas été commités — conservez-les dans `.env` ou dans l'environnement réel lors de l'utilisation du modèle skeleton.

## Journal des modifications
- Documentation – Documente la configuration de type skeleton / `.env` en couches et l'extension de vue Twig par défaut pour les nouveaux projets.
- v3.18.1 - Ajout des options de configuration `flight.debug` et `flight.allow_method_override`.
- v3.5.0 - Ajout de la configuration `flight.v2.output_buffering` pour prendre en charge le comportement de mise en mémoire tampon de sortie hérité.
- v2.0 - Ajout des configurations principales.