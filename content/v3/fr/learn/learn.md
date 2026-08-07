# Apprendre Flight

Flight est un framework PHP rapide, simple et extensible. Il est très polyvalent et peut être utilisé pour construire tout type d'application web.
Il est conçu avec la simplicité à l'esprit et est écrit d'une manière facile à comprendre et à utiliser—par les humains et par les [assistants de codage IA](/learn/ai).

> **Remarque :** Vous verrez des exemples qui utilisent `Flight::` comme variable statique et d'autres qui utilisent l'objet Moteur `$app->`. Les deux fonctionnent de manière interchangeable. `$app` et `$this->app` dans un contrôleur/middleware est l'approche recommandée par l'équipe Flight (et ce que le squelette officiel + `AGENTS.md` standardisent pour les nouveaux projets).

## Composants principaux

### [Routage](/learn/routing)

Apprenez à gérer les routes pour votre application web. Cela inclut également le regroupement des routes, les paramètres de route et le middleware.

### [Middleware](/learn/middleware)

Apprenez à utiliser le middleware pour filtrer les requêtes et les réponses dans votre application.

### [Autochargement](/learn/autoloading)

Apprenez à charger automatiquement vos propres classes. La **casse** du dossier doit correspondre à vos espaces de noms—le squelette utilise `App\` et des dossiers en PascalCase comme `app/Controller/`.

### [Requêtes](/learn/requests)

Apprenez à gérer les requêtes et les réponses dans votre application.

### [Réponses](/learn/responses)

Apprenez à envoyer des réponses à vos utilisateurs.

### [Modèles HTML](/learn/templates)

Apprenez à rendre du HTML avec Twig (par défaut du squelette), Latte, ou d'autres moteurs—pas seulement les vues PHP intégrées.

### [Sécurité](/learn/security)

Apprenez à sécuriser votre application contre les menaces de sécurité courantes.

### [Configuration](/learn/configuration)

Apprenez à configurer le framework pour votre application.

### [Gestionnaire d'événements](/learn/events)

Apprenez à utiliser le système d'événements pour ajouter des événements personnalisés à votre application.

### [Étendre Flight](/learn/extending)

Apprenez à étendre le framework en ajoutant vos propres méthodes et classes.

### [Crochets de méthode et filtrage](/learn/filtering)

Apprenez à ajouter des crochets d'événements à vos méthodes et aux méthodes internes du framework.

### [Conteneur d'injection de dépendances (DIC)](/learn/dependency-injection-container)

Apprenez à utiliser les conteneurs d'injection de dépendances (DIC) pour gérer les dépendances de votre application.

## Classes utilitaires

### [Collections](/learn/collections)

Les collections sont utilisées pour contenir des données et être accessibles sous forme de tableau ou d'objet pour faciliter leur utilisation.

### [Wrapper JSON](/learn/json)

Ceci contient quelques fonctions simples pour rendre l'encodage et le décodage de votre JSON cohérents.

### [SimplePdo](/learn/simple-pdo)

PDO peut parfois ajouter plus de maux de tête que nécessaire. SimplePdo est une classe d'aide PDO moderne avec des méthodes pratiques comme `insert()`, `update()`, `delete()` et `transaction()` pour rendre les opérations de base de données beaucoup plus faciles.

### [PdoWrapper](/learn/pdo-wrapper) (Obsolète)

Le wrapper PDO d'origine est obsolète depuis la v3.18.0. Veuillez utiliser [SimplePdo](/learn/simple-pdo) à la place.

### [Gestionnaire de fichiers téléchargés](/learn/uploaded-file)

Une classe simple pour aider à gérer les fichiers téléchargés et les déplacer vers un emplacement permanent.

## Concepts importants

### [Pourquoi un framework ?](/learn/why-frameworks)

Voici un court article expliquant pourquoi vous devriez utiliser un framework. Il est bon de comprendre les avantages d'utiliser un framework avant de commencer à en utiliser un.

De plus, un excellent tutoriel a été créé par [@lubiana](https://git.php.fail/lubiana). Bien qu'il n'entre pas dans les détails spécifiques à Flight,
ce guide vous aidera à comprendre certains des principaux concepts entourant un framework et pourquoi ils sont bénéfiques à utiliser.
Vous pouvez trouver le tutoriel [ici](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight comparé à d'autres frameworks](/learn/flight-vs-another-framework)

Si vous migrez depuis un autre framework tel que Laravel, Slim, Fat-Free, ou Symfony vers Flight, cette page vous aidera à comprendre les différences entre les deux.

## Autres sujets

### [Tests unitaires](/learn/unit-testing)

Suivez ce guide pour apprendre à tester unitairement votre code Flight afin qu'il soit solide comme un roc.

### [IA et expérience développeur](/learn/ai)

Flight est conçu pour s'associer aux LLM de codage : `AGENTS.md`, commandes Runway `ai:*`, et une disposition de squelette claire pour que les agents restent sur le modèle.

### [Migration v2 -> v3](/learn/migrating-to-v3)

La rétrocompatibilité a été en grande partie maintenue, mais il y a quelques changements dont vous devez être conscient lors de la migration de v2 à v3.