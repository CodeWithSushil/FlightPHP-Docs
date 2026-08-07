# Maršrutēšana

## Pārskats
Maršrutēšana Flight PHP kartē URL modeļus ar atzvanīšanas funkcijām vai klašu metodēm, nodrošinot ātru un vienkāršu pieprasījumu apstrādi. Tā ir veidota ar minimālu papildu slogu, iesācējiem draudzīgu lietošanu un paplašināmību bez ārējām atkarībām.

## Izpratne
Maršrutēšana ir pamata mehānisms, kas savieno HTTP pieprasījumus ar jūsu lietojumprogrammas loģiku Flight. Definējot maršrutus, jūs norādāt, kā dažādi URL aktivizē konkrētu kodu — neatkarīgi no tā, vai tas notiek caur funkcijām, klašu metodēm vai kontrolieru darbībām. Flight maršrutēšanas sistēma ir elastīga, atbalstot pamata modeļus, nosauktus parametrus, regulārās izteiksmes un papildu funkcijas, piemēram, atkarību ievadīšanu un resursu maršrutēšanu. Šī pieeja uztur jūsu kodu organizētu un viegli uzturamu, vienlaikus paliekot ātra un vienkārša iesācējiem, kā arī paplašināma pieredzējušiem lietotājiem.

> **Piezīme:** Vēlaties saprast vairāk par maršrutēšanu? Apskatiet lapu ["kāpēc ietvars?"](/learn/why-frameworks), lai iegūtu padziļinātu skaidrojumu.

## Pamata lietošana

### Vienkārša maršruta definēšana
Pamata maršrutēšana Flight tiek veikta, saskaņojot URL modeli ar atzvanīšanas funkciju vai klases un metodes masīvu.

```php
Flight::route('/', function(){
    echo 'hello world!';
});
```

> Maršruti tiek saskaņoti to definēšanas secībā. Pirmais maršruts, kas atbilst pieprasījumam, tiks izsaukts.

### Funkciju izmantošana kā atzvanīšanas
Atzvanīšana var būt jebkurš objekts, kas ir izsaucams. Tātad varat izmantot parasto funkciju:

```php
function hello() {
    echo 'hello world!';
}

Flight::route('/', 'hello');
```

### Klašu un metožu izmantošana kā kontrolieris
Varat izmantot arī klases metodi (statisku vai nestatisku):

```php
class GreetingController {
    public function hello() {
        echo 'hello world!';
    }
}

Flight::route('/', [ 'GreetingController','hello' ]);
// vai
Flight::route('/', [ GreetingController::class, 'hello' ]); // ieteicamā metode
// vai
Flight::route('/', [ 'GreetingController::hello' ]);
// vai 
Flight::route('/', [ 'GreetingController->hello' ]);
```

Vai arī izveidojot objektu vispirms un pēc tam izsaucot metodi:

```php
use flight\Engine;

// GreetingController.php
class GreetingController
{
	protected Engine $app
    public function __construct(Engine $app) {
		$this->app = $app;
        $this->name = 'John Doe';
    }

    public function hello() {
        echo "Hello, {$this->name}!";
    }
}

// index.php
$app = Flight::app();
$greeting = new GreetingController($app);

Flight::route('/', [ $greeting, 'hello' ]);
```

> **Piezīme:** Pēc noklusējuma, kad kontrolieris tiek izsaukts ietvara ietvaros, `flight\Engine` klase vienmēr tiek ievadīta, ja vien nenorādāt citādi, izmantojot [atkarību ievadīšanas konteineru](/learn/dependency-injection-container)

### Metodei specifiska maršrutēšana

Pēc noklusējuma maršruta modeļi tiek saskaņoti ar visām pieprasījuma metodēm. Varat atbildēt uz konkrētām metodēm, novietojot identifikatoru pirms URL.

```php
Flight::route('GET /', function () {
  echo 'I received a GET request.';
});

Flight::route('POST /', function () {
  echo 'I received a POST request.';
});

// Jūs nevarat izmantot Flight::get() maršrutiem, jo tā ir metode
//    mainīgo iegūšanai, nevis maršruta izveidei.
Flight::post('/', function() { /* kods */ });
Flight::patch('/', function() { /* kods */ });
Flight::put('/', function() { /* kods */ });
Flight::delete('/', function() { /* kods */ });
```

Varat arī kartēt vairākas metodes vienai atzvanīšanai, izmantojot atdalītāju `|`:

```php
Flight::route('GET|POST /', function () {
  echo 'I received either a GET or a POST request.';
});
```

### Īpaša apstrāde HEAD un OPTIONS pieprasījumiem

Flight nodrošina iebūvētu apstrādi `HEAD` un `OPTIONS` HTTP pieprasījumiem:

#### HEAD pieprasījumi

- **HEAD pieprasījumi** tiek apstrādāti tāpat kā `GET` pieprasījumi, bet Flight automātiski noņem atbildes pamattekstu pirms tā nosūtīšanas klientam.
- Tas nozīmē, ka varat definēt maršrutu `GET`, un HEAD pieprasījumi uz to pašu URL atgriezīs tikai galvenes (bez satura), kā to nosaka HTTP standarti.

```php
Flight::route('GET /info', function() {
    echo 'This is some info!';
});
// HEAD pieprasījums uz /info atgriezīs tās pašas galvenes, bet bez pamatteksta.
```

#### OPTIONS pieprasījumi

`OPTIONS` pieprasījumus Flight automātiski apstrādā jebkuram definētam maršrutam.
- Kad tiek saņemts OPTIONS pieprasījums, Flight atbild ar `204 No Content` statusu un `Allow` galveni, kurā uzskaitītas visas atbalstītās HTTP metodes šim maršrutam.
- Jums nav jādefinē atsevišķs maršruts OPTIONS.

```php
// Maršrutam, kas definēts kā:
Flight::route('GET|POST /users', function() { /* ... */ });

// OPTIONS pieprasījums uz /users atbildēs ar:
//
// Statuss: 204 No Content
// Allow: GET, POST, HEAD, OPTIONS
```

### Router objekta izmantošana

Papildus varat iegūt Router objektu, kuram ir dažas palīgmetodes, ko varat izmantot:

```php

$router = Flight::router();

// kartē visas metodes tāpat kā Flight::route()
$router->map('/', function() {
	echo 'hello world!';
});

// GET pieprasījums
$router->get('/users', function() {
	echo 'users';
});
$router->post('/users', 			function() { /* kods */});
$router->put('/users/update/@id', 	function() { /* kods */});
$router->delete('/users/@id', 		function() { /* kods */});
$router->patch('/users/@id', 		function() { /* kods */});
```

### Regulārās izteiksmes (Regex)
Maršrutos varat izmantot regulārās izteiksmes:

```php
Flight::route('/user/[0-9]+', function () {
  // Šis atbilst /user/1234
});
```

Lai gan šī metode ir pieejama, ieteicams izmantot nosauktus parametrus vai nosauktus parametrus ar regulārajām izteiksmēm, jo tie ir lasāmāki un vieglāk uzturami.

### Nosauktie parametri
Maršrutos varat norādīt nosauktus parametrus, kas tiks nodoti jūsu atzvanīšanas funkcijai. **Tas vairāk ir paredzēts maršruta lasāmībai nekā kam citam. Lūdzu, skatiet sadaļu zemāk par svarīgu piezīmi.**

```php
Flight::route('/@name/@id', function (string $name, string $id) {
  echo "hello, $name ($id)!";
});
```

Varat iekļaut arī regulārās izteiksmes ar saviem nosauktajiem parametriem, izmantojot atdalītāju `:`:

```php
Flight::route('/@name/@id:[0-9]{3}', function (string $name, string $id) {
  // Šis atbilst /bob/123
  // Bet neatbilst /bob/12345
});
```

> **Piezīme:** Regulārās izteiksmes grupu `()` saskaņošana ar pozicionālajiem parametriem netiek atbalstīta. Piemēram: `:'\(`

#### Svarīga piezīme

Lai gan iepriekšējā piemērā šķiet, ka `@name` ir tieši saistīts ar mainīgo `$name`, tas tā nav. Parametru secība atzvanīšanas funkcijā nosaka to, kas tiek nodots. Ja jūs apmainītu parametru secību atzvanīšanas funkcijā, mainīgie tiktu apmainīti arī. Šeit ir piemērs:

```php
Flight::route('/@name/@id', function (string $id, string $name) {
  echo "hello, $name ($id)!";
});
```

Un, ja jūs dotos uz šādu URL: `/bob/123`, izvade būtu `hello, 123 (bob)!`. 
_Esiet uzmanīgi_, kad veidojat savus maršrutus un atzvanīšanas funkcijas!

### Neobligātie parametri
Varat norādīt nosauktus parametrus, kas nav obligāti saskaņošanai, iekļaujot segmentus iekavās.

```php
Flight::route(
  '/blog(/@year(/@month(/@day)))',
  function(?string $year, ?string $month, ?string $day) {
    // Šis atbilst šādiem URL:
    // /blog/2012/12/10
    // /blog/2012/12
    // /blog/2012
    // /blog
  }
);
```

Jebkuri neobligātie parametri, kas netiek saskaņoti, tiks nodoti kā `NULL`.

### Aizstājējzīmju maršrutēšana
Saskaņošana tiek veikta tikai atsevišķiem URL segmentiem. Ja vēlaties saskaņot vairākus segmentus, varat izmantot aizstājējzīmi `*`.

```php
Flight::route('/blog/*', function () {
  // Šis atbilst /blog/2000/02/01
});
```

Lai novirzītu visus pieprasījumus uz vienu atzvanīšanu, varat rīkoties šādi:

```php
Flight::route('*', function () {
  // Dariet kaut ko
});
```

### 404 Nav atrasts apstrādātājs

Pēc noklusējuma, ja URL nevar atrast, Flight nosūtīs `HTTP 404 Not Found` atbildi, kas ir ļoti vienkārša un parasta.
Ja vēlaties pielāgotāku 404 atbildi, varat [kartēt](/learn/extending) savu `notFound` metodi:

```php
Flight::map('notFound', function() {
	$url = Flight::request()->url;

	// Varat arī izmantot Flight::render() ar pielāgotu veidni.
    $output = <<<HTML
		<h1>Mana pielāgotā 404 Nav atrasts</h1>
		<h3>Lapa, kuru pieprasījāt ({$url}), netika atrasta.</h3>
		HTML;

	$this->response()
		->clearBody()
		->status(404)
		->write($output)
		->send();
});
```

### Metode nav atrasta apstrādātājs

Pēc noklusējuma, ja URL tiek atrasts, bet metode nav atļauta, Flight nosūtīs `HTTP 405 Method Not Allowed` atbildi, kas ir ļoti vienkārša un parasta (piem., Method Not Allowed. Allowed Methods are: GET, POST). Tā arī iekļaus `Allow` galveni ar atļautajām metodēm šim URL.

Ja vēlaties pielāgotāku 405 atbildi, varat [kartēt](/learn/extending) savu `methodNotFound` metodi:

```php
use flight\net\Route;

Flight::map('methodNotFound', function(Route $route) {
	$url = Flight::request()->url;
	$methods = implode(', ', $route->methods);

	// Varat arī izmantot Flight::render() ar pielāgotu veidni.
	$output = <<<HTML
		<h1>Mana pielāgotā 405 Metode nav atļauta</h1>
		<h3>Metode, kuru pieprasījāt ({$url}), nav atļauta.</h3>
		<p>Atļautās metodes ir: {$methods}</p>
		HTML;

	$this->response()
		->clearBody()
		->status(405)
		->setHeader('Allow', $methods)
		->write($output)
		->send();
});
```

## Papildu lietošana

### Atkarību ievadīšana maršrutos
Ja vēlaties izmantot atkarību ievadīšanu caur konteineru (PSR-11, PHP-DI, Dice utt.), vienīgais maršrutu veids, kur tas ir pieejams, ir vai nu pašam tieši izveidot objektu un izmantot konteineru sava objekta izveidei, vai arī varat izmantot virknes, lai definētu klasi un izsaucamo metodi. Plašāku informāciju skatiet lapā [Atkarību ievadīšana](/learn/dependency-injection-container).

Šeit ir ātrs piemērs:

```php

use flight\database\SimplePdo;

// Greeting.php
class Greeting
{
	protected SimplePdo $db;
	public function __construct(SimplePdo $db) {
		$this->db = $db;
	}

	public function hello(int $id) {
		// darīt kaut ko ar $this->db
		$name = $this->db->fetchField("SELECT name FROM users WHERE id = ?", [ $id ]);
		echo "Hello, world! My name is {$name}!";
	}
}

// index.php

// Iestatiet konteineru ar nepieciešamajiem parametriem
// Skatiet lapu Par atkarību ievadīšanu, lai iegūtu vairāk informācijas par PSR-11
$dice = new \Dice\Dice();

// Neaizmirstiet pārdefinēt mainīgo ar '$dice = '!!!!!
$dice = $dice->addRule(SimplePdo::class, [
	'shared' => true,
	'constructParams' => [ 
		'mysql:host=localhost;dbname=test', 
		'root',
		'password'
	]
]);

// Reģistrējiet konteinera apstrādātāju
Flight::registerContainerHandler(function($class, $params) use ($dice) {
	return $dice->create($class, $params);
});

// Maršruti kā parasti
Flight::route('/hello/@id', [ 'Greeting', 'hello' ]);
// vai
Flight::route('/hello/@id', 'Greeting->hello');
// vai
Flight::route('/hello/@id', 'Greeting::hello');

Flight::start();
```

### Izpildes nodošana nākamajam maršrutam
<span class="badge bg-warning">Novecojis</span>
Varat nodot izpildi nākamajam atbilstošajam maršrutam, atgriežot `true` no savas atzvanīšanas funkcijas.

```php
Flight::route('/user/@name', function (string $name) {
  // Pārbaudiet kādu nosacījumu
  if ($name !== "Bob") {
    // Turpiniet uz nākamo maršrutu
    return true;
  }
});

Flight::route('/user/*', function () {
  // Šis tiks izsaukts
});
```

Tagad ieteicams izmantot [starpprogrammatūru](/learn/middleware), lai apstrādātu sarežģītus gadījumus, piemēram, šo.

### Maršruta aizstājvārdi
Piešķirot maršrutam aizstājvārdu, vēlāk varat šo aizstājvārdu dinamiski izsaukt savā lietotnē, lai to ģenerētu vēlāk kodā (piem., saite HTML veidnē vai pāradresācijas URL ģenerēšana).

```php
Flight::route('/users/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
// vai 
Flight::route('/users/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');

// vēlāk kaut kur kodā
class UserController {
	public function update() {

		// kods lietotāja saglabāšanai...
		$id = $user['id']; // piemēram, 5

		$redirectUrl = Flight::getUrl('user_view', [ 'id' => $id ]); // atgriezīs '/users/5'
		Flight::redirect($redirectUrl);
	}
}

```

Tas ir īpaši noderīgi, ja jūsu URL gadās mainīties. Iepriekšējā piemērā pieņemsim, ka lietotāji tika pārvietoti uz `/admin/users/@id` vietā.
Ar aizstājvārdu maršrutam jums vairs nav jāatrod visi vecie URL savā kodā un jāmaina tie, jo aizstājvārds tagad atgriezīs `/admin/users/5`, kā iepriekšējā piemērā.

Maršruta aizstājvārdi darbojas arī grupās:

```php
Flight::group('/users', function() {
    Flight::route('/@id', function($id) { echo 'user:'.$id; }, false, 'user_view');
	// vai
	Flight::route('/@id', function($id) { echo 'user:'.$id; })->setAlias('user_view');
});
```

### Maršruta informācijas apskate
Ja vēlaties apskatīt atbilstošā maršruta informāciju, to var izdarīt divos veidos:

1. Varat izmantot rekvizītu `executedRoute` uz `Flight::router()` objekta.
2. Varat pieprasīt, lai maršruta objekts tiktu nodots jūsu atzvanīšanai, nododot `true` kā trešo parametru maršruta metodē. Maršruta objekts vienmēr būs pēdējais parametrs, kas tiek nodots jūsu atzvanīšanas funkcijai.

#### `executedRoute`
```php
Flight::route('/', function() {
  $route = Flight::router()->executedRoute;
  // Dariet kaut ko ar $route
  // Saskaņoto HTTP metožu masīvs
  $route->methods;

  // Nosaukto parametru masīvs
  $route->params;

  // Atbilstošā regulārā izteiksme
  $route->regex;

  // Satur jebkura '*' saturu, kas izmantots URL modelī
  $route->splat;

  // Parāda URL ceļu... ja jums tiešām tas ir nepieciešams
  $route->pattern;

  // Parāda, kāda starpprogrammatūra ir piešķirta šim
  $route->middleware;

  // Parāda šim maršrutam piešķirto aizstājvārdu
  $route->alias;
});
```

> **Piezīme:** Rekvizīts `executedRoute` tiks iestatīts tikai pēc tam, kad maršruts ir izpildīts. Ja mēģināsiet tam piekļūt pirms maršruta izpildes, tas būs `NULL`. Varat izmantot `executedRoute` arī [starpprogrammatūrā](/learn/middleware)!

#### Nododiet `true` maršruta definīcijā
```php
Flight::route('/', function(\flight\net\Route $route) {
  // Saskaņoto HTTP metožu masīvs
  $route->methods;

  // Nosaukto parametru masīvs
  $route->params;

  // Atbilstošā regulārā izteiksme
  $route->regex;

  // Satur jebkura '*' saturu, kas izmantots URL modelī
  $route->splat;

  // Parāda URL ceļu... ja jums tiešām tas ir nepieciešams
  $route->pattern;

  // Parāda, kāda starpprogrammatūra ir piešķirta šim
  $route->middleware;

  // Parāda šim maršrutam piešķirto aizstājvārdu
  $route->alias;
}, true);// <-- Šis true parametrs to nodrošina
```

### Maršrutu grupēšana un starpprogrammatūra
Var būt gadījumi, kad vēlaties grupēt saistītus maršrutus (piemēram, `/api/v1`).
To var izdarīt, izmantojot metodi `group`:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Atbilst /api/v1/users
  });

  Flight::route('/posts', function () {
	// Atbilst /api/v1/posts
  });
});
```

Varat pat ligzdot grupu grupas:

```php
Flight::group('/api', function () {
  Flight::group('/v1', function () {
	// Flight::get() iegūst mainīgos, tas neiestata maršrutu! Skatiet objekta kontekstu zemāk
	Flight::route('GET /users', function () {
	  // Atbilst GET /api/v1/users
	});

	Flight::post('/posts', function () {
	  // Atbilst POST /api/v1/posts
	});

	Flight::put('/posts/1', function () {
	  // Atbilst PUT /api/v1/posts
	});
  });
  Flight::group('/v2', function () {

	// Flight::get() iegūst mainīgos, tas neiestata maršrutu! Skatiet objekta kontekstu zemāk
	Flight::route('GET /users', function () {
	  // Atbilst GET /api/v2/users
	});
  });
});
```

#### Grupēšana ar objekta kontekstu

Varat joprojām izmantot maršrutu grupēšanu ar `Engine` objektu šādā veidā:

```php
$app = Flight::app();

$app->group('/api/v1', function (Router $router) {

  // izmantojiet $router mainīgo
  $router->get('/users', function () {
	// Atbilst GET /api/v1/users
  });

  $router->post('/posts', function () {
	// Atbilst POST /api/v1/posts
  });
});
```

> **Piezīme:** Šī ir ieteicamā metode maršrutu un grupu definēšanai ar `$router` objektu.

#### Grupēšana ar starpprogrammatūru

Varat arī piešķirt starpprogrammatūru maršrutu grupai:

```php
Flight::group('/api/v1', function () {
  Flight::route('/users', function () {
	// Atbilst /api/v1/users
  });
}, [ MyAuthMiddleware::class ]); // vai [ new MyAuthMiddleware() ], ja vēlaties izmantot instanci
```

Sīkāku informāciju skatiet lapā [grupas starpprogrammatūra](/learn/middleware#grouping-middleware).

### Resursu maršrutēšana
Varat izveidot maršrutu kopu resursam, izmantojot metodi `resource`. Tas izveidos maršrutu kopu resursam, kas atbilst RESTful konvencijām.

Lai izveidotu resursu, rīkojieties šādi:

```php
Flight::resource('/users', UsersController::class);
```

Un fonā tiks izveidoti šādi maršruti:

```php
[
      'index' => 'GET /users',
      'create' => 'GET /users/create',
      'store' => 'POST /users',
      'show' => 'GET /users/@id',
      'edit' => 'GET /users/@id/edit',
      'update' => 'PUT /users/@id',
      'destroy' => 'DELETE /users/@id'
]
```

Un jūsu kontrolieris izmantos šādas metodes:

```php
class UsersController
{
    public function index(): void
    {
    }

    public function show(string $id): void
    {
    }

    public function create(): void
    {
    }

    public function store(): void
    {
    }

    public function edit(string $id): void
    {
    }

    public function update(string $id): void
    {
    }

    public function destroy(string $id): void
    {
    }
}
```

> **Piezīme:** Jūs varat apskatīt jaunpievienotos maršrutus ar `runway`, izpildot `php runway routes`.

#### Resursu maršrutu pielāgošana

Ir dažas iespējas, kā konfigurēt resursu maršrutus.

##### Aizstājvārda bāze

Varat konfigurēt `aliasBase`. Pēc noklusējuma aizstājvārds ir pēdējā norādītā URL daļa.
Piemēram, `/users/` rezultātā `aliasBase` būs `users`. Kad šie maršruti tiek izveidoti, aizstājvārdi ir `users.index`, `users.create` utt. Ja vēlaties mainīt aizstājvārdu, iestatiet `aliasBase` uz vēlamo vērtību.

```php
Flight::resource('/users', UsersController::class, [ 'aliasBase' => 'user' ]);
```

##### Only un Except

Varat arī norādīt, kurus maršrutus vēlaties izveidot, izmantojot opcijas `only` un `except`.

```php
// Iekļaujiet tikai šīs metodes un izslēdziet pārējās
Flight::resource('/users', UsersController::class, [ 'only' => [ 'index', 'show' ] ]);
```

```php
// Izslēdziet tikai šīs metodes un iekļaujiet pārējās
Flight::resource('/users', UsersController::class, [ 'except' => [ 'create', 'store', 'edit', 'update', 'destroy' ] ]);
```

Tās būtībā ir iekļaušanas un izslēgšanas opcijas, lai jūs varētu norādīt, kurus maršrutus vēlaties izveidot.

##### Starpprogrammatūra

Varat arī norādīt starpprogrammatūru, kas jāpalaiž katram maršrutam, kas izveidots ar `resource` metodi.

```php
Flight::resource('/users', UsersController::class, [ 'middleware' => [ MyAuthMiddleware::class ] ]);
```

### Straumēšanas atbildes

Tagad varat straumēt atbildes klientam, izmantojot `stream()` vai `streamWithHeaders()`. 
Tas ir noderīgi, lai nosūtītu lielus failus, ilgstošus procesus vai ģenerētu lielas atbildes. Maršruta straumēšana tiek apstrādāta nedaudz savādāk nekā parastais maršruts.

> **Piezīme:** Straumēšanas atbildes ir pieejamas tikai tad, ja [`flight.v2.output_buffering`](/learn/migrating-to-v3#output_buffering) ir iestatīts uz `false`.

#### Straumēšana ar manuālām galvenēm

Varat straumēt atbildi klientam, izmantojot metodi `stream()` maršrutā. Ja to darāt, jums pašam jāiestata visas galvenes pirms jebko izvadāt klientam.
Tas tiek darīts ar PHP funkciju `header()` vai `Flight::response()->setRealHeader()` metodi.

```php
Flight::route('/@filename', function($filename) {

	$response = Flight::response();

	// protams, jūs sanitizētu ceļu un tamlīdzīgi.
	$fileNameSafe = basename($filename);

	// Ja jums ir papildu galvenes, kas jāiestata pēc maršruta izpildes,
	// tās jādefinē pirms jebkas tiek izvadīts.
	// Tām visām jābūt tiešam izsaukumam uz header() funkciju vai
	// izsaukumam uz Flight::response()->setRealHeader()
	header('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');
	// vai
	$response->setRealHeader('Content-Disposition: attachment; filename="'.$fileNameSafe.'"');

	$filePath = '/some/path/to/files/'.$fileNameSafe;

	if (!is_readable($filePath)) {
		Flight::halt(404, 'File not found');
	}

	// manuāli iestatiet satura garumu, ja vēlaties
	header('Content-Length: '.filesize($filePath));
	// vai
	$response->setRealHeader('Content-Length: '.filesize($filePath));

	// Straumējiet failu klientam, kamēr tas tiek lasīts
	readfile($filePath);

// Šī ir burvju rindiņa šeit
})->stream();
```

#### Straumēšana ar galvenēm

Varat arī izmantot metodi `streamWithHeaders()`, lai iestatītu galvenes pirms straumēšanas sākšanas.

```php
Flight::route('/stream-users', function() {

	// šeit varat pievienot jebkuras papildu galvenes, kuras vēlaties
	// tikai jāizmanto header() vai Flight::response()->setRealHeader()

	// neatkarīgi no tā, kā iegūstat savus datus, tikai kā piemērs...
	$users_stmt = Flight::db()->query("SELECT id, first_name, last_name FROM users");

	echo '{';
	$user_count = count($users);
	while($user = $users_stmt->fetch(PDO::FETCH_ASSOC)) {
		echo json_encode($user);
		if(--$user_count > 0) {
			echo ',';
		}

		// Tas ir nepieciešams, lai nosūtītu datus klientam
		ob_flush();
	}
	echo '}';

// Šādi jūs iestatīsiet galvenes pirms straumēšanas sākšanas.
})->streamWithHeaders([
	'Content-Type' => 'application/json',
	'Content-Disposition' => 'attachment; filename="users.json"',
	// neobligāts statusa kods, pēc noklusējuma 200
	'status' => 200
]);
```

## Skatīt arī
- [Starpprogrammatūra](/learn/middleware) - Starpprogrammatūras izmantošana ar maršrutiem autentifikācijai, žurnālfailiem utt.
- [Atkarību ievadīšana](/learn/dependency-injection-container) - Objektu izveides un pārvaldības vienkāršošana maršrutos.
- [Kāpēc ietvars?](/learn/why-frameworks) - Izpratne par tāda ietvara kā Flight izmantošanas priekšrocībām.
- [Paplašināšana](/learn/extending) - Kā paplašināt Flight ar savu funkcionalitāti, ieskaitot `notFound` metodi.
- [php.net: preg_match](https://www.php.net/manual/en/function.preg-match.php) - PHP funkcija regulāro izteiksmju saskaņošanai.

## Problēmu novēršana
- Maršruta parametri tiek saskaņoti pēc secības, nevis pēc nosaukuma. Pārliecinieties, ka atzvanīšanas funkcijas parametru secība atbilst maršruta definīcijai.
- `Flight::get()` lietošana nedefinē maršrutu; maršrutēšanai izmantojiet `Flight::route('GET /...')` vai Router objekta kontekstu grupās (piem., `$router->get(...)`).
- Rekvizīts `executedRoute` tiek iestatīts tikai pēc maršruta izpildes; pirms izpildes tas ir `NULL`.
- Straumēšanai ir jāatspējo Flight mantotā izejas buferizācijas funkcionalitāte (`flight.v2.output_buffering = false`).
- Atkarību ievadīšanai tikai dažas maršruta definīcijas atbalsta konteinerā balstītu instancēšanu.

### 404 Nav atrasts vai negaidīta maršruta darbība

Ja redzat 404 Nav atrasts kļūdu (bet jūs zvērat uz savu dzīvību, ka tā tur tiešām ir un tā nav drukas kļūda), tas faktiski var būt problēma ar to, ka maršruta galapunktā atgriežat vērtību, nevis vienkārši to izvadāt. Iemesls tam ir apzināts, bet tas var pārsteigt dažus izstrādātājus.

```php
Flight::route('/hello', function(){
	// Tas var izraisīt 404 Nav atrasts kļūdu
	return 'Hello World';
});

// Tas, ko jūs, iespējams, vēlaties
Flight::route('/hello', function(){
	echo 'Hello World';
});
```

Iemesls tam ir īpašs mehānisms, kas iebūvēts maršrutētājā un kas apstrādā atgriezto izvadi kā signālu "doties uz nākamo maršrutu". 
Šo darbību varat redzēt dokumentētu sadaļā [Maršrutēšana](/learn/routing#passing).

## Izmaiņu žurnāls
- v3: Pievienota resursu maršrutēšana, maršruta aizstājvārdi un straumēšanas atbalsts, maršrutu grupas un starpprogrammatūras atbalsts.
- v1: Lielākā daļa pamata funkciju ir pieejamas.