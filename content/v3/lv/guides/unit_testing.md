# Vienību testēšana Flight PHP ar PHPUnit

Šis ceļvedis iepazīstina ar vienību testēšanu Flight PHP, izmantojot [PHPUnit](https://phpunit.de/), un ir paredzēts iesācējiem, kuri vēlas saprast, *kāpēc* vienību testēšanai ir nozīme un kā to praktiski pielietot. Mēs koncentrēsimies uz *uzvedības* testēšanu — nodrošinot, ka jūsu lietotne dara to, ko jūs sagaidāt, piemēram, nosūta e-pastu vai saglabā ierakstu — nevis uz triviāliem aprēķiniem. Sāksim ar vienkāršu [maršruta apstrādātāju](/learn/routing) un pāriesim uz sarežģītāku [kontrolleri](/learn/routing), iekļaujot [atkarību injekciju](/learn/dependency-injection-container) (DI) un trešo pušu pakalpojumu atdarināšanu.

## Kāpēc vienību testēšana?

Vienību testēšana nodrošina, ka jūsu kods darbojas, kā paredzēts, atklājot kļūdas, pirms tās nonāk ražošanā. Tas ir īpaši vērtīgi Flight, kur vieglā maršrutēšana un elastība var novest pie sarežģītām mijiedarbībām. Atsevišķiem izstrādātājiem vai komandām vienību testi kalpo kā drošības tīkls, dokumentējot paredzamo uzvedību un novēršot regresijas, kad vēlāk atgriežaties pie sava koda. Tie arī uzlabo dizainu: grūti testējams kods bieži norāda uz pārāk sarežģītām vai cieši saistītām klasēm.

Atšķirībā no vienkāršotiem piemēriem (piemēram, testējot `x * y = z`), mēs koncentrēsimies uz reālās pasaules uzvedību, piemēram, ievades validāciju, datu saglabāšanu vai darbību aktivizēšanu, piemēram, e-pastus. Mūsu mērķis ir padarīt testēšanu pieejamu un jēgpilnu.

## Vispārīgi vadošie principi

1. **Testējiet uzvedību, nevis ieviešanu**: Koncentrējieties uz rezultātiem (piemēram, "e-pasts nosūtīts" vai "ieraksts saglabāts"), nevis uz iekšējām detaļām. Tas padara testus izturīgus pret refaktorēšanu.
2. **Pārtrauciet lietot `Flight::`**: Flight statiskās metodes ir ļoti ērtas, bet apgrūtina testēšanu. Jums vajadzētu pierast lietot `$app` mainīgo no `$app = Flight::app();`. `$app` ir visas tās pašas metodes, kas ir `Flight::`. Jūs joprojām varēsiet lietot `$app->route()` vai `$this->app->json()` savā kontrollerī utt. Tāpat izmantojiet īsto Flight maršrutētāju ar `$router = $app->router()` un pēc tam varat lietot `$router->get()`, `$router->post()`, `$router->group()` utt. Skatiet [Maršrutēšana](/learn/routing).
3. **Turiet testus ātrus**: Ātri testi veicina biežu izpildi. Izvairieties no lēnām darbībām, piemēram, datubāzes izsaukumiem vienību testos. Ja jums ir lēns tests, tā ir zīme, ka rakstāt integrācijas testu, nevis vienību testu. Integrācijas testi ir tad, kad faktiski iesaistāt reālas datubāzes, reālus HTTP izsaukumus, reālu e-pasta sūtīšanu utt. Tiem ir sava vieta, bet tie ir lēni un var būt nestabili, kas nozīmē, ka tie dažkārt neizdodas nezināma iemesla dēļ.
4. **Izmantojiet aprakstošus nosaukumus**: Testu nosaukumiem skaidri jāapraksta pārbaudāmā uzvedība. Tas uzlabo lasāmību un uzturējamību.
5. **Izvairieties no globālajiem mainīgajiem kā no mēra**: Samaziniet `$app->set()` un `$app->get()` lietošanu, jo tie darbojas kā globālais stāvoklis, prasot atdarinājumus katrā testā. Dodiet priekšroku DI vai DI konteineram (skatiet [Atkarību injekcijas konteiners](/learn/dependency-injection-container)). Pat `$app->map()` metodes izmantošana tehniski ir "globāla", un no tās vajadzētu izvairīties par labu DI. Izmantojiet sesijas bibliotēku, piemēram, [flightphp/session](https://github.com/flightphp/session), lai testos varētu atdarināt sesijas objektu. **Neizsauciet** [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) tieši savā kodā, jo tā ir globālā mainīgā ieviešana jūsu kodā, kas apgrūtina testēšanu.
6. **Izmantojiet atkarību injekciju**: Injicējiet atkarības (piemēram, [`PDO`](https://www.php.net/manual/en/class.pdo.php), e-pasta sūtītājus) kontrolleros, lai izolētu loģiku un vienkāršotu atdarināšanu. Ja jums ir klase ar pārāk daudz atkarībām, apsveriet iespēju to refaktorēt mazākās klasēs, no kurām katra ir atbildīga par vienu lietu, ievērojot [SOLID principus](https://en.wikipedia.org/wiki/SOLID).
7. **Atdariniet trešo pušu pakalpojumus**: Atdariniet datubāzes, HTTP klientus (cURL) vai e-pasta pakalpojumus, lai izvairītos no ārējiem izsaukumiem. Testējiet vienu vai divus slāņus dziļi, bet ļaujiet savai pamatloģikai darboties. Piemēram, ja jūsu lietotne sūta īsziņas, jūs **NEVĒLATIES** patiešām sūtīt īsziņu katru reizi, kad palaižat testus, jo šīs izmaksas uzkrāsies (un būs lēnāk). Tā vietā atdariniet īsziņu pakalpojumu un vienkārši pārbaudiet, vai jūsu kods izsauca īsziņu pakalpojumu ar pareizajiem parametriem.
8. **Tiecieties uz augstu pārklājumu, nevis pilnību**: 100% rindu pārklājums ir labs, bet tas nenozīmē, ka viss jūsu kodā ir testēts tā, kā vajadzētu (izpētiet [zaru/ceļu pārklājumu PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Prioritāti piešķiriet kritiskajai uzvedībai (piemēram, lietotāja reģistrācijai, API atbildēm un neveiksmīgu atbilžu tveršanai).
9. **Izmantojiet kontrollerus maršrutiem**: Maršrutu definīcijās izmantojiet kontrollerus, nevis slēgumus. `flight\Engine $app` pēc noklusējuma tiek injicēts katrā kontrollerī caur konstruktoru. Testos izmantojiet `$app = new Flight\Engine()`, lai testa ietvaros izveidotu Flight instanci, injicētu to savā kontrollerī un izsauktu metodes tieši (piemēram, `$controller->register()`). Skatiet [Flight paplašināšana](/learn/extending) un [Maršrutēšana](/learn/routing).
10. **Izvēlieties atdarināšanas stilu un pieturieties pie tā**: PHPUnit atbalsta vairākus atdarināšanas stilus (piemēram, prophecy, iebūvētos atdarinājumus), vai arī varat izmantot anonīmās klases, kurām ir savas priekšrocības, piemēram, koda pabeigšana, lūšana, ja maināt metodes definīciju, utt. Vienkārši esiet konsekventi savos testos. Skatiet [PHPUnit Mock Objects](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Izmantojiet `protected` redzamību metodēm/īpašībām, kuras vēlaties testēt apakšklasēs**: Tas ļauj tās pārdefinēt testa apakšklasēs, nepadarot tās publiskas; tas ir īpaši noderīgi anonīmu klašu atdarinājumiem.

## PHPUnit iestatīšana

Pirmkārt, iestatiet [PHPUnit](https://phpunit.de/) savā Flight PHP projektā, izmantojot Composer, lai atvieglotu testēšanu. Skatiet [PHPUnit sākšanas rokasgrāmatu](https://phpunit.readthedocs.io/en/12.3/installation.html), lai iegūtu vairāk informācijas.

1. Sava projekta direktorijā izpildiet:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Tas instalē jaunāko PHPUnit kā izstrādes atkarību.

2. Izveidojiet `tests` direktoriju sava projekta saknē testa failiem.

3. Pievienojiet testa skriptu failam `composer.json` ērtībai:
   ```json
   // cits composer.json saturs
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Izveidojiet failu `phpunit.xml` saknē:
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

Tagad, kad jūsu testi ir izveidoti, varat palaist `composer test`, lai izpildītu testus.

## Vienkārša maršruta apstrādātāja testēšana

Sāksim ar pamata [maršrutu](/learn/routing), kas validē lietotāja e-pasta ievadi. Mēs testēsim tā uzvedību: veiksmes ziņojuma atgriešanu derīgiem e-pastiem un kļūdas ziņojumu nederīgiem. E-pasta validācijai mēs izmantojam [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

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
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'Invalid email'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'Valid email'];
		}

		$this->app->json($responseArray);
	}
}
```

Lai to testētu, izveidojiet testa failu. Skatiet [Vienību testēšana un SOLID principi](/learn/unit-testing-and-solid-principles), lai uzzinātu vairāk par testu strukturēšanu:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simulē POST datus
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Simulē POST datus
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('Invalid email', $output['message']);
	}
}
```

**Galvenie punkti**:
- Mēs simulējam POST datus, izmantojot pieprasījuma klasi. Neizmantojiet globālus mainīgos, piemēram, `$_POST`, `$_GET` utt., jo tas padara testēšanu sarežģītāku (jums vienmēr ir jāatiestata šīs vērtības, pretējā gadījumā citi testi var neizdoties).
- Visi kontrolleri pēc noklusējuma saņem `flight\Engine` instanci, kas tiek injicēta tajos pat bez DIC konteinera iestatīšanas. Tas ievērojami atvieglo kontrolleru tiešu testēšanu.
- Nav vispār izmantots `Flight::`, padarot kodu vieglāk testējamu.
- Testi pārbauda uzvedību: pareizu statusu un ziņojumu derīgiem/nederīgiem e-pastiem.

Izpildiet `composer test`, lai pārliecinātos, ka maršruts darbojas, kā paredzēts. Lai uzzinātu vairāk par [pieprasījumiem](/learn/requests) un [atbildēm](/learn/responses) Flight, skatiet attiecīgo dokumentāciju.

## Atkarību injekcijas izmantošana testējamiem kontrolleriem

Sarežģītākiem scenārijiem izmantojiet [atkarību injekciju](/learn/dependency-injection-container) (DI), lai kontrollerus padarītu testējamus. Izvairieties no Flight globālajiem mainīgajiem (piemēram, `Flight::set()`, `Flight::map()`, `Flight::register()`), jo tie darbojas kā globālais stāvoklis, prasot atdarinājumus katrā testā. Tā vietā izmantojiet Flight DI konteineru, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) vai manuālo DI.

Izmantosim [`flight\database\SimplePdo`](/learn/simple-pdo), nevis tiešu PDO. Šo palīgu ir daudz vieglāk atdarināt un vienību testēt (un tas ir vēlamāks par novecojušo `PdoWrapper`).

Šeit ir kontrolleris, kas saglabā lietotāju datubāzē un nosūta sagaidīšanas e-pastu:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// return pievienošana šeit palīdz apturēt izpildi vienību testēšanā
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

**Galvenie punkti**:
- Kontrolleris ir atkarīgs no [`SimplePdo`](/learn/simple-pdo) instances un `MailerInterface` (izdomāta trešās puses e-pasta pakalpojuma).
- Atkarības tiek injicētas caur konstruktoru, izvairoties no globālajiem mainīgajiem.

### Kontrollera testēšana ar atdarinājumiem (mocks)

Tagad testēsim `UserController` uzvedību: e-pastu validāciju, saglabāšanu datubāzē un e-pastu sūtīšanu. Mēs atdarināsim datubāzi un e-pasta sūtītāju, lai izolētu kontrolleri.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Dažreiz ir nepieciešams sajaukt atdarināšanas stilus
		// Šeit mēs izmantojam PHPUnit iebūvēto atdarinājumu PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Anonīmas klases izmantošana, lai atdarinātu SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Kad mēs to atdarinām šādi, mēs īsti neveicam datubāzes izsaukumu.
			// Mēs varam to tālāk konfigurēt, lai mainītu PDOStatement atdarinājumu, simulējot kļūmes utt.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Tukšs konstruktors apiet vecāka konstruktoru
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Should not be called');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Should not be called');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// Nepieciešams kartēt jsonHalt, lai izvairītos no izejas
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('Invalid email', $result['message']);
    }
}
```

**Galvenie punkti**:
- Mēs atdarinām `SimplePdo` un `MailerInterface`, lai izvairītos no reāliem datubāzes vai e-pasta izsaukumiem.
- Testi pārbauda uzvedību: derīgi e-pasti aktivizē datubāzes ievietošanu un e-pasta sūtīšanu; nederīgi e-pasti izlaiž abus.
- Atdariniet trešo pušu atkarības (piemēram, `SimplePdo`, `MailerInterface`), ļaujot kontrollera loģikai darboties.

### Pārāk liela atdarināšana

Esiet uzmanīgi, lai neatdarinātu pārāk lielu daļu sava koda. Tālāk es sniegšu piemēru, kāpēc tas varētu būt slikti, izmantojot mūsu `UserController`. Mēs mainīsim šo pārbaudi uz metodi ar nosaukumu `isEmailValid` (izmantojot `filter_var`), bet pārējos jaunos papildinājumus — uz atsevišķu metodi `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// return pievienošana šeit palīdz apturēt izpildi vienību testēšanā
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'Invalid email']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'User registered']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

Un tagad pārāk atdarinātais vienību tests, kas īsti neko netesta:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// mēs šeit izlaižam papildu atkarību injekciju, jo tas ir "viegli"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Apiet atkarības konstruktorā
			public function __construct($app) {
				$this->app = $app;
			}

			// Mēs vienkārši piespiedīsim, lai tas būtu derīgs.
			protected function isEmailValid($email) {
				return true; // Vienmēr atgriež true, apejot reālo validāciju
			}

			// Apiet faktiskos datubāzes un e-pasta sūtītāja izsaukumus
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
    }
}
```

Urrā, mums ir vienību testi un tie izdodas! Bet pagaidiet, kas notiktu, ja es faktiski mainītu `isEmailValid` vai `registerUser` iekšējo darbību? Mani testi joprojām izdotos, jo es esmu atdarinājis visu funkcionalitāti. Ļaujiet man parādīt, ko es domāju.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... citas metodes ...

	protected function isEmailValid($email) {
		// Mainītā loģika
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Tagad tam vajadzētu būt tikai noteiktam domēnam
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Ja es palaistu savus iepriekš minētos vienību testus, tie joprojām izdotos! Bet, tā kā es netestēju uzvedību (faktiski neļaujot daļai koda izpildīties), es, iespējams, esmu ieprogrammējis kļūdu, kas gaida, lai notiktu ražošanā. Tests būtu jāmaina, lai ņemtu vērā jauno uzvedību, kā arī pretējo gadījumu, kad uzvedība nav tāda, kādu mēs sagaidām.

## Pilns piemērs

Pilnu Flight PHP projekta piemēru ar vienību testiem varat atrast GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide). Lai iegūtu dziļāku izpratni, skatiet [Vienību testēšana un SOLID principi](/learn/unit-testing-and-solid-principles).

## Biežākās kļūdas

- **Pārāk liela atdarināšana**: Neatdariniet katru atkarību; ļaujiet daļai loģikas (piemēram, kontrollera validācijai) izpildīties, lai testētu reālu uzvedību. Skatiet [Vienību testēšana un SOLID principi](/learn/unit-testing-and-solid-principles).
- **Globālais stāvoklis**: Bieža globālo PHP mainīgo (piemēram, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) izmantošana padara testus trauslus. Tas pats attiecas uz `Flight::`. Refaktorējiet, lai atkarības tiktu nodotas tieši.
- **Sarežģīta iestatīšana**: Ja testa iestatīšana ir apgrūtinoša, jūsu klasei, iespējams, ir pārāk daudz atkarību vai pienākumu, pārkāpjot [SOLID principus](/learn/unit-testing-and-solid-principles).

## Mērogošana ar vienību testiem

Vienību testi noder lielākos projektos vai tad, kad pēc mēnešiem atgriežaties pie koda. Tie dokumentē uzvedību un atklāj regresijas, glābjot jūs no lietotnes no jauna apgūšanas. Atsevišķiem izstrādātājiem testējiet kritiskos ceļus (piemēram, lietotāja reģistrāciju, maksājumu apstrādi). Komandām testi nodrošina konsekventu uzvedību visās izmaiņās. Skatiet [Kāpēc ietvari?](/learn/why-frameworks), lai uzzinātu vairāk par ieguvumiem, ko sniedz ietvari un testi.

Dalieties ar saviem testēšanas padomiem Flight PHP dokumentācijas krātuvē!

_Rakstījis [n0nag0n](https://github.com/n0nag0n) 2025_