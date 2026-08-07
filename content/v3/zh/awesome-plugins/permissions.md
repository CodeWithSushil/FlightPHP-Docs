# FlightPHP/Permissions

这是一个权限模块，如果你的应用中有多个角色且每个角色的功能略有不同，你可以在项目中使用它。这个模块允许你为每个角色定义权限，然后检查当前用户是否有权限访问某个页面或执行某个操作。

点击 [这里](https://github.com/flightphp/permissions) 查看 GitHub 上的仓库。

安装
-------
运行 `composer require flightphp/permissions` 即可开始使用！

用法
-------
首先你需要设置权限，然后告诉你的应用这些权限的含义。最终你将使用 `$Permissions->has()`、`->can()` 或 `is()` 来检查权限。`has()` 和 `can()` 功能相同，但命名不同以使你的代码更具可读性。

## 基础示例

假设你的应用中有一个功能检查用户是否已登录。你可以像这样创建一个权限对象：

```php
// index.php
require 'vendor/autoload.php';

// 一些代码

// 然后你可能有一个告诉当前用户角色的代码
// 通常你会从会话变量中获取当前角色
// 这是用户登录后定义的，否则他们将拥有 'guest' 或 'public' 角色。
$current_role = 'admin';

// 设置权限
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// 你可能希望将这个对象持久化在 Flight 的某个地方
Flight::set('permission', $permission);
```

然后在某个控制器中，你可能有这样的代码。

```php
<?php

// 某个控制器
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// 做一些事情
		} else {
			// 做其他事情
		}
	}
}
```

你也可以用它来跟踪用户是否有权限在应用中执行某些操作。
例如，如果你有用户可以在你的软件上发布内容，你可以
检查他们是否有权限执行某些操作。

```php
$current_role = 'admin';

// 设置权限
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

然后在某个控制器中...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// 做一些事情
		} else {
			// 做其他事情
		}
	}
}
```

## 注入依赖
你可以将依赖注入到定义权限的闭包中。如果你有一些切换、id 或其他你想检查的数据点，这非常有用。Class->Method 类型的调用也适用相同的规则，只是在方法中定义参数。

### 闭包

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... 代码
});

// 在你的控制器文件中
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// 做一些事情
	} else {
		// 做其他事情
	}
}
```

### 类

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... 代码
	}
}
```

## 使用类设置权限的快捷方式
你也可以使用类来定义你的权限。如果你有很多权限且想保持代码整洁，这非常有用。你可以这样做：
```php
<?php

// 引导代码
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// 假设你之前已经设置好了
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // 每个人都可以查看订单
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // 经理可以创建订单
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // 如果用户有特殊切换，他们可以更新订单
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // 管理员可以删除订单
		}
		return $allowed_permissions;
	}
}
```
很酷的部分是还有一个你可以使用的快捷方式（也可以被缓存！！！），你只需告诉权限类将类中的所有方法映射到权限中。所以如果你有一个名为 `order()` 的方法和一个名为 `company()` 的方法，这些将自动映射，这样你就可以运行 `$Permissions->has('order.read')` 或 `$Permissions->has('company.read')`，它就能工作。定义这个很困难，所以请继续关注这里。你只需要这样做：

创建你想要分组的权限类。
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// 确定权限的代码
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// 确定权限的代码
		return $permissions_array;
	}
}
```

然后使用这个库使权限可被发现。

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

最后，在你的代码库中调用权限来检查用户是否被允许执行给定的权限。

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('你不能创建订单。抱歉！');
		}
	}
}
```

### 缓存

要启用缓存，请参阅简单的 [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache) 库。下面是启用它的示例。
```php

// 这个 $app 可以是你的代码的一部分，或者
// 你可以只传递 null，它将在构造函数中
// 从 Flight::app() 中获取
$app = Flight::app();

// 目前它接受文件缓存作为参数。其他缓存可以轻松
// 在未来添加。
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 是缓存的秒数。不使用缓存时可以省略此参数
```

然后就可以开始了！