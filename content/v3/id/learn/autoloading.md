# Autoloading

## Ringkasan

Autoloading adalah konsep dalam PHP di mana Anda menentukan direktori atau direktori-direktori untuk memuat kelas. Ini jauh lebih bermanfaat daripada menggunakan `require` atau `include` untuk memuat kelas. Ini juga merupakan persyaratan untuk menggunakan paket Composer.

Membuat autoloading benar itu penting untuk [pengembangan berbantuan AI](/learn/ai) juga: agen menempatkan file di tempat namespace menunjuk. Jika **huruf besar/kecil** folder dan namespace tidak sesuai, kesalahan class-not-found akan muncul di Linux bahkan ketika hal-hal "berfungsi" di disk Mac yang tidak peka huruf besar/kecil.

## Memahami

Secara default, kelas `Flight` apa pun di-autoload secara otomatis berkat Composer. Untuk kelas aplikasi **Anda**, ada dua pendekatan umum:

1. **Composer PSR-4** (yang digunakan oleh [skeleton resmi](https://github.com/flightphp/skeleton)): memetakan prefiks namespace ke direktori di `composer.json`, lalu `composer dump-autoload`.
2. **`Flight::path()`**: mengarahkan loader Flight ke direktori-direktori (berguna untuk aplikasi sederhana atau saat Anda tidak menggunakan Composer untuk kode aplikasi).

Menggunakan autoloader menyederhanakan kode Anda banyak. Alih-alih dinding `include` / `require` di bagian atas setiap file, kelas dimuat saat pertama kali Anda menggunakannya.

### Sensitivitas huruf besar/kecil (baca ini dua kali)

**Namespace harus cocok dengan struktur direktori *dan* huruf besar/kecil direktori tersebut.**

| Berfungsi | Rusak di Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` dengan folder `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Mencampur `App\` dengan `controllers` huruf kecil |

Namespace PHP tidak peka huruf besar/kecil dalam beberapa konteks, tetapi **Composer dan sistem file tidak**. Skeleton resmi menstandarkan pada:

- Composer: `"App\\": "app/"`
- Folder: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), bukan `controllers` / `middlewares`

Dokumen lama dan contoh komunitas terkadang menggunakan `app\controllers` huruf kecil. Itu masih berfungsi jika folder Anda huruf kecil—tetapi **proyek skeleton baru menggunakan `App\` + folder PascalCase**. Pilih satu konvensi per proyek dan patuhi agar manusia dan alat AI tidak menemukan tata letak kedua.

## Skeleton (direkomendasikan untuk proyek baru)

Setelah `composer create-project flightphp/skeleton`, kode aplikasi di-autoload melalui Composer—tidak perlu `Flight::path()` untuk kelas `App\`:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Dice menyelesaikan App\Controller\… melalui container
$router->get('/', [HomeController::class, 'index']);
```

Lihat [Instalasi](/install) untuk pohon lengkap dan [AI & pengalaman pengembang](/learn/ai) untuk cara `AGENTS.md` mendokumentasikan tata letak ini untuk asisten pengkodean.

## Penggunaan Dasar (`Flight::path()`)

Mari kita asumsikan kita memiliki pohon direktori seperti berikut:

```text
# Contoh path
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - berisi controller untuk proyek ini
│   ├── translations
│   ├── UTILS - berisi kelas untuk aplikasi ini saja (ini sengaja huruf kapital semua untuk contoh nanti)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Anda mungkin telah memperhatikan bahwa ini mirip dengan pohon aplikasi pada umumnya (situs dokumentasi itu sendiri menggunakan tata letak terstruktur). `controllers` huruf kecil di sini adalah *pilihan* yang valid—hanya saja bukan default skeleton saat ini.

Anda dapat menentukan setiap direktori untuk dimuat seperti ini:

```php

/**
 * public/index.php
 */

// Tambahkan path ke autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// tidak perlu namespace

// Semua kelas yang di-autoload disarankan menggunakan Pascal Case (setiap kata diawali huruf kapital, tanpa spasi)
class MyController {

	public function index() {
		// lakukan sesuatu
	}
}
```

## Namespace dengan `Flight::path()`

Jika Anda memiliki namespace, sebenarnya ini menjadi sangat mudah untuk diterapkan. Anda harus menggunakan metode `Flight::path()` untuk menentukan direktori root (bukan document root atau folder `public/`) dari aplikasi Anda.

```php

/**
 * public/index.php
 */

// Tambahkan path ke autoloader
Flight::path(__DIR__.'/../');
```

Sekarang ini adalah contoh tampilan controller Anda. Perhatikan contoh di bawah ini, tetapi perhatikan komentar untuk informasi penting.

```php
/**
 * app/controllers/MyController.php
 */

// namespace diperlukan
// namespace sama dengan struktur direktori
// namespace harus mengikuti huruf besar/kecil yang sama dengan struktur direktori
// namespace dan direktori tidak boleh memiliki garis bawah (kecuali Loader::setV2ClassLoading(false) diatur)
namespace app\controllers;

// Semua kelas yang di-autoload disarankan menggunakan Pascal Case (setiap kata diawali huruf kapital, tanpa spasi)
// Mulai 3.7.2, Anda dapat menggunakan Pascal_Snake_Case untuk nama kelas Anda dengan menjalankan Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// lakukan sesuatu
	}
}
```

Dan jika Anda ingin meng-autoload kelas di direktori utils Anda, Anda pada dasarnya melakukan hal yang sama:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// namespace harus cocok dengan struktur direktori dan huruf besar/kecil (perhatikan direktori UTILS semuanya huruf kapital
//     seperti di pohon file di atas)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// lakukan sesuatu
	}
}
```

### Namespace gaya skeleton (aturan yang sama, huruf besar/kecil berbeda)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

Aturannya tidak berubah—hanya huruf besar/kecil folder/namespace yang dipilih skeleton. **Huruf besar/kecil apa pun yang digunakan folder Anda, baris `namespace` Anda harus cocok.**

## Garis Bawah dalam Nama Kelas

Mulai 3.7.2, Anda dapat menggunakan Pascal_Snake_Case untuk nama kelas Anda dengan menjalankan `Loader::setV2ClassLoading(false);`. 
Ini akan memungkinkan Anda menggunakan garis bawah dalam nama kelas. 
Ini tidak disarankan, tetapi tersedia bagi mereka yang membutuhkannya.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Tambahkan path ke autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// tidak perlu namespace

class My_Controller {

	public function index() {
		// lakukan sesuatu
	}
}
```

## Lihat Juga
- [Instalasi](/install) - Pohon skeleton dan default `App\` untuk proyek baru.
- [Routing](/learn/routing) - Cara memetakan rute ke controller dan merender tampilan.
- [Dependency Injection](/learn/dependency-injection-container) - Cara controller mendapatkan `Engine` dan layanan.
- [AI & Pengalaman Pengembang](/learn/ai) - Jaga agen tetap selaras dengan tata letak Anda melalui `AGENTS.md`.
- [Mengapa Framework?](/learn/why-frameworks) - Memahami manfaat menggunakan framework seperti Flight.

## Pemecahan Masalah
- Jika Anda tidak dapat mengetahui mengapa kelas ber-namespace Anda tidak ditemukan, ingat: dengan `Flight::path()`, arahkan ke **root proyek** (atau basis yang benar untuk namespace Anda), bukan hanya folder bersarang yang Anda lupa untuk dicerminkan di namespace.
- Dengan Composer PSR-4, jalankan `composer dump-autoload` setelah mengubah pemetaan `composer.json`.
- Di CI Linux atau produksi, huruf besar/kecil folder yang salah adalah kegagalan "berfungsi di mesin saya" yang sangat umum.

### Kelas Tidak Ditemukan (autoloading tidak berfungsi)

Mungkin ada beberapa alasan untuk hal ini. Di bawah ini adalah beberapa contoh.

#### Nama File Salah
Yang paling umum adalah nama kelas tidak cocok dengan nama file.

Jika Anda memiliki kelas bernama `MyClass` maka file harus bernama `MyClass.php`. Jika Anda memiliki kelas bernama `MyClass` dan file bernama `myclass.php` 
maka autoloader tidak akan dapat menemukannya.

#### Namespace atau Huruf Besar/Kecil Folder Salah
Jika Anda menggunakan namespace, maka namespace harus cocok dengan struktur direktori **termasuk huruf besar/kecil**.

```php
// ...kode...

// jika MyController Anda ada di app/Controller (skeleton) dan ber-namespace App\Controller
// ini tidak akan berfungsi:
Flight::route('/hello', 'MyController->hello');

// Gaya skeleton:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Tata letak huruf kecil lama (hanya jika folder Anda benar-benar app/controllers):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// atau sepenuhnya memenuhi syarat:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` tidak ditentukan (kode aplikasi non-Composer)

Jika Anda mengandalkan `Flight::path()` alih-alih Composer untuk kelas aplikasi, tentukan path sebelum rute yang merujuk ke kelas tersebut (sering kali di awal bootstrap / `public/index.php`):

```php
// Tambahkan path ke autoloader (root proyek untuk aplikasi ber-namespace)
Flight::path(__DIR__.'/../');
```

Skeleton resmi terutama menggunakan **Composer PSR-4** untuk `App\`, jadi Anda biasanya tidak perlu `Flight::path()` untuk controller dan model di sana.

## Changelog
- Dokumen – Mendokumentasikan skeleton `App\` + folder PascalCase dan jebakan sensitivitas huruf besar/kecil untuk manusia dan alat AI.
- v3.7.2 - Anda dapat menggunakan Pascal_Snake_Case untuk nama kelas Anda dengan menjalankan `Loader::setV2ClassLoading(false);`
- v2.0 - Fungsionalitas Autoload ditambahkan.