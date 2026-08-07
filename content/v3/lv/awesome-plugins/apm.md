# FlightPHP APM Dokumentācija

Laipni lūdzam FlightPHP APM — jūsu lietotnes personīgais veiktspējas treneris! Šī rokasgrāmata ir jūsu ceļvedis, kā iestatīt, izmantot un apgūt Lietotnes veiktspējas uzraudzību (APM) ar FlightPHP. Neatkarīgi no tā, vai meklējat lēnus pieprasījumus vai vēlaties iedziļināties latentuma diagrammās, mēs esam jums palīdzējuši. Padarīsim jūsu lietotni ātrāku, lietotājus laimīgākus un atkļūdošanas sesijas vieglākas!

Skatīt [demo](https://flightphp-docs-apm.sky-9.com/apm/dashboard) Flight Docs vietnes informācijas paneli.

![FlightPHP APM](/images/apm.png)

## Kāpēc APM ir svarīgs

Iedomājieties: jūsu lietotne ir aizņemta restorāna. Bez veida, kā izsekot, cik ilgi pasūtījumi aizņem vai kur virtuve kavējas, jūs minat, kāpēc klienti aiziet neapmierināti. APM ir jūsu sous-chef — tas vēro katru soli, no ienākošajiem pieprasījumiem līdz datubāzes vaicājumiem, un atzīmē visu, kas jūs palēnina. Lēnas lapas zaudē lietotājus (pētījumi saka, ka 53% atstāj, ja vietne aizņem vairāk nekā 3 sekundes, lai ielādētu!), un APM palīdz jums noķert šīs problēmas *pirms* tās sāp. Tas ir proaktīvs miera prāts — mazāk “kāpēc tas ir salauzts?” brīžu, vairāk “skatieties, cik gludi tas darbojas!” uzvaru.

## Uzstādīšana

Sāciet darbu ar Composer:

```bash
composer require flightphp/apm
```

Jums būs nepieciešams:
- **PHP 7.4+**: Saglabā mūs saderīgus ar LTS Linux distro, vienlaikus atbalstot modernu PHP.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: Vieglais ietvars, ko mēs uzlabojam.

## Atbalstītās datubāzes

FlightPHP APM pašlaik atbalsta šādas datubāzes metrikas glabāšanai:

- **SQLite3**: Vienkārša, failu bāzēta un lieliska vietējai izstrādei vai mazām lietotnēm. Noklusējuma opcija lielākajā daļā iestatījumu.
- **MySQL/MariaDB**: Ideāla lielākiem projektiem vai ražošanas vidēm, kur nepieciešama stabila, mērogojama glabāšana.

Jūs varat izvēlēties savu datubāzes tipu konfigurācijas solī (skatīt zemāk). Pārliecinieties, ka jūsu PHP vidē ir instalēti nepieciešamie paplašinājumi (piem., `pdo_sqlite` vai `pdo_mysql`).

## Darba sākšana

Šeit ir jūsu soli pa solim ceļvedis APM izcilībai:

### 1. Reģistrējiet APM

Ievietojiet šo savā `index.php` vai `services.php` failā, lai sāktu izsekošanu:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// If you're adding a database connection
// Prefer SimplePdo (or PdoQueryCapture from Tracy Extensions in dev).
// Enable APM query tracking via the options array (5th argument).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**Kas šeit notiek?**
- `LoggerFactory::create()` paņem jūsu konfigurāciju (vairāk par to drīz) un iestata žurnālu — SQLite pēc noklusējuma.
- `Apm` ir zvaigzne — tas klausās Flight notikumus (pieprasījumi, maršruti, kļūdas utt.) un apkopo metrikas.
- `bindEventsToFlightInstance($app)` piesaista to visu jūsu Flight lietotnei.

**Pro padoms: Paraugu ņemšana**
Ja jūsu lietotne ir aizņemta, katra pieprasījuma reģistrēšana var pārslodzi sistēmu. Izmantojiet parauga ātrumu (0.0 līdz 1.0):

```php
$Apm = new Apm($ApmLogger, 0.1); // Logs 10% of requests
```

Tas saglabā veiktspēju ātru, vienlaikus nodrošinot jums stabilus datus.

### 2. Konfigurējiet to

Palaidiet šo, lai izveidotu savu `.runway-config.json`:

```bash
php vendor/bin/runway apm:init
```

**Ko tas dara?**
- Palaiž vedni, kas jautā, no kurienes nāk neapstrādātās metrikas (avots) un kur nonāk apstrādātie dati (galamērķis).
- Noklusējums ir SQLite — piemēram, `sqlite:/tmp/apm_metrics.sqlite` avotam, cits galamērķim.
- Jūs beigsiet ar konfigurāciju, piemēram:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Šis process arī jautās, vai vēlaties palaist migrācijas šim iestatījumam. Ja iestatāt to pirmo reizi, atbilde ir jā.

**Kāpēc divas atrašanās vietas?**
Neapstrādātās metrikas ātri uzkrājas (domājiet par nefiltrētiem žurnāliem). Strādnieks tās apstrādā strukturētā galamērķī informācijas panelim. Saglabā lietas sakārtotas!

### 3. Apstrādājiet metrikas ar strādnieku

Strādnieks pārvērš neapstrādātās metrikas informācijas panelim gatavos datos. Palaidiet to vienreiz:

```bash
php vendor/bin/runway apm:worker
```

**Ko tas dara?**
- Lasa no jūsu avota (piem., `apm_metrics.sqlite`).
- Apstrādā līdz 100 metrikām (noklusējuma partijas izmērs) jūsu galamērķī.
- Apstājas, kad pabeigts vai ja nav palikušas metrikas.

**Saglabājiet to darboties**
Dzīvām lietotnēm jums būs nepieciešama nepārtraukta apstrāde. Šeit ir jūsu opcijas:

- **Dēmona režīms**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Darbojas mūžīgi, apstrādājot metrikas, kad tās nāk. Lieliski izstrādei vai maziem iestatījumiem.

- **Crontab**:
  Pievienojiet šo savam crontab (`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  Izpildās katru minūti — ideāli ražošanai.

- **Tmux/Screen**:
  Sāciet atvienojamu sesiju:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, then D to detach; `tmux attach -t apm-worker` to reconnect
  ```
  Saglabā to darboties pat tad, ja izrakstāties.

- **Pielāgotas izmaiņas**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: Apstrādāt 50 metrikas vienlaikus.
  - `--max_messages 1000`: Apstāties pēc 1000 metrikām.
  - `--timeout 300`: Beigt pēc 5 minūtēm.

**Kāpēc apgrūtināties?**
Bez strādnieka jūsu informācijas panelis ir tukšs. Tas ir tilts starp neapstrādātiem žurnāliem un darbības ieskatiem.

### 4. Palaidiet informācijas paneli

Skatiet savas lietotnes vitālos rādītājus:

```bash
php vendor/bin/runway apm:dashboard
```

**Ko tas dara?**
- Palaida PHP serveri `http://localhost:8001/apm/dashboard`.
- Rāda pieprasījumu žurnālus, lēnus maršrutus, kļūdu līmeņus un vairāk.

**Pielāgojiet to**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: Pieejams no jebkura IP (noderīgi attālinātai skatīšanai).
- `--port 8080`: Izmantojiet citu portu, ja 8001 ir aizņemts.
- `--php-path`: Norādiet uz PHP, ja tas nav jūsu PATH.

Atveriet URL savā pārlūkprogrammā un izpētiet!

#### Ražošanas režīms

Ražošanai jums var nākties izmēģināt dažas metodes, lai palaistu informācijas paneli, jo tur droši vien ir ugunsmūri un citi drošības pasākumi. Šeit ir dažas opcijas:

- **Izmantojiet reverso proxy**: Iestatiet Nginx vai Apache, lai pārsūtītu pieprasījumus uz informācijas paneli.
- **SSH tunelis**: Ja varat SSH pieslēgties serverim, izmantojiet `ssh -L 8080:localhost:8001
youruser@yourserver` lai tunelētu informācijas paneli uz savu lokālo mašīnu.
- **VPN**: Ja jūsu serveris ir aiz VPN, pieslēdzieties tam un piekļūstiet informācijas panelim tieši.
- **Konfigurējiet ugunsmūri**: Atveriet portu 8001 savam IP vai servera tīklam. (vai jebkuru portu, ko iestatījāt).
- **Konfigurējiet Apache/Nginx**: Ja jums ir tīmekļa serveris jūsu lietotnes priekšā, varat to konfigurēt uz domēnu vai apakšdomēnu. Ja to darāt, iestatiet dokumentu sakni uz `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### Vēlaties citu informācijas paneli?

Jūs varat izveidot savu informācijas paneli, ja vēlaties! Skatieties vendor/flightphp/apm/src/apm/presenter direktoriju idejām, kā prezentēt datus savam informācijas panelim!

## Informācijas paneļa funkcijas

Informācijas panelis ir jūsu APM HQ — šeit ir tas, ko redzēsiet:

- **Pieprasījumu žurnāls**: Katrs pieprasījums ar laika zīmogu, URL, atbildes kodu un kopējo laiku. Noklikšķiniet uz “Detaļas” middleware, vaicājumiem un kļūdām.
- **Lēnākie pieprasījumi**: Top 5 pieprasījumi, kas aizņem laiku (piem., “/api/heavy” ar 2.5s).
- **Lēnākie maršruti**: Top 5 maršruti pēc vidējā laika — lieliski modeļu noteikšanai.
- **Kļūdu līmenis**: Procentuāli pieprasījumi, kas neizdodas (piem., 2.3% 500s).
- **Latentuma procentiles**: 95. (p95) un 99. (p99) atbildes laiki — ziniet savus sliktākos scenārijus.
- **Atbildes koda diagramma**: Vizualizējiet 200s, 404s, 500s laika gaitā.
- **Garie vaicājumi/middleware**: Top 5 lēnie datubāzes zvani un middleware slāņi.
- **Kešatmiņas trāpījums/neveiksme**: Cik bieži jūsu kešatmiņa glābj dienu.

**Papildu**:
- Filtrēt pēc “Pēdējā stunda,” “Pēdējā diena,” vai “Pēdējā nedēļa.”
- Pārslēgt tumšo režīmu tām vēlu nakts sesijām.

**Piemērs**:
Pieprasījums uz `/users` var parādīt:
- Kopējais laiks: 150ms
- Middleware: `AuthMiddleware->handle` (50ms)
- Vaicājums: `SELECT * FROM users` (80ms)
- Kešatmiņa: Trāpījums `user_list` (5ms)

## Pielāgotu notikumu pievienošana

Izsekot jebko — kā API zvanu vai maksājuma procesu:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**Kur tas parādās?**
Informācijas paneļa pieprasījuma detaļās zem “Pielāgotie notikumi” — izvēršams ar skaistu JSON formatējumu.

**Lietošanas gadījums**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
Tagad redzēsiet, vai šis API velk jūsu lietotni uz leju!

## Datubāzes uzraudzība

Izsekot PDO vaicājumus šādi:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // required to capture queries for the APM
]);
$Apm->addPdoConnection($pdo);
```

**Ko jūs saņemat**:
- Vaicājuma teksts (piem., `SELECT * FROM users WHERE id = ?`)
- Izpildes laiks (piem., 0.015s)
- Rindu skaits (piem., 42)

**Uzmanību**:
- **Neobligāti**: Izlaidiet šo, ja jums nav nepieciešama DB izsekošana.
- **SimplePdo (ieteicams)**: Izmantojiet `SimplePdo` ar `trackApmQueries => true`. Novecojušais `PdoWrapper` joprojām darbojas (5. konstruktora arguments `true`). Neapstrādāts core PDO vēl nav saistīts — sekojiet līdzi!
- **Veiktspējas brīdinājums**: Katra vaicājuma reģistrēšana DB smagā vietnē var palēnināt lietas. Izmantojiet paraugu ņemšanu (`$Apm = new Apm($ApmLogger, 0.1)`) lai samazinātu slodzi.

**Piemēra izvade**:
- Vaicājums: `SELECT name FROM products WHERE price > 100`
- Laiks: 0.023s
- Rindas: 15

## Strādnieka opcijas

Pielāgojiet strādnieku pēc savas patikas:

- `--timeout 300`: Apstājas pēc 5 minūtēm — labs testēšanai.
- `--max_messages 500`: Ierobežo līdz 500 metrikām — saglabā to galīgu.
- `--batch_size 200`: Apstrādā 200 vienlaikus — līdzsvaro ātrumu un atmiņu.
- `--daemon`: Darbojas nepārtraukti — ideāli dzīvoai uzraudzībai.

**Piemērs**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Darbojas stundu, apstrādājot 100 metrikas vienlaikus.

## Pieprasījuma ID lietotnē

Katram pieprasījumam ir unikāls pieprasījuma ID izsekošanai. Jūs varat izmantot šo ID savā lietotnē, lai korelētu žurnālus un metrikas. Piemēram, jūs varat pievienot pieprasījuma ID kļūdas lapai:

```php
Flight::map('error', function($message) {
	// Get the request ID from the response header X-Flight-Request-Id
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Additionally you could fetch it from the Flight variable
	// This method won't work well in swoole or other async platforms.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## Jaunināšana

Ja jaunināt uz jaunāku APM versiju, pastāv iespēja, ka ir nepieciešamas datubāzes migrācijas. Jūs varat to izdarīt, palaižot šādu komandu:

```bash
php vendor/bin/runway apm:migrate
```
Tas palaidīs visas migrācijas, kas nepieciešamas, lai atjauninātu datubāzes shēmu uz jaunāko versiju.

**Piezīme:** Ja jūsu APM datubāze ir liela izmēra, šīs migrācijas var aizņemt kādu laiku. Jūs varat vēlēties palaist šo komandu ārpus maksimālās slodzes stundām.

### Jaunināšana no 0.4.3 -> 0.5.0

Ja jaunināt no 0.4.3 uz 0.5.0, jums būs jāpalaiž šāda komanda:

```bash
php vendor/bin/runway apm:config-migrate
```

Tas migrēs jūsu konfigurāciju no vecā formāta, izmantojot `.runway-config.json` failu, uz jauno formātu, kas glabā atslēgu/vērtību `config.php` failā.

## Veco datu tīrīšana

Lai saglabātu savu datubāzi sakārtotu, jūs varat iztīrīt vecos datus. Tas ir īpaši noderīgi, ja vadāt aizņemtu lietotni un vēlaties saglabāt datubāzes izmēru pārvaldāmu.
Jūs varat to izdarīt, palaižot šādu komandu:

```bash
php vendor/bin/runway apm:purge
```
Tas noņems visus datus, kas vecāki par 30 dienām no datubāzes. Jūs varat pielāgot dienu skaitu, nododot citu vērtību `--days` opcijai:

```bash
php vendor/bin/runway apm:purge --days 7
```
Tas noņems visus datus, kas vecāki par 7 dienām no datubāzes.

## Problēmu novēršana

Iestrēguši? Izmēģiniet šos:

- **Nav informācijas paneļa datu?**
  - Vai strādnieks darbojas? Pārbaudiet `ps aux | grep apm:worker`.
  - Vai konfigurācijas ceļi sakrīt? Pārbaudiet, vai `.runway-config.json` DSN norāda uz reāliem failiem.
  - Palaidiet `php vendor/bin/runway apm:worker` manuāli, lai apstrādātu gaidošās metrikas.

- **Strādnieka kļūdas?**
  - Ieskaitieties savos SQLite failos (piem., `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Pārbaudiet PHP žurnālus stack traces.

- **Informācijas panelis nesāksies?**
  - Vai ports 8001 ir aizņemts? Izmantojiet `--port 8080`.
  - Vai PHP nav atrasts? Izmantojiet `--php-path /usr/bin/php`.
  - Vai ugunsmūris bloķē? Atveriet portu vai izmantojiet `--host localhost`.

- **Pārāk lēni?**
  - Samaziniet parauga ātrumu: `$Apm = new Apm($ApmLogger, 0.05)` (5%).
  - Samaziniet partijas izmēru: `--batch_size 20`.

- **Neizseko izņēmumus/kļūdas?**
  - Ja jums ir [Tracy](https://tracy.nette.org/) ieslēgts jūsu projektam, tas pārrakstīs Flight kļūdu apstrādi. Jums būs jāatspējo Tracy un pēc tam jāpārliecinās, ka `Flight::set('flight.handle_errors', true);` ir iestatīts.

- **Neizseko datubāzes vaicājumus?**
  - Dodiet priekšroku `SimplePdo` ar `['trackApmQueries' => true]` kā 5. konstruktora argumentu (opciju masīvs).
  - Ja joprojām izmantojat novecojušo `PdoWrapper`, nododiet `true` kā 5. argumentu.
  - Izsauciet `$Apm->addPdoConnection($pdo)` pēc savienojuma izveides.