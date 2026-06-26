# Keamanan

## Gambaran Umum

Keamanan adalah hal yang sangat penting ketika berurusan dengan aplikasi web. Anda ingin memastikan bahwa aplikasi Anda aman dan data pengguna Anda 
aman. Flight menyediakan sejumlah fitur untuk membantu Anda mengamankan aplikasi web Anda.

## Pemahaman

Ada sejumlah ancaman keamanan umum yang harus Anda waspadai saat membangun aplikasi web. Beberapa ancaman paling umum
meliputi:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Templates](/learn/templates) membantu XSS dengan meng-escape output secara default sehingga Anda tidak perlu mengingat untuk melakukannya. [Sessions](/awesome-plugins/session) dapat membantu CSRF dengan menyimpan token CSRF di sesi pengguna seperti yang diuraikan di bawah ini. Menggunakan prepared statements dengan PDO dapat membantu mencegah serangan SQL injection (atau menggunakan metode yang berguna di kelas [PdoWrapper](/learn/pdo-wrapper)). CORS dapat ditangani dengan hook sederhana sebelum `Flight::start()` dipanggil.

Semua metode ini bekerja bersama untuk membantu menjaga keamanan aplikasi web Anda. Ini harus selalu menjadi prioritas utama dalam pikiran Anda untuk mempelajari dan memahami praktik terbaik keamanan.

## Penggunaan Dasar

### Headers

HTTP headers adalah salah satu cara termudah untuk mengamankan aplikasi web Anda. Anda dapat menggunakan headers untuk mencegah clickjacking, XSS, dan serangan lainnya. 
Ada beberapa cara yang dapat Anda tambahkan headers ini ke aplikasi Anda.

Dua situs web yang bagus untuk memeriksa keamanan headers Anda adalah [securityheaders.com](https://securityheaders.com/) dan 
[observatory.mozilla.org](https://observatory.mozilla.org/). Setelah Anda menyiapkan kode di bawah ini, Anda dapat dengan mudah memverifikasi bahwa headers Anda berfungsi dengan kedua situs web tersebut.

#### Tambahkan Secara Manual

Anda dapat menambahkan headers ini secara manual dengan menggunakan metode `header` pada objek `Flight\Response`.
```php
// Set the X-Frame-Options header to prevent clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Set the Content-Security-Policy header to prevent XSS
// Note: this header can get very complex, so you'll want
//  to consult examples on the internet for your application
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Set the X-XSS-Protection header to prevent XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Set the X-Content-Type-Options header to prevent MIME sniffing
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Set the Referrer-Policy header to control how much referrer information is sent
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Set the Strict-Transport-Security header to force HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Set the Permissions-Policy header to control what features and APIs can be used
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Ini dapat ditambahkan di bagian atas file `routes.php` atau `index.php` Anda.

#### Tambahkan sebagai Filter

Anda juga dapat menambahkannya dalam filter/hook seperti berikut: 

```php
// Add the headers in a filter
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

Anda juga dapat menambahkannya sebagai kelas middleware yang memberikan fleksibilitas terbesar untuk rute mana yang akan menerapkan ini. Secara umum, headers ini harus diterapkan pada semua respons HTML dan API.

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

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
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php or wherever you have your routes
// FYI, this empty string group acts as a global middleware for
// all routes. Of course you could do the same thing and just add
// this only to specific routes.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) adalah jenis serangan di mana situs web berbahaya dapat membuat browser pengguna mengirim permintaan ke situs web Anda. 
Ini dapat digunakan untuk melakukan tindakan di situs web Anda tanpa sepengetahuan pengguna. Flight tidak menyediakan mekanisme perlindungan CSRF bawaan, 
tetapi Anda dapat dengan mudah menerapkan sendiri dengan menggunakan middleware.

#### Setup

Pertama Anda perlu menghasilkan token CSRF dan menyimpannya di sesi pengguna. Anda kemudian dapat menggunakan token ini dalam formulir Anda dan memeriksanya saat 
formulir dikirimkan. Kami akan menggunakan plugin [flightphp/session](/awesome-plugins/session) untuk mengelola sesi.

```php
// Generate a CSRF token and store it in the user's session
// (assuming you've created a session object at attached it to Flight)
// see the session documentation for more information
Flight::register('session', flight\Session::class);

// You only need to generate a single token per session (so it works 
// across multiple tabs and requests for the same user)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Menggunakan Template Flight PHP Default

```html
<!-- Use the CSRF token in your form -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- other form fields -->
</form>
```

##### Menggunakan Latte

Anda juga dapat mengatur fungsi khusus untuk mengeluarkan token CSRF dalam template Latte Anda.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// other configurations...

	// Set a custom function to output the CSRF token
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Dan sekarang dalam template Latte Anda dapat menggunakan fungsi `csrf()` untuk mengeluarkan token CSRF.

```html
<form method="post">
	{csrf()}
	<!-- other form fields -->
</form>
```

#### Periksa Token CSRF

Anda dapat memeriksa token CSRF menggunakan beberapa metode.

##### Middleware

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

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
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// index.php or wherever you have your routes
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// more routes
}, [ CsrfMiddleware::class ]);
```

##### Event Filters

```php
// This middleware checks if the request is a POST request and if it is, it checks if the CSRF token is valid
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// capture the csrf token from the form values
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// or for a JSON response
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) adalah jenis serangan di mana input formulir berbahaya dapat menyuntikkan kode ke dalam situs web Anda. Kebanyakan peluang ini datang 
dari nilai formulir yang akan diisi oleh pengguna akhir Anda. Anda **tidak boleh** mempercayai output dari pengguna Anda! Selalu asumsikan bahwa semua mereka adalah 
peretas terbaik di dunia. Mereka dapat menyuntikkan JavaScript atau HTML berbahaya ke halaman Anda. Kode ini dapat digunakan untuk mencuri informasi dari 
pengguna Anda atau melakukan tindakan di situs web Anda. Dengan menggunakan kelas view Flight atau mesin templating lain seperti [Latte](/awesome-plugins/latte), Anda dapat dengan mudah meng-escape output untuk mencegah serangan XSS.

```php
// Let's assume the user is clever as tries to use this as their name
$name = '<script>alert("XSS")</script>';

// This will escape the output
Flight::view()->set('name', $name);
// This will output: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// If you use something like Latte registered as your view class, it will also auto escape this.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection adalah jenis serangan di mana pengguna berbahaya dapat menyuntikkan kode SQL ke dalam database Anda. Ini dapat digunakan untuk mencuri informasi 
dari database Anda atau melakukan tindakan pada database Anda. Sekali lagi Anda **tidak boleh** mempercayai input dari pengguna Anda! Selalu asumsikan mereka 
sedang mencari masalah. Anda dapat menggunakan prepared statements dalam objek `PDO` Anda akan mencegah SQL injection.

```php
// Assuming you have Flight::db() registered as your PDO object
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// If you use the PdoWrapper class, this can easily be done in one line
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// You can do the same thing with a PDO object with ? placeholders
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### Contoh Tidak Aman

Berikut adalah alasan mengapa kita menggunakan prepared statements SQL untuk melindungi dari contoh-contoh tidak berbahaya seperti di bawah ini:

```php
// end user fills out a web form.
// for the value of the form, the hacker puts in something like this:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// After the query is build it looks like this
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// It looks strange, but it's a valid query that will work. In fact,
// it's a very common SQL injection attack that will return all users.

var_dump($users); // this will dump all users in the database, not just the one single username
```

### Validasi Callback JSONP

Jika Anda menggunakan metode `Flight::jsonp()`, perhatikan bahwa Flight memvalidasi nama parameter callback JSONP terhadap regex allowlist yang ketat (`/^[A-Za-z_$][\w$.]{0,127}$/`). Setiap nama callback yang tidak sesuai dengan pola ini akan menyebabkan Flight melemparkan exception, mencegah injeksi JavaScript sembarang melalui nilai callback berbahaya.

Validasi ini sudah built-in dan tidak memerlukan konfigurasi tambahan, tetapi perlu diketahui saat debugging error yang tidak terduga dari endpoint JSONP.

### CORS

Cross-Origin Resource Sharing (CORS) adalah mekanisme yang memungkinkan banyak sumber daya (misalnya, font, JavaScript, dll.) pada halaman web untuk diminta 
dari domain lain di luar domain dari mana sumber daya berasal. Flight tidak memiliki fungsionalitas bawaan, 
tetapi ini dapat dengan mudah ditangani dengan hook yang berjalan sebelum metode `Flight::start()` dipanggil.

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
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
		// customize your allowed hosts here.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php or wherever you have your routes
$CorsUtil = new CorsUtil();

// This needs to be run before start runs.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Pengerasan Konfigurasi Flight

Flight mengekspos beberapa pengaturan engine yang memiliki implikasi keamanan langsung. Mengatur ini dengan benar adalah salah satu cara termudah untuk mengeraskan aplikasi Anda.

#### `flight.allow_method_override`

Secara default, Flight mengizinkan klien untuk mengganti metode HTTP dari permintaan menggunakan header `X-HTTP-Method-Override` atau field `_method` dalam body POST. Meskipun ini berguna untuk formulir HTML yang hanya dapat mengirim `GET`/`POST`, hal ini dapat berbahaya jika Anda tidak mengharapkannya — penyerang dapat memalsukan permintaan `DELETE` atau `PUT` melalui formulir biasa.

Jika aplikasi Anda tidak bergantung pada perilaku ini (misalnya Anda membangun API yang dikonsumsi oleh klien modern atau frontend JavaScript yang dapat mengirim verb HTTP apa pun), Anda sebaiknya menonaktifkannya:

```php
// In your index.php or bootstrap file, before Flight::start()
Flight::set('flight.allow_method_override', false);
```

Nilai default adalah `true` untuk kompatibilitas ke belakang, tetapi **mengatur ke `false` sangat direkomendasikan** untuk aplikasi apa pun yang tidak secara eksplisit memerlukan fitur override.

#### `flight.debug`

Flight memiliki pengaturan `flight.debug` yang mengontrol apakah informasi error detail (pesan exception, kode, dan stack trace lengkap) dirender di browser ketika exception yang tidak ditangani terjadi. Defaultnya adalah `false`, yang berarti hanya pesan `500 Internal Server Error` generik yang ditampilkan — tidak ada detail internal yang bocor ke klien.

Jangan pernah mengaktifkan ini pada server produksi. Gunakan hanya secara lokal atau di lingkungan staging:

```php
// Safe for local development only — NEVER in production
Flight::set('flight.debug', true);
```

Ketika `flight.debug` adalah `false` (default), Anda masih dapat menangkap error dengan mengaktifkan `flight.log_errors`:

```php
// Log errors server-side without exposing them to the client
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Konfigurasi produksi yang direkomendasikan

```php
// index.php or app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Penanganan Error
Sembunyikan detail error sensitif di produksi untuk menghindari kebocoran info ke penyerang. Di produksi, log error alih-alih menampilkannya dengan `display_errors` diatur ke `0`.

```php
// In your bootstrap.php or index.php

// add this to your app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Disable error display
    ini_set('log_errors', 1);     // Log errors instead
    ini_set('error_log', '/path/to/error.log');
}

// In your routes or controllers
// Use Flight::halt() for controlled error responses
Flight::halt(403, 'Access denied');
```

### Sanitasi Input
Jangan pernah mempercayai input pengguna. Sanitasi menggunakan [filter_var](https://www.php.net/manual/en/function.filter-var.php) sebelum memproses untuk mencegah data berbahaya masuk secara diam-diam.

```php

// Lets assume a $_POST request with $_POST['input'] and $_POST['email']

// Sanitize a string input
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitize an email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Hashing Password
Simpan password dengan aman dan verifikasi dengan aman menggunakan fungsi bawaan PHP seperti [password_hash](https://www.php.net/manual/en/function.password-hash.php) dan [password_verify](https://www.php.net/manual/en/function.password-verify.php). Password tidak boleh disimpan dalam teks biasa, juga tidak boleh dienkripsi dengan metode yang dapat dibalik. Hashing memastikan bahwa bahkan jika database Anda disusupi, password aktual tetap dilindungi.

```php
$password = Flight::request()->data->password;
// Hash a password when storing (e.g., during registration)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verify a password (e.g., during login)
if (password_verify($password, $stored_hash)) {
    // Password matches
}
```

### Rate Limiting
Lindungi dari serangan brute force atau denial-of-service dengan membatasi tingkat permintaan menggunakan cache.

```php
// Assuming you have flightphp/cache installed and registered
// Using flightphp/cache in a filter
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Reset after 60 seconds
});
```

## Lihat Juga
- [Sessions](/awesome-plugins/session) - Cara mengelola sesi pengguna dengan aman.
- [Templates](/learn/templates) - Menggunakan template untuk meng-escape output secara otomatis dan mencegah XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Interaksi database yang disederhanakan dengan prepared statements.
- [Middleware](/learn/middleware) - Cara menggunakan middleware untuk menyederhanakan proses menambahkan security headers.
- [Responses](/learn/responses) - Cara menyesuaikan respons HTTP dengan secure headers.
- [Requests](/learn/requests) - Cara menangani dan membersihkan input pengguna.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Fungsi PHP untuk sanitasi input.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Fungsi PHP untuk hashing password yang aman.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Fungsi PHP untuk memverifikasi password yang di-hash.

## Pemecahan Masalah
- Lihat bagian "Lihat Juga" di atas untuk informasi pemecahan masalah terkait masalah dengan komponen Flight Framework.

## Changelog
- v3.18.1 - Menambahkan bagian Flight Configuration Hardening yang mencakup `flight.allow_method_override`, `flight.debug`, dan validasi callback JSONP.
- v3.1.0 - Menambahkan bagian tentang CORS, Error Handling, Input Sanitization, Password Hashing, dan Rate Limiting.
- v2.0 - Menambahkan escaping untuk default views untuk mencegah XSS.