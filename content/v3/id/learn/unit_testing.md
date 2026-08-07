# Pengujian Unit

## Ringkasan

Pengujian unit di Flight membantu Anda memastikan aplikasi Anda berperilaku seperti yang diharapkan, menemukan bug sejak dini, dan membuat basis kode Anda lebih mudah dipelihara. Flight dirancang untuk bekerja dengan lancar dengan [PHPUnit](https://phpunit.de/), kerangka pengujian PHP yang paling populer.

## Pemahaman

Pengujian unit memeriksa perilaku bagian-bagian kecil dari aplikasi Anda (seperti pengontrol atau layanan) secara terpisah. Di Flight, ini berarti menguji bagaimana rute, pengontrol, dan logika Anda merespons berbagai masukan—tanpa bergantung pada status global atau layanan eksternal nyata.

Prinsip-prinsip utama:
- **Uji perilaku, bukan implementasi:** Fokus pada apa yang dilakukan kode Anda, bukan bagaimana kode itu melakukannya.
- **Hindari status global:** Gunakan injeksi ketergantungan alih-alih `Flight::set()` atau `Flight::get()`.
- **Tirukan layanan eksternal:** Ganti hal-hal seperti basis data atau pengirim email dengan objek pengganti (test doubles).
- **Jaga pengujian tetap cepat dan fokus:** Pengujian unit tidak boleh mengakses basis data atau API nyata.

## Penggunaan Dasar

### Menyiapkan PHPUnit

1. Instal PHPUnit dengan Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Buat direktori `tests` di akar proyek Anda.
3. Tambahkan skrip pengujian ke `composer.json` Anda:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Buat file `phpunit.xml`:
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

Sekarang Anda dapat menjalankan pengujian Anda dengan `composer test`.

### Menguji Penangan Rute Sederhana

Misalkan Anda memiliki rute yang memvalidasi email:

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
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Pengujian sederhana untuk pengontrol ini:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Tips:**
- Simulasikan data POST menggunakan `$app->request()->data`.
- Hindari menggunakan statis `Flight::` dalam pengujian Anda—gunakan instance `$app`.

### Menggunakan Injeksi Ketergantungan untuk Pengontrol yang Dapat Diuji

Suntikkan ketergantungan (seperti basis data atau pengirim email) ke dalam pengontrol Anda agar mudah ditiru (mock) dalam pengujian:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

Dan pengujian dengan mock:

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Penggunaan Lanjutan

- **Mocking:** Gunakan mock bawaan PHPUnit atau kelas anonim untuk menggantikan ketergantungan.
- **Menguji pengontrol secara langsung:** Buat instance pengontrol dengan `Engine` baru dan mock ketergantungan.
- **Hindari mocking berlebihan:** Biarkan logika nyata berjalan jika memungkinkan; hanya mock layanan eksternal.

## Lihat Juga

- [Panduan Pengujian Unit](/guides/unit-testing) - Panduan komprehensif tentang praktik terbaik pengujian unit.
- [Kontainer Injeksi Ketergantungan](/learn/dependency-injection-container) - Cara menggunakan DIC untuk mengelola ketergantungan dan meningkatkan kemampuan pengujian.
- [Memperluas](/learn/extending) - Cara menambahkan helper Anda sendiri atau mengganti kelas inti.
- [SimplePdo](/learn/simple-pdo) - Menyederhanakan interaksi basis data dan lebih mudah ditiru dalam pengujian.
- [Permintaan](/learn/requests) - Menangani permintaan HTTP di Flight.
- [Respons](/learn/responses) - Mengirim respons kepada pengguna.
- [Pengujian Unit dan Prinsip SOLID](/learn/unit-testing-and-solid-principles) - Pelajari bagaimana prinsip SOLID dapat meningkatkan pengujian unit Anda.

## Pemecahan Masalah

- Hindari menggunakan status global (`Flight::set()`, `$_SESSION`, dll.) dalam kode dan pengujian Anda.
- Jika pengujian Anda lambat, Anda mungkin menulis pengujian integrasi—mock layanan eksternal untuk menjaga pengujian unit tetap cepat.
- Jika penyiapan pengujian rumit, pertimbangkan untuk memfaktorkan ulang kode Anda menggunakan injeksi ketergantungan.

## Catatan Perubahan

- v3.15.0 - Menambahkan contoh untuk injeksi ketergantungan dan mocking.