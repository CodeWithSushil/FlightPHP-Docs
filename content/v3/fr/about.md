# Flight PHP Framework

Flight est un framework PHP rapide, simple et extensible—conçu pour les développeurs qui veulent accomplir des tâches rapidement, sans complications. Que vous construisiez une application web classique, une API ultra-rapide, ou que vous travailliez avec des assistants de codage IA, l'empreinte légère et le design direct de Flight en font un choix parfait. Flight est conçu pour être léger, mais peut également gérer les exigences d'une architecture d'entreprise.

## Pourquoi Choisir Flight ?

- **Accessible aux Débutants :** Flight est un excellent point de départ pour les nouveaux développeurs PHP. Sa structure claire et sa syntaxe simple vous aident à apprendre le développement web sans vous perdre dans le code standard.
- **Adoré par les Pros :** Les développeurs expérimentés apprécient Flight pour sa flexibilité et son contrôle. Vous pouvez passer d'un petit prototype à une application complète sans changer de framework.
- **Rétrocompatible :** Nous valorisons votre temps. Flight v3 est une augmentation de v2, conservant presque toute la même API. Nous croyons en l'évolution, pas en la révolution—plus de "casser le monde" à chaque nouvelle version majeure.
- **Zéro Dépendances :** Le cœur de Flight est entièrement exempt de dépendances—pas de polyfills, pas de packages externes, pas même d'interfaces PSR. Cela signifie moins de vecteurs d'attaque, une empreinte plus petite et pas de changements cassants inattendus provenant de dépendances en amont. Les plugins optionnels peuvent inclure des dépendances, mais le cœur restera toujours léger et sécurisé.
- **Compatible IA :** La petite surface d'API de Flight et le [squelette officiel](https://github.com/flightphp/skeleton) (une mise en page, `AGENTS.md`, injection par constructeur) facilitent le travail des outils de codage IA pour rester dans le modèle. Même base de code que vous tapiez chaque ligne ou que vous travailliez avec un agent. [En savoir plus sur l'utilisation de l'IA avec Flight](/learn/ai).

## Aperçu Vidéo

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">C'est assez simple, n'est-ce pas ?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">En savoir plus</a> sur Flight dans la documentation !
    </div>
  </div>
</div>

## Démarrage Rapide

Pour une installation rapide et basique, installez-le avec Composer :

```bash
composer require flightphp/core
```

Ou vous pouvez télécharger un zip du dépôt [ici](https://github.com/flightphp/core). Ensuite, vous aurez un fichier `index.php` basique comme suit :

```php
<?php

// si installé avec composer
require 'vendor/autoload.php';
// ou si installé manuellement par fichier zip
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

C'est tout ! Vous avez une application Flight basique. Vous pouvez maintenant exécuter ce fichier avec `php -S localhost:8000` et visiter `http://localhost:8000` dans votre navigateur pour voir le résultat.

Les exemples courts avec `Flight::` comme celui-ci sont excellents pour l'apprentissage et les micro-applications. Pour une mise en page de projet complète que les humains et les outils IA partagent, utilisez le squelette ci-dessous.

## Application Squelette/Modèle

Il existe un starter officiel pour vous aider à commencer tout nouveau projet Flight. Il configure la structure, la configuration, les scripts Composer et les instructions adaptées à l'IA dès le départ.

Consultez [flightphp/skeleton](https://github.com/flightphp/skeleton) pour un projet prêt à l'emploi, ou visitez la page [exemples](examples) pour l'inspiration. Vous voulez les détails du workflow IA ? [Explorez l'IA et l'expérience développeur](/learn/ai).

Ce que vous obtenez (niveau élevé) :

- **Espaces de noms `App\`** avec des dossiers PascalCase (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—la **casse** du dossier doit correspondre à l'espace de noms (voir [Autoloading](/learn/autoloading))
- **Injection Dice + `Engine`** pour que les contrôleurs restent testables (préférez `$this->app` plutôt que `Flight::` dans le code de l'application)
- Vues **Twig**, échantillon **SimplePdo** + ActiveRecord, **migrate** Runway
- **`AGENTS.md`** à la racine (plus des copies ciblées) et **`SECURITY.md`** pour les assistants et la politique de sécurité

## Installation de l'Application Squelette

C'est assez simple !

```bash
# Créer le nouveau projet
composer create-project flightphp/skeleton my-project/
# Entrer dans le répertoire de votre nouveau projet
cd my-project/
# Démarrer le serveur de développement local pour commencer immédiatement !
composer start
```

Cela crée la structure du projet, copie `config_sample.php` → `config.php` (et `.env.example` → `.env` si présent), et vous êtes prêt à partir. Données d'exemple optionnelles :

```bash
php runway migrate
# puis visitez /posts et /api/posts
```

## Haute Performance

Flight est l'un des frameworks PHP les plus rapides disponibles. Son cœur léger signifie moins de surcharge et plus de vitesse—parfait pour les applications traditionnelles et les workflows assistés par IA modernes. Vous pouvez voir tous les benchmarks sur [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Voyez le benchmark ci-dessous avec d'autres frameworks PHP populaires.

| Framework | Requêtes/sec Texte Brut | Requêtes/sec JSON |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight et l'IA

Curieux de savoir comment Flight s'associe aux LLM de codage ? [Découvrez](/learn/ai) comment `AGENTS.md`, les commandes Runway `ai:*`, et la mise en page du squelette gardent les assistants sur les rails.

## Stabilité et Rétrocompatibilité

Nous valorisons votre temps. Nous avons tous vu des frameworks qui se réinventent complètement tous les quelques années, laissant les développeurs avec du code cassé et des migrations coûteuses. Flight est différent. Flight v3 a été conçu comme une augmentation de v2, ce qui signifie que l'API que vous connaissez et appréciez n'a pas été supprimée. En fait, la plupart des projets v2 fonctionneront sans aucun changement dans v3.

Nous nous engageons à garder Flight stable pour que vous puissiez vous concentrer sur la construction de votre application, pas sur la correction de votre framework. Le squelette peut être opiniâtre pour les *nouveaux* projets ; les API du cœur restent familières pour tout le monde.

# Communauté

Nous sommes sur Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

Et Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Contribution

Il y a deux façons de contribuer à Flight :

1. Contribuer au framework principal en visitant le [dépôt principal](https://github.com/flightphp/core).
2. Aider à améliorer la documentation ! Ce site de documentation est hébergé sur [Github](https://github.com/flightphp/docs). Si vous repérez une erreur ou voulez améliorer quelque chose, n'hésitez pas à soumettre une pull request. Nous adorons les mises à jour et les nouvelles idées—surtout autour de l'IA et des nouvelles technologies !

# Exigences

Flight nécessite PHP 7.4 ou supérieur.

**Note :** PHP 7.4 est supporté car au moment de la rédaction (2024), PHP 7.4 est la version par défaut pour certaines distributions Linux LTS. Forcer un passage à PHP >8 causerait beaucoup de problèmes pour ces utilisateurs. Le framework supporte également PHP >8.

# Licence

Flight est publié sous la licence [MIT](https://github.com/flightphp/core/blob/master/LICENSE).