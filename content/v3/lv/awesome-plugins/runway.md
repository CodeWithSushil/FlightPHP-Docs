# Runway

Runway ir CLI lietotne, kas palīdz pārvaldīt jūsu Flight lietotnes. Tā var ģenerēt kontrolierus, parādīt visas maršrutus, palaist AI iestatīšanas palīgus, migrācijas (skeletonā) un vēl vairāk. Tā ir balstīta uz izcilo [adhocore/php-cli](https://github.com/adhocore/php-cli) bibliotēku.

Klikšķiniet [šeit](https://github.com/flightphp/runway), lai skatītu kodu.

Scaffolding komandas ir apzināti saskaņotas ar [oficiālo skeleton](https://github.com/flightphp/skeleton), lai [AI kodēšanas rīki](/learn/ai) un cilvēki katru reizi iegūtu vienādus ceļus, vārdu telpas un konstruktora injekcijas stilu.

## Instalācija

Instalējiet ar composer.

```bash
composer require flightphp/runway
```

Skeleton jau atkarīgs no Runway; izmantojiet `php runway` no projekta saknes.

## Pamata konfigurācija

Pirmo reizi palaižot Runway, tas mēģinās atrast `runway` konfigurāciju `app/config/config.php` caur `'runway'` atslēgu.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// neobligāti; skeleton arī izmanto index_root publiskajai ieejai
		'index_root' => 'public/index.php',
    ],
];
```

> **PIEZĪME** - Sākot ar **v1.2.0**, `.runway-config.json` ir novecojis par labu `app/config/config.php`. Migrējiet ar `php runway config:migrate`, kad atjaunināt vecākus projektus. Skeleton joprojām var izveidot nelielu `.runway-config.json` create-project laikā saderībai; turpmāk dodiet priekšroku `runway` atslēgai `config.php`.

### Projekta saknes noteikšana

Runway ir pietiekami gudrs, lai noteiktu jūsu projekta sakni, pat ja to palaižat no apakšdirektorijas. Tas meklē indikatorus, piemēram, `composer.json`, `.git` vai `app/config/config.php`, lai noteiktu, kur atrodas projekta sakne. Tas nozīmē, ka Runway komandas varat palaist no jebkuras vietas savā projektā!

## Lietošana

Runway ir vairākas komandas, kuras varat izmantot Flight lietotnes pārvaldīšanai. Ir divi vienkārši veidi, kā izmantot Runway.

1. Ja izmantojat skeleton projektu, varat palaist `php runway [komanda]` no sava projekta saknes.
1. Ja izmantojat Runway kā composer instalētu pakotni, varat palaist `vendor/bin/runway [komanda]` no sava projekta saknes.

### Komandu saraksts

Visu pieejamo komandu sarakstu varat apskatīt, palaižot `php runway` komandu.

```bash
php runway
```

Paļaujieties tikai uz komandām, kas faktiski parādās šajā sarakstā jūsu instalācijā (pamata Runway komandas pret projektu specifiskām, piemēram, skeleton `migrate`).

### Komandas palīdzība

Jebkurai komandai varat nodot `--help` karodziņu, lai iegūtu vairāk informācijas par to, kā izmantot komandu.

```bash
php runway routes --help
php runway make:controller --help
```

Šeit ir daži piemēri:

### Kontroliera ģenerēšana

`make:controller` veido kontroliera skeletu, kas atbilst oficiālajam skeleton izkārtojumam:

| | |
|--|--|
| **Ceļš** | `app/Controller/{Nosaukums}.php` |
| **Vārdu telpa** | `App\Controller` |
| **Stils** | `flight\Engine` konstruktora injekcija (bez `Flight::` klases ķermenī) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Paredzamās formas piemērs (vienkāršots):

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// piem. $this->app->render('…', […]);
	}
}
```

Reģistrējiet to ar klases izsaucamajam, lai Dice varētu izveidot kontrolieri:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Kāpēc šis izkārtojums?** Mapes **reģistram** jāatbilst vārdu telpai (`Controller` nevis `controllers`) Composer PSR-4 Linux sistēmām — skatiet [Autoloadēšanu](/learn/autoloading). Tas pats ceļš ir tas, ko root un scoped `AGENTS.md` faili saka AI rīkiem izmantot, lai ģenerētie un rokām rakstītie kontrolieri paliktu identiski.

> Vecākā dokumentācijā un kopienas projektos dažreiz tika izmantots `app/controllers/` un `app\controllers`. Tas joprojām ir derīgs, ja *jūsu* kokā joprojām tiek izmantotas mazie burti mapēm. **Jauni skeleton projekti un pašreizējā `make:controller` izvade izmanto `app/Controller/` + `App\Controller`.**

### Active Record modeļa ģenerēšana

Vispirms pārliecinieties, ka esat instalējis [Active Record](/awesome-plugins/active-record) spraudni.

```bash
php runway make:record users
```

Oficiālajā skeleton modeļi atrodas **`app/Model/`** ar vārdu telpu **`App\Model`**, un DB savienojums ir **[SimplePdo](/learn/simple-pdo)** (injicējiet to vai nododiet ActiveRecord konstruktoram). Ģenerētie failu nosaukumi/vārdu telpas seko Runway pašreizējiem noklusējumiem un jūsu `runway` konfigurācijai — dodiet priekšroku jaunu modeļu saskaņošanai ar `App\Model`, lai tie atbilstu [autoloadēšanai](/learn/autoloading) un `AGENTS.md`.

Oficiālā skeleton posts demonstrācijas modelim atbilstošs piemērs:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Ja vecāks ģenerators joprojām izvada `app/records` / `app\records`, varat saglabāt šo konvenciju mantojuma lietotnēs vai pārvietot failus uz `app/Model/` un atjaunināt vārdu telpu, lai atbilstu mapes reģistram.

### Migrācijas (skeleton)

Oficiālais skeleton nodrošina projekta komandu (atklātu no `app/commands/`), piemēram:

```bash
php runway migrate
```

Migrācijas ir SQL faili zem `migrations/` (piemēram, `YYYYMMDDHHMMSS_apraksts.sql` SQLite un `…_apraksts.mysql.sql` MySQL), izvēlēti no jūsu datu bāzes draivera konfigurācijas / vides. Precīzi karodziņi un uzvedība ir noteikta šajā projekta komandā — palaidiet `php runway migrate --help` savā lietotnē.

### AI palīgi

Runway atklāj AI orientētas komandas, kas tiek izmantotas ar [AI un izstrādātāja pieredzi](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Tās saglabā LLM akreditācijas datus un ģenerē projekta instrukcijas (galvenokārt **`AGENTS.md`**). Skeletonā uzskatiet `AGENTS.md` (un scoped kopijas zem `app/`) plus **`SECURITY.md`** par patiesības avotu aģentiem.

### Visu maršrutu parādīšana

Tas parādīs visus maršrutus, kas pašlaik ir reģistrēti ar Flight.

```bash
php runway routes
```

Ja vēlaties skatīt tikai specifiskus maršrutus, varat nodot karodziņu maršrutu filtrēšanai.

```bash
# Parādīt tikai GET maršrutus
php runway routes --get

# Parādīt tikai POST maršrutus
php runway routes --post

# utt.
```

## Pielāgotu komandu pievienošana Runway

Ja veidojat pakotni Flight vai vēlaties pievienot savas pielāgotās komandas savam projektam, varat to izdarīt, izveidojot `src/commands/`, `flight/commands/`, `app/commands/` vai `commands/` direktoriju savam projektam/pakotnei. Ja nepieciešama turpmāka pielāgošana, skatiet zemāk esošo sadaļu par konfigurāciju.

Skeletonā projekta komandas atrodas **`app/commands/`** ar vārdu telpu **`App\Command`**. Runway tās atklāj pēc ceļa; saglabājiet šo mapi sinhronizētu ar Composer classmap/PSR-4, kā jūsu projekts jau dara.

Lai izveidotu komandu, vienkārši paplašiniet `AbstractBaseCommand` klasi un īstenojiet vismaz `__construct` metodi un `execute` metodi.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * Konstruktor
     *
     * @param array<string,mixed> $config Konfigurācija no app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Izveidot piemēru dokumentācijai', $config);
        $this->argument('<funny-gif>', 'Smieklīgā gif nosaukums');
    }

	/**
     * Izpilda funkciju
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Veido piemēru...');

		// Dariet kaut ko šeit

		$io->ok('Piemērs izveidots!');
	}
}
```

Skatiet [adhocore/php-cli dokumentāciju](https://github.com/adhocore/php-cli), lai iegūtu vairāk informācijas par to, kā izveidot savas pielāgotās komandas Flight lietotnē!

## Konfigurācijas pārvaldība

Tā kā konfigurācija ir pārvietota uz `app/config/config.php` sākot ar `v1.2.0`, ir dažas palīgfunkcijas konfigurācijas pārvaldīšanai.

> **Skeleton padoms:** Saglabājiet `config.php` kā **burtiskas** PHP vērtības. Noslēpumi pieder `.env`. Izvairieties no `$_ENV[...]` izteiksmēm `config.php` iekšpusē — `config:set` pārraksta šo failu kā statiskus datus un varētu iebāzt noslēpumus failā. Skatiet [Konfigurāciju](/learn/configuration).

### Vecās konfigurācijas migrēšana

Ja jums ir vecs `.runway-config.json` fails, varat viegli migrēt to uz `app/config/config.php` ar šādu komandu:

```bash
php runway config:migrate
```

### Konfigurācijas vērtības iestatīšana

Varat iestatīt konfigurācijas vērtību, izmantojot `config:set` komandu. Tas ir noderīgi, ja vēlaties atjaunināt konfigurācijas vērtību, neatverot failu.

```bash
php runway config:set app_root "app/"
```

### Konfigurācijas vērtības iegūšana

Varat iegūt konfigurācijas vērtību, izmantojot `config:get` komandu.

```bash
php runway config:get app_root
```

## Visas Runway konfigurācijas

Ja nepieciešams pielāgot Runway konfigurāciju, varat iestatīt šīs vērtības `app/config/config.php`. Zemāk ir dažas papildu konfigurācijas, kuras varat iestatīt:

```php
<?php
// app/config/config.php
return [
    // ... citi konfigurācijas vērtības ...

    'runway' => [
        // Šeit atrodas jūsu lietotnes direktorija
        'app_root' => 'app/',

        // Šis ir direktorijs, kur atrodas jūsu root index fails
        'index_root' => 'public/',

        // Tie ir ceļi uz citu projektu saknēm
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Bāzes ceļi visticamāk nav jākonfigurē, bet tas ir šeit, ja vēlaties
        'base_paths' => [
            '/includes/libs/vendor', // ja jums ir patiešām unikāls ceļš jūsu vendor direktorijai vai kaut kam citam
        ],

        // Galīgie ceļi ir atrašanās vietas projektā, kur meklēt komandu failus
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Ja vēlaties vienkārši pievienot pilnu ceļu, dariet to (absolūts vai relatīvs pret projekta sakni)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Konfigurācijas piekļuve

Ja nepieciešams efektīvi piekļūt konfigurācijas vērtībām, varat tām piekļūt caur `__construct` metodi vai `app()` metodi. Ir arī svarīgi atzīmēt, ka ja jums ir `app/config/services.php` fails, šie pakalpojumi arī būs pieejami jūsu komandai.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Piekļūt konfigurācijai
    $app_root = $this->config['runway']['app_root'];
    
    // Piekļūt pakalpojumiem, piemēram, varbūt datu bāzes savienojumam
    $database = $this->config['database']
    
    // ...
}
```

## AI palīgu ietvari

Runway ir daži palīgu ietvari, kas atvieglo AI komandu ģenerēšanu. Varat izmantot `addOption` un `addArgument` veidā, kas jutās līdzīgi Symfony Console. Tas ir noderīgi, ja izmantojat AI rīkus savu komandu ģenerēšanai.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Izveidot piemēru dokumentācijai', $config);
    
    // Mode arguments ir nullable un noklusēti kā pilnībā neobligāti
    $this->addOption('name', 'Piemēra nosaukums', null);
}
```

## Skatiet arī

- [Instalācija](/install) - Skeleton koks un create-project noklusējumi
- [Autoloadēšana](/learn/autoloading) - `App\` un mapes reģistrs
- [Atkarību injekcija](/learn/dependency-injection-container) - Dice + Engine injekcija ģenerētajiem kontrolieriem
- [AI un izstrādātāja pieredze](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Modeļi, kas izmantoti ar `make:record` / skeleton `App\Model`
- [SimplePdo](/learn/simple-pdo) - DB savienojums, ko izmanto skeleton migrācijas un modeļi