# Instructions d'installation

Il y a quelques prérequis de base avant de pouvoir installer Flight. Vous aurez notamment besoin de :

1. [Installer PHP sur votre système](#installing-php)
2. [Installer Composer](https://getcomposer.org) pour la meilleure expérience de développement.

## Installation de base

Si vous utilisez [Composer](https://getcomposer.org), vous pouvez exécuter la commande suivante :

```bash
composer require flightphp/core
```

Cela placera uniquement les fichiers principaux de Flight sur votre système. Vous devrez définir la structure du projet, la [mise en page](/learn/templates), les [dépendances](/learn/dependency-injection-container), les [configurations](/learn/configuration), l'[autoloading](/learn/autoloading), etc. Cette méthode garantit qu'aucune autre dépendance que Flight n'est installée.

Vous pouvez également [télécharger les fichiers](https://github.com/flightphp/core/archive/master.zip) directement et les extraire dans votre répertoire web.

L'installation de base est parfaite pour apprendre, pour les micro-API et pour les expérimentations copier-coller. Pour une structure d'application complète que les humains *et* les [outils de codage IA](/learn/ai) peuvent suivre de la même manière, utilisez le squelette recommandé ci-dessous.

## Installation recommandée

Il est fortement recommandé de commencer avec l'application [flightphp/skeleton](https://github.com/flightphp/skeleton) pour tout nouveau projet. L'installation est un jeu d'enfant.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# base de données exemple facultative + démo des posts
php runway migrate
```

Cette étape configure la structure du projet, l'autoloading Composer PSR-4, la configuration, et des outils comme [Tracy](/awesome-plugins/tracy), [Tracy Extensions](/awesome-plugins/tracy-extensions), et [Runway](/awesome-plugins/runway). Elle fournit également **`AGENTS.md`** à la racine (et des copies dédiées sous `app/`) afin que les assistants IA partagent une même structure avec vous—voir [IA & expérience développeur](/learn/ai).

### Ce que le squelette vous offre

```text
project-root/
├── AGENTS.md              # Source de vérité pour l'IA / les agents
├── SECURITY.md            # Attentes en matière de sécurité
├── .env.example           # Secrets / superpositions de déploiement (copié vers .env)
├── public/index.php       # Point d'entrée web uniquement
├── app/
│   ├── config/            # amorçage, routes, services, config_sample.php
│   ├── Controller/        # App\Controller\*  (dossier PascalCase !)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # Commandes CLI de Runway
│   ├── views/             # Modèles Twig (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # Migrations SQL (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Les espaces de noms suivent la casse du dossier.** Composer mappe `"App\\": "app/"`, donc :

| Chemin sur disque | Espace de noms |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

Sous Linux, `app/controller/` n'est **pas** la même chose que `app/Controller/`. L'autoloading est sensible à la casse—respectez les dossiers PascalCase du squelette. Détails : [Autoloading](/learn/autoloading).

**Pile par défaut (nouveaux projets) :** vues Twig, SimplePdo + ActiveRecord, Dice avec injection `Engine` (préférez l'absence de `Flight::` dans les classes applicatives), SQLite optionnelle après `php runway migrate`.

`create-project` copie généralement `app/config/config_sample.php` → `config.php` et `.env.example` → `.env` lorsqu'ils sont présents. Les routes se trouvent dans `app/config/routes.php` ; les services et l'injection de dépendances se trouvent dans `app/config/services.php`.

> **Documentation ↔ squelette :** Ces documentations enseignent les **API** de Flight (souvent avec de courts exemples `Flight::`). Le squelette fixe la **forme de l'application**. Lorsque vous ajoutez du code sous `app/`, suivez l'arborescence du squelette ; utilisez la documentation pour les noms de méthodes, les options et les plugins.

## Configurer votre serveur web

### Serveur de développement PHP intégré

C'est de loin le moyen le plus simple de démarrer. Vous pouvez utiliser le serveur intégré pour exécuter votre application et même utiliser SQLite comme base de données (tant que sqlite3 est installé sur votre système) sans presque rien configurer ! Exécutez simplement la commande suivante une fois PHP installé :

```bash
php -S localhost:8000
# ou avec l'application squelette
composer start
```

Ensuite, ouvrez votre navigateur et allez sur `http://localhost:8000`.

Si vous souhaitez définir la racine des documents de votre projet dans un répertoire différent (par exemple, votre projet est `~/myproject`, mais votre racine des documents est `~/myproject/public/`), vous pouvez exécuter la commande suivante une fois que vous êtes dans le répertoire `~/myproject` :

```bash
php -S localhost:8000 -t public/
# avec l'application squelette, cela est déjà configuré
composer start
```

Ensuite, ouvrez votre navigateur et allez sur `http://localhost:8000`.

### Apache

Assurez-vous qu'Apache est déjà installé sur votre système. Si ce n'est pas le cas, recherchez sur Google comment installer Apache sur votre système.

Pour Apache, modifiez votre fichier `.htaccess` avec le contenu suivant :

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Remarque** : Si vous devez utiliser Flight dans un sous-répertoire, ajoutez la ligne
> `RewriteBase /subdir/` juste après `RewriteEngine On`.

> **Remarque** : Si vous voulez protéger tous les fichiers serveur, comme un fichier de base de données ou un fichier d'environnement.
> Placez ceci dans votre fichier `.htaccess` :

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Assurez-vous que Nginx est déjà installé sur votre système. Si ce n'est pas le cas, recherchez sur Google comment installer Nginx sur votre système.

Pour Nginx, ajoutez ce qui suit à votre déclaration de serveur :

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Créez votre fichier `index.php`

Si vous effectuez une installation de base, vous aurez besoin d'un peu de code pour démarrer.

```php
<?php

// Si vous utilisez Composer, chargez l'autoloader.
require 'vendor/autoload.php';
// si vous n'utilisez pas Composer, chargez le framework directement
// require 'flight/Flight.php';

// Définissez ensuite une route et assignez une fonction pour gérer la requête.
Flight::route('/', function () {
  echo 'hello world!';
});

// Enfin, démarrez le framework.
Flight::start();
```

Avec l'application squelette, le point d'entrée public ne fait que démarrer l'application. Les routes sont enregistrées dans `app/config/routes.php` (généralement `[App\Controller\…::class, 'method']` afin que Dice puisse injecter les dépendances). Les services, Twig, SimplePdo et le conteneur sont câblés dans `app/config/services.php`. Cette structure est intentionnelle pour que les outils d'IA et les humains modifient les mêmes endroits à chaque fois.

## Installer PHP

Si vous avez déjà `php` installé sur votre système, passez ces instructions et rendez-vous à [la section téléchargement](#download-the-files)

### **macOS**

#### **Installer PHP avec Homebrew**

1. **Installer Homebrew** (s'il n'est pas déjà installé) :
   - Ouvrez Terminal et exécutez :
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Installer PHP** :
   - Installez la dernière version :
     ```bash
     brew install php
     ```
   - Pour installer une version spécifique, par exemple PHP 8.1 :
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Basculer entre les versions de PHP** :
   - Désactivez la version actuelle et activez la version souhaitée :
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Vérifiez la version installée :
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **Installer PHP manuellement**

1. **Télécharger PHP** :
   - Visitez [PHP for Windows](https://windows.php.net/download/) et téléchargez la dernière version ou une version spécifique (par exemple, 7.4, 8.0) sous forme de fichier zip non thread-safe.

2. **Extraire PHP** :
   - Extrayez le fichier zip téléchargé vers `C:\php`.

3. **Ajouter PHP au PATH du système** :
   - Allez dans **Propriétés système** > **Variables d'environnement**.
   - Sous **Variables système**, trouvez **Path** et cliquez sur **Modifier**.
   - Ajoutez le chemin `C:\php` (ou l'endroit où vous avez extrait PHP).
   - Cliquez sur **OK** pour fermer toutes les fenêtres.

4. **Configurer PHP** :
   - Copiez `php.ini-development` vers `php.ini`.
   - Modifiez `php.ini` pour configurer PHP selon vos besoins (par exemple, définir `extension_dir`, activer des extensions).

5. **Vérifier l'installation de PHP** :
   - Ouvrez l'invite de commandes et exécutez :
     ```cmd
     php -v
     ```

#### **Installer plusieurs versions de PHP**

1. **Répétez les étapes ci-dessus** pour chaque version, en plaçant chacune dans un répertoire séparé (par exemple, `C:\php7`, `C:\php8`).

2. **Basculez entre les versions** en ajustant la variable d'environnement PATH du système pour pointer vers le répertoire de la version souhaitée.

### **Ubuntu (20.04, 22.04, etc.)**

#### **Installer PHP avec apt**

1. **Mettre à jour les listes de paquets** :
   - Ouvrez Terminal et exécutez :
     ```bash
     sudo apt update
     ```

2. **Installer PHP** :
   - Installez la dernière version de PHP :
     ```bash
     sudo apt install php
     ```
   - Pour installer une version spécifique, par exemple PHP 8.1 :
     ```bash
     sudo apt install php8.1
     ```

3. **Installer des modules supplémentaires** (facultatif) :
   - Par exemple, pour installer le support MySQL :
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Basculer entre les versions de PHP** :
   - Utilisez `update-alternatives` :
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Vérifiez la version installée** :
   - Exécutez :
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **Installer PHP avec yum/dnf**

1. **Activer le dépôt EPEL** :
   - Ouvrez Terminal et exécutez :
     ```bash
     sudo dnf install epel-release
     ```

2. **Installer le dépôt de Remi** :
   - Exécutez :
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Installer PHP** :
   - Pour installer la version par défaut :
     ```bash
     sudo dnf install php
     ```
   - Pour installer une version spécifique, par exemple PHP 7.4 :
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Basculer entre les versions de PHP** :
   - Utilisez la commande de module `dnf` :
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Vérifiez la version installée** :
   - Exécutez :
     ```bash
     php -v
     ```

### **Remarques générales**

- Pour les environnements de développement, il est important de configurer les paramètres PHP en fonction des exigences de votre projet.
- Lorsque vous changez de version de PHP, assurez-vous que toutes les extensions PHP pertinentes sont installées pour la version spécifique que vous comptez utiliser.
- Redémarrez votre serveur web (Apache, Nginx, etc.) après avoir changé de version de PHP ou mis à jour les configurations pour appliquer les changements.