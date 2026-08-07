# Konfigurācija

## Pārskats

Flight nodrošina vienkāršu veidu, kā konfigurēt dažādus sistēmas aspektus atbilstoši jūsu lietojumprogrammas vajadzībām. Daži iestatījumi ir noteikti pēc noklusējuma, bet tos var pārrakstīt pēc nepieciešamības. Varat arī iestatīt savus mainīgos, ko izmantot visā lietojumprogrammā.

Skaidra, slāņveida konfigurācija (failu noklusējumi + vides noslēpumi) arī palīdz [AI kodēšanas rīkiem](/learn/ai): aģenti mācās vienu vietu literāļiem un vienu vietu noslēpumiem, nevis izdomā `$_ENV` lasījumus kontrolieros.

## Izpratne

Jūs varat pielāgot noteiktu Flight darbību, iestatot konfigurācijas vērtības
ar `set` metodi.

```php
Flight::set('flight.log_errors', true);
```

Strukturētā lietojumprogrammā (ieskaitot [skeleton](https://github.com/flightphp/skeleton)) jūs parasti ielādējat projekta iestatījumus no `app/config/config.php` un pēc tam lietojat atbilstošās atslēgas uz Engine (piemēram, `flight.base_url`, `flight.views.path`). Varat arī ievadīt nelielu konfigurācijas objektu kontrolieros, nevis lasīt globālos mainīgos visur — tas ir draudzīgāk testiem un aģentiem, kas seko `AGENTS.md`.

## Pamata lietošana

### Flight konfigurācijas opcijas

Šeit ir saraksts ar visiem pieejamajiem konfigurācijas iestatījumiem:

- **flight.base_url** `?string` - Pārraksta pieprasījuma bāzes URL, ja Flight darbojas apakšdirektorijā. (noklusējums: null)
- **flight.case_sensitive** `bool` - Reģistrjutīga URL atbilstība. (noklusējums: false)
- **flight.handle_errors** `bool` - Ļauj Flight iekšēji apstrādāt visas kļūdas. (noklusējums: true)
  - Ja vēlaties, lai Flight apstrādā kļūdas, nevis noklusējuma PHP uzvedību, šim jābūt true.
  - Ja jums ir instalēts [Tracy](/awesome-plugins/tracy), vēlaties iestatīt šo uz false, lai Tracy var apstrādāt kļūdas.
  - Ja jums ir instalēts [APM](/awesome-plugins/apm) spraudnis, vēlaties iestatīt šo uz true, lai APM var reģistrēt kļūdas.
- **flight.log_errors** `bool` - Reģistrē kļūdas tīmekļa servera kļūdu žurnālfailā. (noklusējums: false)
  - Ja jums ir instalēts [Tracy](/awesome-plugins/tracy), Tracy reģistrēs kļūdas, pamatojoties uz Tracy konfigurācijām, nevis šo konfigurāciju.
- **flight.debug** `bool` - Izvada detalizētu kļūdu informāciju (izņēmuma ziņojumu, kodu un steka izsekojumu) pārlūkā, kad rodas kļūda. (noklusējums: false)
  - **Nekad neiespējojiet to ražošanā** — tas atklāj iekšējas lietojumprogrammas detaļas. Lietojiet to tikai vietējai izstrādei vai starpposma videi.
  - Ja `false`, tiek rādīta vispārīga `500 Internal Server Error` atbilde. Savienojiet ar `flight.log_errors`, lai uztvertu kļūdas servera pusē.
- **flight.allow_method_override** `bool` - Ļauj pārrakstīt HTTP metodi, izmantojot `X-HTTP-Method-Override` pieprasījuma galveni vai `_method` lauku POST pamattekstā. (noklusējums: true)
  - **Ieteicams iestatīt `false`** lietojumprogrammām, kurām nav nepieciešama HTML veidlapu metodes viltošana, jo tas novērš klientu iespēju viltot `DELETE` vai `PUT` pieprasījumus, izmantojot standarta POST veidlapu.
  - Skatiet [Drošība](/learn/security#flight-configuration-hardening) sīkākai informācijai.
- **flight.views.path** `string` - Direktorija, kurā atrodas skata veidņu faili. (noklusējums: ./views)
- **flight.views.extension** `string` - Skata veidņu faila paplašinājums. (noklusējums: `.php`; oficiālais skeleton iestata `.twig`, ja tiek izmantots Twig)
- **flight.content_length** `bool` - Iestata `Content-Length` galveni. (noklusējums: true)
  - Ja izmantojat [Tracy](/awesome-plugins/tracy), tas ir jāiestata uz false, lai Tracy varētu pareizi renderēt.
- **flight.v2.output_buffering** `bool` - Izmantot mantoto izejas buferizāciju. Skatiet [migrating to v3](migrating-to-v3). (noklusējums: false)

### Ielādētāja konfigurācija

Ir vēl viens konfigurācijas iestatījums ielādētājam. Tas ļauj
automātiski ielādēt klases ar `_` nosaukumā.

```php
// Iespējot klašu ielādi ar apakšsvītrām
// Noklusējums ir true
Loader::$v2ClassLoading = false;
```

Atcerieties, ka [automātiskā ielāde](/learn/autoloading) ir atkarīga arī no **mapju reģistra** atbilstības jūsu nosaukumvietām — īpaši ar skeleton `App\` + `app/Controller/` izkārtojumu.

### Projekta konfigurācija un `.env` (skeleton modelis)

Flight kodolam nav nepieciešami `.env` faili. Daudzas lietojumprogrammas izmanto tikai PHP konfigurācijas masīvu. Oficiālais skeleton noslāņo konfigurāciju, lai noslēpumi paliktu ārpus git, kamēr Runway joprojām var droši pārrakstīt **literālo** konfigurāciju:

1. **`.env` / reālā vide** — noslēpumi un izvietošanas pārrakstīšanas (giģnorēti).
2. **`app/config/config.php`** — literālo PHP masīvu noklusējumi (kopēti no `config_sample.php`). Dodiet priekšroku **bez** `$_ENV[...]` izteiksmēm šajā failā: rīki, piemēram, `runway config:set`, var to pārrakstīt kā statiskas vērtības un iecept noslēpumus failā.
3. **Sapludināšana startēšanas laikā** — vide uzvar kartētajām atslēgām; lietojumprogrammas kods lasa konfigurācijas objektu vai `$app->get()`, nevis `$_ENV` kontrolieros.

Piemērs `config_sample.php` / `config.php` formas (vienkāršots):

```php
<?php
// Tikai literāļi — noslēpumi pieder .env skeleton darbplūsmā
return [
	'app' => [
		'env' => 'development',
		'debug' => true,
		'base_url' => '/',
		'timezone' => 'UTC',
	],
	'database' => [
		'driver' => 'sqlite', // vai mysql, vai '' lai atspējotu
		'host' => 'localhost',
		'dbname' => '',
		'user' => '',
		'password' => '',
		'file_path' => __DIR__ . '/../../database.sqlite',
	],
	// ...
];
```

```bash
# .env.example → .env (skeleton)
APP_ENV=development
APP_DEBUG=true
FLIGHT_BASE_URL=/
DB_DRIVER=sqlite
# DB_PASSWORD=...
```

Šī sadalīšana ir apzināta [AI draudzīgiem projektiem](/learn/ai): instrukcijas var teikt "noklusējumi `config.php`, noslēpumi `.env`, ievadiet Config / Engine — nekad neizgudrojiet env piekļuvi kontrolierī." Esošās lietojumprogrammas var pilnībā ignorēt `.env` un saglabāt vienu konfigurācijas failu.

### Mainīgie

Flight ļauj saglabāt mainīgos, lai tos varētu izmantot jebkurā jūsu lietojumprogrammas vietā.

```php
// Saglabājiet savu mainīgo
Flight::set('id', 123);

// Citur jūsu lietojumprogrammā
$id = Flight::get('id');
```
Lai pārbaudītu, vai mainīgais ir iestatīts, varat:

```php
if (Flight::has('id')) {
  // Izdariet kaut ko
}
```

Varat notīrīt mainīgo, rīkojoties šādi:

```php
// Notīra id mainīgo
Flight::clear('id');

// Notīra visus mainīgos
Flight::clear();
```

> **Piezīme:** Tas, ka varat iestatīt mainīgo, nenozīmē, ka jums tas būtu jādara. Lietojiet šo funkciju taupīgi. Iemesls ir tāds, ka viss, kas šeit glabājas, kļūst par globālo mainīgo. Globālie mainīgie ir slikti, jo tos var mainīt no jebkuras vietas jūsu lietojumprogrammā, apgrūtinot kļūdu izsekošanu. Turklāt tas var sarežģīt tādas lietas kā [vienību testēšana](/guides/unit-testing). Dodiet priekšroku konstruktora ievadīšanai (kā skeleton + Dice iestatījumā) pakalpojumiem un konfigurācijai, kas nepieciešama kontrolieriem.

### Kļūdas un izņēmumi

Visas kļūdas un izņēmumi tiek uztverti ar Flight un nodoti `error` metodei,
ja `flight.handle_errors` ir iestatīts uz true.

Noklusējuma uzvedība ir nosūtīt vispārīgu `HTTP 500 Internal Server Error`
atbildi ar zināmu kļūdas informāciju.

Varat [pārrakstīt](/learn/extending) šo uzvedību savām vajadzībām:

```php
Flight::map('error', function (Throwable $error) {
  // Apstrādāt kļūdu
  echo $error->getTraceAsString();
});
```

Pēc noklusējuma kļūdas netiek reģistrētas tīmekļa serverī. To var iespējot,
mainot konfigurāciju:

```php
Flight::set('flight.log_errors', true);
```

#### 404 Nav atrasts

Ja URL nevar atrast, Flight izsauc `notFound` metodi. Noklusējuma
uzvedība ir nosūtīt `HTTP 404 Not Found` atbildi ar vienkāršu ziņojumu.

Varat [pārrakstīt](/learn/extending) šo uzvedību savām vajadzībām:

```php
Flight::map('notFound', function () {
  // Apstrādāt nav atrasts
});
```

## Skatīt arī
- [Instalācija](/install) - Skeleton konfigurācija, `.env`, un startēšanas izkārtojums.
- [Automātiskā ielāde](/learn/autoloading) - Nosaukumvietas un mapju reģistrs.
- [Flight paplašināšana](/learn/extending) - Kā paplašināt un pielāgot Flight pamata funkcionalitāti.
- [Vienību testēšana](/guides/unit-testing) - Kā rakstīt vienību testus jūsu Flight lietojumprogrammai.
- [AI un izstrādātāja pieredze](/learn/ai) - `AGENTS.md` un konsekventas projekta instrukcijas.
- [Tracy](/awesome-plugins/tracy) - Spraudnis uzlabotai kļūdu apstrādei un atkļūdošanai.
- [Tracy paplašinājumi](/awesome-plugins/tracy_extensions) - Paplašinājumi Tracy integrēšanai ar Flight.
- [APM](/awesome-plugins/apm) - Spraudnis lietojumprogrammas veiktspējas uzraudzībai un kļūdu izsekošanai.
- [Drošība](/learn/security) - Stiprināšanas karodziņi un noslēpumu apstrāde.

## Problēmu novēršana
- Ja jums ir problēmas atrast visas konfigurācijas vērtības, varat izmantot `var_dump(Flight::get());`
- Ja Runway vai izvietošanas rīki pārrakstīja `config.php`, pārliecinieties, ka noslēpumi netika ierakstīti git — turiet tos `.env` vai reālajā vidē, ja izmantojat skeleton modeli.

## Izmaiņu žurnāls
- Dokumentācija — Dokumentēta skeleton stila konfigurācija / `.env` slāņošana un Twig skata paplašinājuma noklusējums jauniem projektiem.
- v3.18.1 - Pievienotas `flight.debug` un `flight.allow_method_override` konfigurācijas opcijas.
- v3.5.0 - Pievienota konfigurācija `flight.v2.output_buffering`, lai atbalstītu mantoto izejas buferizācijas uzvedību.
- v2.0 - Pievienotas pamata konfigurācijas.