# KI & Entwicklererfahrung mit Flight

## Übersicht

Flight macht es einfach, Ihre PHP-Projekte mit KI-gestützten Tools und modernen Entwickler-Workflows zu erweitern. Mit integrierten Befehlen zur Verbindung mit LLM-Anbietern (Large Language Model) und zur Erstellung projektspezifischer KI-Codierungsanweisungen hilft Flight Ihnen und Ihrem Team, das Beste aus KI-Assistenten wie GitHub Copilot, Cursor, Windsurf und Antigravity (Gemini) herauszuholen.

## Verständnis

KI-Codierungsassistenten sind am hilfreichsten, wenn sie den Kontext, die Konventionen und Ziele Ihres Projekts verstehen. Flights KI-Helfer ermöglichen Ihnen:
- Ihr Projekt mit beliebten LLM-Anbietern zu verbinden (OpenAI, Grok, Claude, usw.)
- Projektspezifische Anweisungen für KI-Tools zu generieren und zu aktualisieren, damit alle konsistente, relevante Hilfe erhalten
- Ihr Team ausgerichtet und produktiv zu halten, mit weniger Zeit für die Erklärung des Kontexts

Diese Funktionen sind in das Flight Core-CLI und das offizielle [flightphp/skeleton](https://github.com/flightphp/skeleton) Starter-Projekt integriert.

## Grundlegende Verwendung

### LLM-Anmeldedaten einrichten

Der Befehl `ai:init` führt Sie durch die Verbindung Ihres Projekts mit einem LLM-Anbieter.

```bash
php runway ai:init
```

Sie werden aufgefordert:
- Ihren Anbieter auszuwählen (OpenAI, Grok, Claude, usw.)
- Ihren API-Schlüssel einzugeben
- Die Basis-URL und den Modellnamen festzulegen

Dadurch werden die notwendigen Anmeldedaten für zukünftige LLM-Anfragen erstellt.

**Beispiel:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Projektspezifische KI-Anweisungen generieren

Der Befehl `ai:generate-instructions` hilft Ihnen, Anweisungen für KI-Codierungsassistenten zu erstellen oder zu aktualisieren, die auf Ihr Projekt zugeschnitten sind.

```bash
php runway ai:generate-instructions
```

Sie beantworten einige Fragen zu Ihrem Projekt (Beschreibung, Datenbank, Templating, Sicherheit, Teamgröße, usw.). Flight verwendet Ihren LLM-Anbieter zur Generierung von Anweisungen und schreibt dann denselben Inhalt in:
- `.github/copilot-instructions.md` (für GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (für Cursor)
- `.windsurfrules` (für Windsurf)
- `.gemini/GEMINI.md` (für Antigravity)
- `AGENTS.md` (im Projektstammverzeichnis, für tool-agnostische KI-Assistenten)

**Beispiel:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Jetzt geben Ihre KI-Tools intelligentere, relevantere Vorschläge basierend auf den tatsächlichen Anforderungen Ihres Projekts.

## Erweiterte Verwendung

- Sie können den Speicherort Ihrer Anmeldedaten- oder Anweisungsdateien mit Befehlsoptionen anpassen (siehe `--help` für jeden Befehl).
- Die KI-Helfer sind so konzipiert, dass sie mit jedem LLM-Anbieter funktionieren, der OpenAI-kompatible APIs unterstützt.
- Wenn Sie Ihre Anweisungen aktualisieren möchten, während sich Ihr Projekt weiterentwickelt, führen Sie einfach `ai:generate-instructions` erneut aus und beantworten die Eingabeaufforderungen erneut.

## Siehe auch

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Der offizielle Starter mit KI-Integration
- [Runway CLI](/awesome-plugins/runway) – Mehr über das CLI-Tool, das diese Befehle unterstützt

## Fehlerbehebung

- Wenn Sie "Missing .runway-creds.json" sehen, führen Sie zuerst `php runway ai:init` aus.
- Stellen Sie sicher, dass Ihr API-Schlüssel gültig ist und Zugriff auf das ausgewählte Modell hat.
- Wenn Anweisungen nicht aktualisiert werden, überprüfen Sie die Dateiberechtigungen in Ihrem Projektverzeichnis.

## Änderungsprotokoll

- v3.18.4 – `ai:generate-instructions` schreibt Projektanweisungen jetzt auch in `AGENTS.md` im Projektstammverzeichnis.
- v3.16.0 – Hinzufügung der CLI-Befehle `ai:init` und `ai:generate-instructions` für die KI-Integration.