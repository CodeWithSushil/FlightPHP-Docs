# Membangun Blog Sederhana dengan Flight PHP

Panduan ini memandu Anda membuat blog dasar menggunakan framework PHP Flight. Anda akan menyiapkan proyek, mendefinisikan rute, mengelola posting dengan JSON, dan merendernya dengan mesin templat Latte—semuanya menunjukkan kesederhanaan dan fleksibilitas Flight. Pada akhirnya, Anda akan memiliki blog yang berfungsi dengan beranda, halaman posting individu, dan formulir pembuatan.

## Prasyarat
- **PHP 7.4+**: Terinstal di sistem Anda.
- **Composer**: Untuk manajemen dependensi.
- **Editor Teks**: Editor apa pun seperti VS Code atau PHPStorm.
- Pengetahuan dasar tentang PHP dan pengembangan web.

## Langkah 1: Siapkan Proyek Anda

Mulailah dengan membuat direktori proyek baru dan menginstal Flight melalui Composer.

1. **Buat Direktori**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Instal Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Buat Direktori Publik**:
   Flight menggunakan satu titik masuk (`index.php`). Buat folder `public/` untuk itu:
   ```bash
   mkdir public
   ```

4. **`index.php` Dasar**:
   Buat `public/index.php` dengan rute "hello world" sederhana:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Jalankan Server Bawaan**:
   Uji pengaturan Anda dengan server pengembangan PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Kunjungi `http://localhost:8000` untuk melihat "Hello, Flight!".

## Langkah 2: Atur Struktur Proyek Anda

Untuk pengaturan yang rapi, susun proyek Anda seperti ini:

```text
flight-blog/
├── app/
│   ├── config/
│   └── views/
├── data/
├── public/
│   └── index.php
├── vendor/
└── composer.json
```

- `app/config/`: File konfigurasi (misalnya, events, routes).
- `app/views/`: Templat untuk merender halaman.
- `data/`: File JSON untuk menyimpan posting blog.
- `public/`: Root web dengan `index.php`.

## Langkah 3: Instal dan Konfigurasi Latte

Latte adalah mesin templat ringan yang terintegrasi dengan baik dengan Flight.

1. **Instal Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Konfigurasi Latte di Flight**:
   Perbarui `public/index.php` untuk mendaftarkan Latte sebagai mesin tampilan:
   ```php
   <?php
   require '../vendor/autoload.php';

   use Latte\Engine;

   Flight::register('view', Engine::class, [], function ($latte) {
       $latte->setTempDirectory(__DIR__ . '/../cache/');
       $latte->setLoader(new \Latte\Loaders\FileLoader(__DIR__ . '/../app/views/'));
   });

   Flight::route('/', function () {
       Flight::view()->render('home.latte', ['title' => 'My Blog']);
   });

   Flight::start();
   ```

3. **Buat Templat Layout**:
   Di `app/views/layout.latte`:
   ```html
   <!DOCTYPE html>
   <html>
   <head>
       <title>{$title}</title>
   </head>
   <body>
       <header>
           <h1>My Blog</h1>
           <nav>
               <a href="/">Beranda</a> | 
               <a href="/create">Buat Posting</a>
           </nav>
       </header>
       <main>
           {block content}{/block}
       </main>
       <footer>
           <p>&copy; {date('Y')} Flight Blog</p>
       </footer>
   </body>
   </html>
   ```

4. **Buat Templat Beranda**:
   Di `app/views/home.latte`:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$title}</h2>
		<ul>
		{foreach $posts as $post}
			<li><a href="/post/{$post['slug']}">{$post['title']}</a></li>
		{/foreach}
		</ul>
	{/block}
   ```
   Mulai ulang server jika Anda keluar dan kunjungi `http://localhost:8000` untuk melihat halaman yang dirender.

5. **Buat File Data**:

   Gunakan file JSON untuk mensimulasikan database agar sederhana.

   Di `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Langkah 4: Definisikan Rute

Pisahkan rute Anda ke dalam file konfigurasi untuk organisasi yang lebih baik.

1. **Buat `routes.php`**:
   Di `app/config/routes.php`:
   ```php
   <?php
   Flight::route('/', function () {
       Flight::view()->render('home.latte', ['title' => 'My Blog']);
   });

   Flight::route('/post/@slug', function ($slug) {
       Flight::view()->render('post.latte', ['title' => 'Post: ' . $slug, 'slug' => $slug]);
   });

   Flight::route('GET /create', function () {
       Flight::view()->render('create.latte', ['title' => 'Buat Posting']);
   });
   ```

2. **Perbarui `index.php`**:
   Sertakan file rute:
   ```php
   <?php
   require '../vendor/autoload.php';

   use Latte\Engine;

   Flight::register('view', Engine::class, [], function ($latte) {
       $latte->setTempDirectory(__DIR__ . '/../cache/');
       $latte->setLoader(new \Latte\Loaders\FileLoader(__DIR__ . '/../app/views/'));
   });

   require '../app/config/routes.php';

   Flight::start();
   ```

## Langkah 5: Simpan dan Ambil Posting Blog

Tambahkan metode untuk memuat dan menyimpan posting.

1. **Tambahkan Metode Posts**:
   Di `index.php`, tambahkan metode untuk memuat posting:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Perbarui Rute**:
   Ubah `app/config/routes.php` untuk menggunakan posting:
   ```php
   <?php
   Flight::route('/', function () {
       $posts = Flight::posts();
       Flight::view()->render('home.latte', [
           'title' => 'My Blog',
           'posts' => $posts
       ]);
   });

   Flight::route('/post/@slug', function ($slug) {
       $posts = Flight::posts();
       $post = array_filter($posts, fn($p) => $p['slug'] === $slug);
       $post = reset($post) ?: null;
       if (!$post) {
           Flight::notFound();
           return;
       }
       Flight::view()->render('post.latte', [
           'title' => $post['title'],
           'post' => $post
       ]);
   });

   Flight::route('GET /create', function () {
       Flight::view()->render('create.latte', ['title' => 'Buat Posting']);
   });
   ```

## Langkah 6: Buat Templat

Perbarui templat Anda untuk menampilkan posting.

1. **Halaman Posting (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Langkah 7: Tambahkan Pembuatan Posting

Tangani pengiriman formulir untuk menambahkan posting baru.

1. **Buat Formulir (`app/views/create.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$title}</h2>
		<form method="POST" action="/create">
			<div class="form-group">
				<label for="title">Judul:</label>
				<input type="text" name="title" id="title" required>
			</div>
			<div class="form-group">
				<label for="content">Konten:</label>
				<textarea name="content" id="content" required></textarea>
			</div>
			<button type="submit">Simpan Posting</button>
		</form>
	{/block}
   ```

2. **Tambahkan Rute POST**:
   Di `app/config/routes.php`:
   ```php
   Flight::route('POST /create', function () {
       $request = Flight::request();
       $title = $request->data['title'];
       $content = $request->data['content'];
       $slug = strtolower(str_replace(' ', '-', $title));

       $posts = Flight::posts();
       $posts[] = ['slug' => $slug, 'title' => $title, 'content' => $content];
       file_put_contents(__DIR__ . '/../../data/posts.json', json_encode($posts, JSON_PRETTY_PRINT));

       Flight::redirect('/');
   });
   ```

3. **Uji Coba**:
   - Kunjungi `http://localhost:8000/create`.
   - Kirim posting baru (misalnya, "Second Post" dengan beberapa konten).
   - Periksa beranda untuk melihatnya terdaftar.

## Langkah 8: Tingkatkan dengan Penanganan Kesalahan

Timpa metode `notFound` untuk pengalaman 404 yang lebih baik.

Di `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Halaman Tidak Ditemukan']);
});
```

Buat `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Maaf, halaman itu tidak ada!</p>
{/block}
```

## Langkah Berikutnya
- **Tambahkan Gaya**: Gunakan CSS di templat Anda untuk tampilan yang lebih baik.
- **Database**: Ganti `posts.json` dengan database seperti SQLite menggunakan [SimplePdo](/learn/simple-pdo).
- **Validasi**: Tambahkan pemeriksaan untuk slug duplikat atau input kosong.
- **Middleware**: Terapkan autentikasi untuk pembuatan posting.

## Kesimpulan

Anda telah membangun blog sederhana dengan Flight PHP! Panduan ini menunjukkan fitur inti seperti routing, templating dengan Latte, dan penanganan pengiriman formulir—semuanya dengan tetap ringan. Jelajahi dokumentasi Flight untuk fitur yang lebih lanjut guna membawa blog Anda lebih jauh!