# Keamanan

## Tinjauan

Keamanan adalah hal yang sangat penting dalam aplikasi web. Anda perlu memastikan bahwa aplikasi Anda aman dan data pengguna Anda 
terlindungi. Flight menyediakan sejumlah fitur untuk membantu Anda mengamankan aplikasi web.

[skeleton](https://github.com/flightphp/skeleton) resmi juga menyertakan **`SECURITY.md`** khusus dan middleware keamanan header sehingga [alat coding AI](/learn/ai) (dan manusia) memiliki satu tempat yang disengaja untuk secret, header, serta aturan XSS/SQL—terpisah dari gaya penulisan kode umum di `AGENTS.md`.

## Memahami

Ada sejumlah ancaman keamanan umum yang harus Anda waspadai saat membangun aplikasi web. Beberapa ancaman yang paling umum
meliputi:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates) membantu mengatasi XSS dengan melakukan escape pada output secara default (Twig dan Latte melakukan ini; manfaatkan keunggulan tersebut). [Sessions](/awesome-plugins/session) dapat membantu mengatasi CSRF dengan menyimpan token CSRF di sesi pengguna seperti yang dijelaskan di bawah. Menggunakan prepared statements dengan PDO—atau helper pada [SimplePdo](/learn/simple-pdo)—membantu mencegah SQL injection. CORS dapat ditangani dengan hook sederhana sebelum `Flight::start()` dipanggil.

Semua metode ini bekerja sama untuk membantu menjaga keamanan aplikasi web Anda. Selalu ingat untuk mempelajari dan memahami praktik terbaik keamanan. Jangan meminta asisten AI untuk "menonaktifkan CSP" atau melemahkan header hanya agar halaman dapat dimuat tanpa memahami konsekuensinya.

## Penggunaan Dasar

### Header

Header HTTP adalah salah satu cara termudah untuk mengamankan aplikasi web Anda. Anda dapat menggunakan header untuk mencegah clickjacking, XSS, dan serangan lainnya. 
Ada beberapa cara untuk menambahkan header ini ke aplikasi Anda.

Dua situs web yang bagus untuk memeriksa keamanan header Anda adalah [securityheaders.com](https://securityheaders.com/) dan 
[observatory.mozilla.org](https://observatory.mozilla.org/). Setelah Anda menyiapkan kode di bawah, Anda dapat dengan mudah memverifikasi bahwa header Anda berfungsi dengan kedua situs tersebut.

Skeleton menyertakan `App\Middleware\SecurityHeadersMiddleware` (CSP dengan nonce per permintaan, frame options, HSTS, dan lainnya). Preferensikan untuk memperluasnya secara sengaja daripada mematikan header.

#### Tambahkan Secara Manual

Anda dapat menambahkan header ini secara manual menggunakan metode `header` pada objek `Flight\Response`.
```php
// Setel header X-Frame-Options untuk mencegah clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Setel header Content-Security-Policy untuk mencegah XSS
// Catatan: header ini bisa menjadi sangat kompleks, jadi Anda perlu
//  berkonsultasi dengan contoh-contoh di internet untuk aplikasi Anda
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Setel header X-XSS-Protection untuk mencegah XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Setel header X-Content-Type-Options untuk mencegah MIME sniffing
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Setel header Referrer-Policy untuk mengontrol seberapa banyak informasi referrer yang dikirim
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Setel header Strict-Transport-Security untuk memaksa HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Setel header Permissions-Policy untuk mengontrol fitur dan API apa saja yang dapat digunakan
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Ini dapat ditambahkan di bagian atas file `routes.php` atau `index.php` Anda.

#### Tambahkan sebagai Filter

Anda juga dapat menambahkannya dalam filter/hook seperti berikut:

```php
// Tambahkan header dalam sebuah filter
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### Tambahkan sebagai Middleware

Anda juga dapat menambahkannya sebagai class middleware yang memberikan fleksibilitas terbesar untuk menentukan route mana yang akan diterapkan. Secara umum, header ini harus diterapkan ke semua respons HTML dan API.

Jalur dan namespace ala skeleton (**kapitalisasi folder harus sesuai dengan `App\Middleware`**):

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		// Preferensikan nonce CSP dari bootstrap ketika Anda memiliki skrip inline (skeleton menetapkan csp_nonce)
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — grup string kosong = middleware global untuk semua route
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// lebih banyak route
}, [SecurityHeadersMiddleware::class]);
```

Proyek lama mungkin masih menggunakan `app/middlewares` dan `app\middlewares`; itu berfungsi jika folder sesuai. Aplikasi skeleton baru menggunakan **`app/Middleware/`** dan **`App\Middleware`**. Lihat [Autoloading](/learn/autoloading).

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) adalah jenis serangan di mana situs web berbahaya dapat membuat browser pengguna mengirim permintaan ke situs web Anda. 
Ini dapat digunakan untuk melakukan tindakan di situs web Anda tanpa sepengetahuan pengguna. Flight tidak menyediakan mekanisme perlindungan CSRF 
bawaan, tetapi Anda dapat dengan mudah menerapkannya sendiri menggunakan middleware.

#### Pengaturan

Pertama, Anda perlu menghasilkan token CSRF dan menyimpannya di sesi pengguna. Anda kemudian dapat menggunakan token ini di form Anda dan memeriksanya saat 
form dikirim. Kita akan menggunakan plugin [flightphp/session](/awesome-plugins/session) untuk mengelola sesi.

```php
// Hasilkan token CSRF dan simpan di sesi pengguna
// (dengan asumsi Anda telah membuat objek sesi dan melampirkannya ke Flight)
// lihat dokumentasi sesi untuk informasi lebih lanjut
Flight::register('session', flight\Session::class);

// Anda hanya perlu menghasilkan satu token per sesi (sehingga berfungsi 
// di beberapa tab dan permintaan untuk pengguna yang sama)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Menggunakan Template PHP Flight Default

```html
<!-- Gunakan token CSRF di form Anda -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- field form lainnya -->
</form>
```

##### Menggunakan Twig (default skeleton)

Daftarkan fungsi Twig atau teruskan token ke setiap tampilan form. Contoh minimal dengan global + field form:

```php
// Saat mengonfigurasi Twig (misalnya services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# field lainnya #}
</form>
```

##### Menggunakan Latte

Anda juga dapat mengatur fungsi khusus untuk menampilkan token CSRF di template Latte Anda.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// konfigurasi lainnya...

	// Atur fungsi khusus untuk menampilkan token CSRF
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Dan sekarang di template Latte Anda dapat menggunakan fungsi `csrf()` untuk menampilkan token CSRF.

```html
<form method="post">
	{csrf()}
	<!-- field form lainnya -->
</form>
```

#### Memeriksa Token CSRF

Anda dapat memeriksa token CSRF menggunakan beberapa metode.

##### Middleware

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			// Validasi token
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Token CSRF tidak valid');
			}
		}
	}
}

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// lebih banyak route
}, [CsrfMiddleware::class]);
```

##### Event Filters

```php
// Middleware ini memeriksa apakah permintaan adalah POST dan jika ya, ia memeriksa apakah token CSRF valid
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// ambil token csrf dari nilai form
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Token CSRF tidak valid');
			// atau untuk respons JSON
			Flight::jsonHalt(['error' => 'Token CSRF tidak valid'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) adalah jenis serangan di mana input form berbahaya dapat menyuntikkan kode ke situs web Anda. Sebagian besar peluang ini berasal 
dari nilai form yang akan diisi oleh pengguna akhir Anda. Anda **tidak boleh pernah** mempercayai output dari pengguna Anda! Selalu anggap semuanya adalah 
peretas terbaik di dunia. Mereka dapat menyuntikkan JavaScript atau HTML berbahaya ke halaman Anda. Kode ini dapat digunakan untuk mencuri informasi dari 
pengguna Anda atau melakukan tindakan di situs web Anda. Dengan menggunakan class view Flight atau mesin template seperti [Twig](/awesome-plugins/twig) atau [Latte](/awesome-plugins/latte), Anda dapat dengan mudah melakukan escape output untuk mencegah serangan XSS.

```php
// Anggap pengguna cerdas dan mencoba menggunakan ini sebagai nama mereka
$name = '<script>alert("XSS")</script>';

// Ini akan melakukan escape pada output
Flight::view()->set('name', $name);
// Ini akan menghasilkan output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (default skeleton) dan Latte melakukan auto-escape secara default — preferensikan daripada echo PHP mentah
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → sudah di-escape
// Hindari |raw / output yang tidak di-escape kecuali kontennya sepenuhnya tepercaya
```

### SQL Injection

SQL Injection adalah jenis serangan di mana pengguna berbahaya dapat menyuntikkan kode SQL ke database Anda. Ini dapat digunakan untuk mencuri informasi 
dari database Anda atau melakukan tindakan pada database Anda. Sekali lagi, Anda **tidak boleh pernah** mempercayai input dari pengguna Anda! Selalu anggap mereka 
haus akan data. Gunakan prepared statements—helper [SimplePdo](/learn/simple-pdo) menjadikan ini jalur default.

```php
// Dengan asumsi Anda mendaftarkan Flight::db() sebagai SimplePdo (atau menginjeksi SimplePdo di controller)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (disarankan) — satu baris dengan parameter terikat
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Ide yang sama dengan placeholder ?
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

Di controller bergaya skeleton, preferensikan injeksi konstruktor `SimplePdo` daripada `Flight::db()` sehingga pengujian dan kode yang dihasilkan AI tetap konsisten ([DIC](/learn/dependency-injection-container)).

#### Contoh Tidak Aman

Di bawah ini adalah alasan mengapa kita menggunakan SQL prepared statements untuk melindungi dari contoh sederhana seperti di bawah:

```php
// pengguna akhir mengisi form web.
// untuk nilai form, peretas memasukkan sesuatu seperti ini:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Setelah query dibuat, hasilnya terlihat seperti ini
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Terlihat aneh, tetapi ini adalah query yang valid dan akan berhasil. Faktanya,
// ini adalah serangan SQL injection yang sangat umum yang akan mengembalikan semua pengguna.

var_dump($users); // ini akan menampilkan semua pengguna di database, bukan hanya satu username tersebut
```

### Secret dan Konfigurasi

- Letakkan secret di **`.env`** (atau environment asli), bukan di `config.php` contoh yang di-commit.
- Aturan skeleton: default literal di `config.php`; gabungkan env saat bootstrap; **jangan** membaca `$_ENV` di dalam controller—injeksi konfigurasi sebagai gantinya. Lihat [Configuration](/learn/configuration).
- Jangan pernah meng-commit API keys, password database, atau kunci enkripsi sesi. Arahkan alat AI ke **`SECURITY.md`** sehingga mereka tidak membuat jalan pintas yang tidak aman.

### Validasi Callback JSONP

Jika Anda menggunakan metode `Flight::jsonp()`, perlu diketahui bahwa Flight memvalidasi nama parameter callback JSONP terhadap regex whitelist yang ketat (`/^[A-Za-z_$][\w$.]{0,127}$/`). Nama callback apa pun yang tidak cocok dengan pola ini akan menyebabkan Flight melempar exception, mencegah injeksi JavaScript arbitrer melalui nilai callback berbahaya.

Validasi ini sudah tertanam dan tidak memerlukan konfigurasi tambahan, tetapi perlu diketahui saat men-debug error tak terduga dari endpoint JSONP.

### CORS

Cross-Origin Resource Sharing (CORS) adalah mekanisme yang memungkinkan banyak sumber daya (misalnya, font, JavaScript, dll.) di halaman web untuk 
diminta dari domain lain di luar domain asal sumber daya tersebut. Flight tidak memiliki fungsionalitas bawaan, 
tetapi ini dapat dengan mudah ditangani dengan hook yang berjalan sebelum metode `Flight::start()` dipanggil.

```php
// app/Utils/CorsUtil.php  (skeleton: folder Utils PascalCase → App\Utils)

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// sesuaikan host yang diizinkan di sini.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// bootstrap / routes — jalankan sebelum start
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Penguatan Konfigurasi Flight

Flight mengekspos beberapa pengaturan engine yang memiliki implikasi keamanan langsung. Mengatur ini dengan benar adalah salah satu cara termudah untuk memperkuat aplikasi Anda.

#### `flight.allow_method_override`

Secara default, Flight mengizinkan klien untuk menimpa metode HTTP dari sebuah permintaan menggunakan header `X-HTTP-Method-Override` atau field `_method` di body POST. Meskipun ini berguna untuk form HTML yang hanya dapat mengirim `GET`/`POST`, ini bisa berbahaya jika Anda tidak mengharapkannya — penyerang dapat memalsukan permintaan `DELETE` atau `PUT` melalui form biasa.

Jika aplikasi Anda tidak bergantung pada perilaku ini (misalnya, Anda membangun API yang dikonsumsi oleh klien modern atau frontend JavaScript yang dapat mengirim kata kerja HTTP apa pun), Anda harus menonaktifkannya:

```php
// Di index.php atau file bootstrap Anda, sebelum Flight::start()
Flight::set('flight.allow_method_override', false);
```

Nilai defaultnya adalah `true` untuk kompatibilitas mundur, tetapi **sangat disarankan untuk mengaturnya ke `false`** untuk aplikasi apa pun yang tidak secara eksplisit membutuhkan fitur override.

#### `flight.debug`

Flight memiliki pengaturan `flight.debug` yang mengontrol apakah informasi error terperinci (pesan exception, kode, dan stack trace lengkap) ditampilkan di browser ketika exception yang tidak tertangani terjadi. Defaultnya adalah `false`, yang berarti hanya pesan `500 Internal Server Error` generik yang ditampilkan — tidak ada detail internal yang bocor ke klien.

Jangan pernah mengaktifkan ini di server produksi. Gunakan hanya secara lokal atau di lingkungan staging:

```php
// Aman untuk pengembangan lokal saja — JANGAN di produksi
Flight::set('flight.debug', true);
```

Ketika `flight.debug` adalah `false` (default), Anda masih dapat menangkap error dengan mengaktifkan `flight.log_errors`:

```php
// Catat error di sisi server tanpa mengeksposnya ke klien
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Konfigurasi produksi yang direkomendasikan

```php
// index.php atau diterapkan dari konfigurasi app / bootstrap
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Penanganan Error
Sembunyikan detail error yang sensitif di produksi untuk menghindari kebocoran informasi kepada penyerang. Di produksi, catat error alih-alih menampilkannya dengan `display_errors` diatur ke `0`.

```php
// Di bootstrap.php atau index.php Anda

// tambahkan ini ke app/config/config.php Anda
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Nonaktifkan tampilan error
    ini_set('log_errors', 1);     // Catat error sebagai gantinya
    ini_set('error_log', '/path/to/error.log');
}

// Di routes atau controller Anda
// Gunakan Flight::halt() untuk respons error yang terkontrol
Flight::halt(403, 'Akses ditolak');
```

### Sanitasi Input
Jangan pernah mempercayai input pengguna. Sanitasi menggunakan [filter_var](https://www.php.net/manual/en/function.filter-var.php) sebelum diproses untuk mencegah data berbahaya masuk. Preferensikan membaca input melalui `$app->request()` (atau `Flight::request()`) daripada `$_GET` / `$_POST` mentah di kode aplikasi.

```php

// Anggap ada permintaan $_POST dengan $_POST['input'] dan $_POST['email']

// Sanitasi input string
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitasi email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Hashing Password
Simpan password dengan aman dan verifikasi dengan aman menggunakan fungsi bawaan PHP seperti [password_hash](https://www.php.net/manual/en/function.password-hash.php) dan [password_verify](https://www.php.net/manual/en/function.password-verify.php). Password tidak boleh pernah disimpan dalam teks biasa, juga tidak boleh dienkripsi dengan metode yang dapat dibalik. Hashing memastikan bahwa bahkan jika database Anda disusupi, password sebenarnya tetap terlindungi.

```php
$password = Flight::request()->data->password;
// Hash password saat menyimpannya (misalnya, saat registrasi)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verifikasi password (misalnya, saat login)
if (password_verify($password, $stored_hash)) {
    // Password cocok
}
```

### Pembatasan Laju (Rate Limiting)
Lindungi dari serangan brute force atau serangan denial-of-service dengan membatasi laju permintaan menggunakan cache.

```php
// Dengan asumsi Anda memiliki flightphp/cache yang terinstal dan terdaftar
// Menggunakan flightphp/cache dalam filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Terlalu banyak permintaan');
    }
    
    $cache->set($key, $attempts + 1, 60); // Atur ulang setelah 60 detik
});
```

## Lihat Juga
- [Sessions](/awesome-plugins/session) - Cara mengelola sesi pengguna dengan aman.
- [Templates](/learn/templates) - Twig/Latte auto-escape dan XSS.
- [SimplePdo](/learn/simple-pdo) - Helper database dengan prepared statements.
- [PdoWrapper](/learn/pdo-wrapper) - Tidak digunakan lagi; gunakan SimplePdo untuk kode baru.
- [Middleware](/learn/middleware) - Cara menggunakan middleware untuk menyederhanakan proses penambahan header keamanan.
- [Configuration](/learn/configuration) - `.env` vs konfigurasi literal, flag produksi.
- [AI & Developer Experience](/learn/ai) - Jaga kebijakan keamanan di `SECURITY.md` untuk agen.
- [Responses](/learn/responses) - Cara menyesuaikan respons HTTP dengan header yang aman.
- [Requests](/learn/requests) - Cara menangani dan membersihkan input pengguna.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Fungsi PHP untuk sanitasi input.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Fungsi PHP untuk hashing password yang aman.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Fungsi PHP untuk memverifikasi password yang di-hash.

## Pemecahan Masalah
- Lihat bagian "Lihat Juga" di atas untuk informasi pemecahan masalah terkait komponen Framework Flight.
- Jika CSP memblokir skrip Anda, tambahkan nonce (pola skeleton) atau daftar putih origin spesifik—jangan atur `script-src *` tanpa rencana.

## Changelog
- Docs – Skeleton `App\Middleware`, catatan Twig CSRF/XSS, SimplePdo, secret/`.env`, dan `SECURITY.md` untuk proyek yang ramah AI.
- v3.18.1 - Menambahkan bagian Penguatan Konfigurasi Flight yang mencakup `flight.allow_method_override`, `flight.debug`, dan validasi callback JSONP.
- v3.1.0 - Menambahkan bagian tentang CORS, Penanganan Error, Sanitasi Input, Hashing Password, dan Pembatasan Laju.
- v2.0 - Menambahkan escape untuk tampilan default guna mencegah XSS.