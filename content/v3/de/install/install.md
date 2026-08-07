# Installationsanleitung

Es gibt einige grundlegende Voraussetzungen, bevor Sie Flight installieren können. Nämlich benötigen Sie:

1. [Installieren Sie PHP auf Ihrem System](#installing-php)
2. [Installieren Sie Composer](https://getcomposer.org) für die beste Entwicklererfahrung.

## Grundinstallation

Wenn Sie [Composer](https://getcomposer.org) verwenden, können Sie den folgenden Befehl ausführen:

```bash
composer require flightphp/core
```

Dadurch werden nur die Flight-Kerndateien auf Ihrem System abgelegt. Sie müssen die Projektstruktur, [Layout](/learn/templates), [Abhängigkeiten](/learn/dependency-injection-container), [Konfigurationen](/learn/configuration), [Autoloading](/learn/autoloading) usw. selbst definieren. Diese Methode stellt sicher, dass außer Flight keine weiteren Abhängigkeiten installiert werden.

Sie können die [Dateien auch herunterladen](https://github.com/flightphp/core/archive/master.zip) und direkt in Ihr Webverzeichnis entpacken.

Die Grundinstallation ist perfekt zum Lernen, für Micro-APIs und für Copy-Paste-Experimente. Für ein vollständiges App-Layout, das Menschen *und* [KI-Codierungswerkzeuge](/learn/ai) auf dieselbe Weise nachvollziehen können, verwenden Sie das unten empfohlene Grundgerüst.

## Empfohlene Installation

Es wird dringend empfohlen, für neue Projekte mit der [flightphp/skeleton](https://github.com/flightphp/skeleton)-App zu starten. Die Installation ist ein Kinderspiel.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# optionale Beispiel-DB + Posts-Demo
php runway migrate
```

Dieser Schritt richtet die Projektstruktur, das Composer-PSR-4-Autoloading, die Konfiguration sowie Werkzeuge wie [Tracy](/awesome-plugins/tracy), [Tracy Extensions](/awesome-plugins/tracy-extensions) und [Runway](/awesome-plugins/runway) ein. Außerdem wird eine **`AGENTS.md`** im Root-Verzeichnis (sowie bereichsbezogene Kopien unter `app/`) mitgeliefert, damit KI-Assistenten ein gemeinsames Layout mit Ihnen teilen – siehe [KI & Entwicklererfahrung](/learn/ai).

### Was das Grundgerüst Ihnen bietet

```text
project-root/
├── AGENTS.md              # KI / Agenten-Quelle der Wahrheit
├── SECURITY.md            # Sicherheitserwartungen
├── .env.example           # Geheimnisse / Deploy-Overlays (kopiert nach .env)
├── public/index.php       # Nur Web-Einstiegspunkt
├── app/
│   ├── config/            # Bootstrap, Routen, Services, config_sample.php
│   ├── Controller/        # App\Controller\*  (PascalCase-Ordner!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Runway-CLI-Befehle
│   ├── views/             # Twig-Templates (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # SQL-Migrationen (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Namespaces folgen der Ordner-Schreibweise.** Composer mappt `"App\\": "app/"`, also:

| Pfad auf der Festplatte | Namespace |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Auf Linux ist `app/controller/` **nicht** dasselbe wie `app/Controller/`. Das Autoloading unterscheidet zwischen Groß- und Kleinschreibung – verwenden Sie die PascalCase-Ordner des Grundgerüsts. Details: [Autoloading](/learn/autoloading).

**Standard-Stack (neue Projekte):** Twig-Views, SimplePdo + ActiveRecord, Dice mit `Engine`-Injektion (bevorzugen Sie kein `Flight::` innerhalb von App-Klassen), optional SQLite nach `php runway migrate`.

`create-project` kopiert typischerweise `app/config/config_sample.php` → `config.php` und `.env.example` → `.env`, falls vorhanden. Routen leben in `app/config/routes.php`; Services und DI leben in `app/config/services.php`.

> **Dokumentation ↔ Grundgerüst:** Diese Dokumentation lehrt die Flight-**APIs** (oft mit kurzen `Flight::`-Beispielen). Das Grundgerüst legt die **Anwendungsstruktur** fest. Wenn Sie Code unter `app/` hinzufügen, folgen Sie dem Grundgerüstbaum; nutzen Sie die Dokumentation für Methodennamen, Optionen und Plugins.

## Konfigurieren Sie Ihren Webserver

### Integrierter PHP-Entwicklungsserver

Dies ist bei weitem der einfachste Weg, um loszulegen. Sie können den integrierten Server verwenden, um Ihre Anwendung auszuführen, und sogar SQLite als Datenbank nutzen (solange sqlite3 auf Ihrem System installiert ist), ohne viel zu benötigen! Führen Sie einfach den folgenden Befehl aus, sobald PHP installiert ist:

```bash
php -S localhost:8000
# oder mit der Grundgerüst-App
composer start
```

Öffnen Sie dann Ihren Browser und gehen Sie zu `http://localhost:8000`.

Wenn Sie das Dokumentenverzeichnis Ihres Projekts in ein anderes Verzeichnis legen möchten (z. B. Ihr Projekt ist `~/myproject`, aber Ihr Dokumentenverzeichnis ist `~/myproject/public/`), können Sie den folgenden Befehl ausführen, sobald Sie sich im Verzeichnis `~/myproject` befinden:

```bash
php -S localhost:8000 -t public/
# bei der Grundgerüst-App ist dies bereits konfiguriert
composer start
```

Öffnen Sie dann Ihren Browser und gehen Sie zu `http://localhost:8000`.

### Apache

Stellen Sie sicher, dass Apache bereits auf Ihrem System installiert ist. Wenn nicht, googeln Sie, wie Sie Apache auf Ihrem System installieren.

Bearbeiten Sie für Apache Ihre `.htaccess`-Datei mit folgendem Inhalt:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Hinweis**: Wenn Sie Flight in einem Unterverzeichnis verwenden müssen, fügen Sie die Zeile
> `RewriteBase /subdir/` direkt nach `RewriteEngine On` hinzu.

> **Hinweis**: Wenn Sie alle Serverdateien schützen möchten, z. B. eine DB- oder Umgebungsdatei.
> Fügen Sie dies in Ihre `.htaccess`-Datei ein:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Stellen Sie sicher, dass Nginx bereits auf Ihrem System installiert ist. Wenn nicht, googeln Sie, wie Sie Nginx auf Ihrem System installieren.

Für Nginx fügen Sie Ihrer Server-Deklaration Folgendes hinzu:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Erstellen Sie Ihre `index.php`-Datei

Wenn Sie eine Grundinstallation durchführen, benötigen Sie etwas Code, um loszulegen.

```php
<?php

// Wenn Sie Composer verwenden, binden Sie den Autoloader ein.
require 'vendor/autoload.php';
// Wenn Sie Composer nicht verwenden, laden Sie das Framework direkt
// require 'flight/Flight.php';

// Definieren Sie dann eine Route und weisen Sie eine Funktion zur Bearbeitung der Anfrage zu.
Flight::route('/', function () {
  echo 'hello world!';
});

// Starten Sie schließlich das Framework.
Flight::start();
```

Bei der Grundgerüst-App bootet der öffentliche Einstiegspunkt nur die Anwendung. Routen werden in `app/config/routes.php` registriert (typischerweise `[App\Controller\…::class, 'method']`, damit Dice Abhängigkeiten injizieren kann). Services, Twig, SimplePdo und der Container werden in `app/config/services.php` verdrahtet. Diese Struktur ist beabsichtigt, damit KI-Tools und Menschen jedes Mal dieselben Stellen bearbeiten.

## PHP installieren

Wenn auf Ihrem System bereits `php` installiert ist, überspringen Sie diese Anweisungen und fahren Sie mit [dem Download-Abschnitt](#download-the-files) fort.

### **macOS**

#### **PHP mit Homebrew installieren**

1. **Homebrew installieren** (falls nicht bereits installiert):
   - Öffnen Sie das Terminal und führen Sie aus:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **PHP installieren**:
   - Installieren Sie die neueste Version:
     ```bash
     brew install php
     ```
   - Um eine bestimmte Version zu installieren, z. B. PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Zwischen PHP-Versionen wechseln**:
   - Entfernen Sie die Verknüpfung der aktuellen Version und verknüpfen Sie die gewünschte Version:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Überprüfen Sie die installierte Version:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **PHP manuell installieren**

1. **PHP herunterladen**:
   - Besuchen Sie [PHP für Windows](https://windows.php.net/download/) und laden Sie die neueste oder eine bestimmte Version (z. B. 7.4, 8.0) als Nicht-Thread-sichere ZIP-Datei herunter.

2. **PHP entpacken**:
   - Entpacken Sie die heruntergeladene ZIP-Datei nach `C:\php`.

3. **PHP zum System-PATH hinzufügen**:
   - Gehen Sie zu **Systemeigenschaften** > **Umgebungsvariablen**.
   - Suchen Sie unter **Systemvariablen** den Eintrag **Path** und klicken Sie auf **Bearbeiten**.
   - Fügen Sie den Pfad `C:\php` (oder den Ort, an den Sie PHP entpackt haben) hinzu.
   - Klicken Sie auf **OK**, um alle Fenster zu schließen.

4. **PHP konfigurieren**:
   - Kopieren Sie `php.ini-development` nach `php.ini`.
   - Bearbeiten Sie `php.ini`, um PHP nach Bedarf zu konfigurieren (z. B. `extension_dir` festlegen, Erweiterungen aktivieren).

5. **PHP-Installation überprüfen**:
   - Öffnen Sie die Eingabeaufforderung und führen Sie aus:
     ```cmd
     php -v
     ```

#### **Mehrere PHP-Versionen installieren**

1. **Wiederholen Sie die obigen Schritte** für jede Version und platzieren Sie jede in einem separaten Verzeichnis (z. B. `C:\php7`, `C:\php8`).

2. **Wechseln Sie zwischen den Versionen**, indem Sie die System-PATH-Variable so anpassen, dass sie auf das gewünschte Versionsverzeichnis zeigt.

### **Ubuntu (20.04, 22.04 usw.)**

#### **PHP mit apt installieren**

1. **Paketlisten aktualisieren**:
   - Öffnen Sie das Terminal und führen Sie aus:
     ```bash
     sudo apt update
     ```

2. **PHP installieren**:
   - Installieren Sie die neueste PHP-Version:
     ```bash
     sudo apt install php
     ```
   - Um eine bestimmte Version zu installieren, z. B. PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Zusätzliche Module installieren** (optional):
   - Um beispielsweise MySQL-Unterstützung zu installieren:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Zwischen PHP-Versionen wechseln**:
   - Verwenden Sie `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Installierte Version überprüfen**:
   - Führen Sie aus:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **PHP mit yum/dnf installieren**

1. **EPEL-Repository aktivieren**:
   - Öffnen Sie das Terminal und führen Sie aus:
     ```bash
     sudo dnf install epel-release
     ```

2. **Remi-Repository installieren**:
   - Führen Sie aus:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **PHP installieren**:
   - Um die Standardversion zu installieren:
     ```bash
     sudo dnf install php
     ```
   - Um eine bestimmte Version zu installieren, z. B. PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Zwischen PHP-Versionen wechseln**:
   - Verwenden Sie den `dnf`-Modulbefehl:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Installierte Version überprüfen**:
   - Führen Sie aus:
     ```bash
     php -v
     ```

### **Allgemeine Hinweise**

- Für Entwicklungsumgebungen ist es wichtig, die PHP-Einstellungen entsprechend den Anforderungen Ihres Projekts zu konfigurieren.
- Wenn Sie PHP-Versionen wechseln, stellen Sie sicher, dass alle relevanten PHP-Erweiterungen für die jeweilige Version installiert sind, die Sie verwenden möchten.
- Starten Sie Ihren Webserver (Apache, Nginx usw.) nach dem Wechsel der PHP-Versionen oder dem Aktualisieren von Konfigurationen neu, um die Änderungen zu übernehmen.