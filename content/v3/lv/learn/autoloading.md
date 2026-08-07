# Automātiskā ielāde

## Pārskats

Automatiskā ielāde ir PHP koncepcija, kurā norādāt direktoriju vai direktorijas, no kurām ielādēt klases. Tas ir daudz izdevīgāk nekā izmantot `require` vai `include`, lai ielādētu klases. Tas ir arī priekšnoteikums Composer pakotņu izmantošanai.

Pareiza automātiskās ielādes iestatīšana ir svarīga arī [ar AI atbalstītu izstrādi](/learn/ai): aģenti novieto failus tur, kur norāda nosaukumvieta. Ja mapes **reģistrs** un nosaukumvietas reģistrs nesakrīt, Linux sistēmā parādās kļūdas par klases neatrašanu, pat ja lietas "strādāja" uz Mac diska, kur reģistrs netiek nošķirts.

## Izpratne

Pēc noklusējuma jebkura `Flight` klase tiek ielādēta automātiski, pateicoties Composer. **Jūsu** lietojumprogrammas klasēm ir divas izplatītas pieejas:

1. **Composer PSR-4** (ko izmanto [oficiālais skeletons](https://github.com/flightphp/skeleton)): kartējiet nosaukumvietas prefiksu uz direktoriju `composer.json` failā, pēc tam izpildiet `composer dump-autoload`.
2. **`Flight::path()`**: norādiet Flight ielādētājam direktorijas (noderīgi vienkāršām lietotnēm vai ja neizmantojat Composer lietojumprogrammas kodam).

Automatiskās ielādes izmantošana ievērojami vienkāršo jūsu kodu. Tā vietā, lai katra faila augšpusē būtu vesela siena ar `include` / `require`, klases tiek ielādētas, kad tās pirmo reizi izmantojat.

### Reģistrjutība (izlasiet šo divreiz)

**Nosaukumvietām ir jāatbilst direktoriju struktūrai *un* šo direktoriju burtu reģistram.**

| Strādā | Nedarbojas Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` ar mapi `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Ja sajauc `App\` ar mazajiem burtiem `controllers` |

PHP nosaukumvietas dažos kontekstos nav reģistrjutīgas, bet **Composer un failu sistēma ir**. Oficiālais skeletons standartizē šādi:

- Composer: `"App\\": "app/"`
- Mapes: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), nevis `controllers` / `middlewares`

Vecāki dokumenti un kopienas piemēri dažkārt izmantoja mazos burtus `app\controllers`. Tas joprojām darbojas, ja jūsu mapes ir mazajos burtos—bet **jauni skeletona projekti izmanto `App\` + PascalCase mapes**. Izvēlieties vienu konvenciju projektam un pieturieties pie tās, lai cilvēki un AI rīki neizgudrotu otru izkārtojumu.

## Skeletons (ieteicams jauniem projektiem)

Pēc `composer create-project flightphp/skeleton` komandas, lietojumprogrammas kods tiek ielādēts, izmantojot Composer—`Flight::path()` nav nepieciešams `App\` klasēm:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Dice atrisina App\Controller\… caur konteineru
$router->get('/', [HomeController::class, 'index']);
```

Skatiet [Instalācija](/install), lai redzētu pilnu koku, un [AI & izstrādātāju pieredze](/learn/ai), lai uzzinātu, kā `AGENTS.md` dokumentē šo izkārtojumu kodēšanas asistentiem.

## Pamata lietošana (`Flight::path()`)

Pieņemsim, ka mums ir šāds direktoriju koks:

```text
# Piemēra ceļš
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - satur šī projekta kontrollerus
│   ├── translations
│   ├── UTILS - satur klases tikai šai lietojumprogrammai (tas ir ar lielajiem burtiem speciāli, lai vēlāk izmantotu kā piemēru)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Jūs, iespējams, pamanījāt, ka tas ir līdzīgs tipiskam lietojumprogrammas kokam (pati dokumentācijas vietne izmanto strukturētu izkārtojumu). Mazie burti `controllers` šeit ir derīga *izvēle*—tas vienkārši nav skeletona pašreizējais noklusējums.

Jūs varat norādīt katru direktoriju, no kura ielādēt, šādi:

```php

/**
 * public/index.php
 */

// Pievienojiet ceļu automātiskajai ielādei
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// nosaukumvieta nav nepieciešama

// Visām automātiski ielādētajām klasēm ieteicams izmantot Pascal Case (katrs vārds ar lielo sākumburtu, bez atstarpēm)
class MyController {

	public function index() {
		// dariet kaut ko
	}
}
```

## Nosaukumvietas ar `Flight::path()`

Ja jums ir nosaukumvietas, to faktiski ir ļoti viegli ieviest. Jums vajadzētu izmantot `Flight::path()` metodi, lai norādītu lietojumprogrammas saknes direktoriju (nevis dokumenta sakni vai `public/` mapi).

```php

/**
 * public/index.php
 */

// Pievienojiet ceļu automātiskajai ielādei
Flight::path(__DIR__.'/../');
```

Tagad šādi varētu izskatīties jūsu kontrolieris. Apskatiet zemāk esošo piemēru, bet pievērsiet uzmanību komentāriem — tajos ir svarīga informācija.

```php
/**
 * app/controllers/MyController.php
 */

// nosaukumvietas ir obligātas
// nosaukumvietas ir tādas pašas kā direktoriju struktūra
// nosaukumvietām ir jāievēro tāds pats burtu reģistrs kā direktoriju struktūrai
// nosaukumvietās un direktorijos nevar būt apakšsvītras (ja vien nav iestatīts Loader::setV2ClassLoading(false))
namespace app\controllers;

// Visām automātiski ielādētajām klasēm ieteicams izmantot Pascal Case (katrs vārds ar lielo sākumburtu, bez atstarpēm)
// Sākot ar 3.7.2, varat izmantot Pascal_Snake_Case savu klašu nosaukumos, izpildot Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// dariet kaut ko
	}
}
```

Un, ja vēlaties automātiski ielādēt klasi savā utils direktorijā, jūs darītu pamatā to pašu:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// nosaukumvietai ir jāatbilst direktoriju struktūrai un reģistram (ņemiet vērā, ka UTILS direktorijs ir ar lielajiem burtiem
//     tāpat kā iepriekš redzamajā failu kokā)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// dariet kaut ko
	}
}
```

### Skeletona stila nosaukumvieta (tie paši noteikumi, cits reģistrs)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

Noteikumi nav mainījušies—tikai skeletona izvēlētais mapes/nosaukumvietas reģistrs. **Neatkarīgi no tā, kādu reģistru izmantojat savās mapēs, jūsu `namespace` rindai ir jāatbilst.**

## Apakšsvītras klašu nosaukumos

Sākot ar 3.7.2, varat izmantot Pascal_Snake_Case savu klašu nosaukumos, izpildot `Loader::setV2ClassLoading(false);`. 
Tas ļaus jums izmantot apakšsvītras klašu nosaukumos. 
Tas nav ieteicams, bet ir pieejams tiem, kam tas ir nepieciešams.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Pievienojiet ceļu automātiskajai ielādei
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// nosaukumvieta nav nepieciešama

class My_Controller {

	public function index() {
		// dariet kaut ko
	}
}
```

## Skatīt arī
- [Instalācija](/install) - Skeletona koks un `App\` noklusējumi jauniem projektiem.
- [Maršrutēšana](/learn/routing) - Kā kartēt maršrutus uz kontrolleriem un renderēt skatus.
- [Atkarību ievadīšana](/learn/dependency-injection-container) - Kā kontrolleri saņem `Engine` un pakalpojumus.
- [AI & izstrādātāju pieredze](/learn/ai) - Uzturiet aģentus saskaņotus ar jūsu izkārtojumu, izmantojot `AGENTS.md`.
- [Kāpēc ietvars?](/learn/why-frameworks) - Izpratne par ieguvumiem, ko sniedz tāda ietvara kā Flight izmantošana.

## Problēmu novēršana
- Ja nevarat saprast, kāpēc jūsu nosaukumvietu klases netiek atrastas, atcerieties: ar `Flight::path()` norādiet uz **projekta sakni** (vai pareizo bāzi savai nosaukumvietai), nevis tikai uz ligzdotu mapi, kuru aizmirsāt atspoguļot nosaukumvietā.
- Izmantojot Composer PSR-4, pēc `composer.json` kartējumu maiņas izpildiet `composer dump-autoload`.
- Linux CI vai ražošanas vidē nepareizs mapes reģistrs ir ļoti izplatīta "manā mašīnā tas strādā" kļūme.

### Klase nav atrasta (automātiskā ielāde nedarbojas)

Var būt vairāki iemesli, kāpēc tas nenotiek. Zemāk ir daži piemēri.

#### Nepareizs faila nosaukums

Visizplatītākais ir tas, ka klases nosaukums neatbilst faila nosaukumam.

Ja jums ir klase ar nosaukumu `MyClass`, failam ir jābūt nosauktam `MyClass.php`. Ja jums ir klase ar nosaukumu `MyClass` un fails ir nosaukts `myclass.php`, automātiskā ielāde nevarēs to atrast.

#### Nepareiza nosaukumvieta vai mapes reģistrs

Ja izmantojat nosaukumvietas, tad nosaukumvietai ir jāatbilst direktoriju struktūrai **ieskaitot reģistru**.

```php
// ...kods...

// ja jūsu MyController atrodas app/Controller (skeletons) un ir nosaukumvietā App\Controller
// tas nedarbosies:
Flight::route('/hello', 'MyController->hello');

// Skeletona stils:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Vecāks izkārtojums ar mazajiem burtiem (tikai tad, ja jūsu mapes patiešām ir app/controllers):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// vai pilnībā kvalificēts:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` nav definēts (ne-Composer lietojumprogrammas kods)

Ja paļaujaties uz `Flight::path()` nevis Composer lietojumprogrammas klasēm, definējiet ceļu pirms maršrutiem, kas atsaucas uz šīm klasēm (parasti agri bootstrap / `public/index.php`):

```php
// Pievienojiet ceļu automātiskajai ielādei (projekta sakne nosaukumvietu lietotnēm)
Flight::path(__DIR__.'/../');
```

Oficiālais skeletons galvenokārt izmanto **Composer PSR-4** `App\` klasei, tāpēc parasti jums nebūs nepieciešams `Flight::path()` kontrolleriem un modeļiem.

## Izmaiņu žurnāls
- Dokumentācija – dokumentēts skeletona `App\` + PascalCase mapju izkārtojums un reģistrjutības nepilnības cilvēkiem un AI rīkiem.
- v3.7.2 - Varat izmantot Pascal_Snake_Case savu klašu nosaukumos, izpildot `Loader::setV2ClassLoading(false);`
- v2.0 - Pievienota automātiskās ielādes funkcionalitāte.