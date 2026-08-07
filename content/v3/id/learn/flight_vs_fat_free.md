# Flight vs Fat-Free

## Apa itu Fat-Free?
[Fat-Free](https://fatfreeframework.com) (dikenal dengan penuh kasih sayang sebagai **F3**) adalah mikro-framework PHP yang kuat namun mudah digunakan, dirancang untuk membantu Anda membangun aplikasi web yang dinamis dan tangguh - dengan cepat!

Flight dibandingkan dengan Fat-Free dalam banyak hal dan mungkin merupakan kerabat terdekat dalam hal fitur dan kesederhanaan. Fat-Free memiliki banyak fitur yang tidak dimiliki Flight, tetapi juga memiliki banyak fitur yang dimiliki Flight. Fat-Free mulai menunjukkan usianya dan tidak sepopuler dulu.

Pembaruan menjadi semakin jarang dan komunitas tidak seaktif dulu. Kodenya cukup sederhana, tetapi terkadang kurangnya disiplin sintaks dapat membuatnya sulit dibaca dan dipahami. Ia memang bekerja untuk PHP 8.3, tetapi kodenya sendiri masih terlihat seperti hidup di PHP 5.3.

## Kelebihan dibandingkan Flight

- Fat-Free memiliki bintang di GitHub sedikit lebih banyak daripada Flight.
- Fat-Free memiliki beberapa dokumentasi yang layak, tetapi masih kurang jelas di beberapa area.
- Fat-Free memiliki beberapa sumber daya yang jarang seperti tutorial YouTube dan artikel online yang dapat digunakan untuk mempelajari framework ini.
- Fat-Free memiliki [beberapa plugin yang membantu](https://fatfreeframework.com/3.8/api-reference) bawaan yang terkadang berguna.
- Fat-Free memiliki ORM bawaan yang disebut Mapper yang dapat digunakan untuk berinteraksi dengan database Anda. Flight memiliki [active-record](/awesome-plugins/active-record).
- Fat-Free memiliki Sessions, Caching, dan lokalisasi bawaan. Flight mengharuskan Anda menggunakan library pihak ketiga, tetapi hal ini tercakup dalam [dokumentasi](/awesome-plugins).
- Fat-Free memiliki sekelompok kecil [plugin buatan komunitas](https://fatfreeframework.com/3.8/development#Community) yang dapat digunakan untuk memperluas framework. Flight memiliki beberapa yang tercakup dalam halaman [dokumentasi](/awesome-plugins) dan [contoh](/examples).
- Fat-Free seperti Flight tidak memiliki dependensi.
- Fat-Free seperti Flight diarahkan untuk memberikan kendali kepada pengembang atas aplikasi mereka dan pengalaman pengembang yang sederhana.
- Fat-Free mempertahankan kompatibilitas mundur seperti yang dilakukan Flight (sebagian karena pembaruan semakin [jarang](https://github.com/bcosca/fatfree/releases)).
- Fat-Free seperti Flight ditujukan untuk pengembang yang baru pertama kali menjelajah ke dunia framework.
- Fat-Free memiliki mesin template bawaan yang lebih kuat daripada mesin template Flight. Flight merekomendasikan [Latte](/awesome-plugins/latte) untuk mencapai hal ini.
- Fat-Free memiliki perintah CLI unik bertipe "route" di mana Anda dapat membangun aplikasi CLI di dalam Fat-Free itu sendiri dan memperlakukannya seperti permintaan `GET`. Flight mencapai hal ini dengan [runway](/awesome-plugins/runway).

## Kekurangan dibandingkan Flight

- Fat-Free memiliki beberapa tes implementasi dan bahkan memiliki kelas [test](https://fatfreeframework.com/3.8/test) sendiri yang sangat mendasar. Namun,
  tidak 100% diuji unit seperti Flight.
- Anda harus menggunakan mesin pencari seperti Google untuk benar-benar mencari situs dokumentasi.
- Flight memiliki mode gelap di situs dokumentasi mereka. (mic drop)
- Fat-Free memiliki beberapa modul yang sangat tidak terawat.
- Flight memiliki [SimplePdo](/learn/simple-pdo) untuk akses database, yang sedikit lebih sederhana daripada kelas `DB\SQL` bawaan Fat-Free (dan lebih disukai daripada PdoWrapper yang sudah tidak digunakan lagi).
- Flight memiliki [plugin permissions](/awesome-plugins/permissions) yang dapat digunakan untuk mengamankan aplikasi Anda. Fat Free mengharuskan Anda menggunakan library pihak ketiga.
- Flight memiliki ORM yang disebut [active-record](/awesome-plugins/active-record) yang terasa lebih seperti ORM daripada Mapper milik Fat-Free.
  Manfaat tambahan dari `active-record` adalah Anda dapat mendefinisikan hubungan antar rekaman untuk penggabungan otomatis, sedangkan Mapper Fat-Free
  mengharuskan Anda membuat [SQL views](https://fatfreeframework.com/3.8/databases#ProsandCons).
- Anehnya, Fat-Free tidak memiliki root namespace. Flight diberi namespace sepenuhnya agar tidak bertabrakan dengan kode Anda sendiri.
  Kelas `Cache` adalah pelanggar terbesar di sini.
- Fat-Free tidak memiliki middleware. Sebagai gantinya, ada hook `beforeroute` dan `afterroute` yang dapat digunakan untuk menyaring permintaan dan respons di kontroler.
- Fat-Free tidak dapat mengelompokkan rute.
- Fat-Free memiliki penangan wadah injeksi dependensi, tetapi dokumentasinya sangat minim tentang cara menggunakannya.
- Proses debugging bisa menjadi sedikit rumit karena pada dasarnya semuanya disimpan dalam apa yang disebut [`HIVE`](https://fatfreeframework.com/3.8/quick-reference).