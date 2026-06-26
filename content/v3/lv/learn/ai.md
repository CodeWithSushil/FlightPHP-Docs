# AI un Izstrādātāja Pieredze ar Flight

## Pārskats

Flight atvieglo jūsu PHP projektu uzlabošanu ar AI darbinātiem rīkiem un mūsdienīgām izstrādātāju darbplūsmām. Ar iebūvētām komandām savienošanai ar LLM (Large Language Model) nodrošinātājiem un projektu specifisku AI kodēšanas instrukciju ģenerēšanu, Flight palīdz jums un jūsu komandai maksimāli izmantot AI palīgus, piemēram, GitHub Copilot, Cursor, Windsurf un Antigravity (Gemini).

## Izpratne

AI kodēšanas palīgi ir visnoderīgākie, kad tie izprot jūsu projekta kontekstu, konvencijas un mērķus. Flight AI palīgi ļauj jums:
- Savienot savu projektu ar populāriem LLM nodrošinātājiem (OpenAI, Grok, Claude, utt.)
- Ģenerēt un atjaunināt projektu specifiskas instrukcijas AI rīkiem, lai visi saņemtu konsekventu, atbilstošu palīdzību
- Uzturēt savu komandu saskaņotu un produktīvu, mazāk laika tērējot konteksta skaidrošanai

Šīs funkcijas ir iebūvētas Flight kodola CLI un oficiālajā [flightphp/skeleton](https://github.com/flightphp/skeleton) starter projektā.

## Pamata Lietošana

### LLM Akreditācijas Datu Iestatīšana

`ai:init` komanda palīdz savienot jūsu projektu ar LLM nodrošinātāju.

```bash
php runway ai:init
```

Jums tiks piedāvāts:
- Izvēlēties savu nodrošinātāju (OpenAI, Grok, Claude, utt.)
- Ievadīt savu API atslēgu
- Iestatīt bāzes URL un modeļa nosaukumu

Tas izveido nepieciešamos akreditācijas datus turpmākiem LLM pieprasījumiem.

**Piemērs:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Projekta Specifisku AI Instrukciju Ģenerēšana

`ai:generate-instructions` komanda palīdz izveidot vai atjaunināt instrukcijas AI kodēšanas palīgiem, pielāgotas jūsu projektam.

```bash
php runway ai:generate-instructions
```

Jums būs jāatbild uz dažiem jautājumiem par savu projektu (apraksts, datubāze, veidņu izveide, drošība, komandas izmērs, utt.). Flight izmanto jūsu LLM nodrošinātāju, lai ģenerētu instrukcijas, pēc tam raksta to pašu saturu uz:
- `.github/copilot-instructions.md` (GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (Cursor)
- `.windsurfrules` (Windsurf)
- `.gemini/GEMINI.md` (Antigravity)
- `AGENTS.md` (projekta saknē, rīku-agnostiskiem AI palīgiem)

**Piemērs:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Tagad jūsu AI rīki sniegs gudrākus, atbilstošākus ieteikumus, balstoties uz jūsu projekta reālajām vajadzībām.

## Paplašinātā Lietošana

- Jūs varat pielāgot savu akreditācijas datu vai instrukciju failu atrašanās vietu, izmantojot komandu opcijas (skatiet `--help` katrai komandai).
- AI palīgi ir izstrādāti darbam ar jebkuru LLM nodrošinātāju, kas atbalsta OpenAI saderīgas API.
- Ja vēlaties atjaunināt savas instrukcijas, projektam attīstoties, vienkārši vēlreiz palaidiet `ai:generate-instructions` un atbildiet uz uzvednēm.

## Skatiet Arī

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Oficiālais starteris ar AI integrāciju
- [Runway CLI](/awesome-plugins/runway) – Vairāk par CLI rīku, kas darbina šīs komandas

## Problēmu Novēršana

- Ja redzat "Missing .runway-creds.json", vispirms palaidiet `php runway ai:init`.
- Pārliecinieties, ka jūsu API atslēga ir derīga un tai ir piekļuve izvēlētajam modelim.
- Ja instrukcijas neatjauninās, pārbaudiet failu atļaujas savā projekta direktorijā.

## Izmaiņu Žurnāls

- v3.18.4 – `ai:generate-instructions` tagad arī raksta projekta instrukcijas uz `AGENTS.md` projekta saknē.
- v3.16.0 – Pievienotas `ai:init` un `ai:generate-instructions` CLI komandas AI integrācijai.