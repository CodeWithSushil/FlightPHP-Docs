# FlightPHP/Permisos

Este es un módulo de permisos que se puede usar en sus proyectos si tiene múltiples roles en su aplicación y cada rol tiene una funcionalidad un poco diferente. Este módulo le permite definir permisos para cada rol y luego verificar si el usuario actual tiene permiso para acceder a una determinada página o realizar una determinada acción.

Haga clic [here](https://github.com/flightphp/permissions) para ver el repositorio en GitHub.

Instalación
-------
Ejecute `composer require flightphp/permissions` ¡y listo!

Uso
-------
Primero necesita configurar sus permisos, luego le dice a su aplicación lo que significan los permisos. En última instancia, verificará sus permisos con `$Permissions->has()`, `->can()`, o `is()`. `has()` y `can()` tienen la misma funcionalidad, pero se nombran de manera diferente para hacer su código más legible.

## Ejemplo Básico

Supongamos que tiene una característica en su aplicación que verifica si un usuario ha iniciado sesión. Puede crear un objeto de permisos como este:

```php
// index.php
require 'vendor/autoload.php';

// algún código

// luego probablemente tenga algo que le diga cuál es el rol actual de la persona
// probablemente tenga algo donde extraiga el rol actual
// de una variable de sesión que define esto
// después de que alguien inicie sesión, de lo contrario tendrán un rol 'guest' o 'public'.
$current_role = 'admin';

// configurar permisos
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// Probablemente querrá persistir este objeto en Flight en algún lugar
Flight::set('permission', $permission);
```

Luego en algún controlador, podría tener algo como esto.

```php
<?php

// algún controlador
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// hacer algo
		} else {
			// hacer algo más
		}
	}
}
```

También puede usar esto para rastrear si tienen permiso para hacer algo en su aplicación.
Por ejemplo, si tiene una forma en que los usuarios pueden interactuar con publicaciones en su software, puede
verificar si tienen permiso para realizar ciertas acciones.

```php
$current_role = 'admin';

// configurar permisos
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

Luego en algún controlador...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// hacer algo
		} else {
			// hacer algo más
		}
	}
}
```

## Inyectando dependencias
Puede inyectar dependencias en el cierre que define los permisos. Esto es útil si tiene algún tipo de interruptor, id, o cualquier otro punto de datos que desee verificar. Lo mismo funciona para llamadas de tipo Clase->Método, excepto que define los argumentos en el método.

### Cierres

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... código
});

// en su archivo de controlador
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// hacer algo
	} else {
		// hacer algo más
	}
}
```

### Clases

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... código
	}
}
```

## Atajo para establecer permisos con clases
También puede usar clases para definir sus permisos. Esto es útil si tiene muchos permisos y desea mantener su código limpio. Puede hacer algo como esto:
```php
<?php

// código de bootstrap
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// Asumiendo que configuró esto de antemano
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // todos pueden ver un pedido
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // los gerentes pueden crear pedidos
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // si el usuario tiene un interruptor especial, puede actualizar pedidos
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // los administradores pueden eliminar pedidos
		}
		return $allowed_permissions;
	}
}
```
La parte genial es que también hay un atajo que puede usar (¡que también se puede cachear!) donde simplemente le dice a la clase de permisos que mapee todos los métodos en una clase a permisos. Entonces, si tiene un método llamado `order()` y un método llamado `company()`, estos se mapearán automáticamente para que pueda simplemente ejecutar `$Permissions->has('order.read')` o `$Permissions->has('company.read')` y funcionará. Definir esto es muy difícil, así que quédese conmigo. Solo necesita hacer esto:

Cree la clase de permisos que desea agrupar.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// código para determinar permisos
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// código para determinar permisos
		return $permissions_array;
	}
}
```

Luego haga que los permisos sean descubribles usando esta biblioteca.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Finalmente, llame al permiso en su base de código para verificar si el usuario tiene permitido realizar un permiso dado.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('You can\'t create an order. Sorry!');
		}
	}
}
```

### Caché

Para habilitar el caché, vea la simple biblioteca [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Un ejemplo de cómo habilitar esto está a continuación.
```php

// este $app puede ser parte de su código, o
// puede simplemente pasar null y
// extraerá de Flight::app() en el constructor
$app = Flight::app();

// Por ahora acepta esto como un caché de archivos. Otros pueden
// agregarse fácilmente en el futuro.
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 es cuántos segundos cachear esto. Déjelo en blanco para no usar caché
```

¡Y listo!