# HTML skati un veidnes

## Pārskats

Flight pēc noklusējuma nodrošina pamata HTML veidņu funkcionalitāti. Veidņu izmantošana ir ļoti efektīvs veids, kā atdalīt lietojumprogrammas loģiku no prezentācijas slāņa. Īpašs dzinējs (Twig, Latte utt.) arī sniedz AI kodēšanas rīkiem pazīstamu, ierobežotu sintaksi, tāpēc tie mazāk iemaisīs biznesa loģiku jūsu HTML.

## Izpratne

Veidojot lietojumprogrammu, jums, visticamāk, būs HTML, ko vēlēsities nosūtīt atpakaļ gala lietotājam. PHP pats par sevi ir veidņu valoda, bet tajā ir _ļoti_ viegli iepīt biznesa loģiku, piemēram, datubāzes izsaukumus, API izsaukumus utt., jūsu HTML failā, padarot testēšanu un atsaistīšanu par ļoti sarežģītu procesu. Ievietojot datus veidnē un ļaujot veidnei sevi atveidot, kļūst daudz vieglāk atsaistīt un vienību testēt savu kodu. Jūs mums pateiksieties, ja izmantosiet veidnes!

## Pamata lietošana

Flight ļauj nomainīt noklusējuma skatu dzinēju, vienkārši kartējot `render` (vai reģistrējot skatu klasi). Ritiniet uz leju, lai redzētu Twig, Latte, Smarty, Blade un citus.

> **Skeleton noklusējums:** Oficiālais [flightphp/skeleton](https://github.com/flightphp/skeleton) izmanto **tikai Twig** mapē `app/views/` (`*.twig`). Kontrolleri izsauc `$this->app->render('welcome', $data)` (paplašinājums nav obligāts). Tā ir lietojumprogrammas izvēle jauniem projektiem—nevis Flight kodola prasība. Latte un citi dzinēji joprojām tiek pilnībā atbalstīti.

### Twig

<span class="badge bg-info">skeleton noklusējums</span>

[Twig](https://twig.symfony.com/) ir elastīgs, ātrs un drošs veidņu dzinējs, ko izmanto Symfony un daudzi citi PHP projekti. AI kodēšanas rīki mēdz īpaši labi pārzināt Twig, un tas pēc noklusējuma automātiski izbēg izvadi, kas palīdz aizsargāties pret XSS.

#### Instalēšana

```bash
composer require twig/twig
```

(Jau iekļauts, kad veicat `composer create-project flightphp/skeleton`.)

#### Pamata konfigurācija

Pārrakstiet `render` metodi, lai izmantotu Twig, nevis noklusējuma PHP renderētāju:

```php
// pārrakstiet render metodi, lai izmantotu Twig, nevis noklusējuma PHP renderētāju
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Kur Twig glabā savas kompilētās veidnes
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Atļauj "welcome" vai "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

Skeleton šis savienojums atrodas `app/config/services.php` (kopīga Twig vide, kešatmiņas ceļš, globālie mainīgie, piemēram, `base_url` / CSP nonce). Labāk injicējiet `Engine` un izsauciet `$app->render()` no kontrolleriem, lai kods paliktu [AI- un testiem draudzīgs](/learn/ai).

#### Twig izmantošana Flight

Tagad, kad varat renderēt ar Twig, varat darīt, piemēram, šādi:

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Kad pārlūkprogrammā apmeklējat `/Bob`, izvade būtu:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Papildu lasīšana

Pilnīgāks Twig izmantošanas piemērs ar izkārtojumiem ir parādīts šīs dokumentācijas [awesome plugins](/awesome-plugins/twig) sadaļā. Lai redzētu renderēšanas laika metriku Tracy joslā, skatiet [Twig paneli Tracy Extensions](/awesome-plugins/tracy-extensions#twig-panel-optional).

Vairāk par Twig pilnajām iespējām varat uzzināt, lasot [oficiālo dokumentāciju](https://twig.symfony.com/doc/3.x/).

### Latte

<span class="badge bg-secondary">lieliska alternatīva</span>

[Latte](https://latte.nette.org/) ir pilnvērtīgs dzinējs ar PHP līdzīgu sintaksi. Tas joprojām ir lieliska izvēle Flight lietotnēm; skeleton vienkārši standartizē Twig kā vienu kopīgu noklusējumu (īpaši noderīgi, kad AI rīki ģenerē veidnes).

#### Instalēšana

```bash
composer require latte/latte
```

#### Pamata konfigurācija

Galvenā doma ir pārrakstīt `render` metodi, lai izmantotu Latte, nevis noklusējuma PHP renderētāju.

```php
// pārrakstiet render metodi, lai izmantotu latte, nevis noklusējuma PHP renderētāju
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Kur latte tieši glabā savu kešatmiņu
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Latte izmantošana Flight

Tagad, kad varat renderēt ar Latte, varat darīt, piemēram, šādi:

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Kad pārlūkprogrammā apmeklējat `/Bob`, izvade būtu:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Papildu lasīšana

Sarežģītāks Latte izmantošanas piemērs ar izkārtojumiem ir parādīts šīs dokumentācijas [awesome plugins](/awesome-plugins/latte) sadaļā.

Vairāk par Latte pilnajām iespējām, tostarp tulkošanas un valodu iespējām, varat uzzināt, lasot [oficiālo dokumentāciju](https://latte.nette.org/en/).

### Iebūvētais skatu dzinējs

<span class="badge bg-warning">novecojis</span>

> **Piezīme:** Lai gan tā joprojām ir noklusējuma funkcionalitāte un tehniski joprojām darbojas.

Lai attēlotu skata veidni, izsauciet `render` metodi ar veidnes faila nosaukumu un neobligātiem veidnes datiem:

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Veidnes dati, ko nododat, tiek automātiski ievadīti veidnē, un uz tiem var atsaukties kā uz lokāliem mainīgajiem. Veidņu faili ir vienkārši PHP faili. Ja `hello.php` veidnes faila saturs ir:

```php
Hello, <?= $name ?>!
```

Izvade būtu:

```text
Hello, Bob!
```

Varat arī manuāli iestatīt skata mainīgos, izmantojot set metodi:

```php
Flight::view()->set('name', 'Bob');
```

Mainīgais `name` tagad ir pieejams visos jūsu skatos. Tātad varat vienkārši darīt:

```php
Flight::render('hello');
```

Ņemiet vērā, ka, norādot veidnes nosaukumu `render` metodē, varat izlaist `.php` paplašinājumu.

Pēc noklusējuma Flight meklēs `views` direktoriju veidņu failiem. Varat iestatīt citu ceļu savām veidnēm, iestatot šādu konfigurāciju:

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### Izkārtojumi

Tīmekļa vietnēm ir izplatīts vienots izkārtojuma veidnes fails ar mainīgu saturu. Lai renderētu saturu, kas tiks izmantots izkārtojumā, varat nodot neobligātu parametru `render` metodei.

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

Jūsu skatam tad būs saglabātie mainīgie ar nosaukumiem `headerContent` un `bodyContent`. Pēc tam varat renderēt savu izkārtojumu šādi:

```php
Flight::render('layout', ['title' => 'Home Page']);
```

Ja veidņu faili izskatās šādi:

`header.php`:

```php
<h1><?= $heading ?></h1>
```

`body.php`:

```php
<div><?= $body ?></div>
```

`layout.php`:

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

Izvade būtu:
```html
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <div>World</div>
  </body>
</html>
```

### Smarty

Lūk, kā jūs varētu izmantot [Smarty](http://www.smarty.net/) veidņu dzinēju saviem skatiem:

```php
// Ielādē Smarty bibliotēku
require './Smarty/libs/Smarty.class.php';

// Reģistrē Smarty kā skatu klasi
// Nodod arī atzvanīšanas funkciju, lai konfigurētu Smarty ielādes laikā
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Piešķir veidnes datus
Flight::view()->assign('name', 'Bob');

// Attēlo veidni
Flight::view()->display('hello.tpl');
```

Lai nodrošinātu pilnīgumu, jums vajadzētu arī pārrakstīt Flight noklusējuma render metodi:

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

Lūk, kā jūs varētu izmantot [Blade](https://laravel.com/docs/8.x/blade) veidņu dzinēju saviem skatiem:

Pirmkārt, jums ir jāinstalē BladeOne bibliotēka, izmantojot Composer:

```bash
composer require eftec/bladeone
```

Pēc tam varat konfigurēt BladeOne kā skatu klasi Flight:

```php
<?php
// Ielādē BladeOne bibliotēku
use eftec\bladeone\BladeOne;

// Reģistrē BladeOne kā skatu klasi
// Nodod arī atzvanīšanas funkciju, lai konfigurētu BladeOne ielādes laikā
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Piešķir veidnes datus
Flight::view()->share('name', 'Bob');

// Attēlo veidni
echo Flight::view()->run('hello', []);
```

Lai nodrošinātu pilnīgumu, jums vajadzētu arī pārrakstīt Flight noklusējuma render metodi:

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

Šajā piemērā hello.blade.php veidnes fails varētu izskatīties šādi:

```php
<?php
Hello, {{ $name }}!
```

Izvade būtu:

```
Hello, Bob!
```

## Skatīt arī
- [Instalēšana](/install) - Skeleton izkārtojums (`app/views/*.twig`) jauniem projektiem.
- [Paplašināšana](/learn/extending) - Kā pārrakstīt `render` metodi, lai izmantotu citu veidņu dzinēju.
- [Maršrutēšana](/learn/routing) - Kā kartēt maršrutus uz kontrolleriem un renderēt skatus.
- [Atbildes](/learn/responses) - Kā pielāgot HTTP atbildes.
- [Drošība](/learn/security) - Automātiskā izbēgšana un XSS.
- [AI un izstrādātāja pieredze](/learn/ai) - Kāpēc viens veidņu dzinēja noklusējums palīdz kodēšanas aģentiem.
- [Kāpēc ietvars?](/learn/why-frameworks) - Kā veidnes iekļaujas lielajā attēlā.

## Problēmu novēršana
- Ja starpprogrammatūrā (middleware) ir novirzīšana (redirect), bet jūsu lietotne, šķiet, nenovirza, pārliecinieties, ka starpprogrammatūrā pievienojat `exit;` paziņojumu.
- Ja Twig nevar atrast veidni, pārbaudiet `flight.views.path` un to, vai fails pastāv šajā ceļā ar paredzēto paplašinājumu (skeleton: `app/views/`).

## Izmaiņu žurnāls
- Dokumentācija – Twig dokumentēts kā oficiālais skeleton noklusējums; Latte joprojām ir pirmšķirīga alternatīva.
- v2.0 - Sākotnējais laidiens.