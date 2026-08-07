# Flight pret Slim

## Kas ir Slim?
[Slim](https://slimframework.com) ir PHP mikro ietvars, kas palīdz ātri izveidot vienkāršas, bet jaudīgas tīmekļa lietojumprogrammas un API.

Daudz iedvesmas dažiem Flight v3 funkcijām faktiski nāca no Slim. Maršrutu grupēšana un starpprogrammatūras izpilde noteiktā secībā ir divas funkcijas, kuras iedvesmoja Slim. Slim v3 tika izlaists ar mērķi uz vienkāršību, bet par v4 ir [pretrunīgi vērtējumi](https://github.com/slimphp/Slim/issues/2770).

## Priekšrocības salīdzinājumā ar Flight

- Slim ir lielāka izstrādātāju kopiena, kas savukārt veido ērtus moduļus, lai palīdzētu jums no jauna neizgudrot riteni.
- Slim ievēro daudzas saskarnes un standartus, kas ir izplatīti PHP kopienā, tādējādi palielinot savietojamību.
- Slim ir pienācīga dokumentācija un apmācības, kuras var izmantot, lai apgūtu ietvaru (lai gan nekas salīdzinājumā ar Laravel vai Symfony).
- Slim ir dažādi resursi, piemēram, YouTube apmācības un tiešsaistes raksti, kurus var izmantot ietvara apguvei.
- Slim ļauj izmantot jebkurus komponentus, ko vēlaties, lai apstrādātu galvenās maršrutēšanas funkcijas, jo tas atbilst PSR-7.

## Trūkumi salīdzinājumā ar Flight

- Pārsteidzoši, ka Slim nav tik ātrs, kā jūs domātu, ka tam vajadzētu būt mikro-ietvaram. Skatiet [TechEmpower etalonus](https://www.techempower.com/benchmarks/#hw=ph&test=fortune&section=data-r22&l=zik073-cn3), lai iegūtu vairāk informācijas.
- Flight ir paredzēts izstrādātājam, kurš vēlas izveidot vieglu, ātru un viegli lietojamu tīmekļa lietojumprogrammu.
- Flight nav atkarību, turpretim [Slim ir dažas atkarības](https://github.com/slimphp/Slim/blob/4.x/composer.json), kas jums ir jāinstalē.
- Flight ir vērsts uz vienkāršību un lietošanas ērtumu.
- Viena no Flight pamatfunkcijām ir tā, ka tā dara visu iespējamo, lai saglabātu atpakaļsavietojamību. Slim v3 uz v4 bija pārtraucošas izmaiņas.
- Flight ir paredzēts izstrādātājiem, kuri pirmo reizi iepazīst ietvaru pasauli.
- Flight var veidot arī uzņēmuma līmeņa lietojumprogrammas, bet tam nav tik daudz piemēru un apmācību kā Slim. Tas arī prasīs lielāku disciplīnu no izstrādātāja puses, lai saglabātu lietas organizētas un labi strukturētas.
- Flight dod izstrādātājam lielāku kontroli pār lietojumprogrammu, turpretim Slim var ieviest nedaudz maģijas aizkulisēs.
- Flight ir [SimplePdo](/learn/simple-pdo) datubāzes piekļuvei (ieteicams, nevis novecojušais PdoWrapper). Slim prasa izmantot trešās puses bibliotēku.
- Flight ir [atļauju spraudnis](/awesome-plugins/permissions), ko var izmantot lietojumprogrammas aizsardzībai. Slim prasa izmantot trešās puses bibliotēku.
- Flight ir ORM ar nosaukumu [active-record](/awesome-plugins/active-record), ko var izmantot mijiedarbībai ar datubāzi. Slim prasa izmantot trešās puses bibliotēku.
- Flight ir CLI lietojumprogramma ar nosaukumu [runway](/awesome-plugins/runway), ko var izmantot lietojumprogrammas palaišanai no komandrindas. Slim tādas nav.