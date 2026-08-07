# Tracy

Tracy adalah penanganan kesalahan yang luar biasa yang dapat digunakan dengan Flight. Ia memiliki sejumlah panel yang dapat membantu Anda men-debug aplikasi Anda. Ia juga sangat mudah untuk diperluas dan menambahkan panel Anda sendiri. Tim Flight telah membuat beberapa panel khusus untuk proyek Flight dengan plugin [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) (Flight vars, kueri DB, permintaan, sesi, dan panel **Twig** opsional ketika Anda melewatkan profil profiler—lihat [Ekstensi Tracy](/awesome-plugins/tracy-extensions)).

## Instalasi

Instal dengan composer. Dan Anda sebenarnya ingin menginstal ini tanpa versi dev karena Tracy dilengkapi dengan komponen penanganan kesalahan produksi.

```bash
composer require tracy/tracy
```

## Konfigurasi Dasar

Ada beberapa opsi konfigurasi dasar untuk memulai. Anda dapat membaca lebih lanjut tentang mereka di [Dokumentasi Tracy](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Aktifkan Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // kadang-kadang Anda harus eksplisit (juga Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // Anda juga dapat menyediakan array alamat IP

// Di sinilah kesalahan dan pengecualian akan dicatat. Pastikan direktori ini ada dan dapat ditulis.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // tampilkan semua kesalahan
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // semua kesalahan kecuali pemberitahuan yang sudah tidak digunakan
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // jika bar Debugger terlihat, maka content-length tidak dapat diatur oleh Flight

	// Ini khusus untuk Ekstensi Tracy untuk Flight jika Anda telah menyertakannya
	// jika tidak, komentari ini.
	new TracyExtensionLoader($app);
}
```

## Tips Berguna

Saat Anda men-debug kode Anda, ada beberapa fungsi yang sangat membantu untuk menampilkan data untuk Anda.

- `bdump($var)` - Ini akan membuang variabel ke Bar Tracy dalam panel terpisah.
- `dumpe($var)` - Ini akan membuang variabel dan kemudian mati segera.