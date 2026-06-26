# flightphp/cache

Ringan, sederhana, dan berdiri sendiri kelas caching dalam-file PHP yang di-fork dari [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Keuntungan** 
- Ringan, berdiri sendiri dan sederhana
- Semua kode dalam satu file - tidak ada driver yang tidak perlu.
- Aman - setiap file cache yang dihasilkan memiliki header php dengan die, membuat akses langsung tidak mungkin bahkan jika seseorang mengetahui jalur dan server Anda tidak dikonfigurasi dengan benar
- Didokumentasikan dan diuji dengan baik
- Menangani konkurensi dengan benar melalui flock
- Mendukung PHP 7.4+
- Gratis di bawah lisensi MIT

Situs dokumentasi ini menggunakan library ini untuk meng-cache setiap halaman!

Klik [here](https://github.com/flightphp/cache) untuk melihat kode.

## Instalasi

Instal melalui composer:

```bash
composer require flightphp/cache
```

## Penggunaan

Penggunaan cukup mudah. Ini menyimpan file cache di direktori cache.

```php
use flight\Cache;

$app = Flight::app();

// Anda meneruskan direktori tempat cache akan disimpan ke dalam konstruktor
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Ini memastikan bahwa cache hanya digunakan saat dalam mode produksi
	// ENVIRONMENT adalah konstanta yang ditetapkan di file bootstrap Anda atau di tempat lain di aplikasi Anda
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Mendapatkan Nilai Cache

Anda menggunakan metode `get()` untuk mendapatkan nilai yang di-cache. Jika Anda menginginkan metode yang mudah yang akan menyegarkan cache jika sudah kedaluwarsa, Anda dapat menggunakan `refreshIfExpired()`.

```php

// Dapatkan instance cache
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // return data to be cached
}, 10); // 10 detik

// atau
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 detik
}
```

### Menyimpan Nilai Cache

Anda menggunakan metode `set()` untuk menyimpan nilai dalam cache.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 detik
```

### Menghapus Nilai Cache

Anda menggunakan metode `delete()` untuk menghapus nilai dalam cache.

```php
Flight::cache()->delete('simple-cache-test');
```

### Memeriksa apakah Nilai Cache Ada

Anda menggunakan metode `exists()` untuk memeriksa apakah nilai ada dalam cache.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// lakukan sesuatu
}
```

### Menghapus Cache
Anda menggunakan metode `flush()` untuk menghapus seluruh cache.

```php
Flight::cache()->flush();
```

### Mengambil meta data dengan cache

Jika Anda ingin mengambil timestamp dan meta data lainnya tentang entri cache, pastikan Anda meneruskan `true` sebagai parameter yang benar.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // return data to be cached
}, 10, true); // true = return with metadata
// atau
$data = $cache->get("simple-cache-meta-test", true); // true = return with metadata

/*
Contoh item cache yang diambil dengan metadata:
{
    "time":1511667506, <-- save unix timestamp
    "expire":10,       <-- expire time in seconds
    "data":"04:38:26", <-- unserialized data
    "permanent":false
}

Dengan menggunakan metadata, kita dapat, misalnya, menghitung kapan item disimpan atau kapan kedaluwarsa
Kita juga dapat mengakses data itu sendiri dengan kunci "data"
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // get unix timestamp when data expires and subtract current timestamp from it
$cacheddate = $data["data"]; // we access the data itself with the "data" key

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## Kode Sumber

Kunjungi [https://github.com/flightphp/cache](https://github.com/flightphp/cache) untuk melihat kode.