# Plugins Incroyables

Flight est extrêmement extensible. Il existe un certain nombre de plugins qui peuvent être utilisés pour ajouter des fonctionnalités à votre application Flight. Certains sont officiellement supportés par l'Équipe Flight et d'autres sont des bibliothèques micro/lite pour vous aider à démarrer.

## Outils IA

Flight peut être rendu encore plus cool avec des plugins alimentés par l'IA.

- [Flight MCP](/awesome-plugins/mcp) - Un plugin pour intégrer MCP (Model Control Protocol) avec Flight, permettant une fonctionnalité transparente alimentée par l'IA. Principalement axé sur les pages de documentation, il aide à réduire les coûts de tokens en fournissant les informations les plus à jour sur vos projets Flight.

## Documentation API

La documentation API est cruciale pour toute API. Elle aide les développeurs à comprendre comment interagir avec votre API et à quoi s'attendre en retour. Il existe quelques outils disponibles pour vous aider à générer la documentation API pour vos Projets Flight.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Article de blog écrit par Daniel Schreiber sur comment utiliser la Spécification OpenAPI avec FlightPHP pour construire votre API en utilisant une approche API-first.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI est un excellent outil pour vous aider à générer la documentation API pour vos projets Flight. Il est très facile à utiliser et peut être personnalisé pour répondre à vos besoins. C'est la bibliothèque PHP pour vous aider à générer la documentation Swagger.

## Surveillance des Performances d'Application (APM)

La Surveillance des Performances d'Application (APM) est cruciale pour toute application. Elle vous aide à comprendre comment votre application performe et où se trouvent les goulots d'étranglement. Il existe un certain nombre d'outils APM qui peuvent être utilisés avec Flight.
- <span class="badge bg-primary">officiel</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM est une bibliothèque APM simple qui peut être utilisée pour surveiller vos applications Flight. Elle peut être utilisée pour surveiller les performances de votre application et vous aider à identifier les goulots d'étranglement.

## Asynchrone

Flight est déjà un framework rapide mais lui ajouter un moteur turbo rend tout plus amusant (et stimulant) !

- [flightphp/async](/awesome-plugins/async) - Bibliothèque Asynchrone Flight officielle. Cette bibliothèque est un moyen simple d'ajouter du traitement asynchrone à votre application. Elle utilise Swoole/Openswoole en interne pour fournir un moyen simple et efficace d'exécuter des tâches de manière asynchrone.

## Autorisation/Permissions

L'Autorisation et les Permissions sont cruciales pour toute application qui nécessite des contrôles pour qui peut accéder à quoi.

- <span class="badge bg-primary">officiel</span> [flightphp/permissions](/awesome-plugins/permissions) - Bibliothèque Permissions Flight officielle. Cette bibliothèque est un moyen simple d'ajouter des permissions au niveau utilisateur et application à votre application.

## Authentification

L'Authentification est essentielle pour les applications qui doivent vérifier l'identité des utilisateurs et sécuriser les points de terminaison API.

- [firebase/php-jwt](/awesome-plugins/jwt) - Bibliothèque JSON Web Token (JWT) pour PHP. Un moyen simple et sécurisé d'implémenter l'authentification basée sur les tokens dans vos applications Flight. Parfait pour l'authentification API stateless, protéger les routes avec des middleware, et implémenter des flux d'autorisation de style OAuth.

## Mise en Cache

La mise en cache est un excellent moyen d'accélérer votre application. Il existe un certain nombre de bibliothèques de mise en cache qui peuvent être utilisées avec Flight.

- <span class="badge bg-primary">officiel</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Classe de mise en cache PHP en fichier légère, simple et autonome

## CLI

Les applications CLI sont un excellent moyen d'interagir avec votre application. Vous pouvez les utiliser pour générer des contrôleurs, afficher toutes les routes, et plus encore.

- <span class="badge bg-primary">officiel</span> [flightphp/runway](/awesome-plugins/runway) - Runway est une application CLI qui vous aide à gérer vos applications Flight.

## Cookies

Les cookies sont un excellent moyen de stocker de petits morceaux de données côté client. Ils peuvent être utilisés pour stocker les préférences utilisateur, les paramètres d'application, et plus encore.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie est une bibliothèque PHP qui fournit un moyen simple et efficace de gérer les cookies.

## Débogage

Le débogage est crucial lorsque vous développez dans votre environnement local. Il existe quelques plugins qui peuvent améliorer votre expérience de débogage.

- [tracy/tracy](/awesome-plugins/tracy) - C'est un gestionnaire d'erreurs complet qui peut être utilisé avec Flight. Il a un certain nombre de panneaux qui peuvent vous aider à déboguer votre application. Il est aussi très facile à étendre et ajouter vos propres panneaux.
- <span class="badge bg-primary">officiel</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Utilisé avec le gestionnaire d'erreurs [Tracy](/awesome-plugins/tracy), ce plugin ajoute quelques panneaux supplémentaires pour aider au débogage spécifiquement pour les projets Flight.

## Bases de Données

Les bases de données sont le cœur de la plupart des applications. C'est ainsi que vous stockez et récupérez les données. Certaines bibliothèques de base de données sont simplement des wrappers pour écrire des requêtes et d'autres sont des ORM complets.

- <span class="badge bg-primary">officiel</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Assistant PDO Flight officiel qui fait partie du cœur. C'est un wrapper moderne avec des méthodes d'aide pratiques comme `insert()`, `update()`, `delete()`, et `transaction()` pour simplifier les opérations de base de données. Tous les résultats sont retournés sous forme de Collections pour un accès flexible tableau/objet. Pas un ORM, juste une meilleure façon de travailler avec PDO.
- <span class="badge bg-warning">déprécié</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Wrapper PDO Flight officiel qui fait partie du cœur (déprécié depuis v3.18.0). Utilisez SimplePdo à la place.
- <span class="badge bg-primary">officiel</span> [flightphp/active-record](/awesome-plugins/active-record) - ORM/Mapper ActiveRecord Flight officiel. Excellente petite bibliothèque pour récupérer et stocker facilement les données dans votre base de données.
- [byjg/php-migration](/awesome-plugins/migrations) - Plugin pour suivre tous les changements de base de données pour votre projet.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Constructeur de requêtes SQL fluide et léger qui génère du SQL et des paramètres pour les requêtes préparées. Fonctionne très bien avec [SimplePdo](/learn/simple-pdo).

## Chiffrement

Le chiffrement est crucial pour toute application qui stocke des données sensibles. Chiffrer et déchiffrer les données n'est pas terriblement difficile, mais stocker correctement la clé de chiffrement [peut](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [être](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [difficile](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). La chose la plus importante est de ne jamais stocker votre clé de chiffrement dans un répertoire public ou de la commettre dans votre dépôt de code.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - C'est une bibliothèque qui peut être utilisée pour chiffrer et déchiffrer des données. Démarrer et exécuter est assez simple pour commencer à chiffrer et déchiffrer des données.

## File d'Attente de Tâches

Les files d'attente de tâches sont vraiment utiles pour traiter des tâches de manière asynchrone. Cela peut être l'envoi d'emails, le traitement d'images, ou tout ce qui n'a pas besoin d'être fait en temps réel.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue est une bibliothèque qui peut être utilisée pour traiter des tâches de manière asynchrone. Elle peut être utilisée avec beanstalkd, MySQL/MariaDB, SQLite, et PostgreSQL.

## Session

Les sessions ne sont pas vraiment utiles pour les API mais pour construire une application web, les sessions peuvent être cruciales pour maintenir l'état et les informations de connexion.

- <span class="badge bg-primary">officiel</span> [flightphp/session](/awesome-plugins/session) - Bibliothèque Session Flight officielle. C'est une bibliothèque de session simple qui peut être utilisée pour stocker et récupérer des données de session. Elle utilise la gestion de session intégrée de PHP.
- [Ghostff/Session](/awesome-plugins/ghost-session) - Gestionnaire de Session PHP (non-bloquant, flash, segment, chiffrement de session). Utilise PHP open_ssl pour le chiffrement/déchiffrement optionnel des données de session.

## Templating

Le templating est au cœur de toute application web avec une interface utilisateur. Il existe un certain nombre de moteurs de templating qui peuvent être utilisés avec Flight.

- <span class="badge bg-warning">déprécié</span> [flightphp/core View](/learn#views) - C'est un moteur de templating très basique qui fait partie du cœur. Il n'est pas recommandé de l'utiliser si vous avez plus d'une couple de pages dans votre projet.
- [latte/latte](/awesome-plugins/latte) - Latte est un moteur de templating complet qui est très facile à utiliser et ressemble plus à une syntaxe PHP que Twig ou Smarty. Il est aussi très facile à étendre et ajouter vos propres filtres et fonctions.
- [twig/twig](/awesome-plugins/twig) - Twig est un moteur de template flexible, rapide et sécurisé (le même utilisé par Symfony). Les outils IA et de nombreux développeurs PHP le connaissent bien, il échappe automatiquement la sortie par défaut, et il a un énorme écosystème d'extensions.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate est un puissant moteur de template PHP avec compilation d'assets, héritage de templates, et traitement de variables. Fonctionnalités de minification automatique CSS/JS, mise en cache, encodage Base64, et intégration optionnelle du framework Flight PHP.

## Intégration WordPress

Vous voulez utiliser Flight dans votre projet WordPress ? Il y a un plugin pratique pour ça !

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Ce plugin WordPress vous permet d'exécuter Flight directement aux côtés de WordPress. C'est parfait pour ajouter des API personnalisées, des microservices, ou même des applications complètes à votre site WordPress en utilisant le framework Flight. Super utile si vous voulez le meilleur des deux mondes !

## Contribution

Vous avez un plugin que vous aimeriez partager ? Soumettez une pull request pour l'ajouter à la liste !