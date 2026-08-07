# Konfigurasi

## Gambaran Umum

Flight menyediakan cara sederhana untuk mengonfigurasi berbagai aspek framework agar sesuai dengan kebutuhan aplikasi Anda. Beberapa pengaturan sudah ditetapkan secara bawaan, tetapi Anda dapat menimpanya sesuai kebutuhan. Anda juga dapat mengatur variabel Anda sendiri untuk digunakan di seluruh aplikasi.

Konfigurasi berlapis yang jelas (default file + rahasia lingkungan) juga membantu [alat coding AI](/learn/ai): agen dapat belajar satu tempat untuk literal dan satu tempat untuk rahasia, alih-alih menemukan pembacaan `$_ENV` di dalam controller.

## Pemahaman

Anda dapat menyesuaikan perilaku tertentu dari Flight dengan mengatur nilai konfigurasi melalui metode `set`.

```php
Flight::set('flight.log_errors', true);
```

Dalam aplikasi terstruktur (termasuk [skeleton](https://github.com/flightphp/skeleton)), Anda biasanya memuat pengaturan proyek dari `app/config/config.php` dan kemudian menerapkan kunci yang relevan ke Engine (misalnya `flight.base_url`, `flight.views.path`). Anda juga dapat menyuntikkan objek konfigurasi kecil ke dalam controller alih-alih membaca global di mana-mana—lebih ramah untuk pengujian dan untuk agen yang mengikuti `AGENTS.md`.

## Penggunaan Dasar

### Opsi Konfigurasi Flight

Berikut adalah daftar semua pengaturan konfigurasi yang tersedia:

- **flight.base_url** `?string` - Menimpa URL dasar permintaan jika Flight berjalan di subdirektori. (default: null)
- **flight.case_sensitive** `bool` - Pencocokan peka huruf besar/kecil untuk URL. (default: false)
- **flight.handle_errors** `bool` - Mengizinkan Flight untuk menangani semua kesalahan secara internal. (default: true)
  - Jika Anda ingin Flight menangani kesalahan alih-alih perilaku PHP bawaan, ini perlu disetel ke true.
  - Jika Anda telah menginstal [Tracy](/awesome-plugins/tracy), Anda ingin menyetel ini ke false agar Tracy dapat menangani kesalahan.
  - Jika Anda memiliki plugin [APM](/awesome-plugins/apm) terinstal, Anda ingin menyetel ini ke true agar APM dapat mencatat kesalahan.
- **flight.log_errors** `bool` - Mencatat kesalahan ke file log kesalahan server web. (default: false)
  - Jika Anda telah menginstal [Tracy](/awesome-plugins/tracy), Tracy akan mencatat kesalahan berdasarkan konfigurasi Tracy, bukan konfigurasi ini.
- **flight.debug** `bool` - Menampilkan informasi kesalahan terperinci (pesan pengecualian, kode, dan jejak tumpukan) di browser saat terjadi kesalahan. (default: false)
  - **Jangan pernah mengaktifkan ini di produksi** — ini membocorkan detail aplikasi internal. Gunakan hanya untuk pengembangan lokal atau staging.
  - Ketika `false`, respons umum `500 Internal Server Error` yang ditampilkan sebagai gantinya. Pasangkan dengan `flight.log_errors` untuk menangkap kesalahan di sisi server.
- **flight.allow_method_override** `bool` - Mengizinkan metode HTTP untuk ditimpa melalui header permintaan `X-HTTP-Method-Override` atau bidang `_method` di badan POST. (default: true)
  - **Menyetel ini ke `false` disarankan** untuk aplikasi yang tidak memerlukan spoofing metode berbasis formulir HTML, karena ini mencegah klien memalsukan permintaan `DELETE` atau `PUT` melalui formulir POST standar.
  - Lihat [Keamanan](/learn/security#flight-configuration-hardening) untuk detail lebih lanjut.
- **flight.views.path** `string` - Direktori yang berisi file template tampilan. (default: ./views)
- **flight.views.extension** `string` - Ekstensi file template tampilan. (default: `.php`; skeleton resmi menyetel ini ke `.twig` saat menggunakan Twig)
- **flight.content_length** `bool` - Menyetel header `Content-Length`. (default: true)
  - Jika Anda menggunakan [Tracy](/awesome-plugins/tracy), ini perlu disetel ke false agar Tracy dapat dirender dengan benar.
- **flight.v2.output_buffering** `bool` - Menggunakan buffering keluaran lama. Lihat [migrasi ke v3](migrating-to-v3). (default: false)

### Konfigurasi Loader

Terdapat juga pengaturan konfigurasi tambahan untuk loader. Ini memungkinkan Anda untuk memuat kelas secara otomatis dengan `_` di nama kelas.

```php
// Mengaktifkan pemuatan kelas dengan garis bawah
// Defaultnya true
Loader::$v2ClassLoading = false;
```

Ingat bahwa [autoloading](/learn/autoloading) juga bergantung pada **huruf besar/kecil folder** yang cocok dengan namespace Anda—terutama dengan tata letak `App\` + `app/Controller/` pada skeleton.

### Konfigurasi proyek dan `.env` (pola skeleton)

Inti Flight tidak memerlukan file `.env`. Banyak aplikasi hanya menggunakan array konfigurasi PHP. Skeleton resmi melapisi konfigurasi sehingga rahasia tetap keluar dari git sementara Runway dapat dengan aman menulis ulang konfigurasi **literal**:

1. **`.env` / lingkungan nyata** — rahasia dan penimpaan deploy (diabaikan git).
2. **`app/config/config.php`** — default array PHP literal (disalin dari `config_sample.php`). Sebaiknya **tidak** ada ekspresi `$_ENV[...]` di dalam file ini: alat seperti `runway config:set` dapat menulis ulangnya sebagai nilai statis dan dapat memanggang rahasia ke dalam file.
3. **Gabungkan saat bootstrap** — env menang untuk kunci yang dipetakan; kode aplikasi membaca objek konfigurasi atau `$app->get()`, bukan `$_ENV` di controller.

Contoh bentuk `config_sample.php` / `config.php` (disederhanakan):

```php
<?php
// Hanya literal — rahasia ada di .env untuk alur kerja skeleton
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // atau mysql, atau '' untuk menonaktifkan
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env (skeleton)
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

Pemisahan ini disengaja untuk [proyek yang ramah AI](/learn/ai): instruksi dapat mengatakan "default di `config.php`, rahasia di `.env`, suntikkan Config / Engine—jangan pernah menemukan akses env di controller." Aplikasi yang ada dapat mengabaikan `.env` sepenuhnya dan tetap menggunakan satu file konfigurasi.

### Variabel

Flight memungkinkan Anda menyimpan variabel sehingga dapat digunakan di mana saja dalam aplikasi Anda.

```php
// Simpan variabel Anda
Flight::set('id', 123);

// Di bagian lain aplikasi Anda
$id = Flight::get('id');
```
Untuk melihat apakah suatu variabel telah disetel, Anda dapat melakukan:

```php
if (Flight::has('id')) {
  // Lakukan sesuatu
}
```

Anda dapat menghapus variabel dengan melakukan:

```php
// Menghapus variabel id
Flight::clear('id');

// Menghapus semua variabel
Flight::clear();
```

> **Catatan:** Hanya karena Anda dapat mengatur variabel bukan berarti Anda harus melakukannya. Gunakan fitur ini secukupnya. Alasannya adalah karena apa pun yang disimpan di sini menjadi variabel global. Variabel global buruk karena dapat diubah dari mana saja di aplikasi Anda, sehingga sulit untuk melacak bug. Selain itu, ini dapat memperumit hal-hal seperti [pengujian unit](/guides/unit-testing). Lebih suka injeksi konstruktor (seperti pada setup skeleton + Dice) untuk layanan dan konfigurasi yang dibutuhkan controller.

### Kesalahan dan Pengecualian

Semua kesalahan dan pengecualian ditangkap oleh Flight dan diteruskan ke metode `error` jika `flight.handle_errors` disetel ke true.

Perilaku defaultnya adalah mengirim respons umum `HTTP 500 Internal Server Error` dengan beberapa informasi kesalahan.

Anda dapat [menimpa](/learn/extending) perilaku ini sesuai kebutuhan Anda:

```php
Flight::map('error', function (Throwable $error) {
  // Tangani kesalahan
  echo $error->getTraceAsString();
});
```

Secara default, kesalahan tidak dicatat ke server web. Anda dapat mengaktifkannya dengan mengubah konfigurasi:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

Ketika URL tidak dapat ditemukan, Flight memanggil metode `notFound`. Perilaku defaultnya adalah mengirim respons `HTTP 404 Not Found` dengan pesan sederhana.

Anda dapat [menimpa](/learn/extending) perilaku ini sesuai kebutuhan Anda:

```php
Flight::map('notFound', function () {
  // Tangani tidak ditemukan
});
```

## Lihat Juga
- [Instalasi](/install) - Konfigurasi skeleton, `.env`, dan tata letak bootstrap.
- [Autoloading](/learn/autoloading) - Namespace dan huruf besar/kecil folder.
- [Memperluas Flight](/learn/extending) - Cara memperluas dan menyesuaikan fungsionalitas inti Flight.
- [Pengujian Unit](/guides/unit-testing) - Cara menulis pengujian unit untuk aplikasi Flight Anda.
- [AI & Pengalaman Pengembang](/learn/ai) - `AGENTS.md` dan instruksi proyek yang konsisten.
- [Tracy](/awesome-plugins/tracy) - Plugin untuk penanganan kesalahan dan debugging tingkat lanjut.
- [Ekstensi Tracy](/awesome-plugins/tracy_extensions) - Ekstensi untuk mengintegrasikan Tracy dengan Flight.
- [APM](/awesome-plugins/apm) - Plugin untuk pemantauan kinerja aplikasi dan pelacakan kesalahan.
- [Keamanan](/learn/security) - Bendera penguatan dan penanganan rahasia.

## Pemecahan Masalah
- Jika Anda mengalami masalah dalam menemukan semua nilai konfigurasi Anda, Anda dapat melakukan `var_dump(Flight::get());`
- Jika Runway atau alat deploy menulis ulang `config.php`, pastikan rahasia tidak di-commit—simpan di `.env` atau lingkungan nyata saat menggunakan pola skeleton.

## Changelog
- Dokumen – Mendokumentasikan konfigurasi gaya skeleton / lapisan `.env` dan default ekstensi tampilan Twig untuk proyek baru.
- v3.18.1 - Menambahkan opsi konfigurasi `flight.debug` dan `flight.allow_method_override`.
- v3.5.0 - Menambahkan konfigurasi untuk `flight.v2.output_buffering` untuk mendukung perilaku buffering keluaran lama.
- v2.0 - Konfigurasi inti ditambahkan.