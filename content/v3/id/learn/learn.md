# Belajar tentang Flight

Flight adalah framework PHP yang cepat, sederhana, dan dapat diperluas. Ia cukup serbaguna dan dapat digunakan untuk membangun berbagai jenis aplikasi web.
Dibangun dengan mengedepankan kesederhanaan dan ditulis dengan cara yang mudah dipahami dan digunakan—oleh manusia dan oleh [asisten coding AI](/learn/ai).

> **Catatan:** Anda akan melihat contoh yang menggunakan `Flight::` sebagai variabel statis dan beberapa yang menggunakan objek Engine `$app->`. Keduanya dapat digunakan secara bergantian. `$app` dan `$this->app` dalam controller/middleware adalah pendekatan yang direkomendasikan oleh tim Flight (dan apa yang distandarisasi oleh skeleton resmi + `AGENTS.md` untuk proyek baru).

## Komponen Inti

### [Routing](/learn/routing)

Pelajari cara mengelola rute untuk aplikasi web Anda. Ini juga mencakup pengelompokan rute, parameter rute, dan middleware.

### [Middleware](/learn/middleware)

Pelajari cara menggunakan middleware untuk memfilter permintaan dan respons dalam aplikasi Anda.

### [Autoloading](/learn/autoloading)

Pelajari cara memuat otomatis (autoload) kelas-kelas Anda sendiri. **Kapitalisasi** folder harus sesuai dengan namespace Anda—skeleton menggunakan `App\` dan folder PascalCase seperti `app/Controller/`.

### [Requests](/learn/requests)

Pelajari cara menangani permintaan dan respons dalam aplikasi Anda.

### [Responses](/learn/responses)

Pelajari cara mengirim respons kepada pengguna Anda.

### [HTML Templates](/learn/templates)

Pelajari cara merender HTML dengan Twig (default skeleton), Latte, atau mesin lainnya—tidak hanya tampilan PHP bawaan.

### [Security](/learn/security)

Pelajari cara mengamankan aplikasi Anda dari ancaman keamanan umum.

### [Configuration](/learn/configuration)

Pelajari cara mengonfigurasi framework untuk aplikasi Anda.

### [Event Manager](/learn/events)

Pelajari cara menggunakan sistem event untuk menambahkan event kustom ke aplikasi Anda.

### [Extending Flight](/learn/extending)

Pelajari cara memperluas framework dengan menambahkan metode dan kelas Anda sendiri.

### [Method Hooks and Filtering](/learn/filtering)

Pelajari cara menambahkan kait (hook) event ke metode Anda dan metode internal framework.

### [Dependency Injection Container (DIC)](/learn/dependency-injection-container)

Pelajari cara menggunakan kontainer injeksi dependensi (DIC) untuk mengelola dependensi aplikasi Anda.

## Kelas Utilitas

### [Collections](/learn/collections)

Koleksi digunakan untuk menyimpan data dan dapat diakses sebagai array atau sebagai objek untuk kemudahan penggunaan.

### [JSON Wrapper](/learn/json)

Ini memiliki beberapa fungsi sederhana untuk membuat encoding dan decoding JSON Anda konsisten.

### [SimplePdo](/learn/simple-pdo)

PDO terkadang dapat menambah sakit kepala lebih dari yang diperlukan. SimplePdo adalah kelas pembantu PDO modern dengan metode yang mudah seperti `insert()`, `update()`, `delete()`, dan `transaction()` untuk membuat operasi database jauh lebih mudah.

### [PdoWrapper](/learn/pdo-wrapper) (Tidak digunakan lagi)

Pembungkus PDO asli sudah tidak digunakan lagi (deprecated) sejak v3.18.0. Silakan gunakan [SimplePdo](/learn/simple-pdo) sebagai gantinya.

### [Uploaded File Handler](/learn/uploaded-file)

Kelas sederhana untuk membantu mengelola file yang diunggah dan memindahkannya ke lokasi permanen.

## Konsep Penting

### [Why a Framework?](/learn/why-frameworks)

Berikut adalah artikel singkat tentang mengapa Anda sebaiknya menggunakan framework. Sangat baik untuk memahami manfaat menggunakan framework sebelum Anda mulai menggunakannya.

Selain itu, tutorial yang sangat baik telah dibuat oleh [@lubiana](https://git.php.fail/lubiana). Meskipun tidak membahas secara detail tentang Flight secara khusus,
panduan ini akan membantu Anda memahami beberapa konsep utama seputar framework dan mengapa konsep-konsep tersebut bermanfaat untuk digunakan.
Anda dapat menemukan tutorialnya [di sini](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight Dibandingkan dengan Framework Lain](/learn/flight-vs-another-framework)

Jika Anda bermigrasi dari framework lain seperti Laravel, Slim, Fat-Free, atau Symfony ke Flight, halaman ini akan membantu Anda memahami perbedaan antara keduanya.

## Topik Lainnya

### [Unit Testing](/learn/unit-testing)

Ikuti panduan ini untuk mempelajari cara melakukan unit testing pada kode Flight Anda agar menjadi sangat kokoh.

### [AI & Pengalaman Pengembang](/learn/ai)

Flight dibangun untuk dipasangkan dengan LLM coding: `AGENTS.md`, perintah Runway `ai:*`, dan satu tata letak skeleton yang jelas sehingga agen tetap sesuai pola.

### [Migrasi v2 -> v3](/learn/migrating-to-v3)

Kompatibilitas mundur (backwards compatibility) sebagian besar tetap dipertahankan, tetapi ada beberapa perubahan yang perlu Anda ketahui saat bermigrasi dari v2 ke v3.