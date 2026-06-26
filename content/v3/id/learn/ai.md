# AI & Developer Experience dengan Flight

## Gambaran Umum

Flight memudahkan Anda untuk memperkuat proyek PHP Anda dengan alat berbasis AI dan alur kerja pengembang modern. Dengan perintah bawaan untuk menghubungkan ke penyedia LLM (Large Language Model) dan menghasilkan instruksi pengkodean AI yang spesifik untuk proyek, Flight membantu Anda dan tim Anda memaksimalkan penggunaan asisten AI seperti GitHub Copilot, Cursor, Windsurf, dan Antigravity (Gemini).

## Pemahaman

Asisten pengkodean AI paling membantu ketika mereka memahami konteks, konvensi, dan tujuan proyek Anda. Helper AI Flight memungkinkan Anda untuk:
- Menghubungkan proyek Anda ke penyedia LLM populer (OpenAI, Grok, Claude, dll.)
- Menghasilkan dan memperbarui instruksi spesifik proyek untuk alat AI, sehingga semua orang mendapatkan bantuan yang konsisten dan relevan
- Menjaga tim Anda tetap selaras dan produktif, dengan waktu yang lebih sedikit untuk menjelaskan konteks

Fitur-fitur ini sudah terintegrasi ke dalam Flight core CLI dan proyek starter resmi [flightphp/skeleton](https://github.com/flightphp/skeleton).

## Penggunaan Dasar

### Menyiapkan Kredensial LLM

Perintah `ai:init` memandu Anda untuk menghubungkan proyek Anda ke penyedia LLM.

```bash
php runway ai:init
```

Anda akan diminta untuk:
- Memilih penyedia Anda (OpenAI, Grok, Claude, dll.)
- Memasukkan kunci API Anda
- Mengatur URL dasar dan nama model

Ini membuat kredensial yang diperlukan agar Anda dapat membuat permintaan LLM di masa mendatang.

**Contoh:**
```
Selamat datang di AI Init!
LLM API mana yang ingin Anda gunakan? [1] openai, [2] grok, [3] claude: 1
Masukkan URL dasar untuk LLM API [https://api.openai.com]:
Masukkan kunci API Anda untuk openai: sk-...
Masukkan nama model yang ingin Anda gunakan (misalnya gpt-4, claude-3-opus, dll) [gpt-4o]:
Kredensial disimpan ke .runway-creds.json
```

### Menghasilkan Instruksi AI Spesifik Proyek

Perintah `ai:generate-instructions` membantu Anda membuat atau memperbarui instruksi untuk asisten pengkodean AI, yang disesuaikan dengan proyek Anda.

```bash
php runway ai:generate-instructions
```

Anda akan menjawab beberapa pertanyaan tentang proyek Anda (deskripsi, basis data, templating, keamanan, ukuran tim, dll.). Flight menggunakan penyedia LLM Anda untuk menghasilkan instruksi, kemudian menulis konten yang sama ke:
- `.github/copilot-instructions.md` (untuk GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (untuk Cursor)
- `.windsurfrules` (untuk Windsurf)
- `.gemini/GEMINI.md` (untuk Antigravity)
- `AGENTS.md` (di root proyek, untuk asisten AI yang tidak bergantung pada alat tertentu)

**Contoh:**
```
Silakan jelaskan untuk apa proyek Anda? My awesome API
Basis data apa yang Anda rencanakan untuk digunakan? MySQL
Mesin templating HTML apa yang akan Anda gunakan (jika ada)? latte
Apakah keamanan merupakan elemen penting dari proyek ini? (y/n) y
...
Instruksi AI berhasil diperbarui.
```

Sekarang, alat AI Anda akan memberikan saran yang lebih cerdas dan lebih relevan berdasarkan kebutuhan nyata proyek Anda.

## Penggunaan Lanjutan

- Anda dapat menyesuaikan lokasi file kredensial atau instruksi Anda menggunakan opsi perintah (lihat `--help` untuk setiap perintah).
- Helper AI dirancang untuk bekerja dengan penyedia LLM apa pun yang mendukung API yang kompatibel dengan OpenAI.
- Jika Anda ingin memperbarui instruksi Anda seiring perkembangan proyek, cukup jalankan kembali `ai:generate-instructions` dan jawab pertanyaan lagi.

## Lihat Juga

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Starter resmi dengan integrasi AI
- [Runway CLI](/awesome-plugins/runway) – Lebih lanjut tentang alat CLI yang mendukung perintah-perintah ini

## Pemecahan Masalah

- Jika Anda melihat "Missing .runway-creds.json", jalankan `php runway ai:init` terlebih dahulu.
- Pastikan kunci API Anda valid dan memiliki akses ke model yang dipilih.
- Jika instruksi tidak diperbarui, periksa izin file di direktori proyek Anda.

## Changelog

- v3.18.4 – `ai:generate-instructions` sekarang juga menulis instruksi proyek ke `AGENTS.md` di root proyek.
- v3.16.0 – Menambahkan perintah CLI `ai:init` dan `ai:generate-instructions` untuk integrasi AI.