# FlightPHP/Permissions

이 모듈은 앱에 여러 역할이 있고 각 역할마다 약간 다른 기능이 있는 경우 프로젝트에서 사용할 수 있는 권한 모듈입니다. 이 모듈을 사용하면 각 역할에 대한 권한을 정의한 다음 현재 사용자가 특정 페이지에 액세스하거나 특정 작업을 수행할 수 있는 권한이 있는지 확인할 수 있습니다.

GitHub의 저장소는 [여기](https://github.com/flightphp/permissions)를 클릭하세요.

설치
-------
`composer require flightphp/permissions`를 실행하면 됩니다!

사용법
-------
먼저 권한을 설정한 다음, 앱에 권한이 무엇을 의미하는지 알려야 합니다. 궁극적으로 `$Permissions->has()`, `->can()`, `is()`를 사용하여 권한을 확인하게 됩니다. `has()`와 `can()`은 동일한 기능을 가지지만, 코드의 가독성을 높이기 위해 다르게 이름이 지정되었습니다.

## 기본 예제

애플리케이션에 사용자가 로그인했는지 확인하는 기능이 있다고 가정해 보겠습니다. 다음과 같이 권한 객체를 만들 수 있습니다:

```php
// index.php
require 'vendor/autoload.php';

// 일부 코드 

// 그런 다음 현재 사용자의 역할이 무엇인지 알려주는 코드가 있을 것입니다.
// 세션 변수에서 현재 역할을 가져오는 코드가 있을 것입니다.
// 로그인 후에 정의되며, 그렇지 않으면 'guest' 또는 'public' 역할을 갖게 됩니다.
$current_role = 'admin';

// 권한 설정
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// 이 객체를 Flight의 어딘가에 저장하고 싶을 것입니다
Flight::set('permission', $permission);
```

그런 다음 컨트롤러의 어딘가에서 다음과 같은 코드를 작성할 수 있습니다.

```php
<?php

// 일부 컨트롤러
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// 무언가를 수행
		} else {
			// 다른 작업 수행
		}
	}
}
```

또한 애플리케이션에서 사용자가 무언가를 할 수 있는 권한이 있는지 추적하는 데 사용할 수 있습니다.
예를 들어, 사용자가 소프트웨어에 게시물을 올릴 수 있는 방법이 있다면,
특정 작업을 수행할 수 있는 권한이 있는지 확인할 수 있습니다.

```php
$current_role = 'admin';

// 권한 설정
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

그런 다음 컨트롤러의 어딘가에서...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// 무언가를 수행
		} else {
			// 다른 작업 수행
		}
	}
}
```

## 의존성 주입
권한을 정의하는 클로저에 의존성을 주입할 수 있습니다. 이는 확인하려는 토글, ID 또는 기타 데이터 포인트가 있는 경우 유용합니다. Class->Method 유형 호출에서도 동일하게 작동하지만, 인수를 메서드에 정의합니다.

### 클로저

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... 코드
});

// 컨트롤러 파일에서
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// 무언가를 수행
	} else {
		// 다른 작업 수행
	}
}
```

### 클래스

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... 코드
	}
}
```

## 클래스로 권한 설정하기 위한 단축키
클래스를 사용하여 권한을 정의할 수도 있습니다. 권한이 많고 코드를 깔끔하게 유지하려는 경우 유용합니다. 다음과 같이 할 수 있습니다:
```php
<?php

// 부트스트랩 코드
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// 미리 설정했다고 가정
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // 모든 사용자가 주문을 볼 수 있음
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // 관리자는 주문을 생성할 수 있음
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // 사용자가 특별한 토글이 있는 경우 주문을 업데이트할 수 있음
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // 관리자는 주문을 삭제할 수 있음
		}
		return $allowed_permissions;
	}
}
```
멋진 부분은 (캐시도 가능!!!) 단축키를 사용할 수 있다는 것입니다. 권한 클래스에 클래스 내의 모든 메서드를 권한으로 매핑하도록 지시하기만 하면 됩니다. `order()`라는 메서드와 `company()`라는 메서드가 있는 경우, 자동으로 매핑되므로 `$Permissions->has('order.read')` 또는 `$Permissions->has('company.read')`를 실행하면 작동합니다. 정의하기는 매우 까다로우므로 잘 따라오세요. 다음을 수행하기만 하면 됩니다:

그룹화하려는 권한 클래스를 만듭니다.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// 권한을 결정하는 코드
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// 권한을 결정하는 코드
		return $permissions_array;
	}
}
```

그런 다음 이 라이브러리를 사용하여 권한을 검색 가능하게 만듭니다.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

마지막으로, 코드베이스에서 권한을 호출하여 사용자가 주어진 권한을 수행할 수 있는지 확인합니다.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('주문을 생성할 수 없습니다. 죄송합니다!');
		}
	}
}
```

### 캐싱

캐싱을 활성화하려면 간단한 [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache) 라이브러리를 참조하세요. 활성화하는 예는 아래에 나와 있습니다.
```php

// 이 $app은 코드의 일부이거나,
// null을 전달하면 생성자에서 Flight::app()에서 가져옵니다.
$app = Flight::app();

// 현재로서는 파일 캐시로 이를 수락합니다. 다른 것은 쉽게
// 미래에 추가할 수 있습니다. 
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 캐싱할 시간(초)입니다. 캐싱을 사용하지 않으려면 이 값을 생략하세요.
```

이제 시작하세요!