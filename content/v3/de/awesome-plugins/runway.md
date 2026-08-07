# Runway

Runway ist eine CLI-Anwendung, die Ihnen hilft, Ihre Flight-Anwendungen zu verwalten. Sie kann Controller generieren, alle Routen anzeigen, KI-Setup-Helfer ausführen, Migrationen (im Skeleton) und mehr. Sie basiert auf der ausgezeichneten [adhocore/php-cli](https://github.com/adhocore/php-cli) Bibliothek.

Klicken Sie [hier](https://github.com/flightphp/runway), um den Code anzuzeigen.

Scaffolding-Befehle sind absichtlich mit dem [offiziellen Skeleton](https://github.com/flightphp/skeleton) abgestimmt, damit [KI-Coding-Tools](/learn/ai) und Menschen jedes Mal die gleichen Pfade, Namespaces und Constructor-Injection-Stile erhalten.

## Installation

Mit Composer installieren.

```bash
composer require flightphp/runway
```

Das Skeleton hängt bereits von Runway ab; verwenden Sie `php runway` aus dem Projekt-Root.

## Grundlegende Konfiguration

Beim ersten Ausführen von Runway wird versucht, eine `runway`-Konfiguration in `app/config/config.php` über den Schlüssel `'runway'` zu finden.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// optional; das Skeleton verwendet auch index_root für den öffentlichen Einstieg
		'index_root' => 'public/index.php',
    ],
];
```

> **HINWEIS** - Ab **v1.2.0** ist `.runway-config.json` zugunsten von `app/config/config.php` veraltet. Migrieren Sie mit `php runway config:migrate` beim Upgrade älterer Projekte. Das Skeleton kann beim Erstellen eines Projekts weiterhin eine kleine `.runway-config.json` schreiben, um die Kompatibilität zu gewährleisten; bevorzugen Sie den `runway`-Schlüssel in `config.php` für die Zukunft.

### Projektrouterkennung

Runway ist intelligent genug, um das Root-Verzeichnis Ihres Projekts zu erkennen, auch wenn Sie es aus einem Unterverzeichnis ausführen. Es sucht nach Indikatoren wie `composer.json`, `.git` oder `app/config/config.php`, um zu bestimmen, wo sich das Projekt-Root befindet. Das bedeutet, dass Sie Runway-Befehle von überall in Ihrem Projekt ausführen können!

## Verwendung

Runway verfügt über eine Reihe von Befehlen, die Sie zur Verwaltung Ihrer Flight-Anwendung verwenden können. Es gibt zwei einfache Möglichkeiten, Runway zu verwenden.

1. Wenn Sie das Skeleton-Projekt verwenden, können Sie `php runway [Befehl]` aus dem Root Ihres Projekts ausführen.
1. Wenn Sie Runway als über Composer installiertes Paket verwenden, können Sie `vendor/bin/runway [Befehl]` aus dem Root Ihres Projekts ausführen.

### Befehlsliste

Sie können eine Liste aller verfügbaren Befehle anzeigen, indem Sie den Befehl `php runway` ausführen.

```bash
php runway
```

Verlassen Sie sich nur auf Befehle, die tatsächlich in dieser Liste für Ihre Installation erscheinen (Kern-Runway-Befehle vs. projektspezifische Befehle wie `migrate` des Skeletons).

### Befehlshilfe

Für jeden Befehl können Sie das Flag `--help` übergeben, um weitere Informationen zur Verwendung des Befehls zu erhalten.

```bash
php runway routes --help
php runway make:controller --help
```

Hier sind einige Beispiele:

### Controller generieren

`make:controller` erstellt ein Scaffold für einen Controller, der mit dem offiziellen Skeleton-Layout übereinstimmt:

| | |
|--|--|
| **Pfad** | `app/Controller/{Name}.php` |
| **Namespace** | `App\Controller` |
| **Stil** | Constructor-Injection von `flight\Engine` (kein `Flight::` im Klassenrumpf) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Beispiel der erwarteten Form (vereinfacht):

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// z.B. $this->app->render('…', […]);
	}
}
```

Registrieren Sie es mit einem Klassen-Callable, damit Dice den Controller erstellen kann:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Warum dieses Layout?** Die Ordner**groß-/Kleinschreibung** muss mit dem Namespace übereinstimmen (`Controller` nicht `controllers`) für Composer PSR-4 unter Linux—siehe [Autoloading](/learn/autoloading). Der gleiche Pfad ist es, den Root- und Scoped-`AGENTS.md`-Dateien KI-Tools mitteilen, sie zu verwenden, damit generierte und handgeschriebene Controller identisch bleiben.

> Ältere Dokumentationen und Community-Projekte verwendeten manchmal `app/controllers/` und `app\controllers`. Das bleibt gültig, wenn *Ihr* Baum weiterhin Kleinbuchstaben für Ordner verwendet. **Neue Skeleton-Projekte und aktuelle `make:controller`-Ausgaben verwenden `app/Controller/` + `App\Controller`.**

### Active Record Modell generieren

Stellen Sie zuerst sicher, dass Sie das [Active Record](/awesome-plugins/active-record) Plugin installiert haben.

```bash
php runway make:record users
```

Im offiziellen Skeleton leben Modelle unter **`app/Model/`** mit dem Namespace **`App\Model`**, und die DB-Verbindung ist **[SimplePdo](/learn/simple-pdo)** (injizieren Sie es oder übergeben Sie es an den ActiveRecord-Konstruktor). Generierte Dateinamen/Namespaces folgen den aktuellen Standardeinstellungen von Runway und Ihrer `runway`-Konfiguration—bevorzugen Sie die Ausrichtung neuer Modelle an `App\Model`, damit sie mit [Autoloading](/learn/autoloading) und `AGENTS.md` übereinstimmen.

Beispiel eines Modells, das mit der Skeleton-Posts-Demo übereinstimmt:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Wenn ein älterer Generator immer noch `app/records` / `app\records` ausgibt, können Sie diese Konvention in Legacy-Apps beibehalten oder Dateien in `app/Model/` verschieben und den Namespace an die Ordner-Groß-/Kleinschreibung anpassen.

### Migrationen (Skeleton)

Das offizielle Skeleton enthält einen Projektbefehl (entdeckt aus `app/commands/`), wie zum Beispiel:

```bash
php runway migrate
```

Migrationen sind SQL-Dateien unter `migrations/` (zum Beispiel `YYYYMMDDHHMMSS_description.sql` für SQLite und `…_description.mysql.sql` für MySQL), ausgewählt aus Ihrer Datenbanktreiber-Konfiguration / Umgebung. Exakte Flags und Verhalten werden von diesem Projektbefehl definiert—führen Sie `php runway migrate --help` in Ihrer App aus.

### KI-Helfer

Runway stellt KI-orientierte Befehle bereit, die mit [KI & Entwicklererfahrung](/learn/ai) verwendet werden:

```bash
php runway ai:init
php runway ai:generate-instructions
```

Diese speichern LLM-Anmeldedaten und generieren Projektanweisungen (hauptsächlich **`AGENTS.md`**). Beim Skeleton behandeln Sie `AGENTS.md` (und Scoped-Kopien unter `app/`) plus **`SECURITY.md`** als Quelle der Wahrheit für Agenten.

### Alle Routen anzeigen

Dies zeigt alle Routen an, die derzeit bei Flight registriert sind.

```bash
php runway routes
```

Wenn Sie nur bestimmte Routen ansehen möchten, können Sie ein Flag übergeben, um die Routen zu filtern.

```bash
# Nur GET-Routen anzeigen
php runway routes --get

# Nur POST-Routen anzeigen
php runway routes --post

# usw.
```

## Benutzerdefinierte Befehle zu Runway hinzufügen

Wenn Sie entweder ein Paket für Flight erstellen oder Ihre eigenen benutzerdefinierten Befehle in Ihr Projekt einfügen möchten, können Sie dies tun, indem Sie ein `src/commands/`, `flight/commands/`, `app/commands/` oder `commands/` Verzeichnis für Ihr Projekt/Paket erstellen. Wenn Sie weitere Anpassungen benötigen, siehe den Abschnitt unten zur Konfiguration.

Im Skeleton leben Projektbefehle in **`app/commands/`** mit dem Namespace **`App\Command`**. Runway entdeckt sie über den Pfad; halten Sie diesen Ordner synchron mit dem Composer-Classmap/PSR-4, wie Ihr Projekt es bereits tut.

Um einen Befehl zu erstellen, erweitern Sie einfach die Klasse `AbstractBaseCommand` und implementieren Sie mindestens eine `__construct`-Methode und eine `execute`-Methode.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * Konstruktor
     *
     * @param array<string,mixed> $config Konfiguration aus app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Erstellt ein Beispiel für die Dokumentation', $config);
        $this->argument('<funny-gif>', 'Der Name des lustigen Gifs');
    }

	/**
     * Führt die Funktion aus
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Erstelle Beispiel...');

		// Hier etwas tun

		$io->ok('Beispiel erstellt!');
	}
}
```

Siehe die [adhocore/php-cli Dokumentation](https://github.com/adhocore/php-cli) für weitere Informationen darüber, wie Sie Ihre eigenen benutzerdefinierten Befehle in Ihre Flight-Anwendung integrieren können!

## Konfigurationsverwaltung

Da die Konfiguration ab `v1.2.0` in `app/config/config.php` verschoben wurde, gibt es einige Hilfsbefehle zur Konfigurationsverwaltung.

> **Skeleton-Tipp:** Halten Sie `config.php` als **literalen** PHP-Wert. Geheimnisse gehören in `.env`. Vermeiden Sie `$_ENV[...]`-Ausdrücke innerhalb von `config.php`—`config:set` schreibt diese Datei als statische Daten um und könnte Geheimnisse in die Datei einbetten. Siehe [Konfiguration](/learn/configuration).

### Alte Konfiguration migrieren

Wenn Sie eine alte `.runway-config.json`-Datei haben, können Sie diese einfach mit dem folgenden Befehl zu `app/config/config.php` migrieren:

```bash
php runway config:migrate
```

### Konfigurationswert setzen

Sie können einen Konfigurationswert mit dem Befehl `config:set` setzen. Dies ist nützlich, wenn Sie einen Konfigurationswert aktualisieren möchten, ohne die Datei zu öffnen.

```bash
php runway config:set app_root "app/"
```

### Konfigurationswert abrufen

Sie können einen Konfigurationswert mit dem Befehl `config:get` abrufen.

```bash
php runway config:get app_root
```

## Alle Runway-Konfigurationen

Wenn Sie die Konfiguration für Runway anpassen müssen, können Sie diese Werte in `app/config/config.php` setzen. Hier sind einige zusätzliche Konfigurationen, die Sie setzen können:

```php
<?php
// app/config/config.php
return [
    // ... andere Konfigurationswerte ...

    'runway' => [
        // Dies ist der Ort, an dem sich Ihr Anwendungsverzeichnis befindet
        'app_root' => 'app/',

        // Dies ist das Verzeichnis, in dem sich Ihre Root-Indexdatei befindet
        'index_root' => 'public/',

        // Dies sind die Pfade zu den Roots anderer Projekte
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Basis-Pfade müssen wahrscheinlich nicht konfiguriert werden, aber es ist hier, wenn Sie es wollen
        'base_paths' => [
            '/includes/libs/vendor', // wenn Sie einen wirklich einzigartigen Pfad für Ihr Vendor-Verzeichnis oder so etwas haben
        ],

        // Finale Pfade sind Orte innerhalb eines Projekts, um nach den Befehlsdateien zu suchen
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Wenn Sie einfach den vollständigen Pfad hinzufügen möchten, nur zu (absolut oder relativ zum Projekt-Root)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Konfiguration zugreifen

Wenn Sie die Konfigurationswerte effektiv zugreifen müssen, können Sie über die `__construct`-Methode oder die `app()`-Methode darauf zugreifen. Es ist auch wichtig zu beachten, dass wenn Sie eine `app/config/services.php`-Datei haben, diese Dienste auch für Ihren Befehl verfügbar sein werden.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Konfiguration zugreifen
    $app_root = $this->config['runway']['app_root'];
    
    // Dienste zugreifen wie vielleicht eine Datenbankverbindung
    $database = $this->config['database']
    
    // ...
}
```

## KI-Helfer-Wrapper

Runway hat einige Helfer-Wrapper, die es KI erleichtern, Befehle zu generieren. Sie können `addOption` und `addArgument` auf eine Weise verwenden, die sich ähnlich wie Symfony Console anfühlt. Dies ist hilfreich, wenn Sie KI-Tools zur Generierung Ihrer Befehle verwenden.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Erstellt ein Beispiel für die Dokumentation', $config);
    
    // Das mode-Argument ist nullbar und standardmäßig vollständig optional
    $this->addOption('name', 'Der Name des Beispiels', null);
}
```

## Siehe auch

- [Installation](/install) - Skeleton-Baum und create-project-Standardwerte
- [Autoloading](/learn/autoloading) - `App\` und Ordner-Groß-/Kleinschreibung
- [Dependency Injection](/learn/dependency-injection-container) - Dice + Engine-Injection für generierte Controller
- [KI & Entwicklererfahrung](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Modelle verwendet mit `make:record` / Skeleton `App\Model`
- [SimplePdo](/learn/simple-pdo) - DB-Verbindung verwendet von Skeleton-Migrationen und -Modellen