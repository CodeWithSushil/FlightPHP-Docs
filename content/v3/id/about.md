# Flight PHP Framework

Flight adalah framework PHP yang cepat, sederhana, dan dapat diperluas—dibangun untuk developer yang ingin menyelesaikan tugas dengan cepat tanpa ribet. Baik Anda membangun aplikasi web klasik, API berkecepatan tinggi, atau bekerja sama dengan asisten coding AI, desain ringan dan mudah dipahami dari Flight menjadikannya pilihan yang sempurna. Flight dirancang agar ramping, tetapi juga mampu menangani kebutuhan arsitektur enterprise.

## Mengapa Memilih Flight?

- **Ramah Pemula:** Flight adalah titik awal yang bagus untuk developer PHP baru. Struktur yang jelas dan sintaks sederhana membantu Anda belajar pengembangan web tanpa terjebak dalam boilerplate.
- **Disukai Profesional:** Developer berpengalaman menyukai Flight karena fleksibilitas dan kontrolnya. Anda dapat menskalakan dari prototipe kecil hingga aplikasi lengkap tanpa perlu berganti framework.
- **Kompatibel Mundur:** Kami menghargai waktu Anda. Flight v3 adalah augmentasi dari v2, dengan hampir semua API yang sama. Kami percaya pada evolusi, bukan revolusi—tidak ada lagi "merusak dunia" setiap kali versi mayor dirilis.
- **Tanpa Dependensi:** Inti Flight benar-benar bebas dependensi—tidak ada polyfill, tidak ada paket eksternal, bahkan tidak ada antarmuka PSR. Ini berarti lebih sedikit vektor serangan, jejak yang lebih kecil, dan tidak ada perubahan yang merusak secara tiba-tiba dari dependensi upstream. Plugin opsional mungkin memiliki dependensi, tetapi inti akan selalu tetap ramping dan aman.
- **Ramah AI:** Permukaan API yang kecil dari Flight dan [skeleton resmi](https://github.com/flightphp/skeleton) (satu layout, `AGENTS.md`, constructor injection) memudahkan tools coding AI untuk tetap on-pattern. Kode yang sama baik saat Anda mengetik setiap baris atau bekerja sama dengan agen. [Pelajari lebih lanjut tentang menggunakan AI dengan Flight](/learn/ai).

## Gambaran Video

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">Cukup sederhana, bukan?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">Pelajari lebih lanjut</a> tentang Flight dalam dokumentasi!
    </div>
  </div>
</div>

## Mulai Cepat

Untuk instalasi bare bones yang cepat, instal dengan Composer:

```bash
composer require flightphp/core
```

Atau Anda dapat mengunduh zip dari repo [di sini](https://github.com/flightphp/core). Kemudian Anda akan memiliki file `index.php` dasar seperti berikut:

```php
<?php

// jika diinstal dengan composer
require 'vendor/autoload.php';
// atau jika diinstal secara manual dengan file zip
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

Itu saja! Anda memiliki aplikasi Flight dasar. Sekarang Anda dapat menjalankan file ini dengan `php -S localhost:8000` dan mengunjungi `http://localhost:8000` di browser Anda untuk melihat hasilnya.

Contoh `Flight::` yang singkat seperti ini bagus untuk pembelajaran dan aplikasi mikro. Untuk layout proyek lengkap yang digunakan bersama oleh manusia dan tools AI, gunakan skeleton di bawah ini.

## Aplikasi Skeleton/Boilerplate

Ada starter resmi untuk membantu Anda memulai proyek Flight baru. Ini menyiapkan struktur, konfigurasi, skrip Composer, dan instruksi yang ramah AI sejak awal.

Lihat [flightphp/skeleton](https://github.com/flightphp/skeleton) untuk proyek siap pakai, atau kunjungi halaman [contoh](examples) untuk inspirasi. Ingin detail workflow AI? [Jelajahi AI & pengalaman developer](/learn/ai).

Yang Anda dapatkan (tingkat tinggi):

- **Namespace `App\`** dengan folder PascalCase (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—**huruf besar kecil** folder harus sesuai dengan namespace (lihat [Autoloading](/learn/autoloading))
- **Injeksi Dice + `Engine`** agar controller tetap dapat diuji (lebih suka `$this->app` daripada `Flight::` dalam kode aplikasi)
- **View Twig**, contoh **SimplePdo** + ActiveRecord, Runway **migrate**
- **`AGENTS.md`** root (plus salinan scoped) dan **`SECURITY.md`** untuk asisten dan kebijakan keamanan

## Menginstal Aplikasi Skeleton

Cukup mudah!

```bash
# Buat proyek baru
composer create-project flightphp/skeleton my-project/
# Masuk ke direktori proyek baru Anda
cd my-project/
# Jalankan server pengembangan lokal untuk memulai!
composer start
```

Ini membuat struktur proyek, menyalin `config_sample.php` → `config.php` (dan `.env.example` → `.env` jika ada), dan Anda siap untuk mulai. Data contoh opsional:

```bash
php runway migrate
# kemudian kunjungi /posts dan /api/posts
```

## Performa Tinggi

Flight adalah salah satu framework PHP tercepat yang ada. Inti yang ringan berarti overhead lebih sedikit dan kecepatan lebih tinggi—sempurna untuk aplikasi tradisional dan workflow modern yang dibantu AI. Anda dapat melihat semua benchmark di [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Lihat benchmark di bawah ini dengan beberapa framework PHP populer lainnya.

| Framework | Plaintext Reqs/sec | JSON Reqs/sec |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight dan AI

Penasaran bagaimana Flight berpasangan dengan coding LLM? [Temukan](/learn/ai) bagaimana `AGENTS.md`, perintah Runway `ai:*`, dan layout skeleton menjaga asisten tetap on track.

## Stabilitas dan Kompatibilitas Mundur

Kami menghargai waktu Anda. Kami semua telah melihat framework yang benar-benar mengubah dirinya setiap beberapa tahun, meninggalkan developer dengan kode yang rusak dan migrasi yang mahal. Flight berbeda. Flight v3 dirancang sebagai augmentasi dari v2, yang berarti API yang Anda kenal dan sukai tidak dihilangkan. Bahkan, sebagian besar proyek v2 akan bekerja tanpa perubahan apapun di v3.

Kami berkomitmen untuk menjaga Flight tetap stabil sehingga Anda dapat fokus membangun aplikasi Anda, bukan memperbaiki framework Anda. Skeleton bisa bersifat opinionated untuk proyek *baru*; API inti tetap familiar untuk semua orang lain.

# Komunitas

Kami di Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

Dan Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Berkontribusi

Ada dua cara Anda dapat berkontribusi pada Flight:

1. Berkontribusi pada framework inti dengan mengunjungi [repositori core](https://github.com/flightphp/core).
2. Membantu membuat dokumentasi menjadi lebih baik! Situs dokumentasi ini di-hosting di [Github](https://github.com/flightphp/docs). Jika Anda menemukan kesalahan atau ingin memperbaiki sesuatu, silakan ajukan pull request. Kami menyukai pembaruan dan ide baru—terutama seputar AI dan teknologi baru!

# Persyaratan

Flight memerlukan PHP 7.4 atau yang lebih baru.

**Catatan:** PHP 7.4 didukung karena pada saat penulisan ini (2024) PHP 7.4 adalah versi default untuk beberapa distribusi Linux LTS. Memaksa perpindahan ke PHP >8 akan menyebabkan banyak masalah bagi pengguna tersebut. Framework ini juga mendukung PHP >8.

# Lisensi

Flight dirilis di bawah lisensi [MIT](https://github.com/flightphp/core/blob/master/LICENSE).