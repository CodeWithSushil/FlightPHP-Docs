# Kolekcijas

## Pārskats

`Collection` klase Flight ietvaros ir ērts rīks datu kopu pārvaldīšanai. Tā ļauj piekļūt datiem un tos manipulēt, izmantojot gan masīvu, gan objektu pierakstu, padarot jūsu kodu tīrāku un elastīgāku.

## Izpratne

`Collection` būtībā ir masīva iesaiņotājs, taču ar dažām papildu iespējām. Jūs to varat izmantot kā masīvu, veikt cilpu caur to, saskaitīt tā vienumus un pat piekļūt vienumiem tā, it kā tie būtu objekta īpašības. Tas ir īpaši noderīgi, ja vēlaties nodot strukturētus datus savā lietotnē vai padarīt savu kodu lasāmāku.

Kolekcijas implementē vairākas PHP saskarnes:
- `ArrayAccess` (lai jūs varētu izmantot masīva sintaksi)
- `Iterator` (lai jūs varētu veikt cilpu ar `foreach`)
- `Countable` (lai jūs varētu izmantot `count()`)
- `JsonSerializable` (lai jūs varētu viegli pārveidot uz JSON)

## Pamata Lietošana

### Kolekcijas Izveide

Jūs varat izveidot kolekciju, vienkārši nododot masīvu tās konstruktoram:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Piekļuve Vienumiem

Jūs varat piekļūt vienumiem, izmantojot vai nu masīva, vai objekta pierakstu:

```php
// Masīva pieraksts
echo $collection['name']; // Izvade: FlightPHP

// Objekta pieraksts
echo $collection->version; // Izvade: 3
```

### Vienumu Iestatīšana

Jūs varat iestatīt vienumus, izmantojot arī jebkuru no pierakstiem:

```php
// Masīva pieraksts
$collection['author'] = 'Mike Cao';

// Objekta pieraksts
$collection->license = 'MIT';
```

### Vienumu Pārbaude un Noņemšana

Pārbaudiet, vai vienums pastāv:

```php
if (isset($collection['name'])) {
  // Dariet kaut ko
}

if (isset($collection->version)) {
  // Dariet kaut ko
}
```

Noņemiet vienumu:

```php
unset($collection['author']);
unset($collection->license);
```

### Cilpa Caur Kolekciju

Kolekcijas ir iterējamas, tāpēc jūs tās varat izmantot `foreach` cilpā:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Vienumu Skaitīšana

Jūs varat saskaitīt vienumu skaitu kolekcijā:

```php
echo count($collection); // Izvade: 4
```

### Visu Atslēgu vai Datu Iegūšana

Iegūt visas atslēgas:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Iegūt visus datus kā masīvu:

```php
$data = $collection->getData();
```

### Kolekcijas Notīrīšana

Noņemt visus vienumus:

```php
$collection->clear();
```

### JSON Serializācija

Kolekcijas var viegli pārveidot uz JSON:

```php
echo json_encode($collection);
// Izvade: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Papildu Lietošana

Jūs varat pilnībā aizstāt iekšējo datu masīvu, ja nepieciešams:

```php
$collection->setData(['foo' => 'bar']);
```

Kolekcijas ir īpaši noderīgas, ja vēlaties nodot strukturētus datus starp komponentiem vai nodrošināt vairāk objektorientētu saskarni masīvu datiem.

## Skatīt Arī

- [Pieprasījumi](/learn/requests) - Uzziniet, kā apstrādāt HTTP pieprasījumus un kā kolekcijas var izmantot pieprasījumu datu pārvaldīšanai.
- [SimplePdo](/learn/simple-pdo) - Datu bāzes palīgs, kas atgriež vaicājumu rindas kā kolekcijas.

## Problēmu Novēršana

- Ja mēģināt piekļūt atslēgai, kas neeksistē, jūs saņemsiet `null`, nevis kļūdu.
- Atcerieties, ka kolekcijas nav rekursīvas: ligzdotie masīvi netiek automātiski pārveidoti par kolekcijām.
- Ja nepieciešams atiestatīt kolekciju, izmantojiet `$collection->clear()` vai `$collection->setData([])`.

## Izmaiņu Žurnāls

- v3.0 - Uzlaboti tipu norādījumi un PHP 8+ atbalsts.
- v1.0 - Sākotnējais Collection klases laidiens.