# FlightPHP/Permissions

Это модуль прав доступа, который можно использовать в ваших проектах, если у вас есть несколько ролей в приложении, и каждая роль имеет немного разные функциональные возможности. Этот модуль позволяет определить права доступа для каждой роли, а затем проверить, имеет ли текущий пользователь разрешение на доступ к определенной странице или выполнение определенного действия.

Нажмите [здесь](https://github.com/flightphp/permissions), чтобы перейти к репозиторию на GitHub.

Установка
-------
Выполните `composer require flightphp/permissions` и вы готовы к работе!

Использование
-------
Сначала вам нужно настроить ваши права доступа, затем вы сообщаете вашему приложению, что означают эти права доступа. В конечном итоге вы будете проверять ваши права доступа с помощью `$Permissions->has()`, `->can()` или `is()`. `has()` и `can()` имеют одинаковую функциональность, но называются по-разному, чтобы сделать ваш код более читаемым.

## Базовый пример

Предположим, у вас есть функция в вашем приложении, которая проверяет, вошел ли пользователь в систему. Вы можете создать объект прав доступа следующим образом:

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

Затем где-нибудь в контроллере у вас может быть что-то подобное.

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

Вы также можете использовать это для отслеживания, есть ли у них разрешение на выполнение чего-либо в вашем приложении.
Например, если у вас есть способ, которым пользователи могут взаимодействовать с публикациями в вашем программном обеспечении, вы можете
проверить, имеют ли они разрешение на выполнение определенных действий.

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

Затем где-нибудь в контроллере...

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

## Внедрение зависимостей
Вы можете внедрять зависимости в замыкание, которое определяет права доступа. Это полезно, если у вас есть какой-то переключатель, идентификатор или любая другая точка данных, которую вы хотите проверить. То же самое работает для вызовов типа Class->Method, за исключением того, что вы определяете аргументы в методе.

### Замыкания

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

### Классы

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... code
	}
}
```

## Сокращение для установки прав доступа с помощью классов
Вы также можете использовать классы для определения ваших прав доступа. Это полезно, если у вас много прав доступа и вы хотите сохранить чистоту кода. Вы можете сделать что-то подобное:
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
Крутая часть заключается в том, что есть также сокращение, которое вы можете использовать (которое также можно кэшировать!!!), где вы просто сообщаете классу прав доступа сопоставить все методы в классе с правами доступа. Таким образом, если у вас есть метод с именем `order()` и метод с именем `company()`, они будут автоматически сопоставлены, поэтому вы можете просто запустить `$Permissions->has('order.read')` или `$Permissions->has('company.read')`, и это будет работать. Определить это очень сложно, так что оставайтесь со мной здесь. Вам просто нужно сделать это:

Создайте класс прав доступа, который вы хотите сгруппировать вместе.
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

Затем сделайте права доступа обнаруживаемыми с помощью этой библиотеки.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Наконец, вызовите разрешение в вашей кодовой базе, чтобы проверить, разрешено ли пользователю выполнять данное разрешение.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('You can\'t create an order. Sorry!');
		}
	}
}
```

### Кэширование

Чтобы включить кэширование, см. простую библиотеку [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Пример включения этого приведен ниже.
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

И вперед!