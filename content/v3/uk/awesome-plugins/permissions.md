# FlightPHP/Дозволи

Це модуль дозволів, який можна використовувати у ваших проєктах, якщо у вас є кілька ролей у вашому додатку, і кожна роль має дещо різну функціональність. Цей модуль дозволяє вам визначати дозволи для кожної ролі, а потім перевіряти, чи має поточний користувач дозвіл на доступ до певної сторінки чи виконання певної дії.

Натисніть [тут](https://github.com/flightphp/permissions) для репозиторію на GitHub.

Встановлення
-------
Запустіть `composer require flightphp/permissions` і ви на шляху!

Використання
-------
Спочатку вам потрібно налаштувати ваші дозволи, потім ви вказуєте вашому додатку, що означають ці дозволи. Зрештою ви будете перевіряти ваші дозволи за допомогою `$Permissions->has()`, `->can()` або `is()`. `has()` і `can()` мають однакову функціональність, але називаються по-різному, щоб зробити ваш код більш читабельним.

## Базовий приклад

Припустимо, у вас є функція у вашому додатку, яка перевіряє, чи користувач увійшов у систему. Ви можете створити об'єкт дозволів так:

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

Потім у контролері десь може бути щось таке.

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

Ви також можете використовувати це для відстеження, чи мають вони дозвіл на щось робити у вашому додатку.
Наприклад, якщо у вас є спосіб, яким користувачі можуть взаємодіяти з публікаціями у вашому програмному забезпеченні, ви можете 
перевірити, чи мають вони дозвіл на виконання певних дій.

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

Потім у контролері десь...

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

## Ін'єкція залежностей
Ви можете ін'єктувати залежності у замикання, яке визначає дозволи. Це корисно, якщо у вас є якийсь перемикач, ідентифікатор або будь-яка інша точка даних, яку ви хочете перевірити. Те саме працює для викликів типу Class->Method, за винятком того, що ви визначаєте аргументи у методі.

### Замикання

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

### Класи

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... code
	}
}
```

## Ярлик для встановлення дозволів за допомогою класів
Ви також можете використовувати класи для визначення ваших дозволів. Це корисно, якщо у вас є багато дозволів і ви хочете зберегти ваш код чистим. Ви можете зробити щось таке:
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
Крута частина полягає в тому, що є також ярлик, який ви можете використовувати (який також можна кешувати!!!), де ви просто вказуєте класу дозволів зіставити всі методи у класі з дозволами. Тож якщо у вас є метод з назвою `order()` і метод з назвою `company()`, вони автоматично будуть зіставлені, тож ви можете просто запустити `$Permissions->has('order.read')` або `$Permissions->has('company.read')`, і це спрацює. Визначити це дуже складно, тож тримайтеся за мене. Вам просто потрібно зробити це:

Створіть клас дозволів, який ви хочете згрупувати разом.
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

Потім зробіть дозволи доступними для виявлення за допомогою цієї бібліотеки.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Нарешті, викличте дозвіл у вашій кодовій базі, щоб перевірити, чи користувачу дозволено виконувати даний дозвіл.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('You can\'t create an order. Sorry!');
		}
	}
}
```

### Кешування

Щоб увімкнути кешування, дивіться просту бібліотеку [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Приклад увімкнення цього наведено нижче.
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

І вперед!