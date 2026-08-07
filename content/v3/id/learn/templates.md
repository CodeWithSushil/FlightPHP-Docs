# Tampilan HTML dan Template

## Ringkasan

Flight menyediakan beberapa fungsionalitas templating HTML dasar secara bawaan. Templating adalah cara yang sangat efektif untuk memisahkan logika aplikasi Anda dari lapisan presentasi. Mesin khusus (Twig, Latte, dll.) juga memberikan [alat bantu pengodean AI](/learn/ai) sintaks yang familiar dan terbatas sehingga mereka cenderung tidak membuang logika bisnis ke dalam HTML Anda.

## Pemahaman

Saat Anda membangun aplikasi, kemungkinan Anda akan memiliki HTML yang ingin Anda kirim kembali ke pengguna akhir. PHP sendiri adalah bahasa templating, tetapi sangat mudah untuk membungkus logika bisnis seperti panggilan database, panggilan API, dll. ke dalam file HTML Anda dan membuat pengujian serta pemisahan menjadi proses yang sangat sulit. Dengan mendorong data ke dalam template dan membiarkan template merender dirinya sendiri, menjadi lebih mudah untuk memisahkan dan menguji unit kode Anda. Anda akan berterima kasih kepada kami jika menggunakan template!

## Penggunaan Dasar

Flight memungkinkan Anda untuk mengganti mesin tampilan default cukup dengan memetakan `render` (atau mendaftarkan kelas tampilan). Gulir ke bawah untuk Twig, Latte, Smarty, Blade, dan lainnya.

> **Default Skeleton:** [flightphp/skeleton](https://github.com/flightphp/skeleton) resmi menggunakan **Twig saja** di bawah `app/views/` (`*.twig`). Controller memanggil `$this->app->render('welcome', $data)` (ekstensi opsional). Itu adalah pilihan aplikasi untuk proyek baru—bukan keharusan dari inti Flight. Latte dan mesin lainnya tetap didukung penuh.

### Twig

<span class="badge bg-info">default skeleton</span>

[Twig](https://twig.symfony.com/) adalah mesin template yang fleksibel, cepat, dan aman yang digunakan oleh Symfony dan banyak proyek PHP lainnya. Alat bantu pengodean AI cenderung sangat mengenal Twig, dan secara default melakukan auto-escape pada output yang membantu melindungi dari XSS.

#### Instalasi

```bash
composer require twig/twig
```

(Sudah termasuk saat Anda `composer create-project flightphp/skeleton`.)

#### Konfigurasi Dasar

Timpa metode `render` untuk menggunakan Twig alih-alih renderer PHP default:

```php
// timpa metode render untuk menggunakan Twig daripada renderer PHP default
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Di mana Twig menyimpan template yang dikompilasi
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Izinkan "welcome" atau "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

Di skeleton, pemasangan ini berada di `app/config/services.php` (lingkungan Twig bersama, jalur cache, global seperti `base_url` / nonce CSP). Sebaiknya injeksi `Engine` dan panggil `$app->render()` dari controller agar kode tetap [ramah AI dan pengujian](/learn/ai).

#### Menggunakan Twig di Flight

Sekarang Anda dapat merender dengan Twig, Anda dapat melakukan sesuatu seperti ini:

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Saat Anda mengunjungi `/Bob` di browser, outputnya akan menjadi:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Bacaan Lebih Lanjut

Contoh yang lebih lengkap tentang penggunaan Twig dengan layout ditunjukkan di bagian [plugin keren](/awesome-plugins/twig) dari dokumentasi ini. Untuk metrik waktu render pada bar Tracy, lihat [panel Twig di Ekstensi Tracy](/awesome-plugins/tracy-extensions#twig-panel-optional).

Anda dapat mempelajari lebih lanjut tentang kemampuan penuh Twig dengan membaca [dokumentasi resmi](https://twig.symfony.com/doc/3.x/).

### Latte

<span class="badge bg-secondary">alternatif bagus</span>

[Latte](https://latte.nette.org/) adalah mesin berfitur lengkap dengan sintaks mirip PHP. Ini tetap menjadi pilihan yang sangat baik untuk aplikasi Flight; skeleton hanya menstandarkan pada Twig untuk satu default bersama (terutama membantu saat alat AI menghasilkan template).

#### Instalasi

```bash
composer require latte/latte
```

#### Konfigurasi Dasar

Ide utamanya adalah Anda menimpa metode `render` untuk menggunakan Latte alih-alih renderer PHP default.

```php
// timpa metode render untuk menggunakan latte daripada renderer PHP default
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Di mana latte secara khusus menyimpan cache-nya
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Menggunakan Latte di Flight

Sekarang Anda dapat merender dengan Latte, Anda dapat melakukan sesuatu seperti ini:

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Saat Anda mengunjungi `/Bob` di browser, outputnya akan menjadi:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Bacaan Lebih Lanjut

Contoh yang lebih kompleks tentang penggunaan Latte dengan layout ditunjukkan di bagian [plugin keren](/awesome-plugins/latte) dari dokumentasi ini.

Anda dapat mempelajari lebih lanjut tentang kemampuan penuh Latte termasuk terjemahan dan kemampuan bahasa dengan membaca [dokumentasi resmi](https://latte.nette.org/en/).

### Mesin Tampilan Bawaan

<span class="badge bg-warning">usang</span>

> **Catatan:** Ini masih merupakan fungsionalitas default dan secara teknis masih berfungsi.

Untuk menampilkan template tampilan, panggil metode `render` dengan nama
file template dan data template opsional:

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Data template yang Anda berikan secara otomatis disuntikkan ke dalam template dan dapat
dirujuk seperti variabel lokal. File template hanyalah file PHP. Jika
isi file template `hello.php` adalah:

```php
Hello, <?= $name ?>!
```

Outputnya akan menjadi:

```text
Hello, Bob!
```

Anda juga dapat mengatur variabel tampilan secara manual dengan menggunakan metode set:

```php
Flight::view()->set('name', 'Bob');
```

Variabel `name` sekarang tersedia di semua tampilan Anda. Jadi Anda cukup melakukan:

```php
Flight::render('hello');
```

Perhatikan bahwa saat menentukan nama template dalam metode render, Anda dapat
menghilangkan ekstensi `.php`.

Secara default Flight akan mencari direktori `views` untuk file template. Anda dapat
mengatur jalur alternatif untuk template Anda dengan mengatur konfigurasi berikut:

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### Layout

Umumnya situs web memiliki satu file template layout dengan konten yang
bergantian. Untuk merender konten yang akan digunakan dalam layout, Anda dapat memberikan
parameter opsional ke metode `render`.

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

Tampilan Anda kemudian akan memiliki variabel tersimpan bernama `headerContent` dan `bodyContent`.
Anda kemudian dapat merender layout Anda dengan melakukan:

```php
Flight::render('layout', ['title' => 'Home Page']);
```

Jika file template terlihat seperti ini:

`header.php`:

```php
<h1><?= $heading ?></h1>
```

`body.php`:

```php
<div><?= $body ?></div>
```

`layout.php`:

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

Outputnya akan menjadi:
```html
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <div>World</div>
  </body>
</html>
```

### Smarty

Berikut cara menggunakan mesin template [Smarty](http://www.smarty.net/)
untuk tampilan Anda:

```php
// Muat pustaka Smarty
require './Smarty/libs/Smarty.class.php';

// Daftarkan Smarty sebagai kelas tampilan
// Juga berikan fungsi callback untuk mengonfigurasi Smarty saat dimuat
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Tetapkan data template
Flight::view()->assign('name', 'Bob');

// Tampilkan template
Flight::view()->display('hello.tpl');
```

Untuk kelengkapan, Anda juga harus menimpa metode render default Flight:

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

Berikut cara menggunakan mesin template [Blade](https://laravel.com/docs/8.x/blade) untuk tampilan Anda:

Pertama, Anda perlu menginstal pustaka BladeOne melalui Composer:

```bash
composer require eftec/bladeone
```

Kemudian, Anda dapat mengonfigurasi BladeOne sebagai kelas tampilan di Flight:

```php
<?php
// Muat pustaka BladeOne
use eftec\bladeone\BladeOne;

// Daftarkan BladeOne sebagai kelas tampilan
// Juga berikan fungsi callback untuk mengonfigurasi BladeOne saat dimuat
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Tetapkan data template
Flight::view()->share('name', 'Bob');

// Tampilkan template
echo Flight::view()->run('hello', []);
```

Untuk kelengkapan, Anda juga harus menimpa metode render default Flight:

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

Dalam contoh ini, file template `hello.blade.php` mungkin terlihat seperti ini:

```php
<?php
Hello, {{ $name }}!
```

Outputnya akan menjadi:

```
Hello, Bob!
```

## Lihat Juga
- [Instalasi](/install) - Tata letak skeleton (`app/views/*.twig`) untuk proyek baru.
- [Ekstensi](/learn/extending) - Cara menimpa metode `render` untuk menggunakan mesin template yang berbeda.
- [Routing](/learn/routing) - Cara memetakan rute ke controller dan merender tampilan.
- [Respons](/learn/responses) - Cara menyesuaikan respons HTTP.
- [Keamanan](/learn/security) - Auto-escaping dan XSS.
- [AI & Pengalaman Pengembang](/learn/ai) - Mengapa satu default mesin tampilan membantu agen pengodean.
- [Mengapa Framework?](/learn/why-frameworks) - Bagaimana template masuk ke gambaran besar.

## Pemecahan Masalah
- Jika Anda memiliki pengalihan di middleware, tetapi aplikasi Anda tampaknya tidak mengalihkan, pastikan Anda menambahkan pernyataan `exit;` di middleware Anda.
- Jika Twig tidak dapat menemukan template, periksa `flight.views.path` dan pastikan file tersebut ada di jalur itu dengan ekstensi yang diharapkan (skeleton: `app/views/`).

## Changelog
- Dokumen – Twig didokumentasikan sebagai default skeleton resmi; Latte tetap menjadi alternatif kelas satu.
- v2.0 - Rilis awal.