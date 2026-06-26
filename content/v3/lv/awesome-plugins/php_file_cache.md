# flightphp/cache

Viegla, vienkārša un patstāvīga PHP in-file kešatmiņas klase, kas atdalīta no [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Priekšrocības** 
- Viegla, patstāvīga un vienkārša
- Viss kods vienā failā - nav bezjēdzīgu draiveru.
- Droša - katram ģenerētajam kešatmiņas failam ir php galvene ar die, padarot tiešu piekļuvi neiespējamu pat tad, ja kāds zina ceļu un jūsu serveris nav pareizi konfigurēts
- Labi dokumentēta un testēta
- Pareizi apstrādā vienlaicīgumu, izmantojot flock
- Atbalsta PHP 7.4+
- Bezmaksas saskaņā ar MIT licenci

Šī dokumentācijas vietne izmanto šo bibliotēku, lai kešotu katru lapu!

Noklikšķiniet [šeit](https://github.com/flightphp/cache), lai skatītu kodu.

## Instalācija

Instalējiet, izmantojot composer:

```bash
composer require flightphp/cache
```

## Lietošana

Lietošana ir diezgan vienkārša. Tas saglabā kešatmiņas failu kešatmiņas direktorijā.

```php
use flight\Cache;

$app = Flight::app();

// Jūs nododat direktoriju, kurā tiks saglabāta kešatmiņa, konstruktorā
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Tas nodrošina, ka kešatmiņa tiek izmantota tikai ražošanas režīmā
	// ENVIRONMENT ir konstante, kas ir iestatīta jūsu bootstrap failā vai citur jūsu lietotnē
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Iegūt kešatmiņas vērtību

Jūs izmantojat `get()` metodi, lai iegūtu kešotu vērtību. Ja vēlaties ērtības metodi, kas atsvaidzinās kešatmiņu, ja tā ir beidzies, varat izmantot `refreshIfExpired()`.

```php

// Iegūt kešatmiņas instanci
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // atgriezt datus, kas tiks kešoti
}, 10); // 10 sekundes

// vai
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 sekundes
}
```

### Saglabāt kešatmiņas vērtību

Jūs izmantojat `set()` metodi, lai saglabātu vērtību kešatmiņā.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 sekundes
```

### Dzēst kešatmiņas vērtību

Jūs izmantojat `delete()` metodi, lai dzēstu vērtību kešatmiņā.

```php
Flight::cache()->delete('simple-cache-test');
```

### Pārbaudīt, vai kešatmiņas vērtība pastāv

Jūs izmantojat `exists()` metodi, lai pārbaudītu, vai vērtība pastāv kešatmiņā.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// darīt kaut ko
}
```

### Notīrīt kešatmiņu
Jūs izmantojat `flush()` metodi, lai notīrītu visu kešatmiņu.

```php
Flight::cache()->flush();
```

### Izvilkt meta datus ar kešatmiņu

Ja vēlaties izvilkt laika zīmogus un citus meta datus par kešatmiņas ierakstu, pārliecinieties, ka nododat `true` kā pareizo parametru.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // atgriezt datus, kas tiks kešoti
}, 10, true); // true = atgriezt ar metadatiem
// vai
$data = $cache->get("simple-cache-meta-test", true); // true = atgriezt ar metadatiem

/*
Kešotā vienuma piemērs, kas iegūts ar metadatiem:
{
    "time":1511667506, <-- saglabāt unix laika zīmogu
    "expire":10,       <-- izbeigšanās laiks sekundēs
    "data":"04:38:26", <-- atserializēti dati
    "permanent":false
}

Izmantojot metadatus, mēs varam, piemēram, aprēķināt, kad vienums tika saglabāts vai kad tas beidzas
Mēs varam arī piekļūt pašiem datiem ar atslēgu "data"
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // iegūt unix laika zīmogu, kad dati beidzas, un atņemt no tā pašreizējo laika zīmogu
$cacheddate = $data["data"]; // mēs piekļūstam pašiem datiem ar atslēgu "data"

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## Avota kods

Apmeklējiet [https://github.com/flightphp/cache](https://github.com/flightphp/cache), lai skatītu kodu.