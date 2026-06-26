# Konfigurasi

## Gambaran Umum

Flight menyediakan cara sederhana untuk mengonfigurasi berbagai aspek framework agar sesuai dengan kebutuhan aplikasi Anda. Beberapa sudah diatur secara default, tetapi Anda dapat menggantinya sesuai kebutuhan. Anda juga dapat mengatur variabel sendiri untuk digunakan di seluruh aplikasi.

## Pemahaman

Anda dapat menyesuaikan perilaku tertentu Flight dengan mengatur nilai konfigurasi
melalui metode `set`.

```php
Flight::set('flight.log_errors', true);
```

Di file `app/config/config.php`, Anda dapat melihat semua variabel konfigurasi default yang tersedia untuk Anda.

## Penggunaan Dasar

### Opsi Konfigurasi Flight

Berikut adalah daftar semua pengaturan konfigurasi yang tersedia:

- **flight.base_url** `?string` - Menimpa base url permintaan jika Flight berjalan di subdirektori. (default: null)
- **flight.case_sensitive** `bool` - Pencocokan URL yang sensitif terhadap huruf besar/kecil. (default: false)
- **flight.handle_errors** `bool` - Mengizinkan Flight untuk menangani semua error secara internal. (default: true)
  - Jika Anda ingin Flight menangani error alih-alih perilaku default PHP, ini perlu disetel ke true.
  - Jika Anda memiliki [Tracy](/awesome-plugins/tracy) terinstal, Anda ingin mengatur ini ke false agar Tracy dapat menangani error.
  - Jika Anda memiliki plugin [APM](/awesome-plugins/apm) terinstal, Anda ingin mengatur ini ke true agar APM dapat mencatat error.
- **flight.log_errors** `bool` - Mencatat error ke file log error server web. (default: false)
  - Jika Anda memiliki [Tracy](/awesome-plugins/tracy) terinstal, Tracy akan mencatat error berdasarkan konfigurasi Tracy, bukan konfigurasi ini.
- **flight.debug** `bool` - Menampilkan informasi error detail (pesan exception, kode, dan stack trace) di browser saat terjadi error. (default: false)
  - **Jangan aktifkan ini di production** — ini membocorkan detail internal aplikasi. Gunakan hanya untuk pengembangan lokal atau staging.
  - Saat `false`, akan ditampilkan `500 Internal Server Error` generik. Padukan dengan `flight.log_errors` untuk menangkap error di sisi server.
- **flight.allow_method_override** `bool` - Mengizinkan metode HTTP untuk di-override melalui header permintaan `X-HTTP-Method-Override` atau field `_method` di body POST. (default: true)
  - **Disarankan untuk mengatur ini ke `false`** untuk aplikasi yang tidak memerlukan method spoofing berbasis form HTML, karena mencegah klien memalsukan permintaan `DELETE` atau `PUT` melalui form POST standar.
  - Lihat [Keamanan](/learn/security#flight-configuration-hardening) untuk detail lebih lanjut.
- **flight.views.path** `string` - Direktori yang berisi file template view. (default: ./views)
- **flight.views.extension** `string` - Ekstensi file template view. (default: .php)
- **flight.content_length** `bool` - Mengatur header `Content-Length`. (default: true)
  - Jika Anda menggunakan [Tracy](/awesome-plugins/tracy), ini perlu disetel ke false agar Tracy dapat merender dengan benar.
- **flight.v2.output_buffering** `bool` - Menggunakan legacy output buffering. Lihat [migrasi ke v3](migrating-to-v3). (default: false)

### Konfigurasi Loader

Ada pengaturan konfigurasi lain untuk loader. Ini akan mengizinkan Anda 
untuk memuat otomatis kelas dengan `_` dalam nama kelas.

```php
// Aktifkan pemuatan kelas dengan underscore
// Default ke true
Loader::$v2ClassLoading = false;
```

### Variabel

Flight mengizinkan Anda menyimpan variabel agar dapat digunakan di mana saja dalam aplikasi Anda.

```php
// Simpan variabel Anda
Flight::set('id', 123);

// Di tempat lain dalam aplikasi Anda
$id = Flight::get('id');
```
Untuk melihat apakah sebuah variabel telah disetel, Anda dapat melakukan:

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

> **Catatan:** Hanya karena Anda dapat mengatur variabel tidak berarti Anda harus melakukannya. Gunakan fitur ini dengan hemat. Alasannya adalah apa pun yang disimpan di sini menjadi variabel global. Variabel global buruk karena dapat diubah dari mana saja dalam aplikasi Anda, sehingga sulit untuk melacak bug. Selain itu, ini dapat mempersulit hal-hal seperti [pengujian unit](/guides/unit-testing).

### Error dan Exception

Semua error dan exception ditangkap oleh Flight dan diteruskan ke metode `error`.
jika `flight.handle_errors` diatur ke true.

Perilaku default adalah mengirim respons `HTTP 500 Internal Server Error`
generik dengan beberapa informasi error.

Anda dapat [menimpa](/learn/extending) perilaku ini sesuai kebutuhan Anda:

```php
Flight::map('error', function (Throwable $error) {
  // Tangani error
  echo $error->getTraceAsString();
});
```

Secara default, error tidak dicatat ke server web. Anda dapat mengaktifkan ini dengan
mengubah konfigurasi:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Not Found

Saat URL tidak dapat ditemukan, Flight memanggil metode `notFound`. Perilaku default
adalah mengirim respons `HTTP 404 Not Found` dengan pesan sederhana.

Anda dapat [menimpa](/learn/extending) perilaku ini sesuai kebutuhan Anda:

```php
Flight::map('notFound', function () {
  // Tangani not found
});
```

## Lihat Juga
- [Memperluas Flight](/learn/extending) - Cara memperluas dan menyesuaikan fungsionalitas inti Flight.
- [Pengujian Unit](/guides/unit-testing) - Cara menulis pengujian unit untuk aplikasi Flight Anda.
- [Tracy](/awesome-plugins/tracy) - Plugin untuk penanganan error dan debugging lanjutan.
- [Ekstensi Tracy](/awesome-plugins/tracy_extensions) - Ekstensi untuk mengintegrasikan Tracy dengan Flight.
- [APM](/awesome-plugins/apm) - Plugin untuk pemantauan performa aplikasi dan pelacakan error.

## Pemecahan Masalah
- Jika Anda mengalami masalah dalam menemukan semua nilai konfigurasi Anda, Anda dapat melakukan `var_dump(Flight::get());`

## Changelog
- v3.18.1 - Menambahkan opsi konfigurasi `flight.debug` dan `flight.allow_method_override`.
- v3.5.0 - Menambahkan konfigurasi untuk `flight.v2.output_buffering` untuk mendukung perilaku legacy output buffering.
- v2.0 - Konfigurasi inti ditambahkan.