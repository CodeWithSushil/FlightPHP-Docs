# Flight PHP Framework

Flight ir ātrs, vienkāršs, paplašināms PHP ietvars — izveidots izstrādātājiem, kuri vēlas ātri paveikt darbu bez liekām grūtībām. Neatkarīgi no tā, vai veidojat klasisku tīmekļa lietotni, zibenīgi ātru API vai strādājat kopā ar mākslīgā intelekta kodēšanas palīgiem, Flight nelielais apjoms un vienkāršais dizains padara to par ideālu izvēli. Flight ir veidots kā viegls, bet var arī apmierināt uzņēmuma līmeņa arhitektūras prasības.

## Kāpēc Izvēlēties Flight?

- **Draudzīgs Iesācējiem:** Flight ir lieliska sākuma vieta jauniem PHP izstrādātājiem. Tā skaidrā struktūra un vienkāršā sintakse palīdz apgūt tīmekļa izstrādi, nenokļūstot zaudējumā standarta kodā.
- **Iemīļots Profesionāļu Vidū:** Pieredzējuši izstrādātāji Flight mīl tā elastības un kontroles dēļ. Jūs varat mērogot no neliela prototipa līdz pilnvērtīgai lietotnei, nemainot ietvarus.
- **Atpakaļsavietojams:** Mēs novērtējam jūsu laiku. Flight v3 ir v2 papildinājums, saglabājot gandrīz visu to pašu API. Mēs ticam evolūcijai, nevis revolūcijai — vairs nav "pasaules salaušanas" katru reizi, kad iznāk jauna galvenā versija.
- **Nulles Atkarības:** Flight kodols ir pilnībā bez atkarībām — bez polifilliem, bez ārējām paketēm, pat bez PSR saskarnēm. Tas nozīmē mazāk uzbrukuma vektoru, mazāku pēdu un negaidītas pārtraucošas izmaiņas no augšupējām atkarībām. Izvēles spraudņi var ietvert atkarības, bet kodols vienmēr paliks viegls un drošs.
- **AI Draudzīgs:** Flight mazā API virsma un [oficiālais skelets](https://github.com/flightphp/skeleton) (viens izkārtojums, `AGENTS.md`, konstruktora injekcija) atvieglo AI kodēšanas rīkiem sekot modelim. Tā pati koda bāze neatkarīgi no tā, vai rakstāt katru rindu vai strādājat ar aģentu. [Uzziniet vairāk par AI izmantošanu ar Flight](/learn/ai).

## Video Pārskats

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">Vienkārši, vai ne?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">Uzziniet vairāk</a> par Flight dokumentācijā!
    </div>
  </div>
</div>

## Ātrais Starts

Lai veiktu ātru minimālu instalāciju, instalējiet to ar Composer:

```bash
composer require flightphp/core
```

Vai arī varat lejupielādēt repozitorija zip failu [šeit](https://github.com/flightphp/core). Tad jums būs pamata `index.php` fails kā tālāk:

```php
<?php

// if installed with composer
require 'vendor/autoload.php';
// or if installed manually by zip file
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

Tas ir viss! Jums ir pamata Flight lietotne. Tagad varat palaist šo failu ar `php -S localhost:8000` un apmeklēt `http://localhost:8000` savā pārlūkprogrammā, lai redzētu izvadi.

Īsi `Flight::` piemēri kā šis ir lieliski mācībām un mikro lietotnēm. Pilnīgam projekta izkārtojumam, ko dalās cilvēki un AI rīki, izmantojiet tālāk minēto skeletu.

## Skelets/Boilerplate Lietotne

Ir oficiāls starteris, kas palīdz sākt jebkuru jaunu Flight projektu. Tas izveido struktūru, konfigurāciju, Composer skriptus un AI draudzīgas instrukcijas no paša sākuma.

Apskatiet [flightphp/skeleton](https://github.com/flightphp/skeleton), lai iegūtu gatavu projektu, vai apmeklējiet [piemērus](examples) lapu, lai gūtu iedvesmu. Vēlaties AI darba plūsmas detaļas? [Izpētiet AI un izstrādātāja pieredzi](/learn/ai).

Ko jūs iegūstat (augsta līmeņa):

- **`App\` vārdtelpas** ar PascalCase mapēm (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—mapes **reģistram** jāatbilst vārdtelpai (skatiet [Autoloading](/learn/autoloading))
- **Dice + `Engine` injekcija** lai kontrolieri paliktu testējami (dodiet priekšroku `$this->app` pār `Flight::` lietotnes kodā)
- **Twig** skati, **SimplePdo** + ActiveRecord paraugs, Runway **migrate**
- Saknes **`AGENTS.md`** (plus scoped kopijas) un **`SECURITY.md`** palīgiem un drošības politikai

## Skelet Lietotnes Instalēšana

Pietiekami vienkārši!

```bash
# Create the new project
composer create-project flightphp/skeleton my-project/
# Enter your new project directory
cd my-project/
# Bring up the local dev-server to get started right away!
composer start
```

Tas izveido projekta struktūru, kopē `config_sample.php` → `config.php` (un `.env.example` → `.env`, ja tāds ir), un jūs esat gatavs sākt. Izvēles parauga dati:

```bash
php runway migrate
# then visit /posts and /api/posts
```

## Augsta Veiktspēja

Flight ir viens no ātrākajiem PHP ietvariem. Tā vieglais kodols nozīmē mazāku pieskaitāmību un lielāku ātrumu — ideāli gan tradicionālām lietotnēm, gan modernām AI atbalstītām darba plūsmām. Visus etalonus varat redzēt [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Skatiet etalonu zemāk ar dažiem citiem populāriem PHP ietvariem.

| Framework | Plaintext Reqs/sec | JSON Reqs/sec |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight un AI

Vai esat ziņkārīgs, kā Flight sadarbojas ar kodēšanas LLM? [Atklājiet](/learn/ai), kā `AGENTS.md`, Runway `ai:*` komandas un skeleta izkārtojums uztur palīgus uz pareizā ceļa.

## Stabilitāte un Atpakaļsavietojamība

Mēs novērtējam jūsu laiku. Mēs visi esam redzējuši ietvarus, kas pilnībā pārvērtē sevi ik pēc pāris gadiem, atstājot izstrādātājus ar salauztu kodu un dārgām migrācijām. Flight ir citāds. Flight v3 tika izstrādāts kā v2 papildinājums, kas nozīmē, ka API, ko jūs zināt un mīlat, nav noņemts. Patiesībā lielākā daļa v2 projektu darbosies bez jebkādām izmaiņām v3.

Mēs esam apņēmušies uzturēt Flight stabilu, lai jūs varētu koncentrēties uz savas lietotnes izveidi, nevis sava ietvara labošanu. Skelets var būt uzskatu balstīts *jauniem* projektiem; kodola API paliek pazīstami visiem citiem.

# Kopiena

Mēs esam Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

Un Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Ieguldījums

Ir divi veidi, kā varat ieguldīt Flight:

1. Ieguldīt kodola ietvarā, apmeklējot [kodola repozitoriju](https://github.com/flightphp/core).
2. Palīdzēt padarīt dokumentāciju labāku! Šī dokumentācijas vietne tiek mitināta [Github](https://github.com/flightphp/docs). Ja pamanāt kļūdu vai vēlaties kaut ko uzlabot, droši iesniedziet pull request. Mēs mīlam atjauninājumus un jaunas idejas — īpaši par AI un jaunām tehnoloģijām!

# Prasības

Flight prasa PHP 7.4 vai jaunāku versiju.

**Piezīme:** PHP 7.4 tiek atbalstīts, jo pašreizējā rakstīšanas laikā (2024) PHP 7.4 ir noklusējuma versija dažām LTS Linux distribūcijām. Piespiežot pāriet uz PHP >8, radītu daudz problēmu šiem lietotājiem. Ietvars arī atbalsta PHP >8.

# Licence

Flight tiek izlaists saskaņā ar [MIT](https://github.com/flightphp/core/blob/master/LICENSE) licenci.