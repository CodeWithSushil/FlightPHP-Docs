# Autoloading

## Overview

Autoloading is a concept in PHP where you specify a directory or directories to load classes from. This is much more beneficial than using `require` or `include` to load classes. It is also a requirement for using Composer packages.

Getting autoloading right matters for [AI-assisted development](/learn/ai) too: agents place files where the namespace points. If folder **case** and namespace case disagree, class-not-found errors show up on Linux even when things "worked" on a case-insensitive Mac disk.

## Understanding

By default, any `Flight` class is autoloaded for you automatically thanks to Composer. For **your** application classes you have two common approaches:

1. **Composer PSR-4** (what the [official skeleton](https://github.com/flightphp/skeleton) uses): map a namespace prefix to a directory in `composer.json`, then `composer dump-autoload`.
2. **`Flight::path()`**: point Flight's loader at directories (handy for simple apps or when you are not using Composer for app code).

Using an autoloader simplifies your code a lot. Instead of a wall of `include` / `require` at the top of every file, classes load when you first use them.

### Case sensitivity (read this twice)

**Namespaces must match the directory structure *and* the letter case of those directories.**

| Works | Breaks on Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` with folder `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Mixing `App\` with lowercase `controllers` |

PHP namespaces are case-insensitive in some contexts, but **Composer and the filesystem are not**. The official skeleton standardizes on:

- Composer: `"App\\": "app/"`
- Folders: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), not `controllers` / `middlewares`

Older docs and community examples sometimes used lowercase `app\controllers`. That still works if your folders are lowercase—but **new skeleton projects use `App\` + PascalCase folders**. Pick one convention per project and stick to it so humans and AI tools do not invent a second layout.

## Skeleton (recommended for new projects)

After `composer create-project flightphp/skeleton`, app code is autoloaded via Composer—no `Flight::path()` required for `App\` classes:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Dice resolves App\Controller\… via the container
$router->get('/', [HomeController::class, 'index']);
```

See [Installation](/install) for the full tree and [AI & developer experience](/learn/ai) for how `AGENTS.md` documents this layout for coding assistants.

## Basic Usage (`Flight::path()`)

Let's assume we have a directory tree like the following:

```text
# Example path
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - contains the controllers for this project
│   ├── translations
│   ├── UTILS - contains classes for just this application (this is all caps on purpose for an example later)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

You may have noticed that this is similar to a typical app tree (the docs site itself uses a structured layout). Lowercase `controllers` here is a valid *choice*—it is just not the skeleton's current default.

You can specify each directory to load from like this:

```php

/**
 * public/index.php
 */

// Add a path to the autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// no namespacing required

// All autoloaded classes are recommended to be Pascal Case (each word capitalized, no spaces)
class MyController {

	public function index() {
		// do something
	}
}
```

## Namespaces with `Flight::path()`

If you do have namespaces, it actually becomes very easy to implement this. You should use the `Flight::path()` method to specify the root directory (not the document root or `public/` folder) of your application.

```php

/**
 * public/index.php
 */

// Add a path to the autoloader
Flight::path(__DIR__.'/../');
```

Now this is what your controller might look like. Look at the example below, but pay attention to the comments for important information.

```php
/**
 * app/controllers/MyController.php
 */

// namespaces are required
// namespaces are the same as the directory structure
// namespaces must follow the same case as the directory structure
// namespaces and directories cannot have any underscores (unless Loader::setV2ClassLoading(false) is set)
namespace app\controllers;

// All autoloaded classes are recommended to be Pascal Case (each word capitalized, no spaces)
// As of 3.7.2, you can use Pascal_Snake_Case for your class names by running Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// do something
	}
}
```

And if you wanted to autoload a class in your utils directory, you would do basically the same thing:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// namespace must match the directory structure and case (note the UTILS directory is all caps
//     like in the file tree above)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// do something
	}
}
```

### Skeleton-style namespace (same rules, different case)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

The rule did not change—only the skeleton's chosen folder/namespace casing. **Whatever case your folders use, your `namespace` line must match.**

## Underscores in Class Names

As of 3.7.2, you can use Pascal_Snake_Case for your class names by running `Loader::setV2ClassLoading(false);`. 
This will allow you to use underscores in your class names. 
This is not recommended, but it is available for those who need it.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Add a path to the autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// no namespacing required

class My_Controller {

	public function index() {
		// do something
	}
}
```

## See Also
- [Installation](/install) - Skeleton tree and `App\` defaults for new projects.
- [Routing](/learn/routing) - How to map routes to controllers and render views.
- [Dependency Injection](/learn/dependency-injection-container) - How controllers get `Engine` and services.
- [AI & Developer Experience](/learn/ai) - Keep agents aligned with your layout via `AGENTS.md`.
- [Why a Framework?](/learn/why-frameworks) - Understanding the benefits of using a framework like Flight.

## Troubleshooting
- If you can't seem to figure out why your namespaced classes aren't being found, remember: with `Flight::path()`, point at the **project root** (or the correct base for your namespace), not only a nested folder you forgot to mirror in the namespace.
- With Composer PSR-4, run `composer dump-autoload` after changing `composer.json` mappings.
- On Linux CI or production, a wrong folder case is a very common "works on my machine" failure.

### Class Not Found (autoloading not working)

There could be a couple reasons for this one not happening. Below are some examples.

#### Incorrect File Name
The most common is that the class name doesn't match the file name.

If you have a class named `MyClass` then the file should be named `MyClass.php`. If you have a class named `MyClass` and the file is named `myclass.php` 
then the autoloader won't be able to find it.

#### Incorrect Namespace or Folder Case
If you are using namespaces, then the namespace should match the directory structure **including case**.

```php
// ...code...

// if your MyController is in app/Controller (skeleton) and namespaced App\Controller
// this will not work:
Flight::route('/hello', 'MyController->hello');

// Skeleton-style:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Older lowercase layout (only if your folders are actually app/controllers):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// or fully qualified:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` not defined (non-Composer app code)

If you rely on `Flight::path()` instead of Composer for application classes, define the path before routes that reference those classes (often early in bootstrap / `public/index.php`):

```php
// Add a path to the autoloader (project root for namespaced apps)
Flight::path(__DIR__.'/../');
```

The official skeleton primarily uses **Composer PSR-4** for `App\`, so you usually will not need `Flight::path()` for controllers and models there.

## Changelog
- Docs – Document skeleton `App\` + PascalCase folders and case-sensitivity pitfalls for humans and AI tools.
- v3.7.2 - You can use Pascal_Snake_Case for your class names by running `Loader::setV2ClassLoading(false);`
- v2.0 - Autoload functionality added.
