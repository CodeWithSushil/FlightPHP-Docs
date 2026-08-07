# Uzziniet par Flight

Flight ir ātrs, vienkāršs un paplašināms PHP ietvars. Tas ir diezgan daudzpusīgs, un to var izmantot jebkura veida tīmekļa lietojumprogrammu izveidei. Tas ir izstrādāts, domājot par vienkāršību, un ir rakstīts tā, lai to būtu viegli saprast un lietot — gan cilvēkiem, gan [AI kodēšanas palīgiem](/learn/ai).

> **Piezīme:** Jūs redzēsiet piemērus, kuros `Flight::` tiek izmantots kā statisks mainīgais, un dažus, kuros tiek izmantots `$app->` Engine objekts. Abi savstarpēji aizvietojami. `$app` un `$this->app` kontrolierī/starpprogrammatūrā ir Flight komandas ieteiktā pieeja (un to oficiālais skeletons + `AGENTS.md` nosaka kā standartu jauniem projektiem).

## Pamatkomponenti

### [Maršrutēšana](/learn/routing)

Uzziniet, kā pārvaldīt maršrutus savā tīmekļa lietojumprogrammā. Tas ietver arī maršrutu grupēšanu, maršrutu parametrus un starpprogrammatūru.

### [Starpprogrammatūra](/learn/middleware)

Uzziniet, kā izmantot starpprogrammatūru, lai filtrētu pieprasījumus un atbildes savā lietojumprogrammā.

### [Automātiskā ielāde](/learn/autoloading)

Uzziniet, kā automātiski ielādēt savas klases. Mapes **reģistram** jāatbilst jūsu vārdtelpām—skeletons izmanto `App\` un PascalCase mapes, piemēram, `app/Controller/`.

### [Pieprasījumi](/learn/requests)

Uzziniet, kā apstrādāt pieprasījumus un atbildes savā lietojumprogrammā.

### [Atbildes](/learn/responses)

Uzziniet, kā nosūtīt atbildes saviem lietotājiem.

### [HTML veidnes](/learn/templates)

Uzziniet, kā renderēt HTML, izmantojot Twig (skeletona noklusējums), Latte vai citus dzinējus—ne tikai iebūvētos PHP skatus.

### [Drošība](/learn/security)

Uzziniet, kā aizsargāt savu lietojumprogrammu no izplatītiem drošības apdraudējumiem.

### [Konfigurācija](/learn/configuration)

Uzziniet, kā konfigurēt ietvaru savai lietojumprogrammai.

### [Notikumu pārvaldnieks](/learn/events)

Uzziniet, kā izmantot notikumu sistēmu, lai savai lietojumprogrammai pievienotu pielāgotus notikumus.

### [Flight paplašināšana](/learn/extending)

Uzziniet, kā paplašināt ietvaru, pievienojot savas metodes un klases.

### [Metodes āķi un filtrēšana](/learn/filtering)

Uzziniet, kā pievienot notikumu āķus savām metodēm un iekšējām ietvara metodēm.

### [Atkarību injekcijas konteiners (DIC)](/learn/dependency-injection-container)

Uzziniet, kā izmantot atkarību injekcijas konteinerus (DIC), lai pārvaldītu savas lietojumprogrammas atkarības.

## Palīgklases

### [Kolekcijas](/learn/collections)

Kolekcijas tiek izmantotas, lai saglabātu datus un būtu pieejamas kā masīvs vai kā objekts, lai atvieglotu lietošanu.

### [JSON ietinējs](/learn/json)

Tajā ir dažas vienkāršas funkcijas, lai jūsu JSON kodēšana un atkodēšana būtu konsekventa.

### [SimplePdo](/learn/simple-pdo)

PDO reizēm var sagādāt vairāk galvassāpju, nekā nepieciešams. SimplePdo ir mūsdienīga PDO palīgklase ar ērtām metodēm, piemēram, `insert()`, `update()`, `delete()` un `transaction()`, lai datubāzes darbības būtu daudz vienkāršākas.

### [PdoWrapper](/learn/pdo-wrapper) (Novecojis)

Sākotnējais PDO ietinējs ir novecojis no v3.18.0. Lūdzu, tā vietā izmantojiet [SimplePdo](/learn/simple-pdo).

### [Augšupielādēto failu apstrādātājs](/learn/uploaded-file)

Vienkārša klase, kas palīdz pārvaldīt augšupielādētos failus un pārvietot tos uz pastāvīgu atrašanās vietu.

## Svarīgi jēdzieni

### [Kāpēc ietvars?](/learn/why-frameworks)

Šeit ir īss raksts par to, kāpēc jums vajadzētu izmantot ietvaru. Ir labi saprast ietvaru izmantošanas priekšrocības, pirms sākat to lietot.

Turklāt izcilu apmācību ir izveidojis [@lubiana](https://git.php.fail/lubiana). Lai gan tajā nav detalizēti aprakstīts tieši Flight, šī rokasgrāmata palīdzēs jums saprast dažus galvenos jēdzienus, kas saistīti ar ietvariem, un kāpēc tos ir izdevīgi izmantot. Apmācību varat atrast [šeit](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight salīdzinājumā ar citiem ietvariem](/learn/flight-vs-another-framework)

Ja pārejat no cita ietvara, piemēram, Laravel, Slim, Fat-Free vai Symfony, uz Flight, šī lapa palīdzēs jums saprast atšķirības starp tiem.

## Citas tēmas

### [Vienību testēšana](/learn/unit-testing)

Sekojiet šai rokasgrāmatai, lai uzzinātu, kā veikt vienību testus savam Flight kodam, lai tas būtu ļoti stabils.

### [AI un izstrādātāja pieredze](/learn/ai)

Flight ir veidots, lai sadarbotos ar programmēšanas LLM: `AGENTS.md`, Runway `ai:*` komandas un viens skaidrs skeletona izkārtojums, lai aģenti paliktu uz pareizā ceļa.

### [Pāreja no v2 uz v3](/learn/migrating-to-v3)

Atpakaļsaderība lielākoties ir saglabāta, taču ir dažas izmaiņas, par kurām jums jāzina, pārejot no v2 uz v3.