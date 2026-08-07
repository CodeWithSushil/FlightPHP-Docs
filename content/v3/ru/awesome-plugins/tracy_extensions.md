# Расширения панели Tracy для Flight

Это набор расширений, которые делают работу с Flight немного богаче.

- **Flight** - Анализировать все переменные Flight.
- **Database** - Анализировать все запросы, которые выполнялись на странице (если правильно инициализировать подключение к базе данных)
- **Request** - Анализировать все переменные `$_SERVER` и проверять все глобальные полезные данные (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Анализировать все переменные `$_SESSION`, если сессии активны.
- **Twig** *(опционально)* - Анализировать время рендеринга шаблонов Twig, память и какие шаблоны/блоки/макросы выполнялись (требует `twig/twig` и конфигурацию `twig_profile`)

Это особенно удобно с [официальным скелетом](https://github.com/flightphp/skeleton), который по умолчанию использует Twig: тот же макет [AI tools](/learn/ai) также четко отображается на панели Tracy.

Это панель

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

И каждая панель отображает очень полезную информацию о вашем приложении!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Нажмите [здесь](https://github.com/flightphp/tracy-extensions), чтобы просмотреть код.

## Установка

Выполните `composer require flightphp/tracy-extensions --dev` и вы на пути!

Twig **не** является жесткой зависимостью пакета. Установите `twig/twig` только если хотите панель Twig (скелет уже делает это для представлений).

## Конфигурация

Для начала требуется очень мало конфигурации. Вам нужно будет инициализировать отладчик Tracy перед использованием этого [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// bootstrap code
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Возможно, вам нужно указать окружение с Debugger::enable(Debugger::DEVELOPMENT)

// если вы используете подключения к базе данных в вашем приложении, есть 
// обязательная обертка PDO для использования ТОЛЬКО В РАЗРАБОТКЕ (не в продакшене!)
// Она имеет те же параметры, что и обычное подключение PDO
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// или если вы прикрепляете это к фреймворку Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// теперь при каждом выполнении запроса будет фиксироваться время, запрос и параметры

// Это соединяет точки
if(Debugger::$showBar === true) {
	// Это должно быть false, иначе Tracy не сможет рендерить :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// еще код

Flight::start();
```

## Дополнительная конфигурация

### Данные сессии

Если у вас есть пользовательский обработчик сессий (такой как ghostff/session), вы можете передать любой массив данных сессии в Tracy, и он автоматически выведет их для вас. Вы передаете это с ключом `session_data` во втором параметре конструктора `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// или используйте flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Это должно быть false, иначе Tracy не сможет рендерить :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// маршруты и другие вещи...

Flight::start();
```

### Панель Twig (опционально)

Если ваше приложение использует [Twig](/awesome-plugins/twig) (включая официальный скелет), вы можете показывать метрики шаблонов на панели Tracy. Создайте `Profile` Twig, прикрепите `ProfilerExtension` к вашему окружению, затем передайте этот профиль в загрузчик под ключом **`twig_profile`**. Прикрепляйте профилирование только в разработке.

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

// Опционально: предоставить помощники дампа Tracy в шаблонах
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

// Связать Flight::render() с Twig (пример)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Что показывает панель**

- Общее время рендеринга Twig и память
- Количество вызовов шаблонов / блоков / макросов
- Каждый шаблон, который рендерился, со своим временем и памятью

Вкладка Twig **скрыта**, когда для запроса не рендерились шаблоны, или когда вы опускаете `twig_profile` (или не имеете Twig установленным)—другие панели Flight продолжают работать.

В `services.php` в стиле скелета создайте тот же `$profile` / `ProfilerExtension`, когда отладка включена, передайте `twig_profile` в `TracyExtensionLoader` и продолжайте использовать общее окружение Twig для `$app->render()`.

### Latte

_Для этого раздела требуется PHP 8.1+._

Если у вас установлен Latte в вашем проекте, Tracy имеет нативную интеграцию с Latte для анализа ваших шаблонов. Вы просто регистрируете расширение с вашим экземпляром Latte (это собственный мост Tracy для Latte, а не панель Twig выше).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// другие конфигурации...

	// добавить расширение только если панель отладки Tracy включена
	if(Debugger::$showBar === true) {
		// здесь вы добавляете панель Latte в Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## См. также

- [Tracy](/awesome-plugins/tracy) - Базовая настройка Tracy для Flight
- [Twig](/awesome-plugins/twig) - Шаблонизация, используемая скелетом и панелью Twig
- [Templates](/learn/templates) - Как Flight связывает `render` с Twig/Latte
- [Installation](/install) - Скелет включает tracy-extensions в dev