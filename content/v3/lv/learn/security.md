# Drošība

## Pārskats

Drošība ir ļoti svarīga tīmekļa lietojumprogrammām. Jūs vēlaties pārliecināties, ka jūsu lietojumprogramma ir droša un ka jūsu lietotāju dati ir 
aizsargāti. Flight nodrošina vairākas funkcijas, kas palīdz nodrošināt jūsu tīmekļa lietojumprogrammu drošību.

## Izpratne

Ir vairākas izplatītas drošības draudus, par kurām jums būtu jāzina, veidojot tīmekļa lietojumprogrammas. Daži no visbiežāk sastopamajiem draudiem
ietver:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

[Veidnes](/learn/templates) palīdz novērst XSS, noklusēti izvadot aizbēgušus datus, tāpēc jums nav par to jādomā. [Sesijas](/awesome-plugins/session) var palīdzēt ar CSRF, saglabājot CSRF marķieri lietotāja sesijā, kā aprakstīts zemāk. Sagatavotu vaicājumu izmantošana ar PDO palīdz novērst SQL injekcijas uzbrukumus (vai izmantojot ērtas metodes [PdoWrapper](/learn/pdo-wrapper) klasē). CORS var apstrādāt ar vienkāršu āķi pirms `Flight::start()` izsaukšanas.

Visas šīs metodes darbojas kopā, lai palīdzētu uzturēt jūsu tīmekļa lietojumprogrammas drošību. Jums vienmēr jāpatur prātā, ka jāapgūst un jāsaprot drošības labākā prakse.

## Pamata lietošana

### Galvenes

HTTP galvenes ir viens no vienkāršākajiem veidiem, kā nodrošināt jūsu tīmekļa lietojumprogrammu drošību. Jūs varat izmantot galvenes, lai novērstu klikšķu nolaupīšanu, XSS un citus uzbrukumus. 
Ir vairāki veidi, kā pievienot šīs galvenes savai lietojumprogrammai.

Divas lieliskas tīmekļa vietnes, lai pārbaudītu galveņu drošību, ir [securityheaders.com](https://securityheaders.com/) un 
[observatory.mozilla.org](https://observatory.mozilla.org/). Pēc zemāk esošā koda iestatīšanas varat viegli pārbaudīt, vai jūsu galvenes darbojas, izmantojot šīs divas vietnes.

#### Pievienot manuāli

Jūs varat manuāli pievienot šīs galvenes, izmantojot `header` metodi `Flight\Response` objektā.
```php
// Iestatīt X-Frame-Options galveni, lai novērstu klikšķu nolaupīšanu
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Iestatīt Content-Security-Policy galveni, lai novērstu XSS
// Piezīme: šī galvene var kļūt ļoti sarežģīta, tāpēc jums vajadzēs
//  konsultēties ar piemēriem internetā savai lietojumprogrammai
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Iestatīt X-XSS-Protection galveni, lai novērstu XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Iestatīt X-Content-Type-Options galveni, lai novērstu MIME sniffingu
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Iestatīt Referrer-Policy galveni, lai kontrolētu, cik daudz referrer informācijas tiek nosūtīts
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Iestatīt Strict-Transport-Security galveni, lai piespiestu HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Iestatīt Permissions-Policy galveni, lai kontrolētu, kādas funkcijas un API var izmantot
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Tās var pievienot `routes.php` vai `index.php` failu augšdaļā.

#### Pievienot kā filtru

Jūs varat arī pievienot tās filtrā/āķī šādi: 

```php
// Pievienot galvenes filtrā
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

#### Pievienot kā starpprogrammatūru

Jūs varat arī pievienot tās kā starpprogrammatūras klasi, kas nodrošina vislielāko elastību, kurām maršrutiem to lietot. Parasti šīs galvenes jāpielieto visām HTML un API atbildēm.

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

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
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php vai kur jums ir jūsu maršruti
// FYI, šī tukšā virkne grupa darbojas kā globāla starpprogrammatūra visiem
// maršrutiem. Protams, jūs varētu darīt to pašu un tikai pievienot
// to tikai konkrētiem maršrutiem.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// vairāk maršrutu
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) ir uzbrukuma veids, kurā ļaunprātīga tīmekļa vietne var likt lietotāja pārlūkprogrammai nosūtīt pieprasījumu uz jūsu vietni. 
To var izmantot, lai veiktu darbības jūsu vietnē bez lietotāja ziņas. Flight nenodrošina iebūvētu CSRF aizsardzības 
mehānismu, bet jūs varat viegli ieviest savu, izmantojot starpprogrammatūru.

#### Iestatīšana

Vispirms jums jāģenerē CSRF marķieris un jāglabā tas lietotāja sesijā. Pēc tam jūs varat izmantot šo marķieri savās formās un pārbaudīt to, kad 
forma tiek iesniegta. Mēs izmantosim [flightphp/session](/awesome-plugins/session) spraudni sesiju pārvaldībai.

```php
// Ģenerēt CSRF marķieri un saglabāt to lietotāja sesijā
// (pieņemot, ka esat izveidojis sesijas objektu un pievienojis to Flight)
// skatiet sesijas dokumentāciju, lai iegūtu vairāk informācijas
Flight::register('session', flight\Session::class);

// Jums jāģenerē tikai viens marķieris vienā sesijā (tā tas darbojas 
// vairākās cilnēs un pieprasījumos vienam lietotājam)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Izmantojot noklusēto PHP Flight veidni

```html
<!-- Izmantot CSRF marķieri savā formā -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- citi formas lauki -->
</form>
```

##### Izmantojot Latte

Jūs varat arī iestatīt pielāgotu funkciju, lai izvadītu CSRF marķieri savās Latte veidnēs.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// citas konfigurācijas...

	// Iestatīt pielāgotu funkciju, lai izvadītu CSRF marķieri
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

Un tagad savās Latte veidnēs varat izmantot `csrf()` funkciju, lai izvadītu CSRF marķieri.

```html
<form method="post">
	{csrf()}
	<!-- citi formas lauki -->
</form>
```

#### Pārbaudīt CSRF marķieri

Jūs varat pārbaudīt CSRF marķieri, izmantojot vairākas metodes.

##### Starpprogrammatūra

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

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

// index.php vai kur jums ir jūsu maršruti
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// vairāk maršrutu
}, [ CsrfMiddleware::class ]);
```

##### Notikumu filtri

```php
// Šī starpprogrammatūra pārbauda, vai pieprasījums ir POST pieprasījums un, ja tā ir, pārbauda, vai CSRF marķieris ir derīgs
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// uztvert csrf marķieri no formas vērtībām
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// vai JSON atbildei
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) ir uzbrukuma veids, kurā ļaunprātīga formas ievade var injicēt kodu jūsu vietnē. Lielākā daļa šo iespēju nāk 
no formas vērtībām, kuras aizpildīs jūsu galalietotāji. Jums **nekad** nevajadzētu uzticēties izvadei no lietotājiem! Vienmēr pieņemiet, ka visi no tiem ir 
labākie hakeri pasaulē. Viņi var injicēt ļaunprātīgu JavaScript vai HTML savā lapā. Šo kodu var izmantot, lai zagtu informāciju no jūsu 
lietotājiem vai veiktu darbības jūsu vietnē. Izmantojot Flight skata klasi vai citu veidņu dzinēju, piemēram, [Latte](/awesome-plugins/latte), jūs varat viegli aizbēgt izvadi, lai novērstu XSS uzbrukumus.

```php
// Pieņemsim, ka lietotājs ir gudrs un mēģina izmantot šo kā savu vārdu
$name = '<script>alert("XSS")</script>';

// Tas aizbēgs izvadi
Flight::view()->set('name', $name);
// Tas izvadīs: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Ja izmantojat kaut ko līdzīgu Latte, kas reģistrēts kā jūsu skata klase, tas arī automātiski aizbēgs šo.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection ir uzbrukuma veids, kurā ļaunprātīgs lietotājs var injicēt SQL kodu jūsu datubāzē. To var izmantot, lai zagtu informāciju 
no jūsu datubāzes vai veiktu darbības jūsu datubāzē. Atkal jums **nekad** nevajadzētu uzticēties ievadei no lietotājiem! Vienmēr pieņemiet, ka viņi ir 
uz asiņu medībām. Jūs varat izmantot sagatavotus vaicājumus savos `PDO` objektos, kas novērsīs SQL injekciju.

```php
// Pieņemot, ka jums ir Flight::db() reģistrēts kā jūsu PDO objekts
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// Ja izmantojat PdoWrapper klasi, to var viegli izdarīt vienā rindā
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// To pašu varat darīt ar PDO objektu ar ? vietas aizstājējiem
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### Nedrošs piemērs

Zemāk redzamais ir iemesls, kāpēc mēs izmantojam SQL sagatavotus vaicājumus, lai aizsargātu no nevainīgiem piemēriem, piemēram, zemāk redzamā:

```php
// galalietotājs aizpilda tīmekļa formu.
// formas vērtībai hakeris ievada kaut ko līdzīgu šim:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Pēc vaicājuma izveides tas izskatās šādi
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Tas izskatās dīvaini, bet tas ir derīgs vaicājums, kas darbosies. Patiesībā,
// tas ir ļoti izplatīts SQL injekcijas uzbrukums, kas atgriezīs visus lietotājus.

var_dump($users); // tas izmetīs visus lietotājus datubāzē, ne tikai vienu konkrēto lietotājvārdu
```

### JSONP Callback Validation

Ja izmantojat Flight `Flight::jsonp()` metodi, ņemiet vērā, ka Flight validē JSONP callback parametra nosaukumu pret stingru atļauto sarakstu regulāro izteiksmi (`/^[A-Za-z_$][\w$.]{0,127}$/`). Jebkurš callback nosaukums, kas neatbilst šim modelim, liks Flight izraisīt izņēmumu, novēršot patvaļīga JavaScript injicēšanu caur ļaunprātīgu callback vērtību.

Šī validācija ir iebūvēta un neprasa papildu konfigurāciju, bet par to ir vērts zināt, kad atkļūdojat negaidītas kļūdas no JSONP galapunktiem.

### CORS

Cross-Origin Resource Sharing (CORS) ir mehānisms, kas ļauj daudziem resursiem (piem., fontiem, JavaScript utt.) tīmekļa lapā tikt 
pieprasītiem no cita domēna ārpus domēna, no kura resursu izcelsme. Flight nav iebūvētas funkcionalitātes, 
bet to var viegli apstrādāt ar āķi, kas jāizpilda pirms `Flight::start()` metodes izsaukšanas.

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
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
		// pielāgojiet savus atļautos hostus šeit.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php vai kur jums ir jūsu maršruti
$CorsUtil = new CorsUtil();

// Tas jāizpilda pirms start izsaukšanas.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Flight konfigurācijas nostiprināšana

Flight atklāj vairākus dzinēja iestatījumus, kuriem ir tieša drošības ietekme. To pareiza iestatīšana ir viens no vienkāršākajiem veidiem, kā nostiprināt jūsu lietojumprogrammu.

#### `flight.allow_method_override`

Pēc noklusējuma Flight ļauj klientiem ignorēt HTTP metodes pieprasījumu, izmantojot vai nu `X-HTTP-Method-Override` galveni vai `_method` lauku POST ķermenī. Lai gan tas ir ērti HTML formām, kas var sūtīt tikai `GET`/`POST`, tas var būt bīstami, ja jūs to negaidāt — uzbrucējs var viltot `DELETE` vai `PUT` pieprasījumus caur parasto formu.

Ja jūsu lietojumprogramma nepaļaujas uz šo uzvedību (piem., jūs veidojat API, ko patērē mūsdienu klienti vai JavaScript priekšgala, kas var sūtīt jebkuru HTTP darbības vārdu), jums vajadzētu to atspējot:

```php
// Savā index.php vai bootstrap failā, pirms Flight::start()
Flight::set('flight.allow_method_override', false);
```

Noklusējuma vērtība ir `true` atpakaļsavietojamībai, bet **tā iestatīšana uz `false` ir stingri ieteicama** jebkurai lietojumprogrammai, kurai nav skaidri nepieciešama ignorēšanas funkcija.

#### `flight.debug`

Flight ir `flight.debug` iestatījums, kas kontrolē, vai detalizēta kļūdas informācija (izņēmuma ziņojums, kods un pilns steka izsekojums) tiek renderēta pārlūkprogrammā, kad rodas neapstrādāts izņēmums. Noklusējums ir `false`, kas nozīmē, ka tiek rādīts tikai vispārīgs `500 Internal Server Error` ziņojums — klienta pusei netiek nopludināta iekšējā informācija.

Nekad neiespējojiet to ražošanas serverī. Izmantojiet to tikai lokāli vai izstrādes vidē:

```php
// Droši tikai lokālai izstrādei — NEKAD ražošanā
Flight::set('flight.debug', true);
```

Kad `flight.debug` ir `false` (noklusējums), jūs joprojām varat uztvert kļūdas, iespējojot `flight.log_errors`:

```php
// Reģistrēt kļūdas servera pusē, neizpaužot tās klientam
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Ieteicamā ražošanas konfigurācija

```php
// index.php vai app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Kļūdu apstrāde
Paslēpt jutīgu kļūdu informāciju ražošanā, lai izvairītos no informācijas noplūdes uzbrucējiem. Ražošanā reģistrēt kļūdas, nevis tās parādīt, iestatot `display_errors` uz `0`.

```php
// Savā bootstrap.php vai index.php

// pievienojiet to savam app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Atspējot kļūdu parādīšanu
    ini_set('log_errors', 1);     // Reģistrēt kļūdas tā vietā
    ini_set('error_log', '/path/to/error.log');
}

// Savos maršrutos vai kontrolieros
// Izmantot Flight::halt() kontrolētām kļūdu atbildēm
Flight::halt(403, 'Access denied');
```

### Ievades sanitizācija
Nekad neuzticieties lietotāja ievadei. Sanitizējiet to, izmantojot [filter_var](https://www.php.net/manual/en/function.filter-var.php), pirms apstrādes, lai novērstu ļaunprātīgu datu iekļūšanu.

```php

// Pieņemsim $_POST pieprasījumu ar $_POST['input'] un $_POST['email']

// Sanitizēt virknes ievadi
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitizēt e-pastu
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Paroļu jaukšana
Saglabājiet paroles droši un verificējiet tās droši, izmantojot PHP iebūvētās funkcijas, piemēram, [password_hash](https://www.php.net/manual/en/function.password-hash.php) un [password_verify](https://www.php.net/manual/en/function.password-verify.php). Paroles nekad nedrīkst glabāt vienkāršā tekstā, kā arī tās nedrīkst šifrēt ar atgriezeniskām metodēm. Jaukšana nodrošina, ka pat tad, ja jūsu datubāze tiek kompromitēta, faktiskās paroles paliek aizsargātas.

```php
$password = Flight::request()->data->password;
// Jaukt paroli, kad to saglabā (piem., reģistrācijas laikā)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verificēt paroli (piem., pieteikšanās laikā)
if (password_verify($password, $stored_hash)) {
    // Parole sakrīt
}
```

### Pieprasījumu ierobežošana
Aizsargāt pret brute force uzbrukumiem vai pakalpojuma atteikuma uzbrukumiem, ierobežojot pieprasījumu ātrumus ar kešatmiņu.

```php
// Pieņemot, ka jums ir flightphp/cache instalēts un reģistrēts
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

## Skatīt arī
- [Sesijas](/awesome-plugins/session) - Kā droši pārvaldīt lietotāju sesijas.
- [Veidnes](/learn/templates) - Veidņu izmantošana, lai automātiski aizbēgtu izvadi un novērstu XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Vienkāršota datubāzes mijiedarbība ar sagatavotiem vaicājumiem.
- [Starpprogrammatūra](/learn/middleware) - Kā izmantot starpprogrammatūru, lai vienkāršotu drošības galveņu pievienošanas procesu.
- [Atbildes](/learn/responses) - Kā pielāgot HTTP atbildes ar drošām galvenēm.
- [Pieprasījumi](/learn/requests) - Kā apstrādāt un sanitizēt lietotāja ievadi.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - PHP funkcija ievades sanitizācijai.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - PHP funkcija drošai paroles jaukšanai.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - PHP funkcija jaukto paroļu verificēšanai.

## Problēmu novēršana
- Skatiet iepriekš minēto sadaļu "Skatīt arī", lai iegūtu problēmu novēršanas informāciju, kas saistīta ar Flight Framework komponentu problēmām.

## Izmaiņu žurnāls
- v3.18.1 - Pievienota Flight konfigurācijas nostiprināšanas sadaļa, kas aptver `flight.allow_method_override`, `flight.debug` un JSONP callback validāciju.
- v3.1.0 - Pievienotas sadaļas par CORS, kļūdu apstrādi, ievades sanitizāciju, paroles jaukšanu un pieprasījumu ierobežošanu.
- v2.0 - Pievienota aizbēgšana noklusētajiem skatiem, lai novērstu XSS.