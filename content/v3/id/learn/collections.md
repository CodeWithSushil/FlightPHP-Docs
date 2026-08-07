# Collections

## Ikhtisar

Kelas `Collection` di Flight adalah utilitas yang berguna untuk mengelola kumpulan data. Ini memungkinkan Anda mengakses dan memanipulasi data menggunakan notasi array maupun objek, sehingga kode Anda lebih bersih dan lebih fleksibel.

## Pemahaman

Sebuah `Collection` pada dasarnya adalah pembungkus di sekitar array, tetapi dengan kekuatan ekstra. Anda dapat menggunakannya seperti array, melakukan perulangan di atasnya, menghitung itemnya, dan bahkan mengakses item seolah-olah itu properti objek. Ini sangat berguna ketika Anda ingin meneruskan data terstruktur dalam aplikasi Anda, atau ketika Anda ingin membuat kode Anda sedikit lebih mudah dibaca.

Collection mengimplementasikan beberapa antarmuka PHP:
- `ArrayAccess` (sehingga Anda dapat menggunakan sintaks array)
- `Iterator` (sehingga Anda dapat melakukan perulangan dengan `foreach`)
- `Countable` (sehingga Anda dapat menggunakan `count()`)
- `JsonSerializable` (sehingga Anda dapat dengan mudah mengonversinya ke JSON)

## Penggunaan Dasar

### Membuat Collection

Anda dapat membuat sebuah Collection dengan cukup melewatkan array ke konstruktornya:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Mengakses Item

Anda dapat mengakses item menggunakan notasi array atau objek:

```php
// Notasi array
echo $collection['name']; // Keluaran: FlightPHP

// Notasi objek
echo $collection->version; // Keluaran: 3
```

Jika Anda mencoba mengakses kunci yang tidak ada, Anda akan mendapatkan `null` alih-alih error.

### Mengatur Item

Anda juga dapat mengatur item menggunakan salah satu notasi:

```php
// Notasi array
$collection['author'] = 'Mike Cao';

// Notasi objek
$collection->license = 'MIT';
```

### Memeriksa dan Menghapus Item

Periksa apakah sebuah item ada:

```php
if (isset($collection['name'])) {
  // Lakukan sesuatu
}

if (isset($collection->version)) {
  // Lakukan sesuatu
}
```

Hapus sebuah item:

```php
unset($collection['author']);
unset($collection->license);
```

### Iterasi pada Collection

Collection bersifat iterable, sehingga Anda dapat menggunakannya dalam perulangan `foreach`:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Menghitung Item

Anda dapat menghitung jumlah item dalam sebuah Collection:

```php
echo count($collection); // Keluaran: 4
```

### Mendapatkan Semua Kunci atau Data

Dapatkan semua kunci:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Dapatkan semua data sebagai array:

```php
$data = $collection->getData();
```

### Mengosongkan Collection

Hapus semua item:

```php
$collection->clear();
```

### Serialisasi JSON

Collection dapat dengan mudah dikonversi menjadi JSON:

```php
echo json_encode($collection);
// Keluaran: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Penggunaan Lanjutan

Anda dapat mengganti seluruh array data internal jika diperlukan:

```php
$collection->setData(['foo' => 'bar']);
```

Collection sangat berguna ketika Anda ingin meneruskan data terstruktur antar komponen, atau ketika Anda ingin menyediakan antarmuka yang lebih berorientasi objek untuk data array.

## Lihat Juga

- [Requests](/learn/requests) - Pelajari cara menangani permintaan HTTP dan bagaimana Collection dapat digunakan untuk mengelola data permintaan.
- [SimplePdo](/learn/simple-pdo) - Helper database yang mengembalikan baris kueri sebagai Collection.

## Pemecahan Masalah

- Jika Anda mencoba mengakses kunci yang tidak ada, Anda akan mendapatkan `null` alih-alih error.
- Ingat bahwa Collection tidak bersifat rekursif: array bersarang tidak secara otomatis dikonversi menjadi Collection.
- Jika Anda perlu mengatur ulang Collection, gunakan `$collection->clear()` atau `$collection->setData([])`.

## Log Perubahan

- v3.0 - Peningkatan type hints dan dukungan PHP 8+.
- v1.0 - Rilis awal dari kelas Collection.