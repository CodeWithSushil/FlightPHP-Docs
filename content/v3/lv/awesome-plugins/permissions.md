# FlightPHP/Permissions

Šis ir piekļuves tiesību modulis, kuru var izmantot savos projektos, ja jūsu lietotnē ir vairākas lomas un katrai lomai ir nedaudz atšķirīga funkcionalitāte. Šis modulis ļauj definēt piekļuves tiesības katrai lomai un pēc tam pārbaudīt, vai pašreizējam lietotājam ir tiesības piekļūt noteiktai lapai vai veikt noteiktu darbību.

Noklikšķiniet [šeit](https://github.com/flightphp/permissions), lai atvērtu repozitoriju GitHub.

Instalācija
-------
Izpildiet `composer require flightphp/permissions` un esat gatavs!

Lietošana
-------
Vispirms jāizveido piekļuves tiesības, pēc tam jāpastāsta lietotnei, ko tās nozīmē. Galu galā piekļuves tiesības pārbaudīsiet ar `$Permissions->has()`, `->can()` vai `is()`. `has()` un `can()` ir vienāda funkcionalitāte, bet tiem ir atšķirīgi nosaukumi, lai jūsu kods būtu lasāmāks.

## Pamata piemērs

Pieņemsim, ka jūsu lietotnē ir funkcija, kas pārbauda, vai lietotājs ir pieteicies. Jūs varat izveidot piekļuves tiesību objektu šādi:

```php
// index.php
require 'vendor/autoload.php';

// kāds kods 

// tad jūs droši vien kaut ko izmantojat, kas jums pasaka, kāda ir pašreizējā lietotāja loma
// visticamāk, jums ir kaut kas, kas izvelk pašreizējo lomu
// no sesijas mainīgā, kas to definē
// pēc tam, kad kāds piesakās, pretējā gadījumā viņam būs 'guest' vai 'public' loma.
$current_role = 'admin';

// piekļuves tiesību iestatīšana
$permission = new \flight\Permission($current_role);
$permission->defineRule('loggedIn', function($current_role) {
	return $current_role !== 'guest';
});

// Jūs droši vien vēlaties saglabāt šo objektu Flight kaut kur
Flight::set('permission', $permission);
```

Tad kādā kontrolierī varētu būt kaut kas līdzīgs šim.

```php
<?php

// kāds kontrolieris
class SomeController {
	public function someAction() {
		$permission = Flight::get('permission');
		if ($permission->has('loggedIn')) {
			// darīt kaut ko
		} else {
			// darīt kaut ko citu
		}
	}
}
```

To varat izmantot arī, lai izsekotu, vai viņiem ir tiesības kaut ko darīt jūsu lietotnē.
Piemēram, ja jums ir veids, kā lietotāji var mijiedarboties ar ziņām jūsu programmatūrā, varat 
pārbaudīt, vai viņiem ir tiesības veikt noteiktas darbības.

```php
$current_role = 'admin';

// piekļuves tiesību iestatīšana
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

Tad kādā kontrolierī...

```php
class PostController {
	public function create() {
		$permission = Flight::get('permission');
		if ($permission->can('post.create')) {
			// darīt kaut ko
		} else {
			// darīt kaut ko citu
		}
	}
}
```

## Atkarību ievadīšana
Atkarības var ievadīt slēgtā funkcijā, kas definē piekļuves tiesības. Tas ir noderīgi, ja jums ir kāds slēdzis, id vai jebkurš cits datu punkts, pret kuru vēlaties pārbaudīt. Tas pats attiecas uz Class->Method tipa izsaukumiem, izņemot to, ka argumentus definējat metodē.

### Slēgtās funkcijas

```php
$Permission->defineRule('order', function(string $current_role, MyDependency $MyDependency = null) {
	// ... kods
});

// jūsu kontrolierī
public function createOrder() {
	$MyDependency = Flight::myDependency();
	$permission = Flight::get('permission');
	if ($permission->can('order.create', $MyDependency)) {
		// darīt kaut ko
	} else {
		// darīt kaut ko citu
	}
}
```

### Klases

```php
namespace MyApp;

class Permissions {

	public function order(string $current_role, MyDependency $MyDependency = null) {
		// ... kods
	}
}
```

## Īsceļš piekļuves tiesību iestatīšanai ar klasēm
Piekļuves tiesības var definēt arī ar klasēm. Tas ir noderīgi, ja jums ir daudz piekļuves tiesību un vēlaties saglabāt kodu tīru. Varat darīt kaut ko līdzīgu šim:
```php
<?php

// inicializācijas kods
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRule('order', 'MyApp\Permissions->order');

// myapp/Permissions.php
namespace MyApp;

class Permissions {

	public function order(string $current_role, int $user_id) {
		// Pieņemot, ka esat to iestatījis iepriekš
		/** @var \flight\database\SimplePdo $db */
		$db = Flight::db();
		$allowed_permissions = [ 'read' ]; // visi var skatīt pasūtījumu
		if($current_role === 'manager') {
			$allowed_permissions[] = 'create'; // menedžeri var izveidot pasūtījumus
		}
		$some_special_toggle_from_db = $db->fetchField('SELECT some_special_toggle FROM settings WHERE id = ?', [ $user_id ]);
		if($some_special_toggle_from_db) {
			$allowed_permissions[] = 'update'; // ja lietotājam ir speciāls slēdzis, viņš var atjaunināt pasūtījumus
		}
		if($current_role === 'admin') {
			$allowed_permissions[] = 'delete'; // administratori var dzēst pasūtījumus
		}
		return $allowed_permissions;
	}
}
```
Interesantākais ir tas, ka ir arī īsceļš, ko var izmantot (ko var arī kešot!!!), kur vienkārši pasaka piekļuves tiesību klasei, lai tā kartētu visas metodes klasē uz piekļuves tiesībām. Tātad, ja jums ir metode ar nosaukumu `order()` un metode ar nosaukumu `company()`, tās automātiski tiks kartētas, lai jūs varētu vienkārši palaist `$Permissions->has('order.read')` vai `$Permissions->has('company.read')` un tas darbosies. To definēt ir ļoti grūti, tāpēc sekojiet man. Jums vienkārši jādara šādi:

Izveidojiet piekļuves tiesību klasi, ko vēlaties grupēt kopā.
```php
class MyPermissions {
	public function order(string $current_role, int $order_id = 0): array {
		// kods piekļuves tiesību noteikšanai
		return $permissions_array;
	}

	public function company(string $current_role, int $company_id): array {
		// kods piekļuves tiesību noteikšanai
		return $permissions_array;
	}
}
```

Pēc tam padariet piekļuves tiesības atklājamas, izmantojot šo bibliotēku.

```php
$Permissions = new \flight\Permission($current_role);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class);
Flight::set('permissions', $Permissions);
```

Visbeidzot, izsauciet piekļuves tiesības savā kodā, lai pārbaudītu, vai lietotājam ir atļauts veikt noteiktu darbību.

```php
class SomeController {
	public function createOrder() {
		if(Flight::get('permissions')->can('order.create') === false) {
			die('You can\'t create an order. Sorry!');
		}
	}
}
```

### Kešatmiņa

Lai iespējotu kešatmiņu, skatiet vienkāršo [wruczak/phpfilecache](https://docs.flightphp.com/awesome-plugins/php-file-cache) bibliotēku. Zemāk ir piemērs tās iespējotai.
```php

// šis $app var būt daļa no jūsu koda vai
// varat vienkārši nodot null un tas
// iegūs no Flight::app() konstruktorā
$app = Flight::app();

// Pašlaik tas pieņem failu kešatmiņu. Citi var viegli
// tikt pievienoti nākotnē. 
$Cache = new Wruczek\PhpFileCache\PhpFileCache;

$Permissions = new \flight\Permission($current_role, $app, $Cache);
$Permissions->defineRulesFromClassMethods(MyApp\Permissions::class, 3600); // 3600 ir, cik sekundes to kešot. Atstājiet to tukšu, lai neizmantotu kešatmiņu
```

Un uz priekšu!