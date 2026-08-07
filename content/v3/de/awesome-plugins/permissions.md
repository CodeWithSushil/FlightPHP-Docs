# FlightPHP/Permissions

Dies ist ein Berechtigungsmodul, das in Ihren Projekten verwendet werden kann, wenn Sie mehrere Rollen in Ihrer App haben und jede Rolle eine etwas andere Funktionalität hat. Dieses Modul ermöglicht es Ihnen, Berechtigungen für jede Rolle zu definieren und dann zu überprüfen, ob der aktuelle Benutzer die Berechtigung hat, auf eine bestimmte Seite zuzugreifen oder eine bestimmte Aktion auszuführen.

Klicken Sie [hier](https://github.com/flightphp/permissions) für das Repository auf GitHub.

Installation
-------
Führen Sie `composer require flightphp/permissions` aus und schon sind Sie auf dem Weg!

Usage
-------
Zuerst müssen Sie Ihre Berechtigungen einrichten, dann teilen Sie Ihrer App mit, was die Berechtigungen bedeuten. Letztendlich überprüfen Sie Ihre Berechtigungen mit `$Permissions->has()`, `->can()`, oder `is()`. `has()` und `can()` haben die gleiche Funktionalität, sind aber unterschiedlich benannt, um Ihren Code lesbarer zu machen.

## Basic Example

Nehmen wir an, Sie haben eine Funktion in Ihrer Anwendung, die überprüft, ob ein Benutzer angemeldet ist. Sie können ein Berechtigungsobjekt wie folgt erstellen:

```php
// index.php
require 'vendor/autoload.php';

// some code 

// then you probably have something that tells you who the current role is of the person
// likely you have something where you pull the current role
// from a session variable which defines this
// after someone logs in, otherwise they will have a 'guest' or 'public' role.
$current_role = 'admin';

// setup permissions
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// You'll probably want to persist this object in Flight somewhere
Flight::set('permission', $permission);
```

Dann haben Sie irgendwo in einem Controller so etwas.

```php
<?php

// some controller
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// do something
		} else {
			// do something else
		}
	}
}
```

Sie können dies auch verwenden, um zu verfolgen, ob sie die Berechtigung haben, etwas in Ihrer Anwendung zu tun.
Wenn Sie beispielsweise eine Möglichkeit haben, dass Benutzer mit dem Posten in Ihrer Software interagieren können, können Sie 
überprüfen, ob sie die Berechtigung haben, bestimmte Aktionen auszuführen.

```php
$current_role = 'admin';

// setup permissions
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

Dann irgendwo in einem Controller...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// do something
		} else {
			// do something else
		}
	}
}
```

## Injecting dependencies
Sie können Abhängigkeiten in die Closure injizieren, die die Berechtigungen definiert. Dies ist nützlich, wenn Sie eine Art Umschalter, ID oder einen anderen Datenpunkt haben, gegen den Sie überprüfen möchten. Das Gleiche funktioniert auch für Class->Method-Aufrufe, außer dass Sie die Argumente in der Methode definieren.

### Closures

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... code
});

// in your controller file
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// do something
	} else {
		// do something else
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

## Shortcut to set permissions with classes
Sie können auch Klassen verwenden, um Ihre Berechtigungen zu definieren. Dies ist nützlich, wenn Sie viele Berechtigungen haben und Ihren Code sauber halten möchten. Sie können so etwas tun:
```php
<?php

// bootstrap code
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// Assuming you set this up beforehand
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // everyone can view an order
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // managers can create orders
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // if the user has a special toggle, they can update orders
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // admins can delete orders
		}
		return $allowed_permissions;
	}
}
```
Das Coole daran ist, dass es auch eine Abkürzung gibt, die Sie verwenden können (die auch zwischengespeichert werden kann!!!), bei der Sie der Berechtigungsklasse einfach sagen, alle Methoden in einer Klasse in Berechtigungen zu mappen. Wenn Sie also eine Methode namens `order()` und eine Methode namens `company()` haben, werden diese automatisch gemappt, sodass Sie einfach `$Permissions->has('order.read')` oder `$Permissions->has('company.read')` ausführen können und es funktioniert. Das Definieren ist sehr schwierig, also bleiben Sie bei mir hier. Sie müssen nur Folgendes tun:

Erstellen Sie die Klasse von Berechtigungen, die Sie gruppieren möchten.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// code to determine permissions
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// code to determine permissions
		return $permissions_array;
	}
}
```

Dann machen Sie die Berechtigungen mit dieser Bibliothek erkennbar.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Schließlich rufen Sie die Berechtigung in Ihrer Codebasis auf, um zu überprüfen, ob der Benutzer berechtigt ist, eine bestimmte Berechtigung auszuführen.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('You can\'t create an order. Sorry!');
		}
	}
}
```

### Caching

Um das Caching zu aktivieren, sehen Sie sich die einfache [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache) Bibliothek an. Ein Beispiel für die Aktivierung finden Sie unten.
```php

// this $app can be part of your code, or
// you can just pass null and it will
// pull from Flight::app() in the constructor
$app = Flight::app();

// For now it accepts this as a file cache. Others can easily
// be added in the future. 
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 is how many seconds to cache this for. Leave this off to not use caching
```

Und schon geht's los!