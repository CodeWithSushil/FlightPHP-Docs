# Brīnišķīgi spraudņi

Flight ir neticami paplašināms. Ir pieejami vairāki spraudņi, kurus var izmantot, lai pievienotu funkcionalitāti Flight lietotnei. Daži no tiem ir oficiāli atbalstīti Flight komandas, bet citi ir mikro/viegli risinājumi, kas palīdz sākt darbu.

## MI rīki

Flight var padarīt vēl foršāku ar MI darbināmiem spraudņiem.

- [Flight MCP](/awesome-plugins/mcp) - Spraudnis MCP (Model Control Protocol) integrēšanai ar Flight, nodrošinot vienmērīgu MI darbināmu funkcionalitāti. Galvenokārt vērsts uz dokumentācijas lapām, tas palīdz samazināt token izmaksas, nodrošinot aktuālāko informāciju par Flight projektiem.

## API dokumentācija

API dokumentācija ir būtiska jebkurai API. Tas palīdz izstrādātājiem saprast, kā mijiedarboties ar API un ko sagaidīt pretī. Ir pieejami daži rīki, kas palīdz ģenerēt API dokumentāciju Flight projektiem.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Daniel Schreiber bloga ieraksts par to, kā izmantot OpenAPI Spec ar FlightPHP, lai izveidotu API, izmantojot API pirmo pieeju.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI ir lielisks rīks, kas palīdz ģenerēt API dokumentāciju Flight projektiem. Tas ir ļoti viegli lietojams un var tikt pielāgots jūsu vajadzībām. Šī ir PHP bibliotēka, kas palīdz ģenerēt Swagger dokumentāciju.

## Lietotnes veiktspējas uzraudzība (APM)

Lietotnes veiktspējas uzraudzība (APM) ir būtiska jebkurai lietotnei. Tas palīdz izprast, kā darbojas lietotne un kur ir sastrēgumi. Ir pieejami vairāki APM rīki, kurus var izmantot ar Flight.
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM ir vienkārša APM bibliotēka, ko var izmantot Flight lietotņu uzraudzībai. To var izmantot, lai uzraudzītu lietotnes veiktspēju un palīdzētu identificēt sastrēgumus.

## Asinhronā apstrāde

Flight jau ir ātrs ietvars, bet turbodzinēja pievienošana padara visu vēl aizraujošāku (un izaicināmāku)!

- [flightphp/async](/awesome-plugins/async) - Oficiālā Flight Async bibliotēka. Šī bibliotēka ir vienkāršs veids, kā pievienot asinhronu apstrādi lietotnei. Tā izmanto Swoole/Openswoole dziļākā līmenī, lai nodrošinātu vienkāršu un efektīvu veidu asinhronu uzdevumu izpildei.

## Autorizācija/Pieejas tiesības

Autorizācija un pieejas tiesības ir būtiskas jebkurai lietotnei, kurai nepieciešamas kontroles, lai noteiktu, kurš var piekļūt kam.

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - Oficiālā Flight Permissions bibliotēka. Šī bibliotēka ir vienkāršs veids, kā pievienot lietotāju un lietotnes līmeņa pieejas tiesības lietotnei. 

## Autentifikācija

Autentifikācija ir būtiska lietotnēm, kurām nepieciešams verificēt lietotāja identitāti un nodrošināt API galapunktus.

- [firebase/php-jwt](/awesome-plugins/jwt) - JSON Web Token (JWT) bibliotēka PHP. Vienkāršs un drošs veids, kā ieviest token balstītu autentifikāciju Flight lietotnēs. Ideāli piemērots bezstāvokļa API autentifikācijai, maršrutu aizsardzībai ar starpprogrammatūru un OAuth stila autorizācijas plūsmu ieviešanai.

## Kešatmiņa

Kešatmiņa ir lielisks veids, kā paātrināt lietotni. Ir pieejamas vairākas kešatmiņas bibliotēkas, kuras var izmantot ar Flight.

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Vieglā, vienkāršā un patstāvīgā PHP failu kešatmiņas klase

## CLI

CLI lietotnes ir lielisks veids, kā mijiedarboties ar lietotni. Jūs varat tās izmantot, lai ģenerētu kontrolierus, parādītu visus maršrutus un vairāk.

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway ir CLI lietotne, kas palīdz pārvaldīt Flight lietotnes.

## Sīkdatnes

Sīkdatnes ir lielisks veids, kā saglabāt nelielus datu fragmentus klienta pusē. Tās var izmantot, lai saglabātu lietotāja preferences, lietotnes iestatījumus un vairāk.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie ir PHP bibliotēka, kas nodrošina vienkāršu un efektīvu veidu, kā pārvaldīt sīkdatnes.

## Atkļūdošana

Atkļūdošana ir būtiska, kad izstrādājat lokālajā vidē. Ir daži spraudņi, kas var uzlabot jūsu atkļūdošanas pieredzi.

- [tracy/tracy](/awesome-plugins/tracy) - Šis ir pilnībā aprīkots kļūdu apstrādātājs, ko var izmantot ar Flight. Tam ir vairāki paneļi, kas palīdz atkļūdot lietotni. Tas ir arī ļoti viegli paplašināms un var pievienot savus paneļus.
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Izmantojot [Tracy](/awesome-plugins/tracy) kļūdu apstrādātāju, šis spraudnis pievieno dažus papildu paneļus, lai palīdzētu ar atkļūdošanu specifiski Flight projektiem.

## Datubāzes

Datubāzes ir lielākās daļas lietotņu kodols. Tā ir vieta, kur saglabāt un iegūt datus. Dažas datubāzes bibliotēkas ir vienkārši aploki vaicājumu rakstīšanai, bet dažas ir pilnvērtīgas ORM sistēmas.

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Oficiālais Flight PDO palīgs, kas ir daļa no kodola. Tas ir moderns aploks ar ērtām palīgmetodēm, piemēram, `insert()`, `update()`, `delete()` un `transaction()`, lai vienkāršotu datubāzes operācijas. Visi rezultāti tiek atgriezti kā Collections, lai nodrošinātu elastīgu masīva/objekta piekļuvi. Nav ORM, tikai labāks veids, kā strādāt ar PDO.
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Oficiālais Flight PDO aploks, kas ir daļa no kodola (novecojis kopš v3.18.0). Izmantojiet SimplePdo vietā.
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - Oficiālais Flight ActiveRecord ORM/Mapper. Lieliska maza bibliotēka, lai viegli iegūtu un saglabātu datus datubāzē.
- [byjg/php-migration](/awesome-plugins/migrations) - Spraudnis, lai sekotu līdzi visām datubāzes izmaiņām projektā.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Viegls, plūstošs SQL vaicājumu veidotājs, kas ģenerē SQL un parametrus sagatavotajiem vaicājumiem. Lieliski darbojas ar [SimplePdo](/learn/simple-pdo).

## Šifrēšana

Šifrēšana ir būtiska jebkurai lietotnei, kas saglabā sensitīvus datus. Datus šifrēt un atšifrēt nav īpaši grūti, bet pareizi glabāt šifrēšanas atslēgu [var](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [būt](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [sarežģīti](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). Vissvarīgākais ir nekad neglabāt šifrēšanas atslēgu publiskajā direktorijā vai to commit'ot kodā.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - Šī ir bibliotēka, ko var izmantot, lai šifrētu un atšifrētu datus. Darba sākšana ir diezgan vienkārša, lai sāktu šifrēt un atšifrēt datus.

## Darbu rinda

Darbu rindas ir patiešām noderīgas, lai asinhroni apstrādātu uzdevumus. Tas var būt e-pasta sūtīšana, attēlu apstrāde vai jebkas, kas nav jādara reālā laikā.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue ir bibliotēka, ko var izmantot, lai asinhroni apstrādātu darbus. To var izmantot ar beanstalkd, MySQL/MariaDB, SQLite un PostgreSQL.

## Sesija

Sesijas īsti nav noderīgas API, bet tīmekļa lietotnes izveidei sesijas var būt būtiskas stāvokļa un pieteikšanās informācijas uzturēšanai.

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - Oficiālā Flight Session bibliotēka. Šī ir vienkārša sesijas bibliotēka, ko var izmantot, lai saglabātu un iegūtu sesijas datus. Tā izmanto PHP iebūvēto sesiju apstrādi.
- [Ghostff/Session](/awesome-plugins/ghost-session) - PHP Session Manager (nebloķējošs, flash, segment, sesijas šifrēšana). Izmanto PHP open_ssl sesijas datu papildu šifrēšanai/atšifrēšanai.

## Veidnes

Veidnes ir jebkuras tīmekļa lietotnes ar lietotāja saskarni kodols. Ir pieejami vairāki veidņu dzinēji, kurus var izmantot ar Flight.

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - Šis ir ļoti vienkāršs veidņu dzinējs, kas ir daļa no kodola. Tas nav ieteicams izmantot, ja projektā ir vairāk nekā pāris lapas.
- [latte/latte](/awesome-plugins/latte) - Latte ir pilnībā aprīkots veidņu dzinējs, kas ir ļoti viegli lietojams un jūtās tuvāks PHP sintaksei nekā Twig vai Smarty. Tas ir arī ļoti viegli paplašināms un var pievienot savus filtrus un funkcijas.
- [twig/twig](/awesome-plugins/twig) - Twig ir elastīgs, ātrs un drošs veidņu dzinējs (tāds pats, kādu izmanto Symfony). MI rīki un daudzi PHP izstrādātāji to labi zina, tas automātiski aizsargā izvadi pēc noklusējuma un tam ir milzīga paplašinājumu ekosistēma.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate ir spēcīgs PHP veidņu dzinējs ar aktīvu kompilāciju, veidņu mantošanu un mainīgo apstrādi. Ietver automātisku CSS/JS minimizāciju, kešatmiņu, Base64 kodēšanu un papildu Flight PHP ietvara integrāciju.

## WordPress integrācija

Vai vēlaties izmantot Flight WordPress projektā? Tam ir ērts spraudnis!

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Šis WordPress spraudnis ļauj palaist Flight tieši blakus WordPress. Tas ir ideāli piemērots, lai pievienotu pielāgotas API, mikropakalpojumus vai pat pilnas lietotnes WordPress vietnei, izmantojot Flight ietvaru. Īpaši noderīgi, ja vēlaties labāko no abām pasaulēm!

## Ieguldījums

Vai jums ir spraudnis, ko vēlaties dalīties? Iesniedziet pull request, lai to pievienotu sarakstam!