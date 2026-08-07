# Kontainer Injeksi Dependensi

## Ikhtisar

Kontainer Injeksi Dependensi (DIC) adalah peningkatan yang ampuh yang memungkinkan Anda mengelola
dependensi aplikasi Anda. Ini juga merupakan salah satu alasan terbesar mengapa Flight dapat bekerja dengan baik dengan [alat coding AI](/learn/ai) dan pengujian unit: kontroler mengambil apa yang mereka butuhkan di konstruktor alih-alih mengakses global.

## Pemahaman

Injeksi Dependensi (DI) adalah konsep kunci dalam kerangka kerja PHP modern dan digunakan untuk mengelola instansiasi dan konfigurasi objek. Beberapa contoh pustaka DIC adalah: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/), [PHP-DI](http://php-di.org/), dan [league/container](https://container.thephpleague.com/).

DIC adalah cara yang canggih untuk membuat dan mengelola kelas Anda di satu lokasi terpusat. Ini berguna ketika Anda perlu mengirim objek yang sama ke beberapa kelas (kontroler, middleware, perintah, dan sebagainya).

[skeleton flightphp/skeleton](https://github.com/flightphp/skeleton) resmi menghubungkan **Dice** di `app/config/services.php`, mengganti instance `flight\Engine` yang digunakan bersama, dan menyelesaikan target rute seperti `[App\Controller\HomeController::class, 'index']`. Prefer pola itu untuk proyek baru sehingga manusia dan agen mengedit tempat yang sama.

## Penggunaan Dasar

Cara lama dalam melakukan sesuatu mungkin terlihat seperti ini:
```php

require 'vendor/autoload.php';

// kelas untuk mengelola pengguna dari database
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// di file routes.php Anda

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// rute UserController lainnya...

Flight::start();
```

Anda dapat melihat dari kode di atas bahwa kita membuat objek `PDO` baru dan mengirimkannya ke kelas `UserController` kita. Ini baik-baik saja untuk aplikasi kecil, tetapi seiring aplikasi Anda berkembang, Anda akan menemukan bahwa Anda membuat atau mengirimkan objek `PDO` yang sama di banyak tempat. Di sinilah DIC menjadi berguna.

Berikut adalah contoh yang sama menggunakan DIC (menggunakan Dice):
```php

require 'vendor/autoload.php';

// kelas yang sama seperti di atas. Tidak ada yang berubah
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// buat kontainer baru
$container = new \Dice\Dice;

// tambahkan aturan untuk memberi tahu kontainer cara membuat objek PDO
// jangan lupa untuk menetapkannya kembali ke dirinya sendiri seperti di bawah ini!
$container = $container->addRule('PDO', [
	// shared berarti objek yang sama akan dikembalikan setiap kali
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Ini mendaftarkan penangan kontainer sehingga Flight tahu untuk menggunakannya.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// sekarang kita dapat menggunakan kontainer untuk membuat UserController kita
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Saya yakin Anda mungkin berpikir bahwa ada banyak kode tambahan yang ditambahkan ke contoh ini.
Keajaibannya muncul ketika Anda memiliki kontroler lain yang membutuhkan objek `PDO`.

```php

// Jika semua kontroler Anda memiliki konstruktor yang membutuhkan objek PDO
// setiap rute di bawah ini secara otomatis akan mendapatkannya diinjeksikan!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

Bonus tambahan dari menggunakan DIC adalah pengujian unit menjadi jauh lebih mudah. Anda dapat
membuat objek mock dan mengirimkannya ke kelas Anda. Ini adalah manfaat besar saat Anda
menulis tes untuk aplikasi Anda—dan ketika asisten AI membuat kontroler, injeksi konstruktor memberinya pola yang jelas dan konsisten untuk diikuti ([panduan pengujian unit](/guides/unit-testing)).

### Membuat penangan DIC terpusat

Anda dapat membuat penangan DIC terpusat di file layanan Anda dengan [memperluas](/learn/extending) aplikasi Anda. Berikut adalah contohnya:

```php
// services.php

// buat kontainer baru
$container = new \Dice\Dice;
// jangan lupa untuk menetapkannya kembali ke dirinya sendiri seperti di bawah ini!
$container = $container->addRule('PDO', [
	// shared berarti objek yang sama akan dikembalikan setiap kali
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// sekarang kita dapat membuat metode yang dapat dipetakan untuk membuat objek apa pun. 
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Ini mendaftarkan penangan kontainer sehingga Flight tahu untuk menggunakannya untuk kontroler/middleware
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// katakanlah kita memiliki kelas contoh berikut yang mengambil objek PDO di konstruktor
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// kode yang mengirim email
	}
}

// Dan akhirnya Anda dapat membuat objek menggunakan injeksi dependensi
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight memiliki plugin yang menyediakan kontainer sederhana yang sesuai dengan PSR-11 yang dapat Anda gunakan untuk menangani injeksi dependensi Anda. Berikut adalah contoh cepat cara menggunakannya:

```php

// index.php misalnya
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// akan menampilkan ini dengan benar!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Penggunaan Lanjutan flightphp/container

Anda juga dapat menyelesaikan dependensi secara rekursif. Berikut adalah contohnya:

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Implementasi ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

Anda juga dapat membuat penangan DIC sendiri. Ini berguna jika Anda memiliki kontainer khusus yang ingin Anda gunakan yang tidak sesuai dengan PSR-11 (Dice). Lihat bagian [penggunaan dasar](#penggunaan-dasar) untuk cara melakukannya.

Selain itu, ada beberapa default yang membantu yang akan membuat hidup Anda lebih mudah saat menggunakan Flight.

#### Instance Engine (diperlukan untuk injeksi `$app`)

Jika Anda mengetik-hint `flight\Engine` pada kontroler atau middleware, **Dice tidak boleh membuat Engine baru**. Ganti dengan instance yang sama dari bootstrap. Inilah yang dilakukan skeleton resmi, dan ini adalah pola yang diharapkan `AGENTS.md` untuk kontroler yang dihasilkan AI:

```php
// Di suatu tempat di bootstrap / services.php Anda
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // atau $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Penting: gunakan kembali Engine yang sudah di-bootstrap — jangan biarkan Dice `new Engine()`
		Engine::class => $app,
		// Prefer SimplePdo untuk kode baru
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Helper opsional untuk kode non-rute
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (tata letak skeleton — huruf besar/kecil folder cocok dengan namespace)
namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// Tidak ada fasad Flight:: di lapisan aplikasi — lebih mudah diuji dan lebih jelas untuk alat AI
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Jika Anda melewatkan substitusi `Engine`, Dice dapat membuat Engine kedua dan kontroler Anda tidak akan berbagi rute, konfigurasi, atau `render` Twig yang dipetakan dari bootstrap.

#### Menambahkan layanan bersama lainnya (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Setelah Anda membuat $db, $config, $twig di services.php:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

Kemudian kontroler dapat mengambil `SimplePdo $db` (atau tipe konfigurasi Anda) di konstruktor dan tidak pernah memanggil `Flight::db()`. Itu sesuai dengan panduan [pengujian unit](/guides/unit-testing) dan gaya rumah skeleton.

#### Menambahkan kelas lainnya

Jika Anda memiliki kelas lain yang ingin ditambahkan ke kontainer, dengan Dice itu mudah karena mereka akan diselesaikan secara otomatis oleh kontainer. Berikut adalah contohnya:

```php

$container = new \Dice\Dice;
// Jika Anda tidak perlu menginjeksikan dependensi apa pun ke dalam kelas Anda
// Anda tidak perlu mendefinisikan apa pun!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flight juga dapat menggunakan kontainer apa pun yang sesuai dengan PSR-11. Ini berarti Anda dapat menggunakan kontainer apa pun yang mengimplementasikan antarmuka PSR-11. Berikut adalah contoh menggunakan kontainer PSR-11 dari League:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// Ide UserController yang sama seperti di atas, ketik-hint SimplePdo alih-alih PDO mentah

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

Ini bisa sedikit lebih panjang daripada contoh Dice sebelumnya, tetapi tetap menyelesaikan pekerjaan dengan manfaat yang sama!

## Lihat Juga
- [Instalasi](/install) - Tata letak skeleton dan di mana `services.php` berada.
- [Autoloading](/learn/autoloading) - Namespace `App\` dan **huruf besar/kecil** folder.
- [Memperluas Flight](/learn/extending) - Pelajari cara menambahkan injeksi dependensi ke kelas Anda sendiri dengan memperluas kerangka kerja.
- [Konfigurasi](/learn/configuration) - Pelajari cara mengonfigurasi Flight untuk aplikasi Anda.
- [Routing](/learn/routing) - Pelajari cara mendefinisikan rute untuk aplikasi Anda dan cara kerja injeksi dependensi dengan kontroler.
- [Middleware](/learn/middleware) - Pelajari cara membuat middleware untuk aplikasi Anda dan cara kerja injeksi dependensi dengan middleware.
- [Pengujian Unit](/guides/unit-testing) - Mengapa injeksi konstruktor lebih baik daripada global `Flight::`.
- [AI & Pengalaman Pengembang](/learn/ai) - Satu pola DI untuk manusia dan agen.
- [SimplePdo](/learn/simple-pdo) - Helper basis data yang disarankan untuk injeksi.

## Pemecahan Masalah
- Jika Anda mengalami masalah dengan kontainer Anda, pastikan Anda memberikan nama kelas yang benar ke kontainer.
- Kontroler yang ketik-hint `Engine` tetapi mendapatkan aplikasi "kosong": tambahkan **substitusi Engine** (lihat di atas). Dice tidak boleh `new` Engine kedua.
- Kelas tidak ditemukan untuk `App\Controller\…`: periksa huruf besar/kecil folder di bawah `app/Controller/` — lihat [Autoloading](/learn/autoloading).
- Penangan harus **mengembalikan** objek yang dibuat dari `registerContainerHandler` (jangan memanggil `Flight::make()` tanpa `return`).

## Log Perubahan
- Docs – Dokumentasikan skeleton Dice + substitusi Engine, SimplePdo, dan tata letak `App\Controller` untuk proyek yang ramah AI.
- v3.7.0 - Menambahkan kemampuan untuk mendaftarkan penangan DIC ke Flight.