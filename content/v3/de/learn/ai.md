# KI & Entwicklererfahrung mit Flight

## Übersicht

Flight ist dafür konzipiert, *mit* KI-Programmierwerkzeugen zu arbeiten – nicht gegen sie. Eine kleine, vorhersehbare API, ein klares App-Layout im [offiziellen Skeleton](https://github.com/flightphp/skeleton) und projektspezifische Anweisungsdateien bedeuten, dass Assistenten wie GitHub Copilot, Cursor, Windsurf, Claude Code und Gemini dieselben Muster befolgen können, die du von Hand schreiben würdest.

Mit eingebauten Runway-Befehlen zum Verbinden mit LLM-Anbietern und zum Generieren von Projektanweisungen hilft Flight dir und deinem Team, konsistente, relevante Hilfe zu erhalten, ohne denselben Kontext in jeden Chat einzufügen.

## Verständnis

KI-Programmierassistenten sind am hilfreichsten, wenn sie den Kontext, die Konventionen und die Ziele deines Projekts verstehen. Die KI-Helfer von Flight ermöglichen dir:

- Verbinde dein Projekt mit beliebten LLM-Anbietern (OpenAI, Grok, Claude usw.)
- Generiere und aktualisiere projektspezifische Anweisungen, damit alle dieselbe Anleitung erhalten
- Halte handgeschriebenen und KI-generierten Code in einem einheitlichen Layout (insbesondere mit dem Skeleton)

Diese Funktionen sind in der Flight-Core-CLI enthalten (über [Runway](/awesome-plugins/runway)) und im offiziellen [flightphp/skeleton](https://github.com/flightphp/skeleton) Starter vorkonfiguriert.

### Was das Skeleton für KI mitbringt

Der offizielle Starter behandelt **`AGENTS.md` als die Quelle der Wahrheit** für KI-Tools:

| Datei | Rolle |
|------|------|
| **`AGENTS.md`** (Projektwurzel) | Globale Regeln, Boot-Ablauf, Namensräume, DI, „Was man nicht tun sollte“ |
| **Bereichsbezogene `AGENTS.md`** unter `app/`, `migrations/`, `tests/` usw. | Kompakte, ordnerspezifische Tipps, wenn du in diesem Verzeichnisbaum arbeitest |
| **`SECURITY.md`** | Geheimnisse, Header, XSS/SQL, Meldung – Sicherheit bleibt bewusst getrennt |

Es gibt **keine** separate House-Style-Datei für Copilot / Cursor / Gemini / Windsurf im Skeleton. Weise deinen Assistenten auf die `AGENTS.md` im Wurzelverzeichnis an (und lass ihn Links zu bereichsbezogenen Dateien folgen). Menschen können diese Dateien komplett ignorieren und die [README](https://github.com/flightphp/skeleton) verwenden; das Layout ist in beiden Fällen dasselbe.

> **Dokumente lehren APIs; das Skeleton lehrt Layout.** Kurze `Flight::`-Beispiele in diesen Dokumenten sind hervorragend zum Lernen. In einer Skeleton-App bevorzuge `App\…`-Klassen, Konstruktorinjektion und `$this->app` gegenüber der statischen Fassade in Controllern. Siehe [Installation](/install) und [Autoloading](/learn/autoloading).

## Grundlegende Verwendung

### Einrichten der LLM-Anmeldeinformationen

Der Befehl `ai:init` führt dich durch das Verbinden deines Projekts mit einem LLM-Anbieter.

```bash
php runway ai:init
```

Du wirst aufgefordert:

- Wähle deinen Anbieter (OpenAI, Grok, Claude usw.)
- Gib deinen API-Schlüssel ein
- Lege die Basis-URL und den Modellnamen fest

Dies erstellt die Anmeldeinformationen, die für spätere LLM-Anfragen verwendet werden (zum Beispiel zum Generieren von Anweisungen).

**Beispiel:**

```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Generieren projektspezifischer KI-Anweisungen

Der Befehl `ai:generate-instructions` erstellt oder aktualisiert Anweisungen für KI-Programmierassistenten, zugeschnitten auf *dein* Projekt.

```bash
php runway ai:generate-instructions
```

Du beantwortest ein paar Fragen (Beschreibung, Datenbank, Templating, Sicherheit, Teamgröße usw.). Flight verwendet deinen LLM-Anbieter, um Anweisungen zu generieren, und schreibt sie hauptsächlich in:

- **`AGENTS.md`** im Projektwurzelverzeichnis (werkzeugunabhängig; das, was das offizielle Skeleton und die meisten modernen Agenten erwarten)

Je nach CLI-Version und Optionen kann der Befehl auch toolspezifische Kopien für ältere Workflows schreiben (zum Beispiel Copilot-, Cursor-, Windsurf- oder Gemini-Regeldateien). Behandle bei **neuen Projekten aus dem Skeleton** **`AGENTS.md`** (plus alle bereichsbezogenen `AGENTS.md`-Dateien, die du unter `app/` behältst) als einzige Quelle der Wahrheit – pflege nicht fünf abweichende Anweisungsdateien von Hand.

**Beispiel:**

```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Jetzt können KI-Tools Code vorschlagen, der zu deinem tatsächlichen Stack und Layout passt – nicht zu einem generischen PHP-Tutorial.

## Fortgeschrittene Verwendung

- Passe Anmeldeinformationen oder Ausgabepfade mit Befehlsoptionen an (siehe `--help` bei jedem Befehl).
- Die Helfer funktionieren mit jedem LLM-Anbieter, der eine OpenAI-kompatible API spricht.
- Führe `ai:generate-instructions` erneut aus, wenn sich das Projekt weiterentwickelt, damit Agenten synchron bleiben.
- Im Skeleton solltest du die Sicherheitsrichtlinie in **`SECURITY.md`** und das Code-Layout in **`AGENTS.md`** aufbewahren, damit kein Dokument zu einem Sammelsurium wird.
- Bevorzuge [docs.flightphp.com](https://docs.flightphp.com) und den Flight-MCP-Server, wenn Agenten API-Details benötigen; überprüfe erfundene Methoden anhand von `vendor/flightphp/core`.

## Siehe auch

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Offizieller Starter mit `AGENTS.md`, Twig, SimplePdo und Dice, verdrahtet für eine KI-freundliche Struktur
- [Installation](/install) – Empfohlenes `create-project`-Layout
- [Autoloading](/learn/autoloading) – Ordner-**Großschreibung** entspricht Namensräumen (`App\Controller` ↔ `app/Controller/`)
- [Runway CLI](/awesome-plugins/runway) – CLI, die `ai:*`- und Scaffolding-Befehle antreibt
- [Sicherheit](/learn/security) – Sichere Standardeinstellungen, die Agenten (und Menschen) nicht schwächen sollten

## Fehlerbehebung

- Wenn du „Missing .runway-creds.json“ siehst, führe zuerst `php runway ai:init` aus.
- Stelle sicher, dass dein API-Schlüssel gültig ist und Zugriff auf das ausgewählte Modell hat.
- Wenn Anweisungen nicht aktualisiert werden, überprüfe die Dateiberechtigungen in deinem Projektverzeichnis.
- Wenn ein Agent Flight-APIs oder das falsche Ordnerlayout erfindet, weise ihn auf die **`AGENTS.md`** im Wurzelverzeichnis und diese Dokumentationsseite hin; das Skeleton-Layout hat für Code unter `app/` Vorrang.

## Änderungsprotokoll

- v3.18.4 – `ai:generate-instructions` schreibt Projektanweisungen in `AGENTS.md` im Projektwurzelverzeichnis.
- v3.16.0 – `ai:init`- und `ai:generate-instructions`-CLI-Befehle für die KI-Integration hinzugefügt.