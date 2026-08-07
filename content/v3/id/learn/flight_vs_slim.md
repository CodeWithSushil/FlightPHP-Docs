# Flight vs Slim

## Apa itu Slim?
[Slim](https://slimframework.com) adalah kerangka kerja mikro PHP yang membantu Anda dengan cepat menulis aplikasi web dan API yang sederhana namun powerful.

Banyak inspirasi untuk beberapa fitur v3 Flight sebenarnya berasal dari Slim. Pengelompokan rute (grouping routes), dan mengeksekusi middleware dalam urutan tertentu adalah dua fitur yang terinspirasi dari Slim. Slim v3 dirilis dengan fokus pada kesederhanaan, tetapi ada [ulasan beragam](https://github.com/slimphp/Slim/issues/2770) mengenai v4.

## Kelebihan dibandingkan Flight

- Slim memiliki komunitas pengembang yang lebih besar, yang pada gilirannya membuat modul-modul berguna untuk membantu Anda tidak perlu menciptakan ulang roda.
- Slim mengikuti banyak antarmuka dan standar yang umum di komunitas PHP, yang meningkatkan interoperabilitas.
- Slim memiliki dokumentasi dan tutorial yang layak yang dapat digunakan untuk mempelajari kerangka kerja ini (tidak sebanding dengan Laravel atau Symfony).
- Slim memiliki berbagai sumber daya seperti tutorial YouTube dan artikel online yang dapat digunakan untuk mempelajari kerangka kerja ini.
- Slim memungkinkan Anda menggunakan komponen apa pun yang Anda inginkan untuk menangani fitur routing inti karena sesuai dengan PSR-7.

## Kekurangan dibandingkan Flight

- Anehnya, Slim tidak secepat yang Anda kira untuk sebuah kerangka kerja mikro. Lihat [tolok ukur TechEmpower](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3) untuk informasi lebih lanjut.
- Flight diarahkan untuk pengembang yang ingin membangun aplikasi web yang ringan, cepat, dan mudah digunakan.
- Flight tidak memiliki dependensi, sedangkan [Slim memiliki beberapa dependensi](https://github.com/slimphp/Slim/blob/4.x/composer.json) yang harus Anda instal.
- Flight diarahkan untuk kesederhanaan dan kemudahan penggunaan.
- Salah satu fitur inti Flight adalah ia berusaha semaksimal mungkin untuk menjaga kompatibilitas mundur. Slim v3 ke v4 adalah perubahan yang merusak (breaking change).
- Flight ditujukan untuk pengembang yang baru pertama kali menjelajahi dunia kerangka kerja.
- Flight juga dapat menangani aplikasi tingkat enterprise, tetapi tidak memiliki banyak contoh dan tutorial seperti Slim.
  Ini juga akan membutuhkan lebih banyak disiplin dari pihak pengembang untuk menjaga hal-hal tetap terorganisir dan terstruktur dengan baik.
- Flight memberi pengembang lebih banyak kontrol atas aplikasi, sedangkan Slim dapat menyelipkan beberapa sihir di belakang layar.
- Flight memiliki [SimplePdo](/learn/simple-pdo) untuk akses database (lebih disukai daripada PdoWrapper yang sudah usang). Slim mengharuskan Anda menggunakan pustaka pihak ketiga.
- Flight memiliki [plugin izin](/awesome-plugins/permissions) yang dapat digunakan untuk mengamankan aplikasi Anda. Slim mengharuskan Anda menggunakan pustaka pihak ketiga.
- Flight memiliki ORM bernama [active-record](/awesome-plugins/active-record) yang dapat digunakan untuk berinteraksi dengan database Anda. Slim mengharuskan Anda menggunakan pustaka pihak ketiga.
- Flight memiliki aplikasi CLI bernama [runway](/awesome-plugins/runway) yang dapat digunakan untuk menjalankan aplikasi Anda dari baris perintah. Slim tidak memilikinya.