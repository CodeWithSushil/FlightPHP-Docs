# Runway

Runway is a CLI application that helps you manage your Flight applications. It can generate controllers, display all routes, run AI setup helpers, migrations (in the skeleton), and more. It is based on the excellent [adhocore/php-cli](https://github.com/adhocore/php-cli) library.

Click [here](https://github.com/flightphp/runway) to view the code.

Scaffolding commands are intentionally aligned with the [official skeleton](https://github.com/flightphp/skeleton) so [AI coding tools](/learn/ai) and humans get the same paths, namespaces, and constructor-injection style every time.

## Installation

Install with composer.

```bash
composer require flightphp/runway
```

The skeleton already depends on Runway; use `php runway` from the project root.

## Basic Configuration

The first time you run Runway, it will try and find a `runway` configuration in `app/config/config.php` via the `'runway'` key.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// optional; skeleton also uses index_root for the public entry
		'index_root' => 'public/index.php',
    ],
];
```

> **NOTE** - As of **v1.2.0**, `.runway-config.json` is deprecated in favor of `app/config/config.php`. Migrate with `php runway config:migrate` when upgrading older projects. The skeleton may still write a small `.runway-config.json` on create-project for compatibility; prefer the `runway` key in `config.php` going forward.

### Project Root Detection

Runway is smart enough to detect the root of your project, even if you run it from a subdirectory. It looks for indicators like `composer.json`, `.git`, or `app/config/config.php` to determine where the project root is. This means you can run Runway commands from anywhere in your project! 

## Usage

Runway has a number of commands that you can use to manage your Flight application. There are two easy ways to use Runway.

1. If you are using the skeleton project, you can run `php runway [command]` from the root of your project.
1. If you are using Runway as a package installed via composer, you can run `vendor/bin/runway [command]` from the root of your project.

### Command List

You can view a list of all available commands by running the `php runway` command.

```bash
php runway
```

Only rely on commands that actually appear in that list for your install (core Runway commands vs project-specific ones like the skeleton’s `migrate`).

### Command Help

For any command, you can pass in the `--help` flag to get more information on how to use the command.

```bash
php runway routes --help
php runway make:controller --help
```

Here are a few examples:

### Generate a Controller

`make:controller` scaffolds a controller that matches the official skeleton layout:

| | |
|--|--|
| **Path** | `app/Controller/{Name}.php` |
| **Namespace** | `App\Controller` |
| **Style** | Constructor injection of `flight\Engine` (no `Flight::` in the class body) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Example of the shape you should expect (simplified):

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
		// e.g. $this->app->render('…', […]);
	}
}
```

Register it with a class callable so Dice can build the controller:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Why this layout?** Folder **case** must match the namespace (`Controller` not `controllers`) for Composer PSR-4 on Linux—see [Autoloading](/learn/autoloading). The same path is what root and scoped `AGENTS.md` files tell AI tools to use, so generated and hand-written controllers stay identical.

> Older docs and community projects sometimes used `app/controllers/` and `app\controllers`. That remains valid if *your* tree still uses lowercase folders. **New skeleton projects and current `make:controller` output use `app/Controller/` + `App\Controller`.**

### Generate an Active Record Model

First make sure you've installed the [Active Record](/awesome-plugins/active-record) plugin.

```bash
php runway make:record users
```

In the official skeleton, models live under **`app/Model/`** with namespace **`App\Model`**, and the DB connection is **[SimplePdo](/learn/simple-pdo)** (inject it or pass it into the ActiveRecord constructor). Generated file names/namespaces follow Runway’s current defaults and your `runway` config—prefer aligning new models with `App\Model` so they match [autoloading](/learn/autoloading) and `AGENTS.md`.

Example of a model consistent with the skeleton posts demo:

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

If an older generator still emits `app/records` / `app\records`, you can keep that convention in legacy apps or move files into `app/Model/` and update the namespace to match the folder case.

### Migrations (skeleton)

The official skeleton ships a project command (discovered from `app/commands/`) such as:

```bash
php runway migrate
```

Migrations are SQL files under `migrations/` (for example `YYYYMMDDHHMMSS_description.sql` for SQLite and `…_description.mysql.sql` for MySQL), selected from your database driver config / env. Exact flags and behavior are defined by that project command—run `php runway migrate --help` in your app.

### AI helpers

Runway exposes AI-oriented commands used with [AI & developer experience](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

These store LLM credentials and generate project instructions (primarily **`AGENTS.md`**). On the skeleton, treat `AGENTS.md` (and scoped copies under `app/`) plus **`SECURITY.md`** as the source of truth for agents.

### Display All Routes

This will display all of the routes that are currently registered with Flight.

```bash
php runway routes
```

If you would like to only view specific routes, you can pass in a flag to filter the routes.

```bash
# Display only GET routes
php runway routes --get

# Display only POST routes
php runway routes --post

# etc.
```

## Adding Custom Commands to Runway

If you are either creating a package for Flight, or want to add your own custom commands into your project, you can do so by creating a `src/commands/`, `flight/commands/`, `app/commands/`, or `commands/` directory for your project/package. If you need further customization, see the section below on Configuration.

In the skeleton, project commands live in **`app/commands/`** with namespace **`App\Command`**. Runway discovers them by path; keep that folder in sync with Composer classmap/PSR-4 as your project already does.

To create a command, you simple extend the `AbstractBaseCommand` class, and implement at a minimum a `__construct` method and an `execute` method.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * Construct
     *
     * @param array<string,mixed> $config Config from app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Create an example for the documentation', $config);
        $this->argument('<funny-gif>', 'The name of the funny gif');
    }

	/**
     * Executes the function
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Creating example...');

		// Do something here

		$io->ok('Example created!');
	}
}
```

See the [adhocore/php-cli Documentation](https://github.com/adhocore/php-cli) for more information on how to build your own custom commands into your Flight application!

## Configuration Management

Since configuration has moved to `app/config/config.php` as of `v1.2.0`, there are a few helper commands to manage configuration.

> **Skeleton tip:** Keep `config.php` as **literal** PHP values. Secrets belong in `.env`. Avoid `$_ENV[...]` expressions inside `config.php`—`config:set` rewrites that file as static data and could bake secrets into the file. See [Configuration](/learn/configuration).

### Migrate Old Config

If you have an old `.runway-config.json` file, you can easily migrate it to `app/config/config.php` with the following command:

```bash
php runway config:migrate
```

### Set Configuration Value

You can set a configuration value using the `config:set` command. This is useful if you want to update a configuration value without opening the file.

```bash
php runway config:set app_root "app/"
```

### Get Configuration Value

You can get a configuration value using the `config:get` command.

```bash
php runway config:get app_root
```

## All Runway Configurations

If you need to customize the configuration for Runway, you can set these values in `app/config/config.php`. Below are some additional configurations that you can set:

```php
<?php
// app/config/config.php
return [
    // ... other config values ...

    'runway' => [
        // This is where your application directory is located
        'app_root' => 'app/',

        // This is the directory where your root index file is located
        'index_root' => 'public/',

        // These are the paths to the roots of other projects
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Base paths most likely don't need to be configured, but it's here if you want it
        'base_paths' => [
            '/includes/libs/vendor', // if you have a really unique path for your vendor directory or something
        ],

        // Final paths are locations within a project to search for the command files
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // If you want to just add the full path, go right ahead (absolute or relative to project root)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Accessing Configuration

If you need to access the configuration values effectively, you can access them through the `__construct` method or the `app()` method. It is also important to note that if you have a `app/config/services.php` file, those services will also be available to your command.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Access configuration
    $app_root = $this->config['runway']['app_root'];
    
    // Access services like maybe a database connection
    $database = $this->config['database']
    
    // ...
}
```

## AI Helper Wrappers

Runway has some helper wrappers that make it easier for AI to generate commands. You can use `addOption` and `addArgument` in a way that feels similar to Symfony Console. This is helpful if you are using AI tools to generate your commands.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Create an example for the documentation', $config);
    
    // The mode argument is nullable and defaults to completely optional
    $this->addOption('name', 'The name of the example', null);
}
```

## See Also

- [Installation](/install) - Skeleton tree and create-project defaults
- [Autoloading](/learn/autoloading) - `App\` and folder case
- [Dependency Injection](/learn/dependency-injection-container) - Dice + Engine injection for generated controllers
- [AI & Developer Experience](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Models used with `make:record` / skeleton `App\Model`
- [SimplePdo](/learn/simple-pdo) - DB connection used by skeleton migrations and models
