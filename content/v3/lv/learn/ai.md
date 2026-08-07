# AI un izstrādātāju pieredze ar Flight

## Pārskats

Flight ir veidots, lai strādātu *ar* AI kodēšanas rīkiem—nevis cīnītos pret tiem. Neliels, paredzams API, skaidrs lietotnes izkārtojums [oficiālajā skeleton](https://github.com/flightphp/skeleton) un projektam specifiski instrukciju faili nozīmē, ka asistenti, piemēram, GitHub Copilot, Cursor, Windsurf, Claude Code un Gemini, var sekot tiem pašiem modeļiem, ko jūs rakstītu paši.

Ar iebūvētām Runway komandām savienošanai ar LLM pakalpojumu sniedzējiem un projektu instrukciju ģenerēšanai, Flight palīdz jums un jūsu komandai saņemt konsekventu un atbilstošu palīdzību bez nepieciešamības ielīmēt vienu un to pašu kontekstu katrā tērzēšanā.

## Izpratne

AI kodēšanas asistenti ir visnoderīgākie, kad tie saprot jūsu projekta kontekstu, konvencijas un mērķus. Flight AI palīgi ļauj jums:

- Savienot jūsu projektu ar populāriem LLM pakalpojumu sniedzējiem (OpenAI, Grok, Claude utt.)
- Ģenerēt un atjaunināt projektam specifiskus instrukciju failus, lai visi saņemtu vienus un tos pašus norādījumus
- Saglabāt roku rakstītu un AI ģenerētu kodu vienā izkārtojumā (īpaši ar skeleton)

Šie rīki ir iekļauti Flight pamata CLI (caur [Runway](/awesome-plugins/runway)) un ir iepriekš konfigurēti oficiālajā [flightphp/skeleton](https://github.com/flightphp/skeleton) sākuma projektā.

### Ko skeleton nodrošina AI

Oficiālais sākuma projekts uzskata **`AGENTS.md` par patiesības avotu** AI rīkiem:

| Fails | Loma |
|------|------|
| **`AGENTS.md`** (projekta sakne) | Globālie noteikumi, palaišanas plūsma, namespaces, DI, “ko nedrīkst darīt” |
| **Scoped `AGENTS.md`** zem `app/`, `migrations/`, `tests/` utt. | Vieglas, mapēm specifiskas padomes, kad strādājat šajā kokā |
| **`SECURITY.md`** | Noslēpumi, galvenes, XSS/SQL, ziņošana—drošība paliek apzināta un atsevišķa |

Skeleton **nav** atsevišķa stila faila Copilot / Cursor / Gemini / Windsurf. Norādiet savu asistentu uz saknes `AGENTS.md` (un ļaujiet tam sekot saitēm uz scoped failiem). Cilvēki var pilnībā ignorēt šos failus un izmantot [README](https://github.com/flightphp/skeleton); izkārtojums abos gadījumos ir vienāds.

> **Dokumentācija māca API; skeleton māca izkārtojumu.** Īsi `Flight::` piemēri šajā dokumentācijā ir lieliski mācībām. Skeleton lietotnē dodiet priekšroku `App\…` klasēm, konstruktora injekcijai un `$this->app` nevis statiskajam fasādes stilam kontrolieros. Skatiet [Instalācija](/install) un [Autoloading](/learn/autoloading).

## Pamata lietošana

### LLM akreditācijas datu iestatīšana

Komanda `ai:init` ved jūs cauri projekta savienošanai ar LLM pakalpojumu sniedzēju.

```bash
php runway ai:init
```

Jums tiks prasīts:

- Izvēlēties pakalpojumu sniedzēju (OpenAI, Grok, Claude utt.)
- Ievadīt savu API atslēgu
- Iestatīt bāzes URL un modeļa nosaukumu

Tas izveido akreditācijas datus, kas tiek izmantoti vēlākiem LLM pieprasījumiem (piemēram, instrukciju ģenerēšanai).

**Piemērs:**
```
Laipni lūgti AI Init!
Kuru LLM API vēlaties izmantot? [1] openai, [2] grok, [3] claude: 1
Ievadiet bāzes URL LLM API [https://api.openai.com]:
Ievadiet savu API atslēgu priekš openai: sk-...
Ievadiet modeļa nosaukumu, kuru vēlaties izmantot (piem., gpt-4, claude-3-opus utt.) [gpt-4o]:
Akreditācijas dati saglabāti .runway-creds.json
```

### Projektam specifisku AI instrukciju ģenerēšana

Komanda `ai:generate-instructions` izveido vai atjaunina instrukcijas AI kodēšanas asistentiem, pielāgotas *jūsu* projektam.

```bash
php runway ai:generate-instructions
```

Jums būs jāatbild uz dažiem jautājumiem (apraksts, datubāze, template engine, drošība, komandas lielums utt.). Flight izmanto jūsu LLM pakalpojumu sniedzēju, lai ģenerētu instrukcijas, un ieraksta tās galvenokārt:

- **`AGENTS.md`** projekta saknē (neatkarīgi no rīka; to sagaida oficiālais skeleton un lielākā daļa mūsdienu asistentu)

Atkarībā no CLI versijas un opcijām komanda var ierakstīt arī rīkiem specifiskas kopijas vecākām darba plūsmām (piemēram, Copilot, Cursor, Windsurf vai Gemini noteikumu failus). **Jauniem projektiem, kas veidoti no skeleton**, uzskatiet **`AGENTS.md`** (plus jebkurus scoped `AGENTS.md` failus, ko saglabājat zem `app/`) par vienīgo patiesības avotu—neuzturiet piecus atšķirīgus instrukciju failus manuāli.

**Piemērs:**
```
Lūdzu, aprakstiet, kam jūsu projekts ir paredzēts? Mana lieliskā API
Kuru datubāzi plānojat izmantot? MySQL
Kuru HTML template engine plānojat izmantot (ja tāds ir)? twig
Vai drošība ir svarīgs šī projekta elements? (j/n) j
...
AI instrukcijas veiksmīgi atjauninātas.
```

Tagad AI rīki var ieteikt kodu, kas atbilst jūsu faktiskajai tehnoloģiju kopai un izkārtojumam—nevis vispārīgai PHP apmācībai.

## Papildu lietošana

- Pielāgojiet akreditācijas datus vai izvades ceļus ar komandu opcijām (skatiet `--help` katrai komandai).
- Šie rīki darbojas ar jebkuru LLM pakalpojumu sniedzēju, kas atbalsta OpenAI saderīgu API.
- Pārvietojiet `ai:generate-instructions` atkārtoti, projektam attīstoties, lai asistenti paliktu sinhronizēti.
- Skeleton saglabājiet drošības politiku **`SECURITY.md`** failā un kodēšanas izkārtojumu **`AGENTS.md`** failā, lai neviens no dokumentiem nekļūtu par visu iespējamo krātuvi.
- Dodiet priekšroku [docs.flightphp.com](https://docs.flightphp.com) un Flight MCP serverim, kad asistentiem nepieciešama API informācija; pārbaudiet izgudrotas metodes pret `vendor/flightphp/core`.

## Skatīt arī

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Oficiālais sākuma projekts ar `AGENTS.md`, Twig, SimplePdo un Dice, kas konfigurēti AI draudzīgai struktūrai
- [Instalācija](/install) – Ieteicamais `create-project` izkārtojums
- [Autoloading](/learn/autoloading) – Mapes **reģistrs** atbilst namespaces (`App\Controller` ↔ `app/Controller/`)
- [Runway CLI](/awesome-plugins/runway) – CLI, kas nodrošina `ai:*` un scaffolding komandas
- [Drošība](/learn/security) – Drošie noklusējumi, kurus asistenti (un cilvēki) nedrīkst vājināt

## Problēmu novēršana

- Ja redzat “Missing .runway-creds.json”, vispirms izpildiet `php runway ai:init`.
- Pārliecinieties, ka jūsu API atslēga ir derīga un tai ir piekļuve izvēlētajam modelim.
- Ja instrukcijas netiek atjauninātas, pārbaudiet failu atļaujas projekta direktorijā.
- Ja asistenti izgudro Flight API vai nepareizu mapju izkārtojumu, norādiet tos uz saknes **`AGENTS.md`** un šo dokumentācijas vietni; skeleton izkārtojums ir noteicošais kodam zem `app/`.

## Izmaiņu žurnāls

- v3.18.4 – `ai:generate-instructions` ieraksta projekta instrukcijas `AGENTS.md` projekta saknē.
- v3.16.0 – Pievienotas `ai:init` un `ai:generate-instructions` CLI komandas AI integrācijai.