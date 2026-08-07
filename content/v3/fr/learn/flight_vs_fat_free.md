# Flight vs Fat-Free

## Qu'est-ce que Fat-Free ?
[Fat-Free](https://fatfreeframework.com) (affectueusement connu sous le nom **F3**) est un micro-framework PHP puissant et facile à utiliser, conçu pour vous aider à créer des applications web dynamiques et robustes - rapidement !

Flight se compare à Fat-Free à bien des égards et est probablement le plus proche cousin en termes de fonctionnalités et de simplicité. Fat-Free possède de nombreuses fonctionnalités que Flight n'a pas, mais il possède aussi de nombreuses fonctionnalités que Flight a. Fat-Free commence à montrer son âge et n'est plus aussi populaire qu'avant.

Les mises à jour deviennent moins fréquentes et la communauté n'est plus aussi active qu'avant. Le code est assez simple, mais parfois le manque de discipline syntaxique peut le rendre difficile à lire et à comprendre. Il fonctionne avec PHP 8.3, mais le code lui-même ressemble toujours à du PHP 5.3.

## Avantages par rapport à Flight

- Fat-Free a quelques étoiles de plus que Flight sur GitHub.
- Fat-Free a une documentation correcte, mais elle manque de clarté dans certains domaines.
- Fat-Free dispose de quelques ressources éparses comme des tutoriels YouTube et des articles en ligne qui peuvent être utilisés pour apprendre le framework.
- Fat-Free a [quelques plugins intégrés](https://fatfreeframework.com/3.8/api-reference) qui sont parfois utiles.
- Fat-Free a un ORM intégré appelé Mapper qui peut être utilisé pour interagir avec votre base de données. Flight a [active-record](/awesome-plugins/active-record).
- Fat-Free a les Sessions, le Cache et la localisation intégrés. Flight vous oblige à utiliser des bibliothèques tierces, mais cela est couvert dans la [documentation](/awesome-plugins).
- Fat-Free a un petit groupe de [plugins créés par la communauté](https://fatfreeframework.com/3.8/development#Community) qui peuvent être utilisés pour étendre le framework. Flight en a quelques-uns couverts dans les pages [documentation](/awesome-plugins) et [exemples](/examples).
- Fat-Free comme Flight ne dépend d'aucune bibliothèque externe.
- Fat-Free comme Flight est conçu pour donner au développeur le contrôle sur son application et une expérience de développement simple.
- Fat-Free maintient la rétrocompatibilité comme Flight (en partie parce que les mises à jour deviennent [moins fréquentes](https://github.com/bcosca/fatfree/releases)).
- Fat-Free comme Flight est destiné aux développeurs qui s'aventurent pour la première fois dans le monde des frameworks.
- Fat-Free a un moteur de templates intégré plus robuste que celui de Flight. Flight recommande [Latte](/awesome-plugins/latte) pour y parvenir.
- Fat-Free a une commande unique de type « route » CLI qui permet de créer des applications CLI au sein de Fat-Free lui-même et de la traiter comme une requête `GET`. Flight accomplit cela avec [runway](/awesome-plugins/runway).

## Inconvénients par rapport à Flight

- Fat-Free a quelques tests d'implémentation et possède même sa propre classe [test](https://fatfreeframework.com/3.8/test) très basique. Cependant,
  il n'est pas testé unitairement à 100% comme Flight.
- Vous devez utiliser un moteur de recherche comme Google pour réellement rechercher dans le site de documentation.
- Flight a un mode sombre sur son site de documentation. (mic drop)
- Fat-Free a certains modules qui sont malheureusement non maintenus.
- Flight a [SimplePdo](/learn/simple-pdo) pour l'accès à la base de données, ce qui est un peu plus simple que la classe `DB\SQL` intégrée de Fat-Free (et préféré au PdoWrapper obsolète).
- Flight a un [plugin de permissions](/awesome-plugins/permissions) qui peut être utilisé pour sécuriser votre application. Fat Free vous oblige à utiliser une bibliothèque tierce.
- Flight a un ORM appelé [active-record](/awesome-plugins/active-record) qui ressemble plus à un ORM que le Mapper de Fat-Free.
  L'avantage supplémentaire d'`active-record` est que vous pouvez définir des relations entre les enregistrements pour des jointures automatiques, alors que le Mapper de Fat-Free
  vous oblige à créer des [vues SQL](https://fatfreeframework.com/3.8/databases#ProsandCons).
- Chose étonnante, Fat-Free n'a pas de namespace racine. Flight est entièrement namespacé pour ne pas entrer en collision avec votre propre code.
  La classe `Cache` est le plus grand contrevenant ici.
- Fat-Free n'a pas de middleware. À la place, il y a des hooks `beforeroute` et `afterroute` qui peuvent être utilisés pour filtrer les requêtes et les réponses dans les contrôleurs.
- Fat-Free ne peut pas regrouper les routes.
- Fat-Free a un gestionnaire de conteneur d'injection de dépendances, mais la documentation est incroyablement maigre sur la façon de l'utiliser.
- Le débogage peut devenir un peu délicat car essentiellement tout est stocké dans ce qu'on appelle le [`HIVE`](https://fatfreeframework.com/3.8/quick-reference).