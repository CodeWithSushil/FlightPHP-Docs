# Routing

## Ikhtisar

Routing di Flight PHP memetakan pola URL ke fungsi callback atau metode kelas, memungkinkan penanganan permintaan yang cepat dan sederhana. Ini dirancang dengan overhead minimal, ramah untuk pemula, dan dapat diperluas tanpa dependensi eksternal.

## Pemahaman

Routing adalah mekanisme inti yang menghubungkan permintaan HTTP ke logika aplikasi Anda di Flight. Dengan mendefinisikan rute, Anda menentukan bagaimana URL yang berbeda memicu kode tertentu, baik melalui fungsi, metode kelas, atau aksi kontroler. Sistem routing Flight fleksibel, mendukung pola dasar, parameter bernama, ekspresi reguler, dan fitur lanjutan seperti injeksi dependensi dan routing resourceful. Pendekatan ini menjaga kode Anda tetap terorganisir dan mudah dipelihara, sambil tetap cepat dan sederhana untuk pemula serta dapat diperluas untuk pengguna tingkat lanjut.

> **Catatan:** Ingin memahami lebih lanjut tentang routing? Lihat halaman ["mengapa framework?"](/learn/why-frameworks) untuk penjelasan yang lebih mendalam.

## Penggunaan Dasar

### Mendefinisikan Rute Sederhana

Routing dasar di Flight dilakukan dengan mencocokkan pola URL dengan fungsi callback atau array dari kelas dan metode.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Rute dicocokkan sesuai urutan pendefinisiannya. Rute pertama yang cocok dengan permintaan akan dipanggil.

### Menggunakan Fungsi sebagai Callback

Callback dapat berupa objek apa pun yang dapat dipanggil (callable). Jadi Anda dapat menggunakan fungsi biasa:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Menggunakan Kelas dan Metode sebagai Kontroler

Anda dapat menggunakan metode (statis atau tidak) dari sebuah kelas:

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// atau
Flight::route('/', [ GreetingController::class, 'hello' ]); // metode yang disarankan
// atau
Flight::route('/', [ 'GreetingController::hello' ]);
// atau 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Atau dengan membuat objek terlebih dahulu lalu memanggil metodenya:

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **Catatan:** Secara default ketika sebuah kontroler dipanggil di dalam framework, kelas `flight\Engine` selalu diinjeksikan kecuali Anda menentukan melalui [wadah injeksi dependensi](/learn/dependency-injection-container)

### Routing Khusus Metode

Secara default, pola rute dicocokkan dengan semua metode permintaan. Anda dapat merespons
metode tertentu dengan menempatkan pengidentifikasi sebelum URL.

```php
Flight::route('GET /', function () {
  echo 'Saya menerima permintaan GET.';
});

Flight::route('POST /', function () {
  echo 'Saya menerima permintaan POST.';
});

// Anda tidak dapat menggunakan Flight::get() untuk rute karena itu adalah metode
//    untuk mendapatkan variabel, bukan membuat rute.
Flight::post('/', function() { /* kode */ });
Flight::patch('/', function() { /* kode */ });
Flight::put('/', function() { /* kode */ });
Flight::delete('/', function() { /* kode */ });
```

Anda juga dapat memetakan beberapa metode ke satu callback dengan menggunakan pemisah `|`:

```php
Flight::route('GET|POST /', function () {
  echo 'Saya menerima permintaan GET atau POST.';
});
```

### Penanganan Khusus untuk Permintaan HEAD dan OPTIONS

Flight menyediakan penanganan bawaan untuk permintaan HTTP `HEAD` dan `OPTIONS`:

#### Permintaan HEAD

- **Permintaan HEAD** diperlakukan sama seperti permintaan `GET`, tetapi Flight secara otomatis menghapus body respons sebelum mengirimkannya ke klien.
- Ini berarti Anda dapat mendefinisikan rute untuk `GET`, dan permintaan HEAD ke URL yang sama hanya akan mengembalikan header (tanpa konten), sesuai dengan standar HTTP.

```php
Flight::route('GET /info', function() {
    echo 'Ini adalah beberapa info!';
});
// Permintaan HEAD ke /info akan mengembalikan header yang sama, tetapi tanpa body.
```

#### Permintaan OPTIONS

Permintaan `OPTIONS` ditangani secara otomatis oleh Flight untuk setiap rute yang didefinisikan.
- Ketika permintaan OPTIONS diterima, Flight merespons dengan status `204 No Content` dan header `Allow` yang mencantumkan semua metode HTTP yang didukung untuk rute tersebut.
- Anda tidak perlu mendefinisikan rute terpisah untuk OPTIONS.

```php
// Untuk rute yang didefinisikan sebagai:
Flight::route('GET|POST /users', function() { /* ... */ });

// Permintaan OPTIONS ke /users akan merespons dengan:
//
// Status: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Menggunakan Objek Router

Selain itu, Anda dapat mengambil objek Router yang memiliki beberapa metode pembantu untuk Anda gunakan:

```php

$router = Flight::router();

// memetakan semua metode sama seperti Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// Permintaan GET
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* kode */});
$router->put('/users/update/@id', 	function() { /* kode */});
$router->delete('/users/@id', 		function() { /* kode */});
$router->patch('/users/@id', 		function() { /* kode */});
```

### Ekspresi Reguler (Regex)

Anda dapat menggunakan ekspresi reguler di dalam rute Anda:

```php
Flight::route('/user/[0-9]+', function () {
  // Ini akan cocok dengan /user/1234
});
```

Meskipun metode ini tersedia, disarankan untuk menggunakan parameter bernama, atau
parameter bernama dengan ekspresi reguler, karena lebih mudah dibaca dan dipelihara.

### Parameter Bernama

Anda dapat menentukan parameter bernama di dalam rute Anda yang akan diteruskan ke
fungsi callback Anda. **Ini lebih untuk keterbacaan rute daripada apa pun
lainnya. Silakan lihat bagian tentang catatan penting di bawah ini.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Anda juga dapat menyertakan ekspresi reguler dengan parameter bernama Anda menggunakan
pemisah `:`:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Ini akan cocok dengan /bob/123
  // Tetapi tidak akan cocok dengan /bob/12345
});
```

> **Catatan:** Pencocokan grup regex `()` dengan parameter posisional tidak didukung. Contoh: `:'\(`

#### Catatan Penting

Meskipun pada contoh di atas, tampaknya `@name` terikat langsung dengan variabel `$name`, sebenarnya tidak demikian. Urutan parameter dalam fungsi callback adalah yang menentukan apa yang diteruskan ke fungsi tersebut. Jika Anda menukar urutan parameter dalam fungsi callback, variabel juga akan tertukar. Berikut contohnya:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

Dan jika Anda mengunjungi URL berikut: `/bob/123`, hasilnya akan menjadi `hello, 123 (bob)!`. 
_Harap berhati-hati_ saat Anda menyiapkan rute dan fungsi callback Anda!

### Parameter Opsional

Anda dapat menentukan parameter bernama yang opsional untuk dicocokkan dengan membungkus
segmen dalam tanda kurung.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Ini akan cocok dengan URL berikut:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Parameter opsional apa pun yang tidak cocok akan diteruskan sebagai `NULL`.

### Routing Wildcard

Pencocokan hanya dilakukan pada segmen URL individual. Jika Anda ingin mencocokkan beberapa
segmen, Anda dapat menggunakan wildcard `*`.

```php
Flight::route('/blog/*', function () {
  // Ini akan cocok dengan /blog/2000/02/01
});
```

Untuk mengarahkan semua permintaan ke satu callback, Anda dapat melakukannya:

```php
Flight::route('*', function () {
  // Lakukan sesuatu
});
```

### Penanganan 404 Tidak Ditemukan

Secara default, jika URL tidak dapat ditemukan, Flight akan mengirimkan respons `HTTP 404 Not Found` yang sangat sederhana dan polos.
Jika Anda menginginkan respons 404 yang lebih khusus, Anda dapat [memetakan](/learn/extending) metode `notFound` Anda sendiri:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Anda juga dapat menggunakan Flight::render() dengan template khusus.
    $output = <<<HTML
		<h1>404 Tidak Ditemukan Kustom Saya</h1>
		<h3>Halaman yang Anda minta {$url} tidak dapat ditemukan.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Penanganan Metode Tidak Ditemukan

Secara default, jika URL ditemukan tetapi metode tidak diizinkan, Flight akan mengirimkan respons `HTTP 405 Method Not Allowed` yang sangat sederhana dan polos (Contoh: Metode Tidak Diizinkan. Metode yang Diizinkan adalah: GET, POST). Ini juga akan menyertakan header `Allow` dengan metode yang diizinkan untuk URL tersebut.

Jika Anda menginginkan respons 405 yang lebih khusus, Anda dapat [memetakan](/learn/extending) metode `methodNotFound` Anda sendiri:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Anda juga dapat menggunakan Flight::render() dengan template khusus.
	$output = <<<HTML
		<h1>405 Metode Tidak Diizinkan Kustom Saya</h1>
		<h3>Metode yang Anda minta untuk {$url} tidak diizinkan.</h3>
		<p>Metode yang Diizinkan adalah: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## Penggunaan Lanjutan

### Injeksi Dependensi dalam Rute

Jika Anda ingin menggunakan injeksi dependensi melalui wadah (PSR-11, PHP-DI, Dice, dll), satu-satunya
jenis rute yang tersedia adalah membuat objek secara langsung sendiri
dan menggunakan wadah untuk membuat objek Anda atau Anda dapat menggunakan string untuk mendefinisikan kelas dan
metode yang akan dipanggil. Anda dapat mengunjungi halaman [Injeksi Dependensi](/learn/dependency-injection-container) untuk
informasi lebih lanjut.

Berikut contoh singkatnya:

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// lakukan sesuatu dengan $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! Nama saya adalah {$name}!";
	}
}

// index.php

// Siapkan wadah dengan parameter apa pun yang Anda butuhkan
// Lihat halaman Injeksi Dependensi untuk informasi lebih lanjut tentang PSR-11
$dice = new \Dice\Dice();

// Jangan lupa untuk menetapkan ulang variabel dengan '$dice = '!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Daftarkan penangan wadah
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Rute seperti biasa
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// atau
Flight::route('/hello/@id', 'Greeting->hello');
// atau
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Meneruskan Eksekusi ke Rute Berikutnya

<span class="badge bg-warning">Usang</span>
Anda dapat meneruskan eksekusi ke rute berikutnya yang cocok dengan mengembalikan `true` dari
fungsi callback Anda.

```php
Flight::route('/user/@name', function (string $name) {
  // Periksa beberapa kondisi
  if ($name !== "Bob") {
    // Lanjutkan ke rute berikutnya
    return true;
  }
});

Flight::route('/user/*', function () {
  // Ini akan dipanggil
});
```

Sekarang disarankan untuk menggunakan [middleware](/learn/middleware) untuk menangani kasus penggunaan yang kompleks seperti ini.

### Alias Rute

Dengan menetapkan alias ke sebuah rute, Anda nantinya dapat memanggil alias tersebut di dalam aplikasi Anda secara dinamis untuk dibuat kemudian di kode Anda (contoh: tautan dalam template HTML, atau menghasilkan URL pengalihan).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// atau 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// nanti di suatu tempat dalam kode
class UserController {
	public function update() {

		// kode untuk menyimpan pengguna...
		$id = $user['id']; // 5 sebagai contoh

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // akan mengembalikan '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

Ini sangat membantu jika URL Anda berubah. Pada contoh di atas, misalkan pengguna dipindahkan ke `/admin/users/@id`.
Dengan alias yang terpasang pada rute, Anda tidak perlu lagi mencari semua URL lama di kode Anda dan mengubahnya karena alias sekarang akan mengembalikan `/admin/users/5` seperti pada contoh di atas.

Alias rute juga tetap berfungsi dalam grup:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// atau
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Memeriksa Informasi Rute

Jika Anda ingin memeriksa informasi rute yang cocok, ada 2 cara yang dapat Anda lakukan:

1. Anda dapat menggunakan properti `executedRoute` pada objek `Flight::router()`.
2. Anda dapat meminta objek rute untuk diteruskan ke callback Anda dengan memberikan `true` sebagai parameter ketiga dalam metode rute. Objek rute akan selalu menjadi parameter terakhir yang diteruskan ke fungsi callback Anda.

#### `executedRoute`

```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Lakukan sesuatu dengan $route

  // Array metode HTTP yang dicocokkan
  $route->methods;

  // Array parameter bernama
  $route->params;

  // Ekspresi reguler yang cocok
  $route->regex;

  // Berisi konten dari '*' apa pun yang digunakan dalam pola URL
  $route->splat;

  // Menampilkan jalur url....jika Anda benar-benar membutuhkannya
  $route->pattern;

  // Menampilkan middleware apa yang ditetapkan untuk ini
  $route->middleware;

  // Menampilkan alias yang ditetapkan untuk rute ini
  $route->alias;
});
```

> **Catatan:** Properti `executedRoute` hanya akan diatur setelah sebuah rute dieksekusi. Jika Anda mencoba mengaksesnya sebelum rute dieksekusi, nilainya akan `NULL`. Anda juga dapat menggunakan executedRoute di dalam [middleware](/learn/middleware)!

#### Berikan `true` pada definisi rute

```php
Flight::route('/', function(\flight\net\Route $route) {
  // Array metode HTTP yang dicocokkan
  $route->methods;

  // Array parameter bernama
  $route->params;

  // Ekspresi reguler yang cocok
  $route->regex;

  // Berisi konten dari '*' apa pun yang digunakan dalam pola URL
  $route->splat;

  // Menampilkan jalur url....jika Anda benar-benar membutuhkannya
  $route->pattern;

  // Menampilkan middleware apa yang ditetapkan untuk ini
  $route->middleware;

  // Menampilkan alias yang ditetapkan untuk rute ini
  $route->alias;
}, true);// <-- Parameter true ini yang membuat itu terjadi
```

### Pengelompokan Rute dan Middleware

Mungkin ada saatnya Anda ingin mengelompokkan rute yang terkait (seperti `/api/v1`).
Anda dapat melakukannya dengan menggunakan metode `group`:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Cocok dengan /api/v1/users
  });

  Flight::route('/posts', function () {
	// Cocok dengan /api/v1/posts
  });
});
```

Anda bahkan dapat menumpuk grup di dalam grup:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() mendapatkan variabel, bukan membuat rute! Lihat konteks objek di bawah
	Flight::route('GET /users', function () {
	  // Cocok dengan GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Cocok dengan POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Cocok dengan PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() mendapatkan variabel, bukan membuat rute! Lihat konteks objek di bawah
	Flight::route('GET /users', function () {
	  // Cocok dengan GET /api/v2/users
	});
  });
});
```

#### Pengelompokan dengan Konteks Objek

Anda tetap dapat menggunakan pengelompokan rute dengan objek `Engine` dengan cara berikut:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // gunakan variabel $router
  $router->get('/users', function () {
	// Cocok dengan GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Cocok dengan POST /api/v1/posts
  });
});
```

> **Catatan:** Ini adalah metode yang disarankan untuk mendefinisikan rute dan grup dengan objek `$router`.

#### Pengelompokan dengan Middleware

Anda juga dapat menetapkan middleware ke sekelompok rute:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Cocok dengan /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // atau [ new MyAuthMiddleware() ] jika Anda ingin menggunakan sebuah instance
```

Lihat detail lebih lanjut di halaman [grup middleware](/learn/middleware#grouping-middleware).

### Routing Resource

Anda dapat membuat satu set rute untuk sebuah resource menggunakan metode `resource`. Ini akan membuat
satu set rute untuk sebuah resource yang mengikuti konvensi RESTful.

Untuk membuat resource, lakukan hal berikut:

```php
Flight::resource('/users', UsersController::class);
```

Dan apa yang akan terjadi di latar belakang adalah ia akan membuat rute-rute berikut:

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

Dan kontroler Anda akan menggunakan metode-metode berikut:

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **Catatan:** Anda dapat melihat rute yang baru ditambahkan dengan `runway` dengan menjalankan `php runway routes`.

#### Menyesuaikan Rute Resource

Ada beberapa opsi untuk mengonfigurasi rute resource.

##### Alias Dasar (Alias Base)

Anda dapat mengonfigurasi `aliasBase`. Secara default alias adalah bagian terakhir dari URL yang ditentukan.
Misalnya `/users/` akan menghasilkan `aliasBase` dari `users`. Ketika rute-rute ini dibuat,
aliasnya adalah `users.index`, `users.create`, dst. Jika Anda ingin mengubah aliasnya, atur `aliasBase`
ke nilai yang Anda inginkan.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only dan Except

Anda juga dapat menentukan rute mana yang ingin Anda buat dengan menggunakan opsi `only` dan `except`.

```php
// Whitelist hanya metode ini dan blacklist sisanya
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Blacklist hanya metode ini dan whitelist sisanya
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

Ini pada dasarnya adalah opsi whitelisting dan blacklisting sehingga Anda dapat menentukan rute mana yang ingin dibuat.

##### Middleware

Anda juga dapat menentukan middleware yang akan dijalankan pada setiap rute yang dibuat oleh metode `resource`.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Respons Streaming

Anda sekarang dapat melakukan streaming respons ke klien menggunakan `stream()` atau `streamWithHeaders()`.
Ini berguna untuk mengirim file besar, proses yang berjalan lama, atau menghasilkan respons yang besar.
Streaming rute ditangani sedikit berbeda dari rute biasa.

> **Catatan:** Respons streaming hanya tersedia jika Anda telah mengatur [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) menjadi `false`.

#### Streaming dengan Header Manual

Anda dapat melakukan streaming respons ke klien dengan menggunakan metode `stream()` pada sebuah rute. Jika Anda
melakukan ini, Anda harus mengatur semua header secara manual sebelum Anda mengeluarkan apa pun ke klien.
Ini dilakukan dengan fungsi PHP `header()` atau metode `Flight::response()->setRealHeader()`.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// tentu saja Anda harus membersihkan jalur file dan sebagainya.
	$fileNameSafe = basename($filename);

	// Jika Anda memiliki header tambahan yang perlu diatur di sini setelah rute dieksekusi
	// Anda harus mendefinisikannya sebelum apa pun dicetak.
	// Semuanya harus berupa panggilan mentah ke fungsi header() atau
	// panggilan ke Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// atau
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File tidak ditemukan');
	}

	// atur panjang konten secara manual jika Anda mau
	header('Content-Length: '.filesize($filePath));
	// atau
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Streaming file ke klien saat dibaca
	readfile($filePath);

// Ini adalah baris ajaibnya
})->stream();
```

#### Streaming dengan Header

Anda juga dapat menggunakan metode `streamWithHeaders()` untuk mengatur header sebelum Anda memulai streaming.

```php
Flight::route('/stream-users', function() {

	// Anda dapat menambahkan header tambahan apa pun di sini
	// Anda hanya harus menggunakan header() atau Flight::response()->setRealHeader()

	// bagaimanapun Anda mengambil data Anda, hanya sebagai contoh...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Ini diperlukan untuk mengirim data ke klien
		ob_flush();
	}
	echo '}';

// Ini adalah cara Anda mengatur header sebelum memulai streaming.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// kode status opsional, defaultnya 200
	'status' => 200
]);
```

## Lihat Juga

- [Middleware](/learn/middleware) - Menggunakan middleware dengan rute untuk autentikasi, pencatatan, dll.
- [Injeksi Dependensi](/learn/dependency-injection-container) - Menyederhanakan pembuatan dan pengelolaan objek dalam rute.
- [Mengapa Framework?](/learn/why-frameworks) - Memahami manfaat menggunakan framework seperti Flight.
- [Ekstensi](/learn/extending) - Cara memperluas Flight dengan fungsionalitas Anda sendiri termasuk metode `notFound`.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - Fungsi PHP untuk pencocokan ekspresi reguler.

## Pemecahan Masalah

- Parameter rute dicocokkan berdasarkan urutan, bukan berdasarkan nama. Pastikan urutan parameter callback sesuai dengan definisi rute.
- Menggunakan `Flight::get()` tidak mendefinisikan rute; gunakan `Flight::route('GET /...')` untuk routing atau konteks objek Router dalam grup (misalnya `$router->get(...)`).
- Properti executedRoute hanya diatur setelah rute dieksekusi; nilainya NULL sebelum eksekusi.
- Streaming memerlukan fungsionalitas buffering output lama Flight untuk dinonaktifkan (`flight.v2.output_buffering = false`).
- Untuk injeksi dependensi, hanya definisi rute tertentu yang mendukung pembuatan instance berbasis wadah.

### 404 Tidak Ditemukan atau Perilaku Rute yang Tidak Terduga

Jika Anda melihat kesalahan 404 Not Found (tetapi Anda bersumpah demi hidup Anda bahwa itu benar-benar ada dan itu bukan salah ketik), ini sebenarnya bisa menjadi masalah
dengan Anda mengembalikan nilai di endpoint rute Anda alih-alih hanya mencetaknya. Alasan untuk ini disengaja tetapi bisa mengecoh beberapa pengembang.

```php
Flight::route('/hello', function(){
	// Ini dapat menyebabkan kesalahan 404 Not Found
	return 'Hello World';
});

// Yang mungkin Anda inginkan
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

Alasannya adalah karena mekanisme khusus yang dibangun ke dalam router yang memperlakukan keluaran yang dikembalikan sebagai sinyal untuk "lanjut ke rute berikutnya".
Anda dapat melihat perilaku yang didokumentasikan di bagian [Routing](/learn/routing#passing).

## Riwayat Perubahan

- v3: Menambahkan routing resource, alias rute, dan dukungan streaming, grup rute, dan dukungan middleware.
- v1: Sebagian besar fitur dasar tersedia.