# FlightPHP/Permissions

Ini adalah modul izin yang dapat digunakan dalam proyek Anda jika Anda memiliki beberapa peran dalam aplikasi Anda dan setiap peran memiliki fungsionalitas yang sedikit berbeda. Modul ini memungkinkan Anda untuk mendefinisikan izin untuk setiap peran dan kemudian memeriksa apakah pengguna saat ini memiliki izin untuk mengakses halaman tertentu atau melakukan tindakan tertentu.

Klik [di sini](https://github.com/flightphp/permissions) untuk repositori di GitHub.

Instalasi
-------
Jalankan `composer require flightphp/permissions` dan Anda siap!

Penggunaan
-------
Pertama Anda perlu menyiapkan izin Anda, kemudian Anda memberitahu aplikasi Anda apa arti izin tersebut. Pada akhirnya Anda akan memeriksa izin Anda dengan `$Permissions->has()`, `->can()`, atau `is()`. `has()` dan `can()` memiliki fungsionalitas yang sama, tetapi dinamai berbeda untuk membuat kode Anda lebih mudah dibaca.

## Contoh Dasar

Mari kita asumsikan Anda memiliki fitur dalam aplikasi Anda yang memeriksa apakah pengguna sudah masuk. Anda dapat membuat objek izin seperti ini:

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

Kemudian di suatu pengontrol, Anda mungkin memiliki sesuatu seperti ini.

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

Anda juga dapat menggunakan ini untuk melacak apakah mereka memiliki izin untuk melakukan sesuatu dalam aplikasi Anda.
Misalnya, jika Anda memiliki cara bagi pengguna untuk berinteraksi dengan posting di perangkat lunak Anda, Anda dapat 
memeriksa apakah mereka memiliki izin untuk melakukan tindakan tertentu.

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

Kemudian di suatu pengontrol...

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

## Menyuntikkan dependensi
Anda dapat menyuntikkan dependensi ke dalam closure yang mendefinisikan izin. Ini berguna jika Anda memiliki semacam toggle, id, atau titik data lain yang ingin Anda periksa. Hal yang sama berlaku untuk panggilan tipe Class->Method, kecuali Anda mendefinisikan argumen dalam metode.

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

## Pintasan untuk mengatur izin dengan kelas
Anda juga dapat menggunakan kelas untuk mendefinisikan izin Anda. Ini berguna jika Anda memiliki banyak izin dan ingin menjaga kode Anda tetap bersih. Anda dapat melakukan sesuatu seperti ini:
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
Hal yang keren adalah ada juga pintasan yang dapat Anda gunakan (yang juga dapat di-cache!!!) di mana Anda hanya memberitahu kelas izin untuk memetakan semua metode dalam sebuah kelas menjadi izin. Jadi jika Anda memiliki metode bernama `order()` dan metode bernama `company()`, ini akan secara otomatis dipetakan sehingga Anda hanya dapat menjalankan `$Permissions->has('order.read')` atau `$Permissions->has('company.read')` dan itu akan berhasil. Mendefinisikan ini sangat rumit, jadi ikuti saya di sini. Anda hanya perlu melakukan ini:

Buat kelas izin yang ingin Anda kelompokkan bersama.
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

Kemudian buat izin dapat ditemukan menggunakan library ini.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Akhirnya, panggil izin dalam basis kode Anda untuk memeriksa apakah pengguna diizinkan untuk melakukan izin tertentu.

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

Untuk mengaktifkan caching, lihat library sederhana [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache). Contoh mengaktifkan ini di bawah ini.
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

Dan Anda siap!