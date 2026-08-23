# FlightMail

> **Plugin pihak ketiga** - dikelola oleh [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), dilisensikan MIT). Bukan bagian dari Flight core - silakan laporkan isu di [repositori GitHub-nya](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) memungkinkan Anda mengirim email dari aplikasi Flight tanpa pusing. Library ini membungkus **Symfony Mailer** - library mail yang paling teruji di PHP - dan membuatnya terasa seperti bagian dari Flight. Satu baris untuk instal, satu rantai fluent untuk mengirim:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Berhasil!')
    ->text('Email pertama Anda sedang dalam perjalanan.')
    ->send();
```

## Fitur

- **Penyedia apa pun, masing-masing satu baris.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo dan kawan-kawan semuanya bekerja melalui string DSN sederhana.
- **Gunakan beberapa penyedia sekaligus.** Mail transaksional melalui Postmark, newsletter melalui SMTP Anda sendiri - pilih per pesan.
- **Templat jika Anda menginginkannya.** Render body dengan Twig atau Latte. Tidak mau templat? Cukup kirim string dan tidak perlu instal apa pun extra.
- **Polesan saat kirim.** CSS inlining opsional dan bagian teks biasa otomatis yang diturunkan dari HTML Anda, didukung library yang hanya Anda instal jika digunakan.
- **Membosankan dengan cara terbaik.** Koneksi lazy, error yang jelas alih-alih mail yang ditelan diam-diam, dan semuanya dapat ditukar jika Anda butuh sesuatu yang kustom.

## Persyaratan

| Apa            | Versi                                      |
| -------------- | ------------------------------------------ |
| PHP            | 8.2 atau lebih baru                        |
| Flight PHP     | core ^3.15                                 |
| Symfony Mailer | ^7.2 atau ^8.0 (diinstal secara otomatis)  |

## Instalasi

```bash
composer require ryanstubbs/flightmail
```

Itu saja untuk mengirim email teks biasa dan HTML. Rendering templat bersifat opt-in - tambahkan engine hanya jika Anda akan menggunakannya:

```bash
composer require twig/twig      # untuk templat .twig
composer require latte/latte    # untuk templat .latte
```

Dua library opsional lagi mendukung peningkatan saat kirim yang dibahas [di bawah](#styling-html-dan-menghasilkan-bagian-teks):

```bash
composer require pelago/emogrifier         # untuk CSS inlining ("inline_css")
composer require league/html-to-markdown   # untuk bagian teks Markdown ("text_from_html")
```

Semua ini dapat diinstal berdampingan; FlightMail memilih yang tepat berdasarkan apa yang Anda konfigurasi.

## Email pertama Anda

Tambahkan ini ke bootstrap Anda (tempat yang sama Anda mendefinisikan rute):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Beritahu FlightMail dari mana dan melalui apa mail dikirim.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Selamat bergabung!')
        ->html('<h1>Selamat datang!</h1><p>Kami senang Anda di sini.</p>')
        ->send();
});

Flight::start();
```

Menggunakan [Flight PHP skeleton](https://github.com/flightphp/skeleton)? Daftarkan di `app/config/services.php` dengan gaya instance:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Kedua gaya mengekspos mailer yang sama: `Flight::mail()` dan `$app->mail()` dapat dipertukarkan.

> **Testing secara lokal?** Jika proyek Anda berjalan di [DDEV](https://ddev.com), arahkan DSN ke `smtp://127.0.0.1:1025` dan baca setiap email yang tertangkap di Mailpit pada `http://<project>.ddev.site:8025`. Tidak ada yang keluar dari mesin Anda.

## Mengirim email

### String biasa (tidak perlu mesin templat)

`->text()` dan `->html()` menerima string mentah dan tidak butuh apa pun lain yang terinstal:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Backup selesai')
    ->text('Backup malam selesai dalam 42 menit.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Faktur #123')
    ->html('<h1>Faktur #123</h1><p>Total tagihan: $42.00</p>')
    ->send();
```

### Templat Twig

```php
// welcome.html.twig berisi: Halo {{ name }}, terima kasih sudah mendaftar!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Selamat datang!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Templat Latte

Ide yang sama, ekstensi `.latte`:

```php
// welcome.latte berisi: Halo {$name}, terima kasih sudah mendaftar!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Selamat datang!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + teks biasa bersama

Best practice untuk deliverability - berikan kedua versi kepada klien mail:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Selamat datang!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // versi kaya
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // versi fallback
    ->send();
```

Beberapa hal yang perlu diketahui tentang templat:

- Mereka di-render secara **lazy**, pada saat kirim - compose sekarang, render nanti.
- Engine dipilih berdasarkan ekstensi: `.twig` → Twig, `.latte` → Latte, yang lain → default yang Anda konfigurasi (opsi `renderer`).
- Body `->html()` atau `->text()` eksplisit selalu menang atas templat, jadi Anda bisa set templat default dan override-nya per pesan.

## Styling HTML dan menghasilkan bagian teks

Dua peningkatan opsional saat kirim, keduanya mati secara default dan keduanya didukung library yang hanya Anda instal jika menginginkannya:

| Fitur                    | Instal                    | Kunci config     |
| ------------------------ | ------------------------- | ---------------- |
| CSS inlining             | `pelago/emogrifier`       | `inline_css`     |
| Bagian teks dari HTML    | `league/html-to-markdown` | `text_from_html` |

### Inline CSS ke email HTML Anda

Gmail dan sebagian besar klien webmail membuang blok `<style>` - atribut `style=""` inline adalah satu-satunya styling yang mereka hormati secara andal. Menulis itu secara manual sangat menyebalkan; biarkan [Emogrifier](https://github.com/MyIntervals/emogrifier) melakukannya saat kirim:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Dengan itu aktif, setiap body HTML mendapat CSS-nya di-inline tepat sebelum dikirim - entah berasal dari templat atau `->html()`. Pesan seperti `<style>p { color: red; }</style><p>Hai</p>` keluar sebagai `<p style="color: red;">Hai</p>`.

Untuk menyuntikkan style bersama ke setiap email (warna merek, reset) tanpa mengulanginya di setiap templat, kirim aturan secara langsung atau tunjuk ke file stylesheet:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// atau
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Kontrol per pesan:

```php
$message->inlineCss();          // paksa inlining untuk pesan ini saja
$message->withoutInlineCss();   // lewati meskipun diaktifkan secara global
```

### Hasilkan bagian teks dari HTML Anda

Best practice adalah mengirim versi HTML dan teks biasa bersama, tetapi menulis keduanya membosankan. FlightMail dapat menurunkan bagian teks dari HTML final secara otomatis - konversi dasar tidak butuh dependensi extra, karena converter sudah ikut dengan Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown jika memungkinkan, teks biasa jika tidak
]);
```

Mode:

- `true` atau `'auto'` - output Markdown jika `league/html-to-markdown` terinstal, jika tidak stripping tag sederhana.
- `'markdown'` - paksa Markdown (`composer require league/html-to-markdown`; heading menjadi `==`, tautan `[text](url)`, tebal `**bold**`).
- `'plain'` - selalu strip tag; bekerja tanpa paket extra sama sekali.

Generasi berjalan setelah rendering dan CSS inlining, dan hanya ketika pesan punya body HTML tetapi tidak punya body teks - `->text()` atau `->textTemplate()` eksplisit selalu menang. Override per pesan mencerminkan inlining:

```php
$message->textFromHtml('plain');    // paksa tag-stripping untuk yang ini
$message->withoutTextFromHtml();    // email HTML saja
```

Aktifkan mode yang library-nya belum terinstal dan Anda mendapat error yang jelas yang menyebut `composer require` persis yang harus dijalankan - tidak pernah degradasi diam-diam.

## Memilih penyedia

Penyedia terpasang melalui string DSN. Instal paket bridge, tempel DSN ke `dsns`, selesai.

| Penyedia             | Instal                                       | Contoh DSN                                   |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | bawaan                                       | `smtp://user:pass@host:587`                  |
| Sendmail             | bawaan                                       | `sendmail://default`                         |
| Dev/null (buang mail)| bawaan                                       | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

Daftar lengkap ada di [dokumentasi Symfony Mailer](https://symfony.com/doc/current/mailer.html) - apa pun yang didokumentasikan di sana bekerja di sini tanpa perubahan.

### Beberapa penyedia sekaligus

Beri nama setiap transport, lalu pilih per pesan:

```php
MailPlugin::install([
    'dsns' => [
        'transactional' => 'postmark+api://KEY@api.postmarkapp.com',
        'bulk'          => 'smtp://user:pass@bulk.example.com:587',
    ],
    'from' => 'no-reply@example.com',
]);
```

```php
// Tidak ada pemanggilan ->transport() = kunci pertama di "dsns" ("transactional" di sini).
Flight::mail()->compose()->to('...')->text('struk')->send();

// Pilih rute lain secara eksplisit.
Flight::mail()->compose()->to('...')->text('newsletter')->transport('bulk')->send();
```

## Referensi konfigurasi

Semuanya opsional kecuali `dsns`.

```php
MailPlugin::install([
    // WAJIB - nama transport => Symfony DSN.
    // Entri pertama digunakan ketika pesan tidak menyebutkan satu.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transport yang digunakan ketika pesan tidak punya ->transport() eksplisit dan
    // Anda tidak ingin kunci pertama. Harus ada di "dsns".
    'default_transport' => 'default',

    // Pengirim global. String, Symfony Address, atau ['email' => 'Name'].
    // Diterapkan hanya ketika pesan tidak set ->from() sendiri.
    'from' => ['no-reply@example.com' => 'Aplikasi Saya'],

    // Mesin templat default: 'twig', 'latte', atau nama kustom.
    // Hanya dikonsultasikan untuk templat yang ekstensinya bukan renderer terdaftar.
    'renderer' => 'twig',

    // Tempat templat berada, dicari berurutan; plus direktori cache opsional.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Opsi extra yang diteruskan langsung ke Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Sesuaikan mesin Latte saat boot: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Peningkatan body saat kirim (lihat "Styling HTML dan menghasilkan bagian teks").
    'inline_css' => true,           // atau ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // atau 'plain' / 'markdown'

    // Skema DSN kustom, renderer kustom, hook pra-kirim (lihat di bawah).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Plumbing opsional yang diserahkan ke setiap transport.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Melangkah lebih jauh

Semua di bawah ini opsional. Default sudah mencakup sebagian besar aplikasi.

### Menambahkan skema DSN kustom

Implementasikan `TransportFactoryInterface` milik Symfony dan daftarkan - lalu skema Anda sendiri bekerja persis seperti yang bawaan:

```php
use ryanstubbs\FlightMail\MailPlugin;
use Symfony\Component\Mailer\Transport\Dsn;
use Symfony\Component\Mailer\Transport\TransportFactoryInterface;
use Symfony\Component\Mailer\Transport\TransportInterface;

class MyCarrierFactory implements TransportFactoryInterface
{
    public function supports(Dsn $dsn): bool
    {
        return $dsn->getScheme() === 'mycarrier';
    }

    public function create(Dsn $dsn): TransportInterface
    {
        // ... bangun transport yang berbicara ke carrier Anda
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Menambahkan renderer templat kustom

Apa pun yang mengubah nama templat plus params menjadi string memenuhi syarat:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Templat yang berakhiran .markdown sekarang menggunakannya secara otomatis:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Menjalankan sesuatu tepat sebelum mengirim

Hook menerima pesan yang sudah jadi - setelah rendering, setelah default, tepat sebelum ke kabel:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Event dan logging

Serahkan Symfony event dispatcher dan/atau PSR-3 logger dan setiap transport akan menggunakannya:

```php
$plugin->eventDispatcher($dispatcher); // menerima MessageEvent sebelum setiap kirim
$plugin->logger($logger);              // log tingkat transport
```

## Contekan API

```php
// Setup
MailPlugin::install($config)             // daftarkan pada aplikasi Flight global
MailPlugin::register($app, $config)      // daftarkan pada Engine tertentu
$mailer = Flight::mail();                // instance Mailer bersama

// Membangun pesan
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // metode Symfony Mime standar
$message->text(string)                       // body string biasa
$message->html(string)                       // body string HTML
$message->template($name, $params)           // body HTML dari templat
$message->htmlTemplate($name, $params)       // alias dari template()
$message->textTemplate($name, $params)       // body teks dari templat
$message->inlineCss() / ->withoutInlineCss() // CSS inlining per pesan
$message->textFromHtml($mode)                // bagian teks otomatis: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // email HTML saja
$message->transport($name)                   // rute via DSN bernama
$message->send(): ?SentMessage               // render + kirim

// Pada mailer itu sendiri
$mailer->send($message): ?SentMessage        // alternatif eksplisit untuk $message->send()
$mailer->render($template, $params): string  // render tanpa mengirim
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Karena `Message` extends `Symfony\Component\Mime\Email`, setiap metode Symfony yang sudah Anda kenal - `attach()`, `embed()`, `priority()`, `replyTo()` - langsung bekerja.

## Pemecahan masalah

**"No mail DSNs configured"**
Anda memanggil `Flight::mail()` sebelum mendaftarkan plugin, atau array config tidak menyertakan `dsns`. Error ini disengaja - FlightMail menolak menebak ke mana mail Anda harus pergi alih-alih menjatuhkannya diam-diam.

**"Unknown mail template renderer ..."**
Anda memakai templat yang engine-nya belum terinstal. Perbaiki dengan `composer require twig/twig` atau `composer require latte/latte`, atau daftarkan renderer kustom yang dinamai sesuai ekstensi.

**"Unknown mail transport ..."**
Sebuah `->transport('name')` (atau `default_transport`) tidak cocok dengan kunci mana pun di `dsns`. Periksa ejaan - error-nya mencantumkan nama yang dikonfigurasi.

**Email tidak sampai**
Arahkan `dsns` ke `null://null` untuk memastikan sisa kode Anda bekerja, lalu kembali ke DSN yang sebenarnya. Di DDEV, gunakan `smtp://127.0.0.1:1025` dan periksa pesan di Mailpit pada port 8025.

---

Untuk laporan bug, pull request, dan sumber lengkap, kunjungi [repositori GitHub](https://github.com/ryanstubbs/flightmail).
