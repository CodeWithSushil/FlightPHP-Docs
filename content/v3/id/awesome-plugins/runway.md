# Runway

Runway adalah aplikasi CLI yang membantu Anda mengelola aplikasi Flight. Runway dapat menghasilkan controller, menampilkan semua rute, menjalankan pembantu pengaturan AI, migrasi (dalam skeleton), dan lainnya. Runway didasarkan pada pustaka yang sangat baik [adhocore/php-cli](https://github.com/adhocore/php-cli).

Klik [di sini](https://github.com/flightphp/runway) untuk melihat kode.

Perintah scaffolding sengaja disesuaikan dengan [skeleton resmi](https://github.com/flightphp/skeleton) sehingga [alat pengkodean AI](/learn/ai) dan manusia mendapatkan jalur, namespace, dan gaya constructor-injection yang sama setiap saat.

## Instalasi

Instal dengan composer.

```bash
composer require flightphp/runway
```

Skeleton sudah bergantung pada Runway; gunakan `php runway` dari root proyek.

## Konfigurasi Dasar

Saat pertama kali menjalankan Runway, Runway akan mencoba menemukan konfigurasi `runway` di `app/config/config.php` melalui kunci `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// opsional; skeleton juga menggunakan index_root untuk entri publik
		'index_root' => 'public/index.php',
    ],
];
```

> **CATATAN** - Mulai dari **v1.2.0**, `.runway-config.json` sudah tidak digunakan lagi dan digantikan oleh `app/config/config.php`. Migrasikan dengan `php runway config:migrate` saat mengupgrade proyek lama. Skeleton mungkin masih menulis `.runway-config.json` kecil saat create-project untuk kompatibilitas; preferensikan kunci `runway` di `config.php` ke depannya.

### Deteksi Root Proyek

Runway cukup pintar untuk mendeteksi root proyek Anda, bahkan jika Anda menjalankannya dari subdirektori. Runway mencari indikator seperti `composer.json`, `.git`, atau `app/config/config.php` untuk menentukan di mana root proyek berada. Ini berarti Anda dapat menjalankan perintah Runway dari mana saja dalam proyek Anda!

## Penggunaan

Runway memiliki sejumlah perintah yang dapat Anda gunakan untuk mengelola aplikasi Flight Anda. Ada dua cara mudah untuk menggunakan Runway.

1. Jika Anda menggunakan proyek skeleton, Anda dapat menjalankan `php runway [perintah]` dari root proyek Anda.
1. Jika Anda menggunakan Runway sebagai paket yang diinstal melalui composer, Anda dapat menjalankan `vendor/bin/runway [perintah]` dari root proyek Anda.

### Daftar Perintah

Anda dapat melihat daftar semua perintah yang tersedia dengan menjalankan perintah `php runway`.

```bash
php runway
```

Hanya bergantung pada perintah yang benar-benar muncul dalam daftar tersebut untuk instalasi Anda (perintah Runway inti vs perintah khusus proyek seperti `migrate` milik skeleton).

### Bantuan Perintah

Untuk perintah apa pun, Anda dapat memberikan flag `--help` untuk mendapatkan informasi lebih lanjut tentang cara menggunakan perintah tersebut.

```bash
php runway routes --help
php runway make:controller --help
```

Berikut beberapa contohnya:

### Membuat Controller

`make:controller` membuat scaffold controller yang sesuai dengan tata letak skeleton resmi:

| | |
|--|--|
| **Jalur** | `app/Controller/{Nama}.php` |
| **Namespace** | `App\Controller` |
| **Gaya** | Constructor injection dari `flight\Engine` (tidak ada `Flight::` dalam body kelas) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Contoh bentuk yang seharusnya Anda harapkan (disederhanakan):

```php
<?php

declare(strict_types=1);

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
		// misalnya $this->app->render('…', […]);
	}
}
```

Daftarkan dengan callable kelas sehingga Dice dapat membangun controller:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Mengapa tata letak ini?** Folder **huruf besar** harus sesuai dengan namespace (`Controller` bukan `controllers`) untuk Composer PSR-4 di Linux—lihat [Autoloading](/learn/autoloading). Jalur yang sama adalah yang diberitahu file `AGENTS.md` root dan scoped kepada alat AI untuk digunakan, sehingga controller yang dihasilkan dan ditulis tangan tetap identik.

> Dokumentasi lama dan proyek komunitas kadang-kadang menggunakan `app/controllers/` dan `app\controllers`. Itu tetap valid jika struktur *Anda* masih menggunakan folder huruf kecil. **Proyek skeleton baru dan output `make:controller` saat ini menggunakan `app/Controller/` + `App\Controller`.**

### Membuat Model Active Record

Pastikan dulu Anda telah menginstal plugin [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record users
```

Dalam skeleton resmi, model berada di bawah **`app/Model/`** dengan namespace **`App\Model`**, dan koneksi DB adalah **[SimplePdo](/learn/simple-pdo)** (inject atau lewatkan ke dalam constructor ActiveRecord). Nama file/namespace yang dihasilkan mengikuti default Runway saat ini dan konfigurasi `runway` Anda—preferensikan menyelaraskan model baru dengan `App\Model` agar sesuai dengan [autoloading](/learn/autoloading) dan `AGENTS.md`.

Contoh model yang konsisten dengan demo posts skeleton:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Jika generator lama masih menghasilkan `app/records` / `app\records`, Anda dapat mempertahankan konvensi tersebut dalam aplikasi lama atau memindahkan file ke `app/Model/` dan memperbarui namespace agar sesuai dengan huruf folder.

### Migrasi (skeleton)

Skeleton resmi menyediakan perintah proyek (ditemukan dari `app/commands/`) seperti:

```bash
php runway migrate
```

Migrasi adalah file SQL di bawah `migrations/` (misalnya `YYYYMMDDHHMMSS_description.sql` untuk SQLite dan `…_description.mysql.sql` untuk MySQL), dipilih dari konfigurasi driver database / env Anda. Flag dan perilaku yang tepat didefinisikan oleh perintah proyek tersebut—jalankan `php runway migrate --help` di aplikasi Anda.

### Pembantu AI

Runway menyediakan perintah berorientasi AI yang digunakan dengan [AI & pengalaman pengembang](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Perintah ini menyimpan kredensial LLM dan menghasilkan instruksi proyek (terutama **`AGENTS.md`**). Pada skeleton, perlakukan `AGENTS.md` (dan salinan scoped di bawah `app/`) plus **`SECURITY.md`** sebagai sumber kebenaran untuk agent.

### Menampilkan Semua Rute

Ini akan menampilkan semua rute yang saat ini terdaftar dengan Flight.

```bash
php runway routes
```

Jika Anda ingin hanya melihat rute tertentu, Anda dapat memberikan flag untuk memfilter rute.

```bash
# Menampilkan hanya rute GET
php runway routes --get

# Menampilkan hanya rute POST
php runway routes --post

# dll.
```

## Menambahkan Perintah Kustom ke Runway

Jika Anda membuat paket untuk Flight, atau ingin menambahkan perintah kustom Anda sendiri ke proyek Anda, Anda dapat melakukannya dengan membuat direktori `src/commands/`, `flight/commands/`, `app/commands/`, atau `commands/` untuk proyek/paket Anda. Jika Anda memerlukan kustomisasi lebih lanjut, lihat bagian Konfigurasi di bawah.

Dalam skeleton, perintah proyek berada di **`app/commands/`** dengan namespace **`App\Command`**. Runway menemukannya berdasarkan jalur; pertahankan folder tersebut agar selaras dengan classmap/PSR-4 Composer seperti yang sudah dilakukan proyek Anda.

Untuk membuat perintah, Anda cukup memperluas kelas `AbstractBaseCommand`, dan mengimplementasikan setidaknya metode `__construct` dan metode `execute`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * Construct
     *
     * @param array<string,mixed> $config Config dari app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Buat contoh untuk dokumentasi', $config);
        $this->argument('<funny-gif>', 'Nama gif lucu');
    }

	/**
     * Menjalankan fungsi
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Membuat contoh...');

		// Lakukan sesuatu di sini

		$io->ok('Contoh dibuat!');
	}
}
```

Lihat [Dokumentasi adhocore/php-cli](https://github.com/adhocore/php-cli) untuk informasi lebih lanjut tentang cara membangun perintah kustom Anda sendiri ke dalam aplikasi Flight Anda!

## Manajemen Konfigurasi

Karena konfigurasi telah dipindahkan ke `app/config/config.php` mulai dari `v1.2.0`, ada beberapa perintah pembantu untuk mengelola konfigurasi.

> **Tip Skeleton:** Pertahankan `config.php` sebagai nilai PHP **literal**. Rahasia milik `.env`. Hindari ekspresi `$_ENV[...]` di dalam `config.php`—`config:set` menulis ulang file tersebut sebagai data statis dan dapat membakar rahasia ke dalam file. Lihat [Konfigurasi](/learn/configuration).

### Migrasi Konfigurasi Lama

Jika Anda memiliki file `.runway-config.json` lama, Anda dapat dengan mudah memigrasikannya ke `app/config/config.php` dengan perintah berikut:

```bash
php runway config:migrate
```

### Mengatur Nilai Konfigurasi

Anda dapat mengatur nilai konfigurasi menggunakan perintah `config:set`. Ini berguna jika Anda ingin memperbarui nilai konfigurasi tanpa membuka file.

```bash
php runway config:set app_root "app/"
```

### Mendapatkan Nilai Konfigurasi

Anda dapat mendapatkan nilai konfigurasi menggunakan perintah `config:get`.

```bash
php runway config:get app_root
```

## Semua Konfigurasi Runway

Jika Anda perlu menyesuaikan konfigurasi untuk Runway, Anda dapat mengatur nilai-nilai ini di `app/config/config.php`. Berikut beberapa konfigurasi tambahan yang dapat Anda atur:

```php
<?php
// app/config/config.php
return [
    // ... nilai konfigurasi lainnya ...

    'runway' => [
        // Ini adalah lokasi direktori aplikasi Anda
        'app_root' => 'app/',

        // Ini adalah direktori tempat file index root Anda berada
        'index_root' => 'public/',

        // Ini adalah jalur ke root proyek lain
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Jalur dasar kemungkinan besar tidak perlu dikonfigurasi, tetapi tersedia jika Anda menginginkannya
        'base_paths' => [
            '/includes/libs/vendor', // jika Anda memiliki jalur yang sangat unik untuk direktori vendor Anda atau semacamnya
        ],

        // Jalur akhir adalah lokasi dalam proyek untuk mencari file perintah
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Jika Anda ingin menambahkan jalur lengkap, silakan saja (absolut atau relatif terhadap root proyek)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Mengakses Konfigurasi

Jika Anda perlu mengakses nilai konfigurasi secara efektif, Anda dapat mengaksesnya melalui metode `__construct` atau metode `app()`. Penting juga untuk dicatat bahwa jika Anda memiliki file `app/config/services.php`, layanan tersebut juga akan tersedia untuk perintah Anda.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Mengakses konfigurasi
    $app_root = $this->config['runway']['app_root'];
    
    // Mengakses layanan seperti mungkin koneksi database
    $database = $this->config['database']
    
    // ...
}
```

## Pembungkus Pembantu AI

Runway memiliki beberapa pembungkus pembantu yang membuatnya lebih mudah bagi AI untuk menghasilkan perintah. Anda dapat menggunakan `addOption` dan `addArgument` dengan cara yang terasa mirip dengan Symfony Console. Ini membantu jika Anda menggunakan alat AI untuk menghasilkan perintah Anda.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Buat contoh untuk dokumentasi', $config);
    
    // Argumen mode dapat bernilai null dan default-nya sepenuhnya opsional
    $this->addOption('name', 'Nama contoh', null);
}
```

## Lihat Juga

- [Instalasi](/install) - Pohon skeleton dan default create-project
- [Autoloading](/learn/autoloading) - `App\` dan huruf folder
- [Dependency Injection](/learn/dependency-injection-container) - Dice + Engine injection untuk controller yang dihasilkan
- [AI & Pengalaman Pengembang](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Model yang digunakan dengan `make:record` / skeleton `App\Model`
- [SimplePdo](/learn/simple-pdo) - Koneksi DB yang digunakan oleh migrasi dan model skeleton