# Pengujian Unit di Flight PHP dengan PHPUnit

Panduan ini memperkenalkan pengujian unit di Flight PHP menggunakan [PHPUnit](https://phpunit.de/), ditujukan untuk pemula yang ingin memahami *mengapa* pengujian unit itu penting dan bagaimana menerapkannya secara praktis. Kami akan fokus pada pengujian *perilaku*—memastikan aplikasi Anda melakukan apa yang Anda harapkan, seperti mengirim email atau menyimpan data—bukan pada perhitungan sepele. Kami akan mulai dengan [route handler](/learn/routing) sederhana dan berlanjut ke [controller](/learn/routing) yang lebih kompleks, dengan menyertakan [dependency injection](/learn/dependency-injection-container) (DI) dan mocking layanan pihak ketiga.

## Mengapa Harus Pengujian Unit?

Pengujian unit memastikan kode Anda berperilaku sesuai yang diharapkan, menangkap bug sebelum masuk ke produksi. Ini sangat berharga di Flight, di mana routing yang ringan dan fleksibilitas dapat menyebabkan interaksi yang kompleks. Bagi pengembang individu atau tim, pengujian unit bertindak sebagai jaring pengaman, mendokumentasikan perilaku yang diharapkan dan mencegah regresi saat Anda meninjau kembali kode di kemudian hari. Pengujian unit juga meningkatkan desain: kode yang sulit diuji sering kali menandakan kelas yang terlalu kompleks atau terikat erat.

Berbeda dengan contoh sederhana (misalnya, menguji `x * y = z`), kami akan fokus pada perilaku dunia nyata, seperti memvalidasi input, menyimpan data, atau memicu tindakan seperti email. Tujuan kami adalah membuat pengujian mudah dipahami dan bermakna.

## Prinsip Panduan Umum

1. **Uji Perilaku, Bukan Implementasi**: Fokus pada hasil (misalnya, "email terkirim" atau "data tersimpan") daripada detail internal. Ini membuat pengujian tahan terhadap refactoring.
2. **Berhenti menggunakan `Flight::`**: Metode statis Flight sangat nyaman, tetapi membuat pengujian sulit. Anda harus terbiasa menggunakan variabel `$app` dari `$app = Flight::app();`. `$app` memiliki semua metode yang sama dengan `Flight::`. Anda tetap dapat menggunakan `$app->route()` atau `$this->app->json()` di controller Anda, dll. Anda juga harus menggunakan router Flight yang asli dengan `$router = $app->router()` dan kemudian Anda dapat menggunakan `$router->get()`, `$router->post()`, `$router->group()`, dll. Lihat [Routing](/learn/routing).
3. **Jaga Pengujian Tetap Cepat**: Pengujian yang cepat mendorong eksekusi yang sering. Hindari operasi lambat seperti panggilan basis data dalam pengujian unit. Jika Anda memiliki pengujian yang lambat, itu adalah tanda bahwa Anda sedang menulis pengujian integrasi, bukan pengujian unit. Pengujian integrasi adalah saat Anda benar-benar melibatkan basis data nyata, panggilan HTTP nyata, pengiriman email nyata, dll. Itu memiliki tempatnya, tetapi lambat dan bisa rapuh, artinya kadang gagal karena alasan yang tidak diketahui.
4. **Gunakan Nama yang Deskriptif**: Nama pengujian harus menggambarkan dengan jelas perilaku yang diuji. Ini meningkatkan keterbacaan dan kemudahan pemeliharaan.
5. **Hindari Global Seperti Menghindari Wabah**: Minimalkan penggunaan `$app->set()` dan `$app->get()`, karena mereka bertindak seperti state global, mengharuskan mock di setiap pengujian. Lebih suka DI atau wadah DI (lihat [Dependency Injection Container](/learn/dependency-injection-container)). Bahkan menggunakan metode `$app->map()` secara teknis adalah "global" dan harus dihindari demi DI. Gunakan pustaka sesi seperti [flightphp/session](https://github.com/flightphp/session) sehingga Anda dapat mock objek sesi dalam pengujian Anda. **Jangan** memanggil [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) langsung dalam kode Anda karena itu memasukkan variabel global ke dalam kode Anda, membuat pengujian menjadi sulit.
6. **Gunakan Dependency Injection**: Suntikkan dependensi (misalnya, [`PDO`](https://www.php.net/manual/en/class.pdo.php), pengirim email) ke dalam controller untuk mengisolasi logika dan menyederhanakan mocking. Jika Anda memiliki kelas dengan terlalu banyak dependensi, pertimbangkan untuk refactoring menjadi kelas-kelas yang lebih kecil yang masing-masing memiliki satu tanggung jawab mengikuti [prinsip SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Mock Layanan Pihak Ketiga**: Mock basis data, klien HTTP (cURL), atau layanan email untuk menghindari panggilan eksternal. Uji satu atau dua lapisan ke dalam, tetapi biarkan logika inti Anda berjalan. Misalnya, jika aplikasi Anda mengirim pesan teks, Anda **TIDAK** ingin benar-benar mengirim pesan teks setiap kali menjalankan pengujian karena biayanya akan bertambah (dan akan lebih lambat). Sebagai gantinya, mock layanan pesan teks dan cukup verifikasi bahwa kode Anda memanggil layanan pesan teks dengan parameter yang benar.
8. **Targetkan Cakupan Tinggi, Bukan Kesempurnaan**: Cakupan baris 100% itu bagus, tetapi tidak benar-benar berarti bahwa semua yang ada di kode Anda diuji sebagaimana mestinya (silahkan riset [branch/path coverage di PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Prioritaskan perilaku penting (misalnya, pendaftaran pengguna, respons API, dan menangkap respons yang gagal).
9. **Gunakan Controller untuk Routes**: Dalam definisi route Anda, gunakan controller bukan closure. `flight\Engine $app` disuntikkan ke setiap controller melalui konstruktor secara default. Dalam pengujian, gunakan `$app = new Flight\Engine()` untuk membuat instance Flight dalam pengujian, suntikkan ke controller Anda, dan panggil metode secara langsung (misalnya, `$controller->register()`). Lihat [Extending Flight](/learn/extending) dan [Routing](/learn/routing).
10. **Pilih gaya mocking dan pertahankan**: PHPUnit mendukung beberapa gaya mocking (misalnya, prophecy, mock bawaan), atau Anda dapat menggunakan kelas anonim yang memiliki manfaat sendiri seperti penyelesaian kode, gagal jika Anda mengubah definisi metode, dll. Tetaplah konsisten di seluruh pengujian Anda. Lihat [PHPUnit Mock Objects](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Gunakan visibilitas `protected` untuk metode/properti yang ingin Anda uji di subclass**: Ini memungkinkan Anda untuk menimpanya di subclass pengujian tanpa membuatnya public, ini sangat berguna untuk mock kelas anonim.

## Menyiapkan PHPUnit

Pertama, siapkan [PHPUnit](https://phpunit.de/) di proyek Flight PHP Anda menggunakan Composer untuk pengujian yang mudah. Lihat [panduan Memulai PHPUnit](https://phpunit.readthedocs.io/en/12.3/installation.html) untuk detail lebih lanjut.

1. Di direktori proyek Anda, jalankan:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Ini menginstal PHPUnit terbaru sebagai dependensi pengembangan.

2. Buat direktori `tests` di akar proyek Anda untuk file pengujian.

3. Tambahkan skrip pengujian ke `composer.json` untuk kenyamanan:
   ```json
   // konten composer.json lainnya
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Buat file `phpunit.xml` di akar:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Sekarang setelah pengujian Anda dibuat, Anda dapat menjalankan `composer test` untuk mengeksekusi pengujian.

## Menguji Penangan Route Sederhana

Mari kita mulai dengan [route](/learn/routing) dasar yang memvalidasi input email pengguna. Kita akan menguji perilakunya: mengembalikan pesan sukses untuk email yang valid dan pesan error untuk email yang tidak valid. Untuk validasi email, kita menggunakan [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
	protected $app;

	public function __construct(flight\Engine $app) {
		$this->app = $app;
	}

	public function register() {
		$email = $this->app->request()->data->email;
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

Untuk menguji ini, buat file pengujian. Lihat [Unit Testing dan Prinsip SOLID](/learn/unit-testing-and-solid-principles) untuk lebih lanjut tentang menyusun pengujian:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simulasikan data POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Simulasikan data POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Poin-Poin Penting**:
- Kita mensimulasikan data POST menggunakan kelas request. Jangan gunakan global seperti `$_POST`, `$_GET`, dll karena itu membuat pengujian lebih rumit (Anda harus selalu mengatur ulang nilai-nilai tersebut atau pengujian lain bisa gagal).
- Semua controller secara default akan memiliki instance `flight\Engine` yang disuntikkan ke dalamnya bahkan tanpa pengaturan wadah DI. Ini membuat pengujian controller secara langsung menjadi lebih mudah.
- Tidak ada penggunaan `Flight::` sama sekali, membuat kode lebih mudah diuji.
- Pengujian memverifikasi perilaku: status dan pesan yang benar untuk email valid/tidak valid.

Jalankan `composer test` untuk memverifikasi bahwa route berperilaku sesuai yang diharapkan. Untuk lebih lanjut tentang [requests](/learn/requests) dan [responses](/learn/responses) di Flight, lihat dokumen terkait.

## Menggunakan Dependency Injection untuk Controller yang Dapat Diuji

Untuk skenario yang lebih kompleks, gunakan [dependency injection](/learn/dependency-injection-container) (DI) untuk membuat controller dapat diuji. Hindari global Flight (misalnya, `Flight::set()`, `Flight::map()`, `Flight::register()`) karena mereka bertindak seperti state global, mengharuskan mock untuk setiap pengujian. Sebagai gantinya, gunakan wadah DI Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) atau DI manual.

Mari kita gunakan [`flight\database\SimplePdo`](/learn/simple-pdo) alih-alih PDO mentah. Helper ini jauh lebih mudah untuk di-mock dan diuji unit (dan lebih disukai daripada `PdoWrapper` yang sudah usang).

Berikut adalah controller yang menyimpan pengguna ke basis data dan mengirim email selamat datang:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// menambahkan return di sini membantu pengujian unit untuk menghentikan eksekusi
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Poin-Poin Penting**:
- Controller bergantung pada instance [`SimplePdo`](/learn/simple-pdo) dan `MailerInterface` (sebuah layanan email pihak ketiga pura-pura).
- Dependensi disuntikkan melalui konstruktor, menghindari global.

### Menguji Controller dengan Mock

Sekarang, mari kita uji perilaku `UserController`: memvalidasi email, menyimpan ke basis data, dan mengirim email. Kita akan mock basis data dan mailer untuk mengisolasi controller.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Terkadang mencampur gaya mocking diperlukan
		// Di sini kita menggunakan mock bawaan PHPUnit untuk PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Menggunakan kelas anonim untuk mock SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Saat kita mock dengan cara ini, kita tidak benar-benar melakukan panggilan basis data.
			// Kita dapat mengatur ini lebih lanjut untuk mengubah mock PDOStatement guna mensimulasikan kegagalan, dll.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Sebuah konstruktor kosong melewati konstruktor induk
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// Perlu memetakan jsonHalt untuk menghindari keluar
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**Poin-Poin Penting**:
- Kita mock `SimplePdo` dan `MailerInterface` untuk menghindari panggilan basis data atau email sungguhan.
- Pengujian memverifikasi perilaku: email valid memicu penyisipan basis data dan pengiriman email; email tidak valid melewati keduanya.
- Mock dependensi pihak ketiga (misalnya, `SimplePdo`, `MailerInterface`), membiarkan logika controller berjalan.

### Terlalu Banyak Mocking

Berhati-hatilah untuk tidak terlalu banyak mem-mock kode Anda. Saya akan memberikan contoh di bawah ini tentang mengapa ini bisa menjadi hal yang buruk menggunakan `UserController` kita. Kita akan mengubah pemeriksaan itu menjadi metode bernama `isEmailValid` (menggunakan `filter_var`) dan penambahan baru lainnya menjadi metode terpisah bernama `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// menambahkan return di sini membantu pengujian unit untuk menghentikan eksekusi
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

Dan sekarang pengujian unit yang terlalu di-mock yang sebenarnya tidak menguji apa pun:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// kita melewatkan dependency injection tambahan di sini karena itu "mudah"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Lewati dependensi di konstruktor
			public function __construct($app) {
				$this->app = $app;
			}

			// Kita akan memaksa ini menjadi valid.
			protected function isEmailValid($email) {
				return true; // Selalu kembalikan true, melewati validasi sungguhan
			}

			// Lewati panggilan DB dan mailer yang sebenarnya
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

Hore, kita punya pengujian unit dan semuanya lulus! Tetapi tunggu, bagaimana jika saya benar-benar mengubah cara kerja internal `isEmailValid` atau `registerUser`? Pengujian saya masih akan lulus karena saya sudah mem-mock semua fungsionalitas. Biarkan saya menunjukkan maksud saya.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... metode lainnya ...

	protected function isEmailValid($email) {
		// Logika yang diubah
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Sekarang hanya boleh memiliki domain tertentu
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Jika saya menjalankan pengujian unit di atas, semuanya tetap lulus! Tetapi karena saya tidak menguji perilaku (benar-benar membiarkan sebagian kode berjalan), saya berpotensi membuat bug yang siap terjadi di produksi. Pengujian harus dimodifikasi untuk memperhitungkan perilaku baru, dan juga kebalikan dari saat perilaku tidak sesuai yang kita harapkan.

## Contoh Lengkap

Anda dapat menemukan contoh lengkap proyek Flight PHP dengan pengujian unit di GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide). Untuk pemahaman yang lebih mendalam, lihat [Unit Testing dan Prinsip SOLID](/learn/unit-testing-and-solid-principles).

## Kesalahan Umum

- **Terlalu Banyak Mocking**: Jangan mem-mock setiap dependensi; biarkan sebagian logika (misalnya, validasi controller) berjalan untuk menguji perilaku nyata. Lihat [Unit Testing dan Prinsip SOLID](/learn/unit-testing-and-solid-principles).
- **State Global**: Menggunakan variabel global PHP (misalnya, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) secara berlebihan membuat pengujian rapuh. Hal yang sama berlaku untuk `Flight::`. Refactor untuk mengoper dependensi secara eksplisit.
- **Setup yang Rumit**: Jika setup pengujian merepotkan, kelas Anda mungkin memiliki terlalu banyak dependensi atau tanggung jawab yang melanggar [prinsip SOLID](/learn/unit-testing-and-solid-principles).

## Meningkatkan Skala dengan Pengujian Unit

Pengujian unit sangat berguna dalam proyek yang lebih besar atau saat meninjau kembali kode setelah berbulan-bulan. Pengujian unit mendokumentasikan perilaku dan menangkap regresi, menghemat Anda dari belajar ulang aplikasi. Untuk pengembang solo, uji jalur kritis (misalnya, pendaftaran pengguna, pemrosesan pembayaran). Untuk tim, pengujian memastikan perilaku yang konsisten di seluruh kontribusi. Lihat [Mengapa Framework?](/learn/why-frameworks) untuk lebih lanjut tentang manfaat menggunakan framework dan pengujian.

Kontribusikan tips pengujian Anda sendiri ke repositori dokumentasi Flight PHP!

_Ditulis oleh [n0nag0n](https://github.com/n0nag0n) 2025_