# Tracy

Tracy ir brīnišķīgs kļūdu apstrādātājs, ko var izmantot kopā ar Flight. Tam ir vairāki paneļi, kas var palīdzēt atkļūdot jūsu lietojumprogrammu. Tas ir arī ļoti viegli paplašināms un pievienot savus paneļus. Flight komanda ir izveidojusi dažus paneļus īpaši Flight projektiem ar [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) spraudni (Flight mainīgie, DB vaicājumi, pieprasījums, sesija un izvēles **Twig** panelis, kad nododat profilētāja profilu — skatiet [Tracy Extensions](/awesome-plugins/tracy-extensions)).

## Instalācija

Instalējiet ar composer. Un jūs patiešām vēlēsieties to instalēt bez izstrādes versijas, jo Tracy nāk ar ražošanas kļūdu apstrādes komponenti.

```bash
composer require tracy/tracy
```

## Pamata konfigurācija

Ir dažas pamata konfigurācijas iespējas, lai sāktu darbu. Vairāk par tām varat lasīt [Tracy dokumentācijā](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Iespējot Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // dažreiz jums ir jābūt precīzam (arī Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // jūs varat arī norādīt IP adrešu masīvu

// Šeit tiks reģistrētas kļūdas un izņēmumi. Pārliecinieties, ka šis direktorijs eksistē un ir rakstāms.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // rādīt visas kļūdas
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // visas kļūdas izņemot novecojušus paziņojumus
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // ja Debugger josla ir redzama, tad content-length nevar iestatīt ar Flight

	// Tas ir specifiski Tracy Extension for Flight, ja esat to iekļāvis
	// pretējā gadījumā komentējiet to.
	new TracyExtensionLoader($app);
}
```

## Noderīgi padomi

Kad atkļūdojat savu kodu, ir dažas ļoti noderīgas funkcijas datu izvadīšanai.

- `bdump($var)` - Tas izvadīs mainīgo Tracy joslā atsevišķā panelī.
- `dumpe($var)` - Tas izvadīs mainīgo un pēc tam nekavējoties beigs darbu.