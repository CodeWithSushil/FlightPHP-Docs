# Flight vs Slim

## Qu'est-ce que Slim ?
[Slim](https://slimframework.com) est un micro framework PHP qui vous aide à écrire rapidement des applications web et des API simples mais puissantes.

Une grande partie de l'inspiration pour certaines des fonctionnalités de la v3 de Flight vient en réalité de Slim. Le regroupement des routes et l'exécution du middleware dans un ordre spécifique sont deux fonctionnalités inspirées de Slim. Slim v3 est sorti avec une orientation vers la simplicité, mais il y a eu [des avis mitigés](https://github.com/slimphp/Slim/issues/2770) concernant la v4.

## Avantages par rapport à Flight

- Slim a une plus grande communauté de développeurs, qui créent à leur tour des modules pratiques pour vous aider à ne pas réinventer la roue.
- Slim suit de nombreuses interfaces et normes courantes dans la communauté PHP, ce qui augmente l'interopérabilité.
- Slim possède une documentation et des tutoriels décents qui peuvent être utilisés pour apprendre le framework (rien de comparable à Laravel ou Symfony cependant).
- Slim dispose de diverses ressources comme des tutoriels YouTube et des articles en ligne qui peuvent être utilisés pour apprendre le framework.
- Slim vous permet d'utiliser les composants que vous voulez pour gérer les fonctionnalités de routage de base car il est conforme à la norme PSR-7.

## Inconvénients par rapport à Flight

- Étonnamment, Slim n'est pas aussi rapide que vous pourriez le penser pour un micro framework. Voir les [benchmarks TechEmpower](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) pour plus d'informations.
- Flight est conçu pour un développeur qui cherche à créer une application web légère, rapide et facile à utiliser.
- Flight n'a aucune dépendance, alors que [Slim a quelques dépendances](https://github.com/slimphp/Slim/blob/4.x/composer.json) que vous devez installer.
- Flight est orienté vers la simplicité et la facilité d'utilisation.
- L'une des fonctionnalités clés de Flight est qu'il fait de son mieux pour maintenir la rétrocompatibilité. Le passage de Slim v3 à v4 a été une rupture de compatibilité.
- Flight est destiné aux développeurs qui s'aventurent pour la première fois dans le monde des frameworks.
- Flight peut également gérer des applications de niveau entreprise, mais il n'a pas autant d'exemples et de tutoriels que Slim. Cela exigera également plus de discipline de la part du développeur pour garder les choses organisées et bien structurées.
- Flight donne au développeur plus de contrôle sur l'application, tandis que Slim peut introduire un peu de magie en coulisses.
- Flight dispose de [SimplePdo](/learn/simple-pdo) pour l'accès à la base de données (préféré à l'ancien PdoWrapper obsolète). Slim vous oblige à utiliser une bibliothèque tierce.
- Flight dispose d'un [plugin de permissions](/awesome-plugins/permissions) qui peut être utilisé pour sécuriser votre application. Slim vous oblige à utiliser une bibliothèque tierce.
- Flight dispose d'un ORM appelé [active-record](/awesome-plugins/active-record) qui peut être utilisé pour interagir avec votre base de données. Slim vous oblige à utiliser une bibliothèque tierce.
- Flight dispose d'une application CLI appelée [runway](/awesome-plugins/runway) qui peut être utilisée pour exécuter votre application depuis la ligne de commande. Slim n'en a pas.