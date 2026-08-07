# Instruksi Instalasi

Ada beberapa prasyarat dasar sebelum Anda dapat menginstal Flight. Yaitu Anda perlu:

1. [Instal PHP di sistem Anda](#installing-php)
2. [Instal Composer](https://getcomposer.org) untuk pengalaman pengembang terbaik.

## Instalasi Dasar

Jika Anda menggunakan [Composer](https://getcomposer.org), Anda dapat menjalankan perintah berikut:

```bash
composer require flightphp/core
```

Ini hanya akan menempatkan file inti Flight di sistem Anda. Anda perlu menentukan struktur proyek, [tata letak](/learn/templates), [dependensi](/learn/dependency-injection-container), [konfigurasi](/learn/configuration), [autoloading](/learn/autoloading), dll. Metode ini memastikan tidak ada dependensi lain selain Flight yang diinstal.

Anda juga dapat [mengunduh file](https://github.com/flightphp/core/archive/master.zip) secara langsung dan mengekstraknya ke direktori web Anda.

Instalasi dasar sangat cocok untuk belajar, API mikro, dan eksperimen salin-tempel. Untuk tata letak aplikasi lengkap yang dapat diikuti oleh manusia *dan* [alat coding AI](/learn/ai) dengan cara yang sama, gunakan skeleton yang direkomendasikan di bawah ini.

## Instalasi yang Disarankan

Sangat disarankan untuk memulai dengan aplikasi [flightphp/skeleton](https://github.com/flightphp/skeleton) untuk proyek baru apa pun. Instalasinya sangat mudah.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# opsional DB contoh + demo postingan
php runway migrate
```

Langkah tersebut mengatur struktur proyek, autoloading Composer PSR-4, konfigurasi, dan alat seperti [Tracy](/awesome-plugins/tracy), [Ekstensi Tracy](/awesome-plugins/tracy-extensions), dan [Runway](/awesome-plugins/runway). Ini juga menyertakan **`AGENTS.md`** di root (dan salinan scoped di bawah `app/`) sehingga asisten AI berbagi satu tata letak dengan Anda—lihat [Pengalaman AI & pengembang](/learn/ai).

### Apa yang diberikan skeleton kepada Anda

```text
project-root/
├── AGENTS.md              # Sumber kebenaran AI / agen
├── SECURITY.md            # Ekspektasi keamanan
├── .env.example           # Rahasia / overlay deploy (disalin ke .env)
├── public/index.php       # Hanya entri web
├── app/
│   ├── config/            # bootstrap, rute, layanan, config_sample.php
│   ├── Controller/        # App\Controller\*  (folder PascalCase!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Perintah CLI Runway
│   ├── views/             # Template Twig (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # Migrasi SQL (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Namespace mengikuti huruf besar/kecil folder.** Composer memetakan `"App\\": "app/"`, jadi:

| Path di disk | Namespace |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Di Linux, `app/controller/` **tidak** sama dengan `app/Controller/`. Autoloading peka huruf besar/kecil—cocokkan folder PascalCase milik skeleton. Detail: [Autoloading](/learn/autoloading).

**Default stack (proyek baru):** View Twig, SimplePdo + ActiveRecord, Dice dengan injeksi `Engine` (lebih baik tanpa `Flight::` di dalam kelas aplikasi), SQLite opsional setelah `php runway migrate`.

`create-project` biasanya menyalin `app/config/config_sample.php` → `config.php` dan `.env.example` → `.env` jika ada. Rute berada di `app/config/routes.php`; layanan dan DI berada di `app/config/services.php`.

> **Dokumen ↔ skeleton:** Dokumen ini mengajarkan **API** Flight (sering dengan contoh `Flight::` singkat). Skeleton menetapkan **bentuk aplikasi**. Saat menambahkan kode di bawah `app/`, ikuti struktur skeleton; gunakan dokumen untuk nama metode, opsi, dan plugin.

## Konfigurasi Server Web Anda

### Server Pengembangan PHP Bawaan

Ini adalah cara paling sederhana untuk memulai. Anda dapat menggunakan server bawaan untuk menjalankan aplikasi Anda dan bahkan menggunakan SQLite sebagai basis data (selama sqlite3 terinstal di sistem Anda) dan tidak memerlukan banyak hal! Cukup jalankan perintah berikut setelah PHP terinstal:

```bash
php -S localhost:8000
# atau dengan aplikasi skeleton
composer start
```

Kemudian buka browser Anda dan pergi ke `http://localhost:8000`.

Jika Anda ingin menjadikan direktori root dokumen proyek Anda sebagai direktori yang berbeda (Contoh: proyek Anda berada di `~/myproject`, tetapi root dokumen Anda adalah `~/myproject/public/`), Anda dapat menjalankan perintah berikut setelah berada di direktori `~/myproject`:

```bash
php -S localhost:8000 -t public/
# dengan aplikasi skeleton, ini sudah dikonfigurasi
composer start
```

Kemudian buka browser Anda dan pergi ke `http://localhost:8000`.

### Apache

Pastikan Apache sudah terinstal di sistem Anda. Jika belum, cari di Google cara menginstal Apache di sistem Anda.

Untuk Apache, edit file `.htaccess` Anda dengan berikut:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Catatan**: Jika Anda perlu menggunakan Flight di subdirektori, tambahkan baris `RewriteBase /subdir/` tepat setelah `RewriteEngine On`.

> **Catatan**: Jika Anda ingin melindungi semua file server, seperti file db atau env. Letakkan ini di file `.htaccess` Anda:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Pastikan Nginx sudah terinstal di sistem Anda. Jika belum, cari di Google cara menginstal Nginx di sistem Anda.

Untuk Nginx, tambahkan berikut ke deklarasi server Anda:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Membuat File `index.php` Anda

Jika Anda melakukan instalasi dasar, Anda perlu memiliki beberapa kode untuk memulai.

```php
<?php

// Jika Anda menggunakan Composer, require autoloader.
require 'vendor/autoload.php';
// jika Anda tidak menggunakan Composer, muat framework secara langsung
// require 'flight/Flight.php';

// Kemudian tentukan rute dan tetapkan fungsi untuk menangani permintaan.
Flight::route('/', function () {
  echo 'hello world!';
});

// Terakhir, jalankan framework.
Flight::start();
```

Dengan aplikasi skeleton, entri publik hanya mem-boot aplikasi. Rute didaftarkan di `app/config/routes.php` (biasanya `[App\Controller\…::class, 'method']` sehingga Dice dapat menyuntikkan dependensi). Layanan, Twig, SimplePdo, dan kontainer dihubungkan di `app/config/services.php`. Struktur tersebut disengaja agar alat AI dan manusia mengedit tempat yang sama setiap saat.

<a id="installing-php"></a>
## Menginstal PHP

Jika Anda sudah memiliki `php` yang terinstal di sistem Anda, silakan lewati instruksi ini dan lanjutkan ke [bagian unduhan](#download-the-files)

### **macOS**

#### **Menginstal PHP menggunakan Homebrew**

1. **Instal Homebrew** (jika belum terinstal):
   - Buka Terminal dan jalankan:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Instal PHP**:
   - Instal versi terbaru:
     ```bash
     brew install php
     ```
   - Untuk menginstal versi tertentu, misalnya PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Berpindah antar versi PHP**:
   - Lepas tautan versi saat ini dan tautkan versi yang diinginkan:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Verifikasi versi yang terinstal:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **Menginstal PHP secara manual**

1. **Unduh PHP**:
   - Kunjungi [PHP untuk Windows](https://windows.php.net/download/) dan unduh versi terbaru atau versi tertentu (mis., 7.4, 8.0) sebagai file zip non-thread-safe.

2. **Ekstrak PHP**:
   - Ekstrak file zip yang diunduh ke `C:\php`.

3. **Tambahkan PHP ke PATH sistem**:
   - Buka **System Properties** > **Environment Variables**.
   - Di bawah **System variables**, temukan **Path** dan klik **Edit**.
   - Tambahkan path `C:\php` (atau di mana pun Anda mengekstrak PHP).
   - Klik **OK** untuk menutup semua jendela.

4. **Konfigurasi PHP**:
   - Salin `php.ini-development` ke `php.ini`.
   - Edit `php.ini` untuk mengonfigurasi PHP sesuai kebutuhan (mis., mengatur `extension_dir`, mengaktifkan ekstensi).

5. **Verifikasi instalasi PHP**:
   - Buka Command Prompt dan jalankan:
     ```cmd
     php -v
     ```

#### **Menginstal Beberapa Versi PHP**

1. **Ulangi langkah-langkah di atas** untuk setiap versi, letakkan masing-masing di direktori terpisah (mis., `C:\php7`, `C:\php8`).

2. **Berpindah antar versi** dengan menyesuaikan variabel PATH sistem agar menunjuk ke direktori versi yang diinginkan.

### **Ubuntu (20.04, 22.04, dll.)**

#### **Menginstal PHP menggunakan apt**

1. **Perbarui daftar paket**:
   - Buka Terminal dan jalankan:
     ```bash
     sudo apt update
     ```

2. **Instal PHP**:
   - Instal versi PHP terbaru:
     ```bash
     sudo apt install php
     ```
   - Untuk menginstal versi tertentu, misalnya PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Instal modul tambahan** (opsional):
   - Misalnya, untuk menginstal dukungan MySQL:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Berpindah antar versi PHP**:
   - Gunakan `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Verifikasi versi yang terinstal**:
   - Jalankan:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **Menginstal PHP menggunakan yum/dnf**

1. **Aktifkan repositori EPEL**:
   - Buka Terminal dan jalankan:
     ```bash
     sudo dnf install epel-release
     ```

2. **Instal repositori Remi**:
   - Jalankan:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Instal PHP**:
   - Untuk menginstal versi default:
     ```bash
     sudo dnf install php
     ```
   - Untuk menginstal versi tertentu, misalnya PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Berpindah antar versi PHP**:
   - Gunakan perintah modul `dnf`:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Verifikasi versi yang terinstal**:
   - Jalankan:
     ```bash
     php -v
     ```

### **Catatan Umum**

- Untuk lingkungan pengembangan, penting untuk mengonfigurasi pengaturan PHP sesuai kebutuhan proyek Anda.
- Saat berpindah versi PHP, pastikan semua ekstensi PHP yang relevan terinstal untuk versi tertentu yang ingin Anda gunakan.
- Restart server web Anda (Apache, Nginx, dll.) setelah berpindah versi PHP atau memperbarui konfigurasi agar perubahan diterapkan.