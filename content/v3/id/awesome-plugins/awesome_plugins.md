# Plugin Luar Biasa

Flight sangat dapat diperluas. Ada sejumlah plugin yang dapat digunakan untuk menambahkan fungsionalitas ke aplikasi Flight Anda. Beberapa didukung secara resmi oleh Tim Flight dan yang lainnya adalah library mikro/lite untuk membantu Anda memulai.

## Alat AI

Flight dapat menjadi lebih keren dengan plugin berbasis AI.

- [Flight MCP](/awesome-plugins/mcp) - Plugin untuk mengintegrasikan MCP (Model Control Protocol) dengan Flight, memungkinkan fungsionalitas berbasis AI yang mulus. Sebagian besar berfokus pada halaman dokumentasi, membantu menjaga biaya token tetap rendah dengan menyediakan informasi terkini tentang proyek Flight Anda.

## Dokumentasi API

Dokumentasi API sangat penting untuk API apa pun. Ini membantu pengembang memahami cara berinteraksi dengan API Anda dan apa yang diharapkan sebagai balasan. Ada beberapa alat yang tersedia untuk membantu Anda menghasilkan dokumentasi API untuk Proyek Flight Anda.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Tulisan blog oleh Daniel Schreiber tentang cara menggunakan OpenAPI Spec dengan FlightPHP untuk membangun API Anda menggunakan pendekatan API first.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI adalah alat yang bagus untuk membantu Anda menghasilkan dokumentasi API untuk proyek Flight Anda. Sangat mudah digunakan dan dapat disesuaikan untuk memenuhi kebutuhan Anda. Ini adalah library PHP untuk membantu Anda menghasilkan dokumentasi Swagger.

## Pemantauan Performa Aplikasi (APM)

Pemantauan Performa Aplikasi (APM) sangat penting untuk aplikasi apa pun. Ini membantu Anda memahami bagaimana aplikasi Anda berkinerja dan di mana bottleneck-nya. Ada sejumlah alat APM yang dapat digunakan dengan Flight.
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM adalah library APM sederhana yang dapat digunakan untuk memantau aplikasi Flight Anda. Dapat digunakan untuk memantau performa aplikasi Anda dan membantu mengidentifikasi bottleneck.

## Async

Flight sudah merupakan framework yang cepat tetapi menambahkan mesin turbo membuat segalanya lebih menyenangkan (dan menantang)!

- [flightphp/async](/awesome-plugins/async) - Library Flight Async resmi. Library ini adalah cara sederhana untuk menambahkan pemrosesan asinkron ke aplikasi Anda. Menggunakan Swoole/Openswoole di balik layar untuk menyediakan cara sederhana dan efektif menjalankan tugas secara asinkron.

## Otorisasi/Permission

Otorisasi dan Permission sangat penting untuk aplikasi apa pun yang memerlukan kontrol untuk siapa yang dapat mengakses apa.

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - Library Flight Permissions resmi. Library ini adalah cara sederhana untuk menambahkan permission tingkat user dan aplikasi ke aplikasi Anda.

## Autentikasi

Autentikasi penting untuk aplikasi yang perlu memverifikasi identitas user dan mengamankan endpoint API.

- [firebase/php-jwt](/awesome-plugins/jwt) - Library JSON Web Token (JWT) untuk PHP. Cara sederhana dan aman untuk mengimplementasikan autentikasi berbasis token dalam aplikasi Flight Anda. Sempurna untuk autentikasi API stateless, melindungi route dengan middleware, dan mengimplementasikan alur otorisasi gaya OAuth.

## Caching

Caching adalah cara yang bagus untuk mempercepat aplikasi Anda. Ada sejumlah library caching yang dapat digunakan dengan Flight.

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Class caching dalam file PHP yang ringan, sederhana dan mandiri

## CLI

Aplikasi CLI adalah cara yang bagus untuk berinteraksi dengan aplikasi Anda. Anda dapat menggunakannya untuk menghasilkan controller, menampilkan semua route, dan lainnya.

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway adalah aplikasi CLI yang membantu Anda mengelola aplikasi Flight Anda.

## Cookie

Cookie adalah cara yang bagus untuk menyimpan potongan data kecil di sisi client. Mereka dapat digunakan untuk menyimpan preferensi user, pengaturan aplikasi, dan lainnya.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie adalah library PHP yang menyediakan cara sederhana dan efektif untuk mengelola cookie.

## Debugging

Debugging sangat penting ketika Anda mengembangkan di lingkungan lokal Anda. Ada beberapa plugin yang dapat meningkatkan pengalaman debugging Anda.

- [tracy/tracy](/awesome-plugins/tracy) - Ini adalah error handler yang lengkap yang dapat digunakan dengan Flight. Memiliki sejumlah panel yang dapat membantu Anda debug aplikasi Anda. Juga sangat mudah untuk diperluas dan menambahkan panel Anda sendiri.
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Digunakan dengan error handler [Tracy](/awesome-plugins/tracy), plugin ini menambahkan beberapa panel tambahan untuk membantu debugging khusus untuk proyek Flight.

## Database

Database adalah inti dari sebagian besar aplikasi. Ini adalah cara Anda menyimpan dan mengambil data. Beberapa library database hanyalah wrapper untuk menulis query dan beberapa adalah ORM yang lengkap.

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Helper PDO Flight resmi yang merupakan bagian dari core. Ini adalah wrapper modern dengan metode helper yang nyaman seperti `insert()`, `update()`, `delete()`, dan `transaction()` untuk menyederhanakan operasi database. Semua hasil dikembalikan sebagai Collection untuk akses array/object yang fleksibel. Bukan ORM, hanya cara yang lebih baik untuk bekerja dengan PDO.
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Wrapper PDO Flight resmi yang merupakan bagian dari core (deprecated sejak v3.18.0). Gunakan SimplePdo sebagai gantinya.
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - Flight ActiveRecord ORM/Mapper resmi. Library kecil yang bagus untuk mengambil dan menyimpan data dengan mudah di database Anda.
- [byjg/php-migration](/awesome-plugins/migrations) - Plugin untuk melacak semua perubahan database untuk proyek Anda.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Query builder SQL yang ringan dan fluent yang menghasilkan SQL dan parameter untuk prepared statement. Bekerja dengan baik dengan [SimplePdo](/learn/simple-pdo).

## Enkripsi

Enkripsi sangat penting untuk aplikasi apa pun yang menyimpan data sensitif. Mengenkripsi dan mendekripsi data tidak terlalu sulit, tetapi menyimpan kunci enkripsi dengan benar [bisa](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [menjadi](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [sulit](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). Hal yang paling penting adalah untuk tidak pernah menyimpan kunci enkripsi Anda di direktori publik atau meng-commitnya ke repositori kode Anda.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - Ini adalah library yang dapat digunakan untuk mengenkripsi dan mendekripsi data. Memulai dan menjalankan cukup sederhana untuk mulai mengenkripsi dan mendekripsi data.

## Job Queue

Job queue sangat membantu untuk memproses tugas secara asinkron. Ini bisa berupa mengirim email, memproses gambar, atau apa pun yang tidak perlu dilakukan secara real-time.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue adalah library yang dapat digunakan untuk memproses job secara asinkron. Dapat digunakan dengan beanstalkd、MySQL/MariaDB、SQLite、dan PostgreSQL.

## Session

Session tidak terlalu berguna untuk API tetapi untuk membangun aplikasi web, session bisa sangat penting untuk mempertahankan state dan informasi login.

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - Library Flight Session resmi. Ini adalah library session sederhana yang dapat digunakan untuk menyimpan dan mengambil data session. Menggunakan penanganan session bawaan PHP.
- [Ghostff/Session](/awesome-plugins/ghost-session) - PHP Session Manager (non-blocking, flash, segment, session encryption). Menggunakan PHP open_ssl untuk enkripsi/dekripsi data session opsional.

## Templating

Templating adalah inti dari aplikasi web apa pun dengan UI. Ada sejumlah mesin templating yang dapat digunakan dengan Flight.

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - Ini adalah mesin templating yang sangat dasar yang merupakan bagian dari core. Tidak disarankan untuk digunakan jika Anda memiliki lebih dari beberapa halaman dalam proyek Anda.
- [latte/latte](/awesome-plugins/latte) - Latte adalah mesin templating yang lengkap yang sangat mudah digunakan dan terasa lebih dekat dengan sintaks PHP daripada Twig atau Smarty. Juga sangat mudah untuk diperluas dan menambahkan filter dan fungsi Anda sendiri.
- [twig/twig](/awesome-plugins/twig) - Twig adalah mesin template yang fleksibel, cepat, dan aman (yang sama digunakan oleh Symfony). Alat AI dan banyak pengembang PHP mengenalnya dengan baik, secara otomatis escape output secara default, dan memiliki ekosistem ekstensi yang besar.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate adalah mesin template PHP yang kuat dengan kompilasi asset, pewarisan template, dan pemrosesan variabel. Fitur minifikasi CSS/JS otomatis, caching, encoding Base64, dan integrasi opsional framework Flight PHP.

## Integrasi WordPress

Ingin menggunakan Flight dalam proyek WordPress Anda? Ada plugin yang berguna untuk itu!

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Plugin WordPress ini memungkinkan Anda menjalankan Flight bersama dengan WordPress. Sempurna untuk menambahkan API khusus, microservice, atau bahkan aplikasi lengkap ke situs WordPress Anda menggunakan framework Flight. Sangat berguna jika Anda ingin yang terbaik dari kedua dunia!

## Berkontribusi

Punya plugin yang ingin Anda bagikan? Kirim pull request untuk menambahkannya ke daftar!