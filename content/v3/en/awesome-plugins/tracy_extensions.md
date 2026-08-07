# Tracy Flight Panel Extensions

This is a set of extensions to make working with Flight a little richer.

- **Flight** - Analyze all Flight variables.
- **Database** - Analyze all queries that have run on the page (if you correctly initiate the database connection)
- **Request** - Analyze all `$_SERVER` variables and examine all global payloads (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Analyze all `$_SESSION` variables if sessions are active.
- **Twig** *(optional)* - Analyze Twig template render time, memory, and which templates/blocks/macros ran (requires `twig/twig` and a `twig_profile` config)

This is especially handy with the [official skeleton](https://github.com/flightphp/skeleton), which defaults to Twig: the same layout [AI tools](/learn/ai) follow also shows up clearly on the Tracy bar.

This is the Panel

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

And each panel displays very helpful information about your application!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Click [here](https://github.com/flightphp/tracy-extensions) to view the code.

## Installation

Run `composer require flightphp/tracy-extensions --dev` and you're on your way!

Twig is **not** a hard dependency of the package. Install `twig/twig` only if you want the Twig panel (the skeleton already does for views).

## Configuration

There is very little configuration you need to do to get this started. You will need to initiate the Tracy debugger prior to using this [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// bootstrap code
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// You may need to specify your environment with Debugger::enable(Debugger::DEVELOPMENT)

// if you use database connections in your app, there is a 
// required PDO wrapper to use ONLY IN DEVELOPMENT (not production please!)
// It has the same parameters as a regular PDO connection
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// or if you attach this to the Flight framework
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// now whenever you make a query it will capture the time, query, and parameters

// This connects the dots
if(Debugger::$showBar === true) {
	// This needs to be false or Tracy can't actually render :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// more code

Flight::start();
```

## Additional Configuration

### Session Data

If you have a custom session handler (such as ghostff/session), you can pass any array of session data to Tracy and it will automatically output it for you. You pass it in with the `session_data` key in the second parameter of the `TracyExtensionLoader` constructor.

```php

use Ghostff\Session\Session;
// or use flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// This needs to be false or Tracy can't actually render :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// routes and other things...

Flight::start();
```

### Twig panel (optional)

If your app uses [Twig](/awesome-plugins/twig) (including the official skeleton), you can show template metrics on the Tracy bar. Create a Twig `Profile`, attach `ProfilerExtension` to your environment, then pass that profile into the loader under the **`twig_profile`** key. Attach profiling only in development.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Optional: expose Tracy dump helpers in templates
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Map Flight::render() to Twig (example)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**What the panel shows**

- Total Twig render time and memory
- Template / block / macro call counts
- Each template that rendered, with its own time and memory

The Twig tab is **hidden** when no templates were rendered for the request, or when you omit `twig_profile` (or do not have Twig installed)—other Flight panels keep working.

In a skeleton-style `services.php`, build the same `$profile` / `ProfilerExtension` when debug is on, pass `twig_profile` into `TracyExtensionLoader`, and keep using your shared Twig environment for `$app->render()`.

### Latte

_PHP 8.1+ is required for this section._

If you have Latte installed in your project, Tracy has a native integration with Latte to analyze your templates. You simply register the extension with your Latte instance (this is Latte’s own Tracy bridge, not the Twig panel above).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// other configurations...

	// only add the extension if Tracy Debug Bar is enabled
	if(Debugger::$showBar === true) {
		// this is where you add the Latte Panel to Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## See Also

- [Tracy](/awesome-plugins/tracy) - Base Tracy setup for Flight
- [Twig](/awesome-plugins/twig) - Templating used by the skeleton and the Twig panel
- [Templates](/learn/templates) - How Flight maps `render` to Twig/Latte
- [Installation](/install) - Skeleton includes tracy-extensions in dev
