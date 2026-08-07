# AI & Pengalaman Pengembang dengan Flight

## Ringkasan

Flight dirancang untuk bekerja *dengan* alat coding AI—bukan melawannya. API yang kecil dan dapat diprediksi, tata letak aplikasi yang jelas di [skeleton resmi](https://github.com/flightphp/skeleton), dan file instruksi khusus proyek berarti asisten seperti GitHub Copilot, Cursor, Windsurf, Claude Code, dan Gemini dapat mengikuti pola yang sama seperti yang Anda tulis secara manual.

Dengan perintah Runway bawaan untuk menghubungkan ke penyedia LLM dan menghasilkan instruksi proyek, Flight membantu Anda dan tim mendapatkan bantuan yang konsisten dan relevan tanpa harus menempelkan konteks yang sama di setiap obrolan.

## Memahami

Asisten coding AI paling membantu ketika mereka memahami konteks, konvensi, dan tujuan proyek Anda. Bantuan AI Flight memungkinkan Anda untuk:

- Menghubungkan proyek Anda ke penyedia LLM populer (OpenAI, Grok, Claude, dll.)
- Menghasilkan dan memperbarui instruksi khusus proyek sehingga semua orang mendapatkan panduan yang sama
- Menjaga kode tulisan tangan dan kode yang dihasilkan AI berada dalam satu tata letak (terutama dengan skeleton)

Fitur-fitur ini tersedia di CLI inti Flight (melalui [Runway](/awesome-plugins/runway)) dan sudah terhubung sebelumnya di starter resmi [flightphp/skeleton](https://github.com/flightphp/skeleton).

### Apa yang disertakan skeleton untuk AI

Starter resmi memperlakukan **`AGENTS.md` sebagai sumber kebenaran** untuk alat AI:

| File | Peran |
|------|------|
| **`AGENTS.md`** (akar proyek) | Aturan global, alur boot, namespace, DI, "apa yang tidak boleh dilakukan" |
| **`AGENTS.md` khusus** di bawah `app/`, `migrations/`, `tests/`, dll. | Tip ringan khusus folder saat Anda bekerja di direktori tersebut |
| **`SECURITY.md`** | Rahasia, header, XSS/SQL, pelaporan—keamanan tetap disengaja dan terpisah |

Tidak ada **file gaya rumah** terpisah untuk Copilot / Cursor / Gemini / Windsurf di skeleton. Arahkan asisten Anda ke `AGENTS.md` di akar proyek (dan biarkan ia mengikuti tautan ke file khusus). Manusia dapat mengabaikan file-file ini sepenuhnya dan menggunakan [README](https://github.com/flightphp/skeleton); tata letaknya tetap sama apa pun caranya.

> **Dokumen mengajarkan API; skeleton mengajarkan tata letak.** Contoh singkat `Flight::` di dokumentasi ini bagus untuk belajar. Di aplikasi skeleton, lebih suka kelas `App\…`, injeksi konstruktor, dan `$this->app` daripada fasad statis di dalam kontroler. Lihat [Instalasi](/install) dan [Autoloading](/learn/autoloading).

## Penggunaan Dasar

### Menyiapkan Kredensial LLM

Perintah `ai:init` memandu Anda menghubungkan proyek ke penyedia LLM.

```bash
php runway ai:init
```

Anda akan diminta untuk:

- Memilih penyedia Anda (OpenAI, Grok, Claude, dll.)
- Memasukkan kunci API Anda
- Menetapkan URL dasar dan nama model

Ini membuat kredensial yang digunakan untuk permintaan LLM selanjutnya (misalnya menghasilkan instruksi).

**Contoh:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Menghasilkan Instruksi AI Khusus Proyek

Perintah `ai:generate-instructions` membuat atau memperbarui instruksi untuk asisten coding AI, disesuaikan dengan *proyek* Anda.

```bash
php runway ai:generate-instructions
```

Anda akan menjawab beberapa pertanyaan (deskripsi, basis data, templating, keamanan, ukuran tim, dll.). Flight menggunakan penyedia LLM Anda untuk menghasilkan instruksi dan menuliskannya terutama ke:

- **`AGENTS.md`** di akar proyek (tidak bergantung pada alat; yang diharapkan oleh skeleton resmi dan sebagian besar agen modern)

Tergantung pada versi CLI dan opsi, perintah tersebut juga dapat menulis salinan khusus alat untuk alur kerja lama (misalnya file aturan Copilot, Cursor, Windsurf, atau Gemini). Untuk **proyek baru dari skeleton**, perlakukan **`AGENTS.md`** (plus file `AGENTS.md` khusus yang Anda simpan di bawah `app/`) sebagai satu-satunya sumber kebenaran—jangan memelihara lima file instruksi yang berbeda-beda secara manual.

**Contoh:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Sekarang alat AI dapat menyarankan kode yang sesuai dengan tumpukan dan tata letak nyata Anda—bukan tutorial PHP generik.

## Penggunaan Lanjutan

- Sesuaikan kredensial atau jalur keluaran dengan opsi perintah (lihat `--help` pada setiap perintah).
- Alat bantu ini bekerja dengan penyedia LLM apa pun yang mendukung API yang kompatibel dengan OpenAI.
- Jalankan ulang `ai:generate-instructions` seiring berkembangnya proyek agar agen tetap sinkron.
- Di skeleton, simpan kebijakan keamanan di **`SECURITY.md`** dan tata letak kode di **`AGENTS.md`** sehingga kedua dokumen tidak menjadi tempat campur aduk.
- Utamakan [docs.flightphp.com](https://docs.flightphp.com) dan server MCP Flight saat agen membutuhkan detail API; verifikasi metode yang dibuat-buat terhadap `vendor/flightphp/core`.

## Lihat Juga

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Starter resmi dengan `AGENTS.md`, Twig, SimplePdo, dan Dice yang terhubung untuk struktur yang ramah AI
- [Instalasi](/install) – Tata letak `create-project` yang direkomendasikan
- [Autoloading](/learn/autoloading) – **Huruf besar/kecil** folder cocok dengan namespace (`App\Controller` ↔ `app/Controller/`)
- [CLI Runway](/awesome-plugins/runway) – CLI yang mendukung perintah `ai:*` dan pembuatan kerangka
- [Keamanan](/learn/security) – Default yang aman yang seharusnya tidak diperlemah oleh agen (dan manusia)

## Pemecahan Masalah

- Jika Anda melihat "Missing .runway-creds.json", jalankan `php runway ai:init` terlebih dahulu.
- Pastikan kunci API Anda valid dan memiliki akses ke model yang dipilih.
- Jika instruksi tidak diperbarui, periksa izin file di direktori proyek Anda.
- Jika agen mengarang API Flight atau tata letak folder yang salah, arahkan ke **`AGENTS.md`** di akar proyek dan situs dokumentasi ini; tata letak skeleton berlaku untuk kode di bawah `app/`.

## Catatan Perubahan

- v3.18.4 – `ai:generate-instructions` menulis instruksi proyek ke `AGENTS.md` di akar proyek.
- v3.16.0 – Menambahkan perintah CLI `ai:init` dan `ai:generate-instructions` untuk integrasi AI.