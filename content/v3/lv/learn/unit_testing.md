# Vienību testēšana

## Pārskats

Vienību testēšana Flight lietotnē palīdz nodrošināt, ka jūsu lietotne darbojas, kā paredzēts, agri atklāt kļūdas un padarīt koda bāzi vieglāk uzturējamu. Flight ir izstrādāts, lai nevainojami sadarbotos ar [PHPUnit](https://phpunit.de/), kas ir populārākais PHP testēšanas ietvars.

## Izpratne

Vienību testi pārbauda atsevišķu jūsu lietotnes daļu (piemēram, kontrolleru vai pakalpojumu) uzvedību izolēti. Flight kontekstā tas nozīmē pārbaudīt, kā jūsu maršruti, kontrolleri un loģika reaģē uz dažādiem ievaddatiem, nepaļaujoties uz globālo stāvokli vai reāliem ārējiem pakalpojumiem.

Galvenie principi:
- **Testējiet uzvedību, nevis ieviešanu:** Koncentrējieties uz to, ko jūsu kods dara, nevis to, kā tas to dara.
- **Izvairieties no globālā stāvokļa:** Izmantojiet atkarību injekciju, nevis `Flight::set()` vai `Flight::get()`.
- **Mockojiet ārējos pakalpojumus:** Aizstājiet tādas lietas kā datu bāzes vai e-pasta sūtītājus ar testa aizstājējiem.
- **Saglabājiet testus ātrus un fokusētus:** Vienību testiem nevajadzētu pieskarties reālām datu bāzēm vai API.

## Pamata lietošana

### PHPUnit iestatīšana

1. Instalējiet PHPUnit ar Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Izveidojiet `tests` direktoriju jūsu projekta saknē.
3. Pievienojiet testa skriptu savam `composer.json`:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Izveidojiet `phpunit.xml` failu:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Tagad varat palaist savus testus ar `composer test`.

### Vienkārša maršruta apstrādātāja testēšana

Pieņemsim, ka jums ir maršruts, kas validē e-pasta adresi:

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
    protected $app;
    public function __construct(flight\Engine $app) {
        $this->app = $app;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Vienkāršs tests šim kontrollerim:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Padomi:**
- Imitējiet POST datus, izmantojot `$app->request()->data`.
- Izvairieties izmantot `Flight::` statiskos elementus savos testos — izmantojiet `$app` instanci.

### Atkarību injekcijas izmantošana testējamiem kontrolleriem

Ievadiet atkarības (piemēram, datu bāzi vai e-pasta sūtītāju) savos kontrolleros, lai tos būtu viegli mockot testos:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

Un tests ar mock objektiem:

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Papildu lietošana

- **Mockošana:** Izmantojiet PHPUnit iebūvētos mock objektus vai anonīmas klases, lai aizstātu atkarības.
- **Kontrolleru tieša testēšana:** Izveidojiet kontrolleru instanci ar jaunu `Engine` un mockojiet atkarības.
- **Izvairieties no pārmērīgas mockošanas:** Ļaujiet reālajai loģikai darboties, kur iespējams; mockojiet tikai ārējos pakalpojumus.

## Skatīt arī

- [Vienību testēšanas ceļvedis](/guides/unit-testing) - Visaptverošs ceļvedis par vienību testēšanas labāko praksi.
- [Atkarību injekcijas konteiners](/learn/dependency-injection-container) - Kā izmantot DIC, lai pārvaldītu atkarības un uzlabotu testējamību.
- [Paplašināšana](/learn/extending) - Kā pievienot savus palīgus vai pārdefinēt pamata klases.
- [SimplePdo](/learn/simple-pdo) - Vienkāršo datu bāzes mijiedarbību un ir vieglāk mockojams testos.
- [Pieprasījumi](/learn/requests) - HTTP pieprasījumu apstrāde Flight.
- [Atbildes](/learn/responses) - Atbilžu sūtīšana lietotājiem.
- [Vienību testēšana un SOLID principi](/learn/unit-testing-and-solid-principles) - Uzziniet, kā SOLID principi var uzlabot jūsu vienību testus.

## Problēmu novēršana

- Izvairieties no globālā stāvokļa (`Flight::set()`, `$_SESSION` utt.) izmantošanas savā kodā un testos.
- Ja jūsu testi ir lēni, iespējams, rakstāt integrācijas testus — mockojiet ārējos pakalpojumus, lai vienību testi būtu ātri.
- Ja testu iestatīšana ir sarežģīta, apsveriet iespēju refaktorēt savu kodu, lai izmantotu atkarību injekciju.

## Izmaiņu žurnāls

- v3.15.0 - Pievienoti piemēri atkarību injekcijai un mockošanai.