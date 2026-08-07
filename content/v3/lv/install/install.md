# Instalēšanas instrukcijas

Pirms varat instalēt Flight, ir daži pamata priekšnosacījumi. Proti, jums būs nepieciešams:

1. [Instalēt PHP savā sistēmā](#installing-php)
2. [Instalēt Composer](https://getcomposer.org), lai iegūtu vislabāko izstrādātāja pieredzi.

## Pamata instalācija

Ja izmantojat [Composer](https://getcomposer.org), varat izpildīt šo komandu:

```bash
composer require flightphp/core
```

Tas jūsu sistēmā ievietos tikai Flight kodola failus. Jums būs jādefinē projekta struktūra, [izkārtojums](/learn/templates), [atkarības](/learn/dependency-injection-container), [konfigurācijas](/learn/configuration), [automātiskā ielāde](/learn/autoloading) utt. Šī metode nodrošina, ka netiek instalētas citas atkarības, izņemot Flight.

Varat arī [lejupielādēt failus](https://github.com/flightphp/core/archive/master.zip) tieši un izvilkt tos savā tīmekļa direktorijā.

Pamata instalācija ir lieliski piemērota mācībām, mikro API un kopēšanas-ielīmēšanas eksperimentiem. Lai iegūtu pilnu lietotnes izkārtojumu, kam cilvēki *un* [AI kodēšanas rīki](/learn/ai) var sekot vienādi, izmantojiet zemāk ieteikto skeletu.

## Ieteicamā instalācija

Ļoti ieteicams sākt ar [flightphp/skeleton](https://github.com/flightphp/skeleton) lietotni jebkuram jaunam projektam. Instalācija ir vienkārša.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# neobligāta parauga DB + ierakstu demonstrācija
php runway migrate
```

Šis solis izveido projekta struktūru, Composer PSR-4 automātisko ielādi, konfigurāciju un rīkus, piemēram, [Tracy](/awesome-plugins/tracy), [Tracy paplašinājumus](/awesome-plugins/tracy-extensions) un [Runway](/awesome-plugins/runway). Tas ietver arī saknes **`AGENTS.md`** failu (un ierobežotas kopijas zem `app/`), lai AI asistenti dalītos ar jums vienā izkārtojumā — skatiet [AI un izstrādātāja pieredze](/learn/ai).

### Ko sniedz skeleton lietotne

```text
project-root/
├── AGENTS.md              # AI / aģentu patiesības avots
├── SECURITY.md            # Drošības gaidības
├── .env.example           # Noslēpumi / izvietošanas pārklājumi (kopēts uz .env)
├── public/index.php       # Tikai tīmekļa ieejas punkts
├── app/
│   ├── config/            # bootstrap, maršruti, pakalpojumi, config_sample.php
│   ├── Controller/        # App\Controller\*  (PascalCase mape!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Runway CLI komandas
│   ├── views/             # Twig veidnes (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # SQL migrācijas (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Nosaukumvietas seko mapes reģistram.** Composer kartē `"App\\": "app/"`, tāpēc:

| Ceļš uz diska | Nosaukumvieta |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Operētājsistēmā Linux `app/controller/` **nav** tas pats, kas `app/Controller/`. Automātiskā ielāde ir reģistrjutīga — atbilst skeleton PascalCase mapēm. Detalizēti: [Automātiskā ielāde](/learn/autoloading).

**Noklusējuma kopums (jauni projekti):** Twig skati, SimplePdo + ActiveRecord, Dice ar `Engine` ievadīšanu (vēlams izvairīties no `Flight::` lietotnes klasēs), neobligāta SQLite pēc `php runway migrate`.

`create-project` parasti kopē `app/config/config_sample.php` → `config.php` un `.env.example` → `.env`, ja tādi ir. Maršruti atrodas `app/config/routes.php`; pakalpojumi un DI atrodas `app/config/services.php`.

> **Dokumentācija ↔ skeleton:** Šie dokumenti māca Flight **API** (bieži ar īsiem `Flight::` piemēriem). Skeleton nosaka **lietotnes struktūru**. Pievienojot kodu zem `app/`, sekojiet skeleton kokam; izmantojiet dokumentus metožu nosaukumiem, opcijām un spraudņiem.

## Konfigurējiet savu tīmekļa serveri

### Iebūvētais PHP izstrādes serveris

Šis ir visvienkāršākais veids, kā sākt darbu. Varat izmantot iebūvēto serveri, lai palaistu savu lietotni, un pat izmantot SQLite kā datubāzi (ja vien sqlite3 ir instalēts jūsu sistēmā), un nekas daudz nav nepieciešams! Vienkārši izpildiet šo komandu pēc PHP instalēšanas:

```bash
php -S localhost:8000
# vai ar skeleton lietotni
composer start
```

Pēc tam atveriet pārlūkprogrammu un dodieties uz `http://localhost:8000`.

Ja vēlaties, lai jūsu projekta dokumentu sakne būtu cita direktorija (piem., jūsu projekts ir `~/myproject`, bet dokumentu sakne ir `~/myproject/public/`), varat izpildīt šo komandu, kad esat `~/myproject` direktorijā:

```bash
php -S localhost:8000 -t public/
# ar skeleton lietotni tas jau ir konfigurēts
composer start
```

Pēc tam atveriet pārlūkprogrammu un dodieties uz `http://localhost:8000`.

### Apache

Pārliecinieties, ka Apache ir instalēts jūsu sistēmā. Ja nav, meklējiet internetā, kā instalēt Apache savā sistēmā.

Apache gadījumā rediģējiet savu `.htaccess` failu ar šādu saturu:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Piezīme**: Ja jums jāizmanto Flight apakšdirektorijā, pievienojiet rindu
> `RewriteBase /subdir/` tūlīt pēc `RewriteEngine On`.

> **Piezīme**: Ja vēlaties aizsargāt visus servera failus, piemēram, db vai env failu.
> Ievietojiet to savā `.htaccess` failā:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Pārliecinieties, ka Nginx ir instalēts jūsu sistēmā. Ja nav, meklējiet internetā, kā instalēt Nginx savā sistēmā.

Nginx gadījumā pievienojiet savai servera deklarācijai šādu konfigurāciju:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Izveidojiet savu `index.php` failu

Ja veicat pamata instalāciju, jums būs nepieciešams kods, lai sāktu darbu.

```php
<?php

// Ja izmantojat Composer, iekļaujiet autoloader.
require 'vendor/autoload.php';
// ja neizmantojat Composer, ielādējiet ietvaru tieši
// require 'flight/Flight.php';

// Pēc tam definējiet maršrutu un piešķiriet funkciju pieprasījuma apstrādei.
Flight::route('/', function () {
  echo 'hello world!';
});

// Visbeidzot, palaidiet ietvaru.
Flight::start();
```

Skeleton lietotnē publiskais ieejas punkts tikai palaiž lietotni. Maršruti tiek reģistrēti `app/config/routes.php` (parasti `[App\Controller\…::class, 'method']`, lai Dice varētu ievadīt atkarības). Pakalpojumi, Twig, SimplePdo un konteiners ir savienoti `app/config/services.php`. Šī struktūra ir apzināta, lai AI rīki un cilvēki katru reizi rediģētu tās pašas vietas.

## PHP instalēšana

Ja jūsu sistēmā jau ir instalēts `php`, izlaidiet šīs instrukcijas un pārejiet uz [lejupielādes sadaļu](#download-the-files)

### **macOS**

#### **PHP instalēšana, izmantojot Homebrew**

1. **Instalējiet Homebrew** (ja vēl nav instalēts):
   - Atveriet termināli un izpildiet:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Instalējiet PHP**:
   - Instalējiet jaunāko versiju:
     ```bash
     brew install php
     ```
   - Lai instalētu konkrētu versiju, piemēram, PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Pārslēgties starp PHP versijām**:
   - Atsaistiet pašreizējo versiju un saistiet vēlamo versiju:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Pārbaudiet instalēto versiju:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **PHP manuāla instalēšana**

1. **Lejupielādējiet PHP**:
   - Apmeklējiet [PHP for Windows](https://windows.php.net/download/) un lejupielādējiet jaunāko vai konkrētu versiju (piem., 7.4, 8.0) kā ne-thread-safe zip failu.

2. **Izvelciet PHP**:
   - Izvelciet lejupielādēto zip failu uz `C:\php`.

3. **Pievienojiet PHP sistēmas PATH**:
   - Dodieties uz **Sistēmas rekvizīti** > **Vides mainīgie**.
   - Sadaļā **Sistēmas mainīgie** atrodiet **Path** un noklikšķiniet **Rediģēt**.
   - Pievienojiet ceļu `C:\php` (vai kur vien izvilkāt PHP).
   - Noklikšķiniet **OK**, lai aizvērtu visus logus.

4. **Konfigurējiet PHP**:
   - Kopējiet `php.ini-development` uz `php.ini`.
   - Rediģējiet `php.ini`, lai konfigurētu PHP pēc vajadzības (piem., iestatot `extension_dir`, iespējojot paplašinājumus).

5. **Pārbaudiet PHP instalāciju**:
   - Atveriet komandu uzvedni un izpildiet:
     ```cmd
     php -v
     ```

#### **Vairāku PHP versiju instalēšana**

1. **Atkārtojiet iepriekš minētās darbības** katrai versijai, ievietojot katru atsevišķā direktorijā (piem., `C:\php7`, `C:\php8`).

2. **Pārslēdzieties starp versijām**, pielāgojot sistēmas PATH mainīgo, lai tas norādītu uz vēlamo versijas direktoriju.

### **Ubuntu (20.04, 22.04 utt.)**

#### **PHP instalēšana, izmantojot apt**

1. **Atjauniniet pakotņu sarakstus**:
   - Atveriet termināli un izpildiet:
     ```bash
     sudo apt update
     ```

2. **Instalējiet PHP**:
   - Instalējiet jaunāko PHP versiju:
     ```bash
     sudo apt install php
     ```
   - Lai instalētu konkrētu versiju, piemēram, PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Instalējiet papildu moduļus** (neobligāti):
   - Piemēram, lai instalētu MySQL atbalstu:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Pārslēgties starp PHP versijām**:
   - Izmantojiet `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Pārbaudiet instalēto versiju**:
   - Izpildiet:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **PHP instalēšana, izmantojot yum/dnf**

1. **Iespējojiet EPEL krātuvi**:
   - Atveriet termināli un izpildiet:
     ```bash
     sudo dnf install epel-release
     ```

2. **Instalējiet Remi krātuvi**:
   - Izpildiet:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Instalējiet PHP**:
   - Lai instalētu noklusējuma versiju:
     ```bash
     sudo dnf install php
     ```
   - Lai instalētu konkrētu versiju, piemēram, PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Pārslēgties starp PHP versijām**:
   - Izmantojiet `dnf` moduļa komandu:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Pārbaudiet instalēto versiju**:
   - Izpildiet:
     ```bash
     php -v
     ```

### **Vispārīgas piezīmes**

- Izstrādes vidēs ir svarīgi konfigurēt PHP iestatījumus atbilstoši jūsu projekta prasībām.
- Pārslēdzot PHP versijas, pārliecinieties, ka visi attiecīgie PHP paplašinājumi ir instalēti konkrētajai versijai, kuru plānojat izmantot.
- Restartējiet savu tīmekļa serveri (Apache, Nginx utt.) pēc PHP versiju maiņas vai konfigurāciju atjaunināšanas, lai izmaiņas stātos spēkā.