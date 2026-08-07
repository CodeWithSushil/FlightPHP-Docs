# Dokumentasi FlightPHP APM

Selamat datang di FlightPHP APM—pelatih performa pribadi untuk aplikasi Anda! Panduan ini adalah peta jalan untuk menyiapkan, menggunakan, dan menguasai Application Performance Monitoring (APM) dengan FlightPHP. Baik Anda sedang mencari permintaan lambat atau hanya ingin mengeksplorasi grafik latensi, kami siap membantu. Mari buat aplikasi Anda lebih cepat, pengguna lebih bahagia, dan sesi debugging lebih mudah!

Lihat [demo](https://flightphp-docs-apm.sky-9.com/apm/dashboard) dari dashboard untuk situs Flight Docs.

![FlightPHP APM](/images/apm.png)

## Mengapa APM Penting

Bayangkan ini: aplikasi Anda adalah restoran yang sibuk. Tanpa cara untuk melacak berapa lama pesanan diproses atau di mana dapur mengalami hambatan, Anda hanya bisa menebak mengapa pelanggan pergi dengan tidak senang. APM adalah koki kedua Anda—memantau setiap langkah, dari permintaan masuk hingga kueri database, dan menandai apa pun yang memperlambat Anda. Halaman yang lambat kehilangan pengguna (studi menunjukkan 53% bounce jika situs membutuhkan waktu lebih dari 3 detik untuk memuat!), dan APM membantu Anda menangkap masalah tersebut *sebelum* berdampak. Ini adalah ketenangan pikiran yang proaktif—lebih sedikit momen "kenapa ini rusak?" dan lebih banyak kemenangan "lihat betapa lancarnya ini berjalan!"

## Instalasi

Mulai dengan Composer:

```bash
composer require flightphp/apm
```

Anda akan memerlukan:
- **PHP 7.4+**: Menjaga kami kompatibel dengan distro Linux LTS sambil mendukung PHP modern.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: Framework ringan yang kami tingkatkan.

## Database yang Didukung

FlightPHP APM saat ini mendukung database berikut untuk menyimpan metrik:

- **SQLite3**: Sederhana, berbasis file, dan bagus untuk pengembangan lokal atau aplikasi kecil. Opsi default dalam sebagian besar pengaturan.
- **MySQL/MariaDB**: Ideal untuk proyek yang lebih besar atau lingkungan produksi yang memerlukan penyimpanan yang kuat dan dapat diskalakan.

Anda dapat memilih jenis database selama langkah konfigurasi (lihat di bawah). Pastikan lingkungan PHP Anda memiliki ekstensi yang diperlukan (misalnya, `pdo_sqlite` atau `pdo_mysql`).

## Memulai

Berikut langkah demi langkah untuk kehebatan APM:

### 1. Daftarkan APM

Letakkan ini ke dalam file `index.php` atau `services.php` Anda untuk mulai melacak:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// If you're adding a database connection
// Prefer SimplePdo (or PdoQueryCapture from Tracy Extensions in dev).
// Enable APM query tracking via the options array (5th argument).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**Apa yang terjadi di sini?**
- `LoggerFactory::create()` mengambil konfigurasi Anda (lebih lanjut segera) dan menyiapkan logger—SQLite secara default.
- `Apm` adalah bintangnya—mendengarkan event Flight (permintaan, rute, error, dll.) dan mengumpulkan metrik.
- `bindEventsToFlightInstance($app)` menghubungkan semuanya ke aplikasi Flight Anda.

**Tips Pro: Sampling**
Jika aplikasi Anda sibuk, mencatat *setiap* permintaan mungkin membebani sistem. Gunakan tingkat sampel (0.0 hingga 1.0):

```php
$Apm = new Apm($ApmLogger, 0.1); // Logs 10% of requests
```

Ini menjaga performa tetap cepat sambil tetap memberikan data yang solid.

### 2. Konfigurasikan

Jalankan ini untuk membuat `.runway-config.json` Anda:

```bash
php vendor/bin/runway apm:init
```

**Apa yang dilakukan ini?**
- Meluncurkan wizard yang menanyakan dari mana metrik mentah berasal (sumber) dan ke mana data yang diproses pergi (tujuan).
- Default adalah SQLite—misalnya, `sqlite:/tmp/apm_metrics.sqlite` untuk sumber, yang lain untuk tujuan.
- Anda akan berakhir dengan konfigurasi seperti:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Proses ini juga akan menanyakan apakah Anda ingin menjalankan migrasi untuk pengaturan ini. Jika Anda mengatur ini untuk pertama kalinya, jawabannya adalah ya.

**Mengapa dua lokasi?**
Metrik mentah menumpuk cepat (pikirkan log yang tidak difilter). Worker memprosesnya menjadi tujuan terstruktur untuk dashboard. Menjaga segalanya tetap rapi!

### 3. Proses Metrik dengan Worker

Worker mengubah metrik mentah menjadi data siap dashboard. Jalankan sekali:

```bash
php vendor/bin/runway apm:worker
```

**Apa yang dilakukannya?**
- Membaca dari sumber Anda (misalnya, `apm_metrics.sqlite`).
- Memproses hingga 100 metrik (ukuran batch default) ke tujuan Anda.
- Berhenti ketika selesai atau jika tidak ada metrik yang tersisa.

**Jaga Tetap Berjalan**
Untuk aplikasi langsung, Anda akan menginginkan pemrosesan berkelanjutan. Berikut opsi Anda:

- **Mode Daemon**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Berjalan selamanya, memproses metrik saat datang. Bagus untuk dev atau pengaturan kecil.

- **Crontab**:
  Tambahkan ini ke crontab Anda (`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  Menembak setiap menit—sempurna untuk produksi.

- **Tmux/Screen**:
  Mulai sesi yang dapat dilepas:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, then D to detach; `tmux attach -t apm-worker` to reconnect
  ```
  Menjaganya tetap berjalan bahkan jika Anda logout.

- **Penyesuaian Kustom**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: Proses 50 metrik sekaligus.
  - `--max_messages 1000`: Berhenti setelah 1000 metrik.
  - `--timeout 300`: Keluar setelah 5 menit.

**Mengapa repot?**
Tanpa worker, dashboard Anda kosong. Ini adalah jembatan antara log mentah dan wawasan yang dapat ditindaklanjuti.

### 4. Luncurkan Dashboard

Lihat vital aplikasi Anda:

```bash
php vendor/bin/runway apm:dashboard
```

**Apa ini?**
- Memulai server PHP di `http://localhost:8001/apm/dashboard`.
- Menampilkan log permintaan, rute lambat, tingkat error, dan lainnya.

**Kustomisasi**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: Dapat diakses dari IP mana pun (berguna untuk melihat dari jarak jauh).
- `--port 8080`: Gunakan port berbeda jika 8001 sudah digunakan.
- `--php-path`: Tunjuk ke PHP jika tidak ada di PATH Anda.

Buka URL di browser Anda dan jelajahi!

#### Mode Produksi

Untuk produksi, Anda mungkin harus mencoba beberapa teknik untuk menjalankan dashboard karena mungkin ada firewall dan langkah keamanan lainnya. Berikut beberapa opsi:

- **Gunakan Reverse Proxy**: Siapkan Nginx atau Apache untuk meneruskan permintaan ke dashboard.
- **SSH Tunnel**: Jika Anda dapat SSH ke server, gunakan `ssh -L 8080:localhost:8001
youruser@yourserver` untuk men-tunnel dashboard ke mesin lokal Anda.
- **VPN**: Jika server Anda berada di belakang VPN, hubungkan ke sana dan akses dashboard secara langsung.
- **Konfigurasi Firewall**: Buka port 8001 untuk IP Anda atau jaringan server. (atau port apa pun yang Anda atur).
- **Konfigurasi Apache/Nginx**: Jika Anda memiliki web server di depan aplikasi Anda, Anda dapat mengonfigurasinya ke domain atau subdomain. Jika Anda melakukan ini, Anda akan mengatur document root ke `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### Ingin dashboard yang berbeda?

Anda dapat membangun dashboard sendiri jika mau! Lihat direktori vendor/flightphp/apm/src/apm/presenter untuk ide tentang cara menyajikan data untuk dashboard Anda sendiri!

## Fitur Dashboard

Dashboard adalah markas besar APM Anda—berikut yang akan Anda lihat:

- **Request Log**: Setiap permintaan dengan timestamp, URL, kode respons, dan total waktu. Klik "Details" untuk middleware, query, dan error.
- **Slowest Requests**: 5 permintaan teratas yang menghabiskan waktu (misalnya, "/api/heavy" dalam 2.5s).
- **Slowest Routes**: 5 rute teratas berdasarkan waktu rata-rata—bagus untuk menemukan pola.
- **Error Rate**: Persentase permintaan yang gagal (misalnya, 2.3% 500s).
- **Latency Percentiles**: Waktu respons ke-95 (p95) dan ke-99 (p99)—ketahui skenario terburuk Anda.
- **Response Code Chart**: Visualisasikan 200s, 404s, 500s dari waktu ke waktu.
- **Long Queries/Middleware**: 5 panggilan database lambat dan lapisan middleware teratas.
- **Cache Hit/Miss**: Seberapa sering cache Anda menyelamatkan hari.

**Ekstra**:
- Filter berdasarkan "Last Hour," "Last Day," atau "Last Week."
- Toggle dark mode untuk sesi larut malam.

**Contoh**:
Permintaan ke `/users` mungkin menunjukkan:
- Total Time: 150ms
- Middleware: `AuthMiddleware->handle` (50ms)
- Query: `SELECT * FROM users` (80ms)
- Cache: Hit pada `user_list` (5ms)

## Menambahkan Event Kustom

Lacak apa saja—seperti panggilan API atau proses pembayaran:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**Di mana itu muncul?**
Dalam detail permintaan dashboard di bawah "Custom Events"—dapat diperluas dengan pemformatan JSON yang cantik.

**Use Case**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
Sekarang Anda akan melihat apakah API tersebut memperlambat aplikasi Anda!

## Monitoring Database

Lacak query PDO seperti ini:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**Apa yang Anda Dapatkan**:
- Teks query (misalnya, `SELECT * FROM users WHERE id = ?`)
- Waktu eksekusi (misalnya, 0.015s)
- Jumlah baris (misalnya, 42)

**Perhatian**:
- **Opsional**: Lewati ini jika Anda tidak memerlukan pelacakan DB.
- **SimplePdo (preferred)**: Gunakan `SimplePdo` dengan `trackApmQueries => true`. `PdoWrapper` yang sudah deprecated masih berfungsi (argumen konstruktor ke-5 `true`). PDO core mentah belum di-hook—tetap pantau!
- **Performance Warning**: Mencatat setiap query pada situs yang berat DB dapat memperlambat hal-hal. Gunakan sampling (`$Apm = new Apm($ApmLogger, 0.1)`) untuk meringankan beban.

**Contoh Output**:
- Query: `SELECT name FROM products WHERE price > 100`
- Time: 0.023s
- Rows: 15

## Opsi Worker

Sesuaikan worker sesuai keinginan Anda:

- `--timeout 300`: Berhenti setelah 5 menit—bagus untuk pengujian.
- `--max_messages 500`: Batas pada 500 metrik—menjaganya tetap terbatas.
- `--batch_size 200`: Memproses 200 sekaligus—menyeimbangkan kecepatan dan memori.
- `--daemon`: Berjalan tanpa henti—ideal untuk monitoring langsung.

**Contoh**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Berjalan selama satu jam, memproses 100 metrik sekaligus.

## Request ID dalam Aplikasi

Setiap permintaan memiliki request ID unik untuk pelacakan. Anda dapat menggunakan ID ini dalam aplikasi Anda untuk mengkorelasikan log dan metrik. Misalnya, Anda dapat menambahkan request ID ke halaman error:

```php
Flight::map('error', function($message) {
	// Get the request ID from the response header X-Flight-Request-Id
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Additionally you could fetch it from the Flight variable
	// This method won't work well in swoole or other async platforms.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## Upgrade

Jika Anda melakukan upgrade ke versi APM yang lebih baru, ada kemungkinan ada migrasi database yang perlu dijalankan. Anda dapat melakukannya dengan menjalankan perintah berikut:

```bash
php vendor/bin/runway apm:migrate
```
Ini akan menjalankan migrasi apa pun yang diperlukan untuk memperbarui skema database ke versi terbaru.

**Catatan:** Jika database APM Anda berukuran besar, migrasi ini mungkin memakan waktu. Anda mungkin ingin menjalankan perintah ini di luar jam sibuk.

### Upgrade dari 0.4.3 -> 0.5.0

Jika Anda melakukan upgrade dari 0.4.3 ke 0.5.0, Anda perlu menjalankan perintah berikut:

```bash
php vendor/bin/runway apm:config-migrate
```

Ini akan memigrasikan konfigurasi Anda dari format lama menggunakan file `.runway-config.json` ke format baru yang menyimpan key/values dalam file `config.php`.

## Menghapus Data Lama

Untuk menjaga database Anda tetap rapi, Anda dapat menghapus data lama. Ini sangat berguna jika Anda menjalankan aplikasi yang sibuk dan ingin menjaga ukuran database tetap manageable.
Anda dapat melakukannya dengan menjalankan perintah berikut:

```bash
php vendor/bin/runway apm:purge
```
Ini akan menghapus semua data yang lebih lama dari 30 hari dari database. Anda dapat menyesuaikan jumlah hari dengan memberikan nilai berbeda ke opsi `--days`:

```bash
php vendor/bin/runway apm:purge --days 7
```
Ini akan menghapus semua data yang lebih lama dari 7 hari dari database.

## Troubleshooting

Buntu? Coba ini:

- **Tidak Ada Data Dashboard?**
  - Apakah worker sedang berjalan? Periksa `ps aux | grep apm:worker`.
  - Apakah path konfigurasi cocok? Verifikasi DSN `.runway-config.json` menunjuk ke file yang sebenarnya.
  - Jalankan `php vendor/bin/runway apm:worker` secara manual untuk memproses metrik yang tertunda.

- **Worker Errors?**
  - Lihat file SQLite Anda (misalnya, `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Periksa log PHP untuk stack trace.

- **Dashboard Tidak Mulai?**
  - Port 8001 sedang digunakan? Gunakan `--port 8080`.
  - PHP tidak ditemukan? Gunakan `--php-path /usr/bin/php`.
  - Firewall memblokir? Buka port atau gunakan `--host localhost`.

- **Terlalu Lambat?**
  - Turunkan tingkat sampel: `$Apm = new Apm($ApmLogger, 0.05)` (5%).
  - Kurangi ukuran batch: `--batch_size 20`.

- **Tidak Melacak Exception/Error?**
  - Jika Anda memiliki [Tracy](https://tracy.nette.org/) diaktifkan untuk proyek Anda, itu akan menggantikan penanganan error Flight. Anda perlu menonaktifkan Tracy dan kemudian pastikan bahwa `Flight::set('flight.handle_errors', true);` sudah diatur.

- **Tidak Melacak Query Database?**
  - Lebih suka `SimplePdo` dengan `['trackApmQueries' => true]` sebagai argumen konstruktor ke-5 (array opsi).
  - Jika Anda masih menggunakan `PdoWrapper` yang sudah deprecated, lewatkan `true` sebagai argumen ke-5.
  - Panggil `$Apm->addPdoConnection($pdo)` setelah membuat koneksi.