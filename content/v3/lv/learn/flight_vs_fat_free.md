# Flight pret Fat-Free

## Kas ir Fat-Free?
[Fat-Free](https://fatfreeframework.com) (mīļi saukts par **F3**) ir jaudīgs, tomēr viegli lietojams PHP mikroietvars, kas izstrādāts, lai palīdzētu jums veidot dinamiskas un stabilas tīmekļa lietotnes - ātri!

Flight daudzējādā ziņā salīdzinās ar Fat-Free un, iespējams, ir vistuvākais radinieks funkciju un vienkāršības ziņā. Fat-Free ir daudz funkciju, kuru Flight nav, bet tam ir arī daudz funkciju, kuras Flight ir. Fat-Free sāk parādīt savu vecumu un vairs nav tik populārs kā tas bija kādreiz.

Atjauninājumi kļūst arvien retāki, un kopiena vairs nav tik aktīva kā agrāk. Kods ir pietiekami vienkāršs, bet dažkārt sintakses disciplīnas trūkums var apgrūtināt tā lasīšanu un saprašanu. Tas darbojas PHP 8.3, bet pats kods joprojām izskatās tā, it kā tas būtu no PHP 5.3 laikiem.

## Plusi salīdzinājumā ar Flight

- Fat-Free GitHub ir nedaudz vairāk zvaigžņu nekā Flight.
- Fat-Free ir diezgan laba dokumentācija, bet dažās jomās tai trūkst skaidrības.
- Fat-Free ir daži reti resursi, piemēram, YouTube apmācības un tiešsaistes raksti, kurus var izmantot, lai apgūtu šo ietvaru.
- Fat-Free ir iebūvēti [daži noderīgi spraudņi](https://fatfreeframework.com/3.8/api-reference), kas dažkārt ir noderīgi.
- Fat-Free ir iebūvēts ORM ar nosaukumu Mapper, ko var izmantot, lai mijiedarbotos ar datu bāzi. Flight ir [active-record](/awesome-plugins/active-record).
- Fat-Free ir iebūvētas sesijas, kešatmiņa un lokalizācija. Flight prasa izmantot trešo pušu bibliotēkas, bet tas ir aprakstīts [dokumentācijā](/awesome-plugins).
- Fat-Free ir neliela [kopienas veidotu spraudņu](https://fatfreeframework.com/3.8/development#Community) grupa, ko var izmantot ietvara paplašināšanai. Flight dažus no tiem apraksta [dokumentācijas](/awesome-plugins) un [piemēru](/examples) lapās.
- Fat-Free, tāpat kā Flight, nav atkarību.
- Fat-Free, tāpat kā Flight, ir vērsts uz to, lai izstrādātājam dotu kontroli pār savu lietotni un vienkāršu izstrādātāja pieredzi.
- Fat-Free saglabā atpakaļejošu saderību, tāpat kā Flight (daļēji tāpēc, ka atjauninājumi kļūst [retāki](https://github.com/bcosca/fatfree/releases)).
- Fat-Free, tāpat kā Flight, ir paredzēts izstrādātājiem, kuri pirmo reizi iepazīst ietvaru pasauli.
- Fat-Free ir iebūvēts veidņu dzinis, kas ir robustāks nekā Flight veidņu dzinis. Flight iesaka izmantot [Latte](/awesome-plugins/latte), lai to sasniegtu.
- Fat-Free ir unikāla CLI tipa "route" komanda, kurā var veidot CLI lietotnes pašā Fat-Free un apstrādāt to līdzīgi kā `GET` pieprasījumu. Flight to panāk ar [runway](/awesome-plugins/runway).

## Mīnusi salīdzinājumā ar Flight

- Fat-Free ir daži ieviešanas testi un pat sava [testa](https://fatfreeframework.com/3.8/test) klase, kas ir ļoti vienkārša. Tomēr tas nav 100% vienību testēts kā Flight.
- Lai meklētu dokumentācijas vietnē, jums ir jāizmanto meklētājprogramma, piemēram, Google.
- Flight dokumentācijas vietnē ir tumšais režīms. (mic drop)
- Fat-Free ir daži moduļi, kas ir nožēlojami nekopti.
- Flight ir [SimplePdo](/learn/simple-pdo) datu bāzes piekļuvei, kas ir nedaudz vienkāršāks par Fat-Free iebūvēto `DB\SQL` klasi (un ir ieteicams, nevis novecojušais PdoWrapper).
- Flight ir [tiesību spraudnis](/awesome-plugins/permissions), ko var izmantot, lai aizsargātu savu lietotni. Fat-Free prasa izmantot trešās puses bibliotēku.
- Flight ir ORM ar nosaukumu [active-record](/awesome-plugins/active-record), kas vairāk izskatās pēc ORM nekā Fat-Free Mapper. `active-record` papildu ieguvums ir tas, ka varat definēt attiecības starp ierakstiem automātiskai savienošanai, kamēr Fat-Free Mapper prasa izveidot [SQL skatus](https://fatfreeframework.com/3.8/databases#ProsandCons).
- Pārsteidzoši, bet Fat-Free nav saknes vārdtelpas. Flight ir pilnībā vārdtelpots, lai netiktu konfliktēts ar jūsu pašu kodu. `Cache` klase ir lielākais pārkāpējs šeit.
- Fat-Free nav middleware. Tā vietā ir `beforeroute` un `afterroute` āķi, kurus var izmantot, lai kontrolētu pieprasījumus un atbildes kontrolieros.
- Fat-Free nevar grupēt maršrutus.
- Fat-Free ir atkarību injekcijas konteinera apstrādātājs, bet dokumentācija par tā lietošanu ir ārkārtīgi skopa.
- Atkļūdošana var kļūt nedaudz sarežģīta, jo pamatā viss tiek glabāts tā sauktajā [`HIVE`](https://fatfreeframework.com/3.8/quick-reference).