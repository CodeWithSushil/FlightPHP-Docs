# Tracy

Tracy est un gestionnaire d'erreurs incroyable qui peut être utilisé avec Flight. Il possède un certain nombre de panneaux qui peuvent vous aider à déboguer votre application. Il est également très facile à étendre et à ajouter vos propres panneaux. L'équipe Flight a créé quelques panneaux spécifiquement pour les projets Flight avec le plugin [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) (variables Flight, requêtes DB, requête, session, et un panneau **Twig** optionnel lorsque vous passez un profil de profiler—voir [Extensions Tracy](/awesome-plugins/tracy-extensions)).

## Installation

Installer avec composer. Et vous voudrez réellement installer cela sans la version dev car Tracy vient avec un composant de gestion d'erreurs de production.

```bash
composer require tracy/tracy
```

## Configuration de base

Il y a quelques options de configuration de base pour commencer. Vous pouvez en savoir plus à leur sujet dans la [Documentation Tracy](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Activer Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // parfois vous devez être explicite (aussi Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // vous pouvez aussi fournir un tableau d'adresses IP

// C'est là où les erreurs et exceptions seront journalisées. Assurez-vous que ce répertoire existe et est accessible en écriture.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // afficher toutes les erreurs
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // toutes les erreurs sauf les notices dépréciées
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // si la barre Debugger est visible, alors content-length ne peut pas être défini par Flight

	// C'est spécifique à l'Extension Tracy pour Flight si vous l'avez incluse
	// sinon commentez ceci.
	new TracyExtensionLoader($app);
}
```

## Conseils utiles

Lorsque vous déboguez votre code, il y a des fonctions très utiles pour afficher des données pour vous.

- `bdump($var)` - Cela va vider la variable dans la barre Tracy dans un panneau séparé.
- `dumpe($var)` - Cela va vider la variable et puis mourir immédiatement.