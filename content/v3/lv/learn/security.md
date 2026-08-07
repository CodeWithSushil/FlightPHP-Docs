# Drošība

## Pārskats

Drošība ir liela lieta, runājot par tīmekļa lietojumprogrammām. Jūs vēlaties pārliecināties, ka jūsu lietojumprogramma ir droša un ka jūsu lietotāju dati ir drošībā. Flight piedāvā vairākas funkcijas, lai palīdzētu jums aizsargāt savas tīmekļa lietojumprogrammas.

Oficiālais [skelets](https://github.com/flightphp/skeleton) ietver arī īpašu **`SECURITY.md`** un drošības galveņu starpprogrammatūru, lai [AI kodēšanas rīkiem](/learn/ai) (un cilvēkiem) būtu viena apzināta vieta noslēpumiem, galvenēm un XSS/SQL noteikumiem — atsevišķi no vispārējā koda stila failā `AGENTS.md`.

## Izpratne

Ir vairāki izplatīti drošības apdraudējumi, par kuriem jums vajadzētu zināt, veidojot tīmekļa lietojumprogrammas. Daži no visizplatītākajiem apdraudējumiem ir:
- Starpsaites pieprasījuma viltošana (CSRF)
- Starpsaites skriptošana (XSS)
- SQL injekcija
- Cross Origin Resource Sharing (CORS)

[Veidnes](/learn/templates) palīdz pret XSS, pēc noklusējuma atdalot izvadi (Twig un Latte to dara; izmantojiet šo priekšrocību). [Sesijas](/awesome-plugins/session) var palīdzēt pret CSRF, saglabājot CSRF tokenu lietotāja sesijā, kā aprakstīts tālāk. Sagatavotu vaicājumu izmantošana ar PDO — vai [SimplePdo](/learn/simple-pdo) palīgmetodes — palīdz novērst SQL injekcijas. CORS var apstrādāt ar vienkāršu āķi pirms `Flight::start()` izsaukšanas.

Visas šīs metodes darbojas kopā, lai palīdzētu uzturēt jūsu tīmekļa lietojumprogrammas drošas. Jums vienmēr priekšplānā jābūt apgūt un izprast drošības paraugpraksi. Nelūdziet AI asistentam "atspējot CSP" vai vājināt galvenes tikai tāpēc, lai lapa ielādētos, nesaprotot kompromisu.

## Pamata lietošana

### Galvenes

HTTP galvenes ir viens no vienkāršākajiem veidiem, kā aizsargāt savas tīmekļa lietojumprogrammas. Varat izmantot galvenes, lai novērstu klikšķināšanas nolaupīšanu (clickjacking), XSS un citus uzbrukumus. Ir vairāki veidi, kā pievienot šīs galvenes savai lietojumprogrammai.

Divas lieliskas vietnes, kur pārbaudīt savu galveņu drošību, ir [securityheaders.com](https://securityheaders.com/) un [observatory.mozilla.org](https://observatory.mozilla.org/). Pēc tālāk norādītā koda iestatīšanas jūs varat viegli pārbaudīt, vai jūsu galvenes darbojas, izmantojot šīs divas vietnes.

Skelets ietver `App\Middleware\SecurityHeadersMiddleware` (CSP ar pieprasījumam atbilstošu nonce, frame opcijām, HSTS un vairāk). Dodiet priekšroku tā apzinātai paplašināšanai, nevis galveņu atspējošanai.

#### Pievienošana ar rokām

Šīs galvenes varat pievienot manuāli, izmantojot `header` metodi uz `Flight\Response` objekta.
```php
// Iestatiet X-Frame-Options galveni, lai novērstu clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Iestatiet Content-Security-Policy galveni, lai novērstu XSS
// Piezīme: šī galvene var kļūt ļoti sarežģīta, tāpēc jūs vēlēsities
//  apskatīt piemērus internetā savai lietojumprogrammai
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Iestatiet X-XSS-Protection galveni, lai novērstu XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Iestatiet X-Content-Type-Options galveni, lai novērstu MIME uzminēšanu
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Iestatiet Referrer-Policy galveni, lai kontrolētu, cik daudz referrer informācijas tiek nosūtīts
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Iestatiet Strict-Transport-Security galveni, lai piespiestu HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Iestatiet Permissions-Policy galveni, lai kontrolētu, kādas funkcijas un API var tikt izmantotas
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Tās var pievienot savu `routes.php` vai `index.php` failu augšpusē.

#### Pievienošana kā filtrs

Varat tās arī pievienot filtrā/āķī, piemēram:

```php
// Pievienojiet galvenes filtrā
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### Pievienošana kā starpprogrammatūra

Varat tās pievienot arī kā starpprogrammatūras klasi, kas nodrošina vislielāko elastību, nosakot, kuriem maršrutiem to piemērot. Parasti šīs galvenes būtu jāpiemēro visām HTML un API atbildēm.

Skeleta stila ceļš un nosaukumvieta (**mapes reģistrs atbilst `App\Middleware`**):

```php
// app/Middleware/SecurityHeadersMiddleware.php

namespace App\Middleware;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		// Dodiet priekšroku CSP nonce no bootstrap, ja jums ir iekļauti skripti (skelets iestata csp_nonce)
		$nonce = $this->app->get('csp_nonce');
		$csp = $nonce
			? "default-src 'self'; script-src 'self' 'nonce-{$nonce}'; style-src 'self' 'nonce-{$nonce}'"
			: "default-src 'self'";

		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header('Content-Security-Policy', $csp);
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// app/config/routes.php — tukšas virknes grupa = globālā starpprogrammatūra visiem maršrutiem
use App\Middleware\SecurityHeadersMiddleware;
use flight\net\Router;

$router->group('', function (Router $router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// vairāk maršrutu
}, [SecurityHeadersMiddleware::class]);
```

Vecāki projekti joprojām var izmantot `app/middlewares` un `app\middlewares`; tas darbojas, ja mapes sakrīt. Jaunās skeleta lietotnes izmanto **`app/Middleware/`** un **`App\Middleware`**. Skatīt [Automātiskā ielāde](/learn/autoloading).

### Starpsaites pieprasījuma viltošana (CSRF)

Starpsaites pieprasījuma viltošana (CSRF) ir uzbrukuma veids, kurā ļaunprātīga vietne var likt lietotāja pārlūkprogrammai nosūtīt pieprasījumu jūsu vietnei. To var izmantot, lai veiktu darbības jūsu vietnē bez lietotāja ziņas. Flight nepiedāvā iebūvētu CSRF aizsardzības mehānismu, taču jūs to viegli varat ieviest pats, izmantojot starpprogrammatūru.

#### Iestatīšana

Vispirms jums ir jāģenerē CSRF tokens un jāsaglabā tas lietotāja sesijā. Pēc tam varat izmantot šo tokenu savās veidlapās un pārbaudīt to, kad veidlapa tiek iesniegta. Mēs izmantosim [flightphp/session](/awesome-plugins/session) spraudni, lai pārvaldītu sesijas.

```php
// Ģenerējiet CSRF tokenu un saglabājiet to lietotāja sesijā
// (pieņemot, ka esat izveidojis sesijas objektu un pievienojis to Flight)
// skatiet sesijas dokumentāciju, lai iegūtu vairāk informācijas
Flight::register('session', flight\Session::class);

// Jums ir jāģenerē tikai viens tokens katrā sesijā (lai tas darbotos 
// vairākās cilnēs un pieprasījumos vienam un tam pašam lietotājam)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Izmantojot noklusējuma PHP Flight veidni

```html
<!-- Izmantojiet CSRF tokenu savā veidlapā -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- citi veidlapas lauki -->
</form>
```

##### Izmantojot Twig (skeleta noklusējums)

Reģistrējiet Twig funkciju vai nododiet tokenu katrā veidlapas skatā. Minimāls piemērs ar globālo mainīgo + veidlapas lauku:

```php
// Konfigurējot Twig (piem., services.php)
$twig->addGlobal('csrf_token', $app->session()->get('csrf_token'));
```

```html
{# app/views/form.twig #}
<form method="post">
	<input type="hidden" name="csrf_token" value="{{ csrf_token }}">
	{# citi lauki #}
</form>
```

##### Izmantojot Latte

Varat arī iestatīt pielāgotu funkciju, lai izvadītu CSRF tokenu savās Latte veidnēs.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// citas konfigurācijas...

	// Iestatiet pielāgotu funkciju, lai izvadītu CSRF tokenu
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Un tagad savās Latte veidnēs varat izmantot `csrf()` funkciju, lai izvadītu CSRF tokenu.

```html
<form method="post">
	{csrf()}
	<!-- citi veidlapas lauki -->
</form>
```

#### CSRF tokena pārbaude

CSRF tokenu var pārbaudīt vairākos veidos.

##### Starpprogrammatūra

```php
// app/Middleware/CsrfMiddleware.php

namespace App\Middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// routes.php
use App\Middleware\CsrfMiddleware;

$router->group('', function ($router) {
	$router->get('/users', [ \App\Controller\UserController::class, 'getUsers' ]);
	// vairāk maršrutu
}, [CsrfMiddleware::class]);
```

##### Notikumu filtri

```php
// Šī starpprogrammatūra pārbauda, vai pieprasījums ir POST pieprasījums, un, ja ir, tā pārbauda, vai CSRF tokens ir derīgs
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// iegūstiet csrf tokenu no veidlapas vērtībām
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// vai JSON atbildei
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Starpsaites skriptošana (XSS)

Starpsaites skriptošana (XSS) ir uzbrukuma veids, kurā ļaunprātīgs veidlapas ievads var ievietot kodu jūsu vietnē. Lielākā daļa šo iespēju rodas no veidlapas vērtībām, ko aizpilda jūsu galalietotāji. Jums **nekad** nevajadzētu uzticēties lietotāju ievadei! Vienmēr pieņemiet, ka viņi visi ir labākie hakeri pasaulē. Viņi var ievietot ļaunprātīgu JavaScript vai HTML jūsu lapā. Šo kodu var izmantot, lai nozagtu informāciju no jūsu lietotājiem vai veiktu darbības jūsu vietnē. Izmantojot Flight skatu klasi vai veidņu dzinēju, piemēram, [Twig](/awesome-plugins/twig) vai [Latte](/awesome-plugins/latte), jūs varat viegli aizsargāt izvadi, lai novērstu XSS uzbrukumus.

```php
// Pieņemsim, ka lietotājs ir viltīgs un mēģina to izmantot kā savu vārdu
$name = '<script>alert("XSS")</script>';

// Tas atdalīs izvadi (escape)
Flight::view()->set('name', $name);
// Tas izvadīs: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Twig (skeleta noklusējums) un Latte pēc noklusējuma automātiski atdala izvadi — dodiet tiem priekšroku, nevis neapstrādātam PHP echo
Flight::render('template', ['name' => $name]);
// Twig: {{ name }}  → atdalīts
// Izvairieties no |raw / neatdalītas izvades, ja vien saturs nav pilnībā uzticams
```

### SQL injekcija

SQL injekcija ir uzbrukuma veids, kurā ļaunprātīgs lietotājs var ievietot SQL kodu jūsu datubāzē. To var izmantot, lai nozagtu informāciju no jūsu datubāzes vai veiktu darbības jūsu datubāzē. Atkal jums **nekad** nevajadzētu uzticēties lietotāju ievadei! Vienmēr pieņemiet, ka viņi ir izslāpuši pēc asinīm. Izmantojiet sagatavotus vaicājumus — [SimplePdo](/learn/simple-pdo) palīgmetodes padara to par noklusējuma ceļu.

```php
// Pieņemot, ka Flight::db() ir reģistrēts kā SimplePdo (vai injicējiet SimplePdo kontrolierī)
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// SimplePdo (ieteicams) — vienas rindas vaicājumi ar piesaistītiem parametriem
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Tā pati ideja ar ? vietturiem
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

Skeleta stila kontrolieros dodiet priekšroku konstruktora injekcijai ar `SimplePdo`, nevis `Flight::db()`, lai testi un AI ģenerēts kods paliktu konsekventi ([DIC](/learn/dependency-injection-container)).

#### Nedrošs piemērs

Tālāk redzams, kāpēc mēs izmantojam SQL sagatavotus vaicājumus, lai aizsargātos no nekaitīgiem piemēriem, kā zemāk:

```php
// galalietotājs aizpilda tīmekļa veidlapu.
// kā veidlapas vērtību hakeris ievada aptuveni šo:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Pēc vaicājuma izveides tas izskatās šādi
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Tas izskatās dīvaini, bet tas ir derīgs vaicājums, kas darbosies. Patiesībā,
// tas ir ļoti izplatīts SQL injekcijas uzbrukums, kas atgriezīs visus lietotājus.

var_dump($users); // tas izdrukās visus datubāzes lietotājus, nevis tikai vienu konkrēto lietotājvārdu
```

### Noslēpumi un konfigurācija

- Ievietojiet noslēpumus **`.env`** (vai īstajā vidē), nevis iekļautajos `config.php` paraugos.
- Skeleta noteikums: burtiskās noklusējuma vērtības `config.php`; bootstrap laikā apvienojiet env; **nedariet** lasiet `$_ENV` kontrolieros — tā vietā injicējiet konfigurāciju. Skatiet [Konfigurācija](/learn/configuration).
- Nekad neiekļaujiet API atslēgas, datubāzes paroles vai sesijas šifrēšanas atslēgas. Norādiet AI rīkus uz **`SECURITY.md`**, lai tie neizdomā nedrošus īsceļus.

### JSONP atzvanīšanas validācija

Ja izmantojat Flight `Flight::jsonp()` metodi, ņemiet vērā, ka Flight pārbauda JSONP atzvanīšanas parametra nosaukumu pret stingru atļauto sarakstu ar regulāro izteiksmi (`/^[A-Za-z_$][\w$.]{0,127}$/`). Jebkurš atzvanīšanas nosaukums, kas neatbilst šim paraugam, liks Flight izmest izņēmumu, tādējādi novēršot patvaļīga JavaScript ievadīšanu caur ļaunprātīgu atzvanīšanas vērtību.

Šī validācija ir iebūvēta un neprasa papildu konfigurāciju, taču ir vērts par to zināt, atkļūdojot negaidītas kļūdas no JSONP galapunktiem.

### CORS

Cross-Origin Resource Sharing (CORS) ir mehānisms, kas ļauj daudzus resursus (piemēram, fontus, JavaScript u.c.) tīmekļa lapā pieprasīt no cita domēna ārpus domēna, no kura resurss nācis. Flight nav iebūvētas funkcionalitātes, taču to var viegli apstrādāt ar āķi, kas tiek palaists pirms `Flight::start()` metodes izsaukšanas.

```php
// app/Utils/CorsUtil.php  (skelets: PascalCase Utils mape → App\Utils)

namespace App\Utils;

use flight\Engine;

class CorsUtil
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function set(array $params = []): void
	{
		$request = $this->app->request();
		$response = $this->app->response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// pielāgojiet savus atļauto resursdatoru sarakstu šeit.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = $this->app->request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = $this->app->response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// bootstrap / maršruti — izpildīt pirms start
$app = Flight::app();
$cors = new \App\Utils\CorsUtil($app);
$app->before('start', [ $cors, 'set' ]);
```

### Flight konfigurācijas nostiprināšana

Flight atklāj vairākus dzinēja iestatījumus, kuriem ir tieša ietekme uz drošību. Pareiza šo iestatījumu konfigurēšana ir viens no vienkāršākajiem veidiem, kā nostiprināt savu lietojumprogrammu.

#### `flight.allow_method_override`

Pēc noklusējuma Flight ļauj klientiem mainīt pieprasījuma HTTP metodi, izmantojot vai nu `X-HTTP-Method-Override` galveni, vai `_method` lauku POST pamattekstā. Lai gan tas ir ērti HTML veidlapām, kas var nosūtīt tikai `GET`/`POST`, tas var būt bīstami, ja jūs to negaidāt — uzbrucējs varētu viltot `DELETE` vai `PUT` pieprasījumus caur parastu veidlapu.

Ja jūsu lietojumprogramma nepaļaujas uz šo uzvedību (piem., veidojat API, ko izmanto mūsdienīgi klienti vai JavaScript priekšgali, kas var nosūtīt jebkuru HTTP metodi), jums tas būtu jāatspējo:

```php
// Savā index.php vai bootstrap failā, pirms Flight::start()
Flight::set('flight.allow_method_override', false);
```

Noklusējuma vērtība ir `true` atpakaļsaderības dēļ, taču **ieteicams to iestatīt uz `false`** jebkurai lietojumprogrammai, kurai nav tieši nepieciešama aizstāšanas funkcija.

#### `flight.debug`

Flight ir `flight.debug` iestatījums, kas nosaka, vai detalizēta kļūdas informācija (izņēmuma ziņojums, kods un pilna steka izsekojums) tiek parādīta pārlūkprogrammā, kad rodas neapstrādāts izņēmums. Noklusējums ir `false`, kas nozīmē, ka tiek rādīts tikai vispārīgs ziņojums `500 Internal Server Error` — klientam netiek nopludinātas iekšējās detaļas.

Nekad neiespējojiet to uz produkcijas servera. Izmantojiet to tikai lokāli vai staging vidē:

```php
// Drošs tikai lokālai izstrādei — NEKAD produkcijā
Flight::set('flight.debug', true);
```

Kad `flight.debug` ir `false` (noklusējums), jūs joprojām varat reģistrēt kļūdas, iespējojot `flight.log_errors`:

```php
// Reģistrējiet kļūdas servera pusē, nepadarot tās redzamas klientam
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Ieteicamā produkcijas konfigurācija

```php
// index.php vai lietots no lietotnes konfigurācijas / bootstrap
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Kļūdu apstrāde

Slēpiet sensitīvas kļūdu detaļas produkcijas vidē, lai neizpaustu informāciju uzbrucējiem. Produkcijas vidē reģistrējiet kļūdas, nevis rādiet tās, iestatot `display_errors` uz `0`.

```php
// Savā bootstrap.php vai index.php failā

// pievienojiet to savam app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Atspējo kļūdu attēlošanu
    ini_set('log_errors', 1);     // Tā vietā reģistrē kļūdas
    ini_set('error_log', '/path/to/error.log');
}

// Savos maršrutos vai kontrolieros
// Izmantojiet Flight::halt() kontrolētām kļūdu atbildēm
Flight::halt(403, 'Access denied');
```

### Ievades sanitizācija

Nekad neuzticieties lietotāja ievadei. Sanitizējiet to, izmantojot [filter_var](https://www.php.net/manual/en/function.filter-var.php), pirms apstrādes, lai novērstu ļaunprātīgu datu iekļūšanu. Dodiet priekšroku ievades lasīšanai ar `$app->request()` (vai `Flight::request()`), nevis neapstrādātiem `$_GET` / `$_POST` lietotnes kodā.

```php

// Pieņemsim, ka ir $_POST pieprasījums ar $_POST['input'] un $_POST['email']

// Sanitizējiet virknes ievadi
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitizējiet e-pasta adresi
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Paroļu hešēšana

Glabājiet paroles droši un pārbaudiet tās drošā veidā, izmantojot PHP iebūvētās funkcijas, piemēram, [password_hash](https://www.php.net/manual/en/function.password-hash.php) un [password_verify](https://www.php.net/manual/en/function.password-verify.php). Paroles nekad nedrīkst glabāt vienkāršā tekstā, un tās nedrīkst šifrēt ar atgriezeniskām metodēm. Hešēšana nodrošina, ka pat tad, ja jūsu datubāze tiek kompromitēta, faktiskās paroles paliek aizsargātas.

```php
$password = Flight::request()->data->password;
// Hešējiet paroli, kad to saglabājat (piem., reģistrācijas laikā)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Pārbaudiet paroli (piem., pieteikšanās laikā)
if (password_verify($password, $stored_hash)) {
    // Parole atbilst
}
```

### Pieprasījumu ātruma ierobežošana

Aizsargājieties pret brutāla spēka uzbrukumiem vai pakalpojuma atteikuma uzbrukumiem, ierobežojot pieprasījumu ātrumu ar kešatmiņu.

```php
// Pieņemot, ka flightphp/cache ir instalēts un reģistrēts
// Izmantojot flightphp/cache filtrā
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Atiestatīt pēc 60 sekundēm
});
```

### Skatīt arī

- [Sesijas](/awesome-plugins/session) - Kā droši pārvaldīt lietotāju sesijas.
- [Veidnes](/learn/templates) - Twig/Latte automātiska izvades atdalīšana un XSS.
- [SimplePdo](/learn/simple-pdo) - Datubāzes palīgmetodes ar sagatavotiem vaicājumiem.
- [PdoWrapper](/learn/pdo-wrapper) - Novecojis; izmantojiet SimplePdo jaunam kodam.
- [Starpprogrammatūra](/learn/middleware) - Kā izmantot starpprogrammatūru, lai vienkāršotu drošības galveņu pievienošanas procesu.
- [Konfigurācija](/learn/configuration) - `.env` salīdzinājumā ar burtisko konfigurāciju, produkcijas karodziņi.
- [AI un izstrādātāju pieredze](/learn/ai) - Glabājiet drošības politiku `SECURITY.md` aģentiem.
- [Atbildes](/learn/responses) - Kā pielāgot HTTP atbildes ar drošām galvenēm.
- [Pieprasījumi](/learn/requests) - Kā apstrādāt un sanitizēt lietotāja ievadi.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - PHP funkcija ievades sanitizācijai.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - PHP funkcija drošai paroļu hešēšanai.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - PHP funkcija hešēto paroļu pārbaudei.

### Problēmu novēršana

- Skatiet iepriekšējo sadaļu "Skatīt arī", lai iegūtu informāciju par problēmu novēršanu saistībā ar Flight Framework komponentu jautājumiem.
- Ja CSP bloķē jūsu skriptus, pievienojiet nonce (skeleta paraugs) vai iekļaujiet atļauto sarakstā konkrētas izcelsmes — neiestatiet `script-src *` bez plāna.

### Izmaiņu žurnāls

- Dokumentācija — Skeleta `App\Middleware`, Twig CSRF/XSS piezīmes, SimplePdo, noslēpumi/`.env` un `SECURITY.md` AI draudzīgiem projektiem.
- v3.18.1 - Pievienota Flight konfigurācijas nostiprināšanas sadaļa, kas aptver `flight.allow_method_override`, `flight.debug` un JSONP atzvanīšanas validāciju.
- v3.1.0 - Pievienotas sadaļas par CORS, kļūdu apstrādi, ievades sanitizāciju, paroļu hešēšanu un pieprasījumu ātruma ierobežošanu.
- v2.0 - Pievienota izvades atdalīšana noklusējuma skatiem, lai novērstu XSS.