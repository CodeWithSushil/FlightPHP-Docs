# Atkarību ievadīšanas konteiners

## Pārskats

Atkarību ievadīšanas konteiners (DIC) ir jaudīgs papildinājums, kas ļauj pārvaldīt
jūsu lietojumprogrammas atkarības. Tas ir arī viens no lielākajiem iemesliem, kāpēc Flight labi sadarbojas ar [AI kodēšanas rīkiem](/learn/ai) un vienību testiem: kontrolleri konstruktorā saņem to, kas tiem nepieciešams, nevis piekļūst globālajiem mainīgajiem.

## Izpratne

Atkarību ievadīšana (DI) ir galvenā koncepcija mūsdienu PHP ietvaros un tiek
izmantota, lai pārvaldītu objektu izveidi un konfigurāciju. Daži DIC
bibliotēku piemēri: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/),
[PHP-DI](http://php-di.org/) un [league/container](https://container.thephpleague.com/).

DIC ir elegantīgs veids, kā izveidot un pārvaldīt savas klases centralizētā
vietā. Tas ir noderīgi, ja jums viens un tas pats objekts jānodod vairākām
klasēm (kontrolleriem, starpprogrammatūrai, komandām utt.).

Oficiālais [flightphp/skeleton](https://github.com/flightphp/skeleton) savieno **Dice** failā `app/config/services.php`, aizstāj kopīgo `flight\Engine` instanci un atrisina maršrutu mērķus, piemēram, `[App\Controller\HomeController::class, 'index']`. Jauniem projektiem izmantojiet šo paraugu, lai cilvēki un aģenti rediģētu vienas un tās pašas vietas.

## Pamata lietošana

Vecais veids, kā to darīt, varētu izskatīties šādi:
```php

require 'vendor/autoload.php';

// klase, lai pārvaldītu lietotājus no datubāzes
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// jūsu routes.php failā

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// citi UserController maršruti...

Flight::start();
```

No augšējā koda var redzēt, ka mēs izveidojam jaunu `PDO` objektu un nododam to
mūsu `UserController` klasei. Mazai lietojumprogrammai tas ir labi, bet, kad
jūsu lietojumprogramma aug, jūs atklāsiet, ka vienu un to pašu `PDO` objektu
izveidojat vai nododat vairākās vietās. Šeit noder DIC.

Šis ir tas pats piemērs, izmantojot DIC (ar Dice):
```php

require 'vendor/autoload.php';

// tā pati klase kā iepriekš. Nekas nav mainījies
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// izveido jaunu konteineru
$container = new \Dice\Dice;

// pievieno kārtulu, lai pateiktu konteineram, kā izveidot PDO objektu
// neaizmirsti to atkārtoti piešķirt sev, kā parādīts zemāk!
$container = $container->addRule('PDO', [
	// shared nozīmē, ka katru reizi tiks atgriezts tas pats objekts
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Tas reģistrē konteinera apstrādātāju, lai Flight zinātu to izmantot.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// tagad mēs varam izmantot konteineru, lai izveidotu mūsu UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Varbūt jūs domājat, ka piemēram tika pievienots daudz lieka koda.
Burvība parādās, kad jums ir cits kontrolleris, kuram nepieciešams `PDO` objekts.

```php

// Ja visiem jūsu kontrolleriem konstruktorā ir nepieciešams PDO objekts,
// katram no šiem maršrutiem tas tiks automātiski ievadīts!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

Papildu ieguvums, izmantojot DIC, ir tas, ka vienību testēšana kļūst daudz vienkāršāka. Jūs varat
izveidot viltojuma (mock) objektu un nodot to savai klasei. Tas ir milzīgs ieguvums, rakstot
testus savai lietojumprogrammai — un, kad AI palīgs ģenerē kontrolleri, konstruktora ievadīšana sniedz tam skaidru, konsekventu paraugu, kam sekot ([vienību testēšanas ceļvedis](/guides/unit-testing)).

### Centralizēta DIC apstrādātāja izveide

Jūs varat izveidot centralizētu DIC apstrādātāju savā pakalpojumu failā, [paplašinot](/learn/extending) savu lietojumprogrammu. Šeit ir piemērs:

```php
// services.php

// izveido jaunu konteineru
$container = new \Dice\Dice;
// neaizmirsti to atkārtoti piešķirt sev, kā parādīts zemāk!
$container = $container->addRule('PDO', [
	// shared nozīmē, ka katru reizi tiks atgriezts tas pats objekts
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// tagad mēs varam izveidot kartējamu metodi jebkura objekta izveidei.
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Tas reģistrē konteinera apstrādātāju, lai Flight zinātu to izmantot kontrolleriem/starpprogrammatūrai
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// pieņemsim, ka mums ir šāda parauga klase, kas konstruktorā saņem PDO objektu
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// kods, kas nosūta e-pastu
	}
}

// Un visbeidzot jūs varat izveidot objektus, izmantojot atkarību ievadīšanu
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

Flight ir spraudnis, kas nodrošina vienkāršu PSR-11 saderīgu konteineru, kuru varat izmantot, lai pārvaldītu
savu atkarību ievadīšanu. Šeit ir ātrs piemērs, kā to lietot:

```php

// index.php, piemēram
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// izvadīs šo pareizi!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Papildu lietošana ar flightphp/container

Jūs varat arī rekursīvi atrisināt atkarības. Šeit ir piemērs:

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Implementācija ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

Jūs varat arī izveidot savu DIC apstrādātāju. Tas ir noderīgi, ja jums ir pielāgots
konteiners, kuru vēlaties izmantot un kurš nav PSR-11 (Dice). Skatiet
[ pamata lietošanas ](#basic-usage) sadaļu, lai uzzinātu, kā to izdarīt.

Turklāt ir daži noderīgi noklusējumi, kas atvieglos jūsu darbu, lietojot Flight.

#### Engine instance (nepieciešama `$app` ievadīšanai)

Ja kontrolleros vai starpprogrammatūrā norādāt tipu `flight\Engine`, **Dice nedrīkst izveidot jaunu Engine**. Aizstājiet to ar to pašu instanci no sāknēšanas faila. To dara oficiālais skeleton, un tas ir paraugs, ko `AGENTS.md` sagaida no AI ģenerētiem kontrolleriem:

```php
// Kaut kur jūsu sāknēšanas / services.php failā
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // vai $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Svarīgi: atkārtoti izmantot sāknēto Engine — neļaujiet Dice veikt `new Engine()`
		Engine::class => $app,
		// Jaunam kodam vēlams izmantot SimplePdo
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Neobligāts palīgs maršrutu ārpus koda
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (skeleton izkārtojums — mapes nosaukums atbilst namespace)
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
		// Nav Flight:: fasādes lietojumprogrammas slānī — vieglāk testēt un skaidrāk AI rīkiem
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Ja izlaižat `Engine` aizstāšanu, Dice var izveidot otru Engine, un jūsu kontrolleris nedalīs maršrutus, konfigurāciju vai kartēto Twig `render` no sāknēšanas faila.

#### Citu koplietotu pakalpojumu pievienošana (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Pēc tam, kad services.php izveidojat $db, $config, $twig:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

Tad kontrolleri var konstruktorā pieņemt `SimplePdo $db` (vai jūsu config tipu) un nekad neizsaukt `Flight::db()`. Tas atbilst [vienību testēšanas](/guides/unit-testing) norādēm un skeleton stila paraugam.

#### Citu klašu pievienošana

Ja jums ir citas klases, kuras vēlaties pievienot konteineram, ar Dice tas ir vienkārši, jo konteiners tās atrisinās automātiski. Šeit ir piemērs:

```php

$container = new \Dice\Dice;
// Ja jums nav nepieciešams ievadīt atkarības savās klasēs,
// jums nekas nav jādefinē!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

Flight var izmantot arī jebkuru PSR-11 saderīgu konteineru. Tas nozīmē, ka varat izmantot jebkuru
konteineru, kas implementē PSR-11 saskarni. Šeit ir piemērs, izmantojot League
PSR-11 konteineru:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// tā pati UserController ideja kā iepriekš, bet ar SimplePdo tipu, nevis neapstrādātu PDO

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

Tas var būt nedaudz garāks nekā iepriekšējais Dice piemērs, taču tas joprojām
sasniedz to pašu rezultātu ar tādiem pašiem ieguvumiem!

## Skatīt arī
- [Instalācija](/install) — Skeleton izkārtojums un kur atrodas `services.php`.
- [Automātiskā ielāde](/learn/autoloading) — `App\` namespace un mapju **reģistrs**.
- [Flight paplašināšana](/learn/extending) — Uzziniet, kā pievienot atkarību ievadīšanu savām klasēm, paplašinot ietvaru.
- [Konfigurācija](/learn/configuration) — Uzziniet, kā konfigurēt Flight savai lietojumprogrammai.
- [Maršrutēšana](/learn/routing) — Uzziniet, kā definēt maršrutus savai lietojumprogrammai un kā atkarību ievadīšana darbojas ar kontrolleriem.
- [Starpprogrammatūra](/learn/middleware) — Uzziniet, kā izveidot starpprogrammatūru savai lietojumprogrammai un kā atkarību ievadīšana darbojas ar starpprogrammatūru.
- [Vienību testēšana](/guides/unit-testing) — Kāpēc konstruktora ievadīšana ir labāka par `Flight::` globālajiem mainīgajiem.
- [AI un izstrādātāju pieredze](/learn/ai) — Viens DI paraugs cilvēkiem un aģentiem.
- [SimplePdo](/learn/simple-pdo) — Ieteicamais datubāzes palīgs ievadīšanai.

## Problēmu novēršana
- Ja jums ir problēmas ar konteineru, pārliecinieties, ka konteineram nododat pareizos klašu nosaukumus.
- Kontrolleri, kas norāda tipu `Engine`, bet saņem "tukšu" lietotni: pievienojiet **Engine aizstāšanu** (skatīt iepriekš). Dice nedrīkst veikt `new` otrajam Engine.
- Klase netiek atrasta `App\Controller\…`: pārbaudiet mapju reģistru zem `app/Controller/` — skatiet [Automātiskā ielāde](/learn/autoloading).
- Apstrādātājam ir **jāatgriež** izveidotais objekts no `registerContainerHandler` (neizsauciet `Flight::make()` bez `return`).

## Izmaiņu žurnāls
- Dokumentācija — Dokumentēta skeleton Dice + Engine aizstāšana, SimplePdo un `App\Controller` izkārtojums AI draudzīgiem projektiem.
- v3.7.0 — Pievienota iespēja reģistrēt DIC apstrādātāju Flight.