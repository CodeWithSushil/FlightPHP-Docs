# Ekstensi Panel Tracy Flight

Ini adalah sekumpulan ekstensi untuk membuat bekerja dengan Flight sedikit lebih kaya.

- **Flight** - Menganalisis semua variabel Flight.
- **Database** - Menganalisis semua query yang telah dijalankan pada halaman (jika Anda menginisiasi koneksi database dengan benar)
- **Request** - Menganalisis semua variabel `$_SERVER` dan memeriksa semua payload global (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Menganalisis semua variabel `$_SESSION` jika sesi aktif.
- **Twig** *(opsional)* - Menganalisis waktu render template Twig, memori, dan template/blok/makro mana yang dijalankan (memerlukan `twig/twig` dan konfigurasi `twig_profile`)

Ini sangat berguna dengan [kerangka resmi](https://github.com/flightphp/skeleton), yang secara default menggunakan Twig: tata letak yang sama [alat AI](/learn/ai) ikuti juga ditampilkan dengan jelas pada bar Tracy.

Ini adalah Panel

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

Dan setiap panel menampilkan informasi yang sangat membantu tentang aplikasi Anda!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Klik [di sini](https://github.com/flightphp/tracy-extensions) untuk melihat kode.

## Instalasi

Jalankan `composer require flightphp/tracy-extensions --dev` dan Anda siap melangkah!

Twig **bukan** dependensi keras dari paket ini. Instal `twig/twig` hanya jika Anda ingin panel Twig (kerangka sudah melakukannya untuk tampilan).

## Konfigurasi

Ada sangat sedikit konfigurasi yang perlu Anda lakukan untuk memulai ini. Anda perlu menginisiasi debugger Tracy sebelum menggunakan ini [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// kode bootstrap
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Anda mungkin perlu menentukan lingkungan Anda dengan Debugger::enable(Debugger::DEVELOPMENT)

// jika Anda menggunakan koneksi database dalam aplikasi, ada 
// wrapper PDO yang diperlukan untuk digunakan HANYA DALAM PENGEMBANGAN (bukan produksi mohon!)
// Ini memiliki parameter yang sama dengan koneksi PDO biasa
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// atau jika Anda melampirkannya ke kerangka Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// sekarang setiap kali Anda membuat query, itu akan menangkap waktu, query, dan parameter

// Ini menghubungkan titik-titik
if(Debugger::$showBar === true) {
	// Ini perlu false atau Tracy tidak bisa benar-benar render :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// kode lainnya

Flight::start();
```

## Konfigurasi Tambahan

### Data Sesi

Jika Anda memiliki handler sesi khusus (seperti ghostff/session), Anda dapat meneruskan array data sesi apa pun ke Tracy dan itu akan secara otomatis menampilkannya untuk Anda. Anda meneruskannya dengan kunci `session_data` dalam parameter kedua konstruktor `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// atau gunakan flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Ini perlu false atau Tracy tidak bisa benar-benar render :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// rute dan hal-hal lainnya...

Flight::start();
```

### Panel Twig (opsional)

Jika aplikasi Anda menggunakan [Twig](/awesome-plugins/twig) (termasuk kerangka resmi), Anda dapat menampilkan metrik template pada bar Tracy. Buat `Profile` Twig, lampirkan `ProfilerExtension` ke lingkungan Anda, lalu lewatkan profil tersebut ke loader di bawah kunci **`twig_profile`**. Lampirkan profiling hanya dalam pengembangan.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Opsional: expose helper dump Tracy dalam template
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Petakan Flight::render() ke Twig (contoh)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Apa yang ditampilkan panel**

- Total waktu render Twig dan memori
- Jumlah panggilan template / blok / makro
- Setiap template yang dirender, dengan waktu dan memorinya sendiri

Tab Twig **tersembunyi** ketika tidak ada template yang dirender untuk permintaan, atau ketika Anda menghilangkan `twig_profile` (atau tidak memiliki Twig terinstal)—panel Flight lainnya tetap berfungsi.

Dalam `services.php` gaya kerangka, bangun `$profile` / `ProfilerExtension` yang sama ketika debug aktif, lewatkan `twig_profile` ke `TracyExtensionLoader`, dan terus gunakan lingkungan Twig bersama Anda untuk `$app->render()`.

### Latte

_PHP 8.1+ diperlukan untuk bagian ini._

Jika Anda memiliki Latte terinstal dalam proyek Anda, Tracy memiliki integrasi asli dengan Latte untuk menganalisis template Anda. Anda cukup mendaftarkan ekstensi dengan instance Latte Anda (ini adalah bridge Tracy Latte sendiri, bukan panel Twig di atas).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// konfigurasi lainnya...

	// hanya tambahkan ekstensi jika Tracy Debug Bar diaktifkan
	if(Debugger::$showBar === true) {
		// ini adalah tempat Anda menambahkan Latte Panel ke Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Lihat Juga

- [Tracy](/awesome-plugins/tracy) - Setup Tracy dasar untuk Flight
- [Twig](/awesome-plugins/twig) - Templating yang digunakan oleh kerangka dan panel Twig
- [Templates](/learn/templates) - Bagaimana Flight memetakan `render` ke Twig/Latte
- [Installation](/install) - Kerangka menyertakan tracy-extensions dalam dev