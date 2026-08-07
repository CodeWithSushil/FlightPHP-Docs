# Tracy Flight Panel Paplašinājumi

Šis ir paplašinājumu komplekts, lai padarītu darbu ar Flight nedaudz bagātāku.

- **Flight** - Analizēt visas Flight mainīgās vērtības.
- **Database** - Analizēt visus vaicājumus, kas izpildīti lapā (ja pareizi inicializē datu bāzes savienojumu)
- **Request** - Analizēt visus `$_SERVER` mainīgos un pārbaudīt visus globālos datus (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Analizēt visus `$_SESSION` mainīgos, ja sesijas ir aktīvas.
- **Twig** *(neobligāti)* - Analizēt Twig veidnes renderēšanas laiku, atmiņu un to, kuras veidnes/bloki/makro tika izpildīti (nepieciešams `twig/twig` un `twig_profile` konfigurācija)

Tas ir īpaši noderīgi ar [oficiālo skeletu](https://github.com/flightphp/skeleton), kurš pēc noklusējuma izmanto Twig: tas pats izkārtojums [AI rīki](/learn/ai) seko arī skaidri parādās Tracy joslā.

Šis ir Panelis

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

Un katrs panelis attēlo ļoti noderīgu informāciju par jūsu aplikāciju!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Klikšķiniet [šeit](https://github.com/flightphp/tracy-extensions) lai skatītu kodu.

## Instalācija

Izpildiet `composer require flightphp/tracy-extensions --dev` un jūs esat ceļā!

Twig nav **stingra** šī pakotnes atkarība. Instalējiet `twig/twig` tikai tad, ja vēlaties Twig paneli (skelets to jau dara skatiem).

## Konfigurācija

Ir ļoti maz konfigurācijas, kas jums jāveic, lai to sāktu. Jums būs jāinicializē Tracy atkļūdotājs pirms šī lietošanas [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// bootstrap kods
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Jums var būt nepieciešams norādīt savu vidi ar Debugger::enable(Debugger::DEVELOPMENT)

// ja savā app izmantojat datu bāzes savienojumus, ir 
// nepieciešams PDO wrapper, ko izmantot TIKAI IZSTRĀDES laikā (nevis ražošanā!)
// Tam ir tie paši parametri kā parastam PDO savienojumam
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// vai ja pievienojat to Flight framework
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// tagad, kad veicat vaicājumu, tas uztvers laiku, vaicājumu un parametrus

// Tas savieno punktus
if(Debugger::$showBar === true) {
	// Tam jābūt false vai Tracy nevar pat renderēt :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// vairāk koda

Flight::start();
```

## Papildu Konfigurācija

### Sesijas Dati

Ja jums ir pielāgots sesiju apstrādātājs (piemēram, ghostff/session), varat nodot jebkuru sesiju datu masīvu Tracy, un tas automātiski to izvadīs jums. Jūs to nododat ar `session_data` atslēgu otrajā parametrā `TracyExtensionLoader` konstruktorā.

```php

use Ghostff\Session\Session;
// vai izmantot flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Tam jābūt false vai Tracy nevar pat renderēt :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// maršruti un citas lietas...

Flight::start();
```

### Twig panelis (neobligāts)

Ja jūsu app izmanto [Twig](/awesome-plugins/twig) (ieskaitot oficiālo skeletu), varat parādīt veidnes metrikas Tracy joslā. Izveidojiet Twig `Profile`, pievienojiet `ProfilerExtension` savai videi, pēc tam nododiet šo profilu ielādētājam zem **`twig_profile`** atslēgas. Pievienojiet profilēšanu tikai izstrādes laikā.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Neobligāti: atklāt Tracy dump palīgfunkcijas veidnēs
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Kartēt Flight::render() uz Twig (piemērs)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Ko panelis parāda**

- Kopējais Twig renderēšanas laiks un atmiņa
- Veidnes / bloka / makro izsaukumu skaits
- Katra veidne, kas tika renderēta, ar savu laiku un atmiņu

Twig cilne ir **paslēpta**, kad vaicājumam netika renderētas veidnes, vai kad izlaižat `twig_profile` (vai jums nav Twig instalēts) — citi Flight paneļi turpina darboties.

Skeletveida `services.php`, izveidojiet to pašu `$profile` / `ProfilerExtension`, kad atkļūdošana ir ieslēgta, nododiet `twig_profile` uz `TracyExtensionLoader`, un turpiniet izmantot savu kopīgo Twig vidi priekš `$app->render()`.

### Latte

_PHP 8.1+ ir nepieciešams šai sadaļai._

Ja jums ir Latte instalēts savā projektā, Tracy ir vietējā integrācija ar Latte, lai analizētu jūsu veidnes. Jūs vienkārši reģistrējat paplašinājumu ar savu Latte instanci (šī ir Latte paša Tracy tilts, nevis Twig panelis augšā).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// citas konfigurācijas...

	// pievienot paplašinājumu tikai tad, ja Tracy Debug josla ir ieslēgta
	if(Debugger::$showBar === true) {
		// šeit jūs pievienojat Latte Panel Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Skatīt Arī

- [Tracy](/awesome-plugins/tracy) - Tracy bāzes iestatījums Flight
- [Twig](/awesome-plugins/twig) - Veidņošana, ko izmanto skelets un Twig panelis
- [Templates](/learn/templates) - Kā Flight kartē `render` uz Twig/Latte
- [Installation](/install) - Skelets ietver tracy-extensions izstrādes laikā