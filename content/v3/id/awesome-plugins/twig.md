# Twig

[Twig](https://twig.symfony.com/) adalah mesin templat PHP yang fleksibel, cepat, dan aman. Ini adalah bahasa templating yang digunakan oleh Symfony dan banyak proyek lainnya, yang berarti alat coding AI dan sebagian besar pengembang PHP sudah sangat mengenal sintaksnya. Twig mengompilasi templat menjadi PHP yang dioptimalkan, meng-escape output secara otomatis secara default (bagus untuk perlindungan XSS), dan mudah diperluas dengan filter, fungsi, dan ekstensi.

## Instalasi

Instal dengan composer.

```bash
composer require twig/twig
```

## Konfigurasi Dasar

Ada beberapa opsi konfigurasi dasar untuk memulai. Anda dapat membaca lebih lanjut tentang opsi-opsi tersebut di [Dokumentasi Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Tempat Twig menyimpan templat yang telah dikompilasi
		'cache' => __DIR__ . '/../cache/twig',
		// Kompilasi ulang templat ketika sumber berubah (berguna saat pengembangan)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Mendaftarkan Twig sebagai Kelas View

Jika Anda lebih suka menggunakan ulang satu Twig environment (direkomendasikan untuk produksi), daftarkan dan arahkan `render` ke sana:

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	],
]);

$app->map('render', function(string $template, array $data): void {
	echo Flight::view()->render($template, $data);
});
```

## Contoh Layout Sederhana

Berikut adalah contoh sederhana dari file layout. Ini adalah file yang akan digunakan untuk membungkus semua view Anda yang lain.

```html
{# app/views/layout.twig #}
<!doctype html>
<html lang="en">
	<head>
		<title>{% if title %}{{ title }} - {% endif %}My App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				{# elemen nav Anda di sini #}
			</nav>
		</header>
		<div id="content">
			{# Ini adalah keajaibannya #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

Dan sekarang kita memiliki file Anda yang akan dirender di dalam blok konten tersebut:

```html
{# app/views/home.twig #}
{# Ini memberitahu Twig bahwa file ini "di dalam" file layout.twig #}
{% extends 'layout.twig' %}

{# Ini adalah konten yang akan dirender di dalam layout di dalam blok konten #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Kemudian ketika Anda pergi untuk merender ini di dalam fungsi atau controller Anda, Anda akan melakukan sesuatu seperti ini:

```php
// rute sederhana
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// atau jika Anda menggunakan controller
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Home Page'
		]);
	}
}
```

Lihat [Dokumentasi Twig](https://twig.symfony.com/doc/3.x/) untuk informasi lebih lanjut tentang cara menggunakan Twig secara maksimal!

## Debugging

Twig dilengkapi dengan [Ekstensi Debug](https://twig.symfony.com/doc/3.x/functions/dump.html) yang menambahkan fungsi `dump()` yang dapat Anda gunakan di dalam templat. Aktifkan hanya saat pengembangan:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // diperlukan untuk fungsi dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Kemudian dalam sebuah templat:

```html
{{ dump(user) }}
```

Anda juga dapat menggabungkan Twig dengan [Tracy](/awesome-plugins/tracy) untuk debugging tingkat PHP. Untuk metrik tingkat templat (waktu render, memori, templat/blok mana yang berjalan), gunakan **panel Twig** opsional di [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): lewati `Twig\Profiler\Profile` sebagai `twig_profile` ke `TracyExtensionLoader`. `TwigTracyExtension` opsional mengekspos `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` di templat ketika Tracy aktif.

## Catatan Keamanan

Twig secara otomatis meng-escape output secara default, yang membantu melindungi dari serangan XSS. Lebih suka `{{ variable }}` untuk teks. Hanya gunakan filter `|raw` ketika Anda sengaja mempercayai konten HTML (misalnya, markdown yang telah disanitasi yang telah Anda proses di sisi server).