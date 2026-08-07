# Documentation APM FlightPHP

Bienvenue sur FlightPHP APM — votre coach personnel de performance d’application ! Ce guide est votre feuille de route pour installer, utiliser et maîtriser la Surveillance des Performances des Applications (APM) avec FlightPHP. Que vous traquiez des requêtes lentes ou que vous souhaitiez simplement vous amuser avec des graphiques de latence, nous avons tout prévu. Rendons votre application plus rapide, vos utilisateurs plus heureux et vos sessions de débogage un jeu d’enfant !

Consultez une [démo](https://flightphp-docs-apm.sky-9.com/apm/dashboard) du tableau de bord pour le site des docs Flight.

![FlightPHP APM](/images/apm.png)

## Pourquoi l’APM est important

Imaginez votre application comme un restaurant très fréquenté. Sans moyen de suivre le temps que prennent les commandes ou là où la cuisine ralentit, vous devinez pourquoi les clients partent mécontents. L’APM est votre sous-chef — il surveille chaque étape, des requêtes entrantes aux requêtes de base de données, et signale tout ce qui vous ralentit. Les pages lentes font perdre des utilisateurs (les études disent que 53 % des visiteurs partent si un site met plus de 3 secondes à charger !), et l’APM vous aide à repérer ces problèmes *avant* qu’ils ne fassent mal. C’est une tranquillité d’esprit proactive — moins de moments « pourquoi est-ce cassé ? », plus de victoires « regardez comme ça tourne bien ! ».

## Installation

Commencez avec Composer :

```bash
composer require flightphp/apm
```

Vous aurez besoin de :
- **PHP 7.4+** : Nous restons compatibles avec les distributions Linux LTS tout en prenant en charge le PHP moderne.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+** : Le framework léger que nous boostons.

## Bases de données prises en charge

FlightPHP APM prend actuellement en charge les bases de données suivantes pour stocker les métriques :

- **SQLite3** : Simple, basée sur des fichiers et idéale pour le développement local ou les petites applications. Option par défaut dans la plupart des configurations.
- **MySQL/MariaDB** : Idéale pour les projets plus importants ou les environnements de production où vous avez besoin d’un stockage robuste et évolutif.

Vous pouvez choisir votre type de base de données lors de l’étape de configuration (voir ci-dessous). Assurez-vous que votre environnement PHP dispose des extensions nécessaires installées (par ex. `pdo_sqlite` ou `pdo_mysql`).

## Démarrage rapide

Voici votre guide étape par étape pour l’excellence APM :

### 1. Enregistrer l’APM

Ajoutez ceci dans votre fichier `index.php` ou `services.php` pour commencer le suivi :

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// Si vous ajoutez une connexion à la base de données
// Préférez SimplePdo (ou PdoQueryCapture des Extensions Tracy en dev).
// Activez le suivi des requêtes APM via le tableau d’options (5e argument).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // obligatoire pour capturer les requêtes pour l’APM
]);
$Apm->addPdoConnection($pdo);
```

**Que se passe-t-il ici ?**
- `LoggerFactory::create()` récupère votre configuration (plus d’informations bientôt) et configure un logger — SQLite par défaut.
- `Apm` est la star — il écoute les événements de Flight (requêtes, routes, erreurs, etc.) et collecte les métriques.
- `bindEventsToFlightInstance($app)` lie le tout à votre application Flight.

**Astuce pro : Échantillonnage**
Si votre application est très sollicitée, journaliser *chaque* requête pourrait surcharger le système. Utilisez un taux d’échantillonnage (0.0 à 1.0) :

```php
$Apm = new Apm($ApmLogger, 0.1); // Enregistre 10 % des requêtes
```

Cela garde les performances fluides tout en vous donnant des données solides.

### 2. Configurer

Exécutez ceci pour générer votre `.runway-config.json` :

```bash
php vendor/bin/runway apm:init
```

**Que fait ceci ?**
- Lance un assistant demandant d’où proviennent les métriques brutes (source) et où les données traitées vont (destination).
- La valeur par défaut est SQLite — par ex. `sqlite:/tmp/apm_metrics.sqlite` pour la source, une autre pour la destination.
- Vous obtiendrez une configuration comme :
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Ce processus demandera également si vous souhaitez exécuter les migrations pour cette configuration. Si vous configurez cela pour la première fois, la réponse est oui.

**Pourquoi deux emplacements ?**
Les métriques brutes s’accumulent rapidement (pensez aux journaux non filtrés). Le worker les traite dans une destination structurée pour le tableau de bord. Tout reste propre !

### 3. Traiter les métriques avec le worker

Le worker transforme les métriques brutes en données prêtes pour le tableau de bord. Exécutez-le une fois :

```bash
php vendor/bin/runway apm:worker
```

**Que fait-il ?**
- Lit depuis votre source (par ex. `apm_metrics.sqlite`).
- Traite jusqu’à 100 métriques (taille de lot par défaut) vers votre destination.
- S’arrête quand c’est fait ou s’il n’y a plus de métriques.

**Le garder en cours d’exécution**
Pour les applications en direct, vous voudrez un traitement continu. Voici vos options :

- **Mode démon** :
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Fonctionne indéfiniment, traitant les métriques au fur et à mesure. Idéal pour le dev ou les petites configurations.

- **Crontab** :
  Ajoutez ceci à votre crontab (`crontab -e`) :
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  S’exécute chaque minute — parfait pour la production.

- **Tmux/Screen** :
  Démarrez une session détachable :
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, puis D pour détacher ; `tmux attach -t apm-worker` pour se reconnecter
  ```
  Continue à fonctionner même si vous vous déconnectez.

- **Personnalisations** :
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50` : Traite 50 métriques à la fois.
  - `--max_messages 1000` : S’arrête après 1000 métriques.
  - `--timeout 300` : Quitte après 5 minutes.

**Pourquoi s’en soucier ?**
Sans le worker, votre tableau de bord est vide. C’est le pont entre les journaux bruts et les informations exploitables.

### 4. Lancer le tableau de bord

Voyez les indicateurs vitaux de votre application :

```bash
php vendor/bin/runway apm:dashboard
```

**Que fait ceci ?**
- Lance un serveur PHP sur `http://localhost:8001/apm/dashboard`.
- Affiche les journaux de requêtes, les routes lentes, les taux d’erreur, et plus encore.

**Personnalisez-le** :
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0` : Accessible depuis n’importe quelle IP (pratique pour la visualisation à distance).
- `--port 8080` : Utilisez un port différent si 8001 est pris.
- `--php-path` : Indiquez le chemin vers PHP s’il n’est pas dans votre PATH.

Saisissez l’URL dans votre navigateur et explorez !

#### Mode production

Pour la production, vous devrez peut-être essayer plusieurs techniques pour faire fonctionner le tableau de bord car il y a probablement des pare-feu et d’autres mesures de sécurité en place. Voici quelques options :

- **Utiliser un proxy inverse** : Configurez Nginx ou Apache pour transférer les requêtes vers le tableau de bord.
- **Tunnel SSH** : Si vous pouvez vous connecter en SSH au serveur, utilisez `ssh -L 8080:localhost:8001
youruser@yourserver` pour tunnelliser le tableau de bord vers votre machine locale.
- **VPN** : Si votre serveur est derrière un VPN, connectez-vous et accédez directement au tableau de bord.
- **Configurer le pare-feu** : Ouvrez le port 8001 pour votre IP ou le réseau du serveur. (ou tout autre port que vous avez défini).
- **Configurer Apache/Nginx** : Si vous avez un serveur web devant votre application, vous pouvez le configurer sur un domaine ou un sous-domaine. Si vous faites cela, vous définirez la racine du document sur `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### Vous voulez un tableau de bord différent ?

Vous pouvez créer votre propre tableau de bord si vous le souhaitez ! Regardez le répertoire vendor/flightphp/apm/src/apm/presenter pour des idées sur la façon de présenter les données pour votre propre tableau de bord !

## Fonctionnalités du tableau de bord

Le tableau de bord est votre QG APM — voici ce que vous verrez :

- **Journal des requêtes** : Chaque requête avec horodatage, URL, code de réponse et temps total. Cliquez sur « Détails » pour voir les middlewares, les requêtes et les erreurs.
- **Requêtes les plus lentes** : Top 5 des requêtes qui prennent le plus de temps (par ex. « /api/heavy » à 2.5s).
- **Routes les plus lentes** : Top 5 des routes par temps moyen — idéal pour repérer des modèles.
- **Taux d’erreur** : Pourcentage de requêtes en échec (par ex. 2.3 % de 500).
- **Percentiles de latence** : Temps de réponse au 95e (p95) et 99e (p99) centile — connaissez vos pires scénarios.
- **Graphique des codes de réponse** : Visualisez les 200, 404, 500 au fil du temps.
- **Requêtes/Middleware longs** : Top 5 des appels de base de données et des couches de middleware les plus lents.
- **Cache Hit/Miss** : À quelle fréquence votre cache sauve la mise.

**Extras** :
- Filtrer par « Dernière heure », « Dernier jour » ou « Dernière semaine ».
- Basculer en mode sombre pour les sessions tardives.

**Exemple** :
Une requête vers `/users` pourrait afficher :
- Temps total : 150 ms
- Middleware : `AuthMiddleware->handle` (50 ms)
- Requête : `SELECT * FROM users` (80 ms)
- Cache : Hit sur `user_list` (5 ms)

## Ajout d’événements personnalisés

Suivez n’importe quoi — comme un appel API ou un processus de paiement :

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**Où cela apparaît-il ?**
Dans les détails de la requête du tableau de bord sous « Événements personnalisés » — extensible avec un formatage JSON élégant.

**Cas d’utilisation** :
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
Maintenant vous verrez si cette API ralentit votre application !

## Surveillance des bases de données

Suivez les requêtes PDO comme ceci :

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // obligatoire pour capturer les requêtes pour l’APM
]);
$Apm->addPdoConnection($pdo);
```

**Ce que vous obtenez** :
- Texte de la requête (par ex. `SELECT * FROM users WHERE id = ?`)
- Temps d’exécution (par ex. 0.015 s)
- Nombre de lignes (par ex. 42)

**Attention** :
- **Optionnel** : Ignorez ceci si vous n’avez pas besoin du suivi DB.
- **SimplePdo (préféré)** : Utilisez `SimplePdo` avec `trackApmQueries => true`. Le `PdoWrapper` déprécié fonctionne toujours (5e argument du constructeur `true`). Le PDO brut du cœur n’est pas encore connecté — restez à l’écoute !
- **Avertissement de performance** : Journaliser chaque requête sur un site riche en DB peut ralentir les choses. Utilisez l’échantillonnage (`$Apm = new Apm($ApmLogger, 0.1)`) pour alléger la charge.

**Exemple de sortie** :
- Requête : `SELECT name FROM products WHERE price > 100`
- Temps : 0.023 s
- Lignes : 15

## Options du worker

Ajustez le worker à votre convenance :

- `--timeout 300` : S’arrête après 5 minutes — bien pour les tests.
- `--max_messages 500` : Limite à 500 métriques — garde ça fini.
- `--batch_size 200` : Traite 200 à la fois — équilibre vitesse et mémoire.
- `--daemon` : Fonctionne sans arrêt — idéal pour la surveillance en direct.

**Exemple** :
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Fonctionne pendant une heure, traitant 100 métriques à la fois.

## Request ID dans l’application

Chaque requête a un identifiant de requête unique pour le suivi. Vous pouvez utiliser cet identifiant dans votre application pour corréler les journaux et les métriques. Par exemple, vous pouvez ajouter l’identifiant de requête à une page d’erreur :

```php
Flight::map('error', function($message) {
	// Obtenez l’identifiant de requête depuis l’en-tête de réponse X-Flight-Request-Id
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Vous pourriez également le récupérer depuis la variable Flight
	// Cette méthode ne fonctionnera pas bien dans swoole ou d’autres plateformes asynchrones.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Erreur : $message (Request ID : $requestId)";
});
```

## Mise à niveau

Si vous mettez à niveau vers une nouvelle version de l’APM, il y a une chance qu’il y ait des migrations de base de données à exécuter. Vous pouvez le faire en exécutant la commande suivante :

```bash
php vendor/bin/runway apm:migrate
```
Cela exécutera toutes les migrations nécessaires pour mettre à jour le schéma de la base de données vers la dernière version.

**Remarque :** Si votre base de données APM est volumineuse, ces migrations peuvent prendre du temps. Vous voudrez peut-être exécuter cette commande en dehors des heures de pointe.

### Mise à niveau de 0.4.3 -> 0.5.0

Si vous mettez à niveau de 0.4.3 vers 0.5.0, vous devrez exécuter la commande suivante :

```bash
php vendor/bin/runway apm:config-migrate
```

Cela migrera votre configuration de l’ancien format utilisant le fichier `.runway-config.json` vers le nouveau format qui stocke les clés/valeurs dans le fichier `config.php`.

## Purge des anciennes données

Pour garder votre base de données propre, vous pouvez purger les anciennes données. C’est particulièrement utile si vous exécutez une application très sollicitée et que vous voulez garder la taille de la base de données gérable.
Vous pouvez le faire en exécutant la commande suivante :

```bash
php vendor/bin/runway apm:purge
```
Cela supprimera toutes les données de plus de 30 jours de la base de données. Vous pouvez ajuster le nombre de jours en passant une valeur différente à l’option `--days` :

```bash
php vendor/bin/runway apm:purge --days 7
```
Cela supprimera toutes les données de plus de 7 jours de la base de données.

## Dépannage

Bloqué ? Essayez ceci :

- **Aucune donnée dans le tableau de bord ?**
  - Le worker fonctionne-t-il ? Vérifiez `ps aux | grep apm:worker`.
  - Les chemins de configuration correspondent-ils ? Vérifiez que les DSN dans `.runway-config.json` pointent vers des fichiers réels.
  - Exécutez `php vendor/bin/runway apm:worker` manuellement pour traiter les métriques en attente.

- **Erreurs du worker ?**
  - Jetez un œil à vos fichiers SQLite (par ex. `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Vérifiez les journaux PHP pour les traces de pile.

- **Le tableau de bord ne démarre pas ?**
  - Le port 8001 est-il utilisé ? Utilisez `--port 8080`.
  - PHP introuvable ? Utilisez `--php-path /usr/bin/php`.
  - Pare-feu bloquant ? Ouvrez le port ou utilisez `--host localhost`.

- **Trop lent ?**
  - Réduisez le taux d’échantillonnage : `$Apm = new Apm($ApmLogger, 0.05)` (5 %).
  - Réduisez la taille du lot : `--batch_size 20`.

- **Ne suit pas les exceptions/erreurs ?**
  - Si vous avez [Tracy](https://tracy.nette.org/) activé pour votre projet, il remplacera la gestion des erreurs de Flight. Vous devrez désactiver Tracy et vous assurer que `Flight::set('flight.handle_errors', true);` est défini.

- **Ne suit pas les requêtes de base de données ?**
  - Préférez `SimplePdo` avec `['trackApmQueries' => true]` comme 5e argument du constructeur (tableau d’options).
  - Si vous utilisez toujours le `PdoWrapper` déprécié, passez `true` comme 5e argument.
  - Appelez `$Apm->addPdoConnection($pdo)` après avoir créé la connexion.