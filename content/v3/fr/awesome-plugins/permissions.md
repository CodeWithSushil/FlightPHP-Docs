# FlightPHP/Permissions

Ceci est un module de permissions qui peut être utilisé dans vos projets si vous avez plusieurs rôles dans votre application et que chaque rôle a une fonctionnalité légèrement différente. Ce module vous permet de définir des permissions pour chaque rôle et ensuite vérifier si l'utilisateur actuel a la permission d'accéder à une certaine page ou d'effectuer une certaine action. 

Cliquez [ici](https://github.com/flightphp/permissions) pour le dépôt sur GitHub.

Installation
-------
Exécutez `composer require flightphp/permissions` et vous êtes prêt !

Utilisation
-------
D'abord, vous devez configurer vos permissions, puis vous indiquez à votre application ce que signifient les permissions. En fin de compte, vous vérifierez vos permissions avec `$Permissions->has()`, `->can()`, ou `is()`. `has()` et `can()` ont la même fonctionnalité, mais portent des noms différents pour rendre votre code plus lisible.

## Exemple de base

Supposons que vous avez une fonctionnalité dans votre application qui vérifie si un utilisateur est connecté. Vous pouvez créer un objet permissions comme ceci :

```php
// index.php
require 'vendor/autoload.php';

// du code 

// puis vous avez probablement quelque chose qui vous indique quel est le rôle actuel de la personne
// vous avez probablement quelque chose où vous récupérez le rôle actuel
// depuis une variable de session qui définit cela
// après qu'une personne se connecte, sinon elle aura un rôle 'guest' ou 'public'.
$current_role = 'admin';

// configuration des permissions
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// Vous voudrez probablement persister cet objet dans Flight quelque part
Flight::set('permission', $permission);
```

Ensuite, dans un contrôleur quelque part, vous pourriez avoir quelque chose comme ceci.

```php
<?php

// un contrôleur
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// faire quelque chose
		} else {
			// faire autre chose
		}
	}
}
```

Vous pouvez également l'utiliser pour suivre s'ils ont la permission de faire quelque chose dans votre application.
Par exemple, si vous avez un moyen pour que les utilisateurs puissent interagir avec la publication sur votre logiciel, vous pouvez 
vérifier s'ils ont la permission d'effectuer certaines actions.

```php
$current_role = 'admin';

// configuration des permissions
$permission = new \flight\Permission($current_role);
$permission->defineRule('post', function($current_role) {
	if($current_role === 'admin') {
		$permissions = ['create', 'read', 'update', 'delete'];
	} else if($current_role === 'editor') {
		$permissions = ['create', 'read', 'update'];
	} else if($current_role === 'author') {
		$permissions = ['create', 'read'];
	} else if($current_role === 'contributor') {
		$permissions = ['create'];
	} else {
		$permissions = [];
	}
	return $permissions;
});
Flight::set('permission', $permission);
```

Ensuite, dans un contrôleur quelque part...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// faire quelque chose
		} else {
			// faire autre chose
		}
	}
}
```

## Injection de dépendances
Vous pouvez injecter des dépendances dans la closure qui définit les permissions. C'est utile si vous avez une sorte de bascule, d'id, ou tout autre point de données que vous voulez vérifier. La même chose fonctionne pour les appels de type Class->Method, sauf que vous définissez les arguments dans la méthode.

### Closures

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... code
});

// dans votre fichier contrôleur
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// faire quelque chose
	} else {
		// faire autre chose
	}
}
```

### Classes

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... code
	}
}
```

## Raccourci pour définir les permissions avec des classes
Vous pouvez également utiliser des classes pour définir vos permissions. C'est utile si vous avez beaucoup de permissions et que vous voulez garder votre code propre. Vous pouvez faire quelque chose comme ceci :
```php
<?php

// code de bootstrap
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// En supposant que vous l'avez configuré au préalable
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // tout le monde peut voir une commande
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // les managers peuvent créer des commandes
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // si l'utilisateur a une bascule spéciale, il peut mettre à jour les commandes
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // les administrateurs peuvent supprimer des commandes
		}
		return $allowed_permissions;
	}
}
```
Le point intéressant est qu'il existe également un raccourci que vous pouvez utiliser (qui peut également être mis en cache !!!) où vous dites simplement à la classe de permissions de mapper toutes les méthodes d'une classe en permissions. Donc si vous avez une méthode nommée `order()` et une méthode nommée `company()`, celles-ci seront automatiquement mappées afin que vous puissiez simplement exécuter `$Permissions->has('order.read')` ou `$Permissions->has('company.read')` et cela fonctionnera. Définir cela est très difficile, alors restez avec moi ici. Vous devez juste faire ceci :

Créez la classe de permissions que vous voulez regrouper.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// code pour déterminer les permissions
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// code pour déterminer les permissions
		return $permissions_array;
	}
}
```

Ensuite, rendez les permissions découvrables en utilisant cette bibliothèque.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Enfin, appelez la permission dans votre codebase pour vérifier si l'utilisateur est autorisé à effectuer une permission donnée.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('Vous ne pouvez pas créer de commande. Désolé !');
		}
	}
}
```

### Mise en cache

Pour activer la mise en cache, consultez la simple bibliothèque [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Un exemple d'activation est ci-dessous.
```php

// cette $app peut faire partie de votre code, ou
// vous pouvez simplement passer null et elle
// récupérera depuis Flight::app() dans le constructeur
$app = Flight::app();

// Pour l'instant, elle accepte ceci comme un cache de fichiers. D'autres peuvent facilement
// être ajoutés à l'avenir. 
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 est le nombre de secondes pour mettre ceci en cache. Laissez vide pour ne pas utiliser la mise en cache
```

Et c'est parti !