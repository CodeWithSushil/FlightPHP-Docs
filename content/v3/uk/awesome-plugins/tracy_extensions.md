# Розширення панелі Tracy для Flight

Це набір розширень, які роблять роботу з Flight ще багатшою.

- **Flight** - Аналіз усіх змінних Flight.
- **Database** - Аналіз усіх запитів, які виконувалися на сторінці (якщо ви правильно ініціювали з'єднання з базою даних)
- **Request** - Аналіз усіх змінних `$_SERVER` та перевірка всіх глобальних даних (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Аналіз усіх змінних `$_SESSION`, якщо сесії активні.
- **Twig** *(необов'язково)* - Аналіз часу рендерингу шаблонів Twig, пам'яті та того, які шаблони/блоки/макроси виконувалися (вимагає `twig/twig` та конфігурацію `twig_profile`)

Це особливо зручно з [офіційним скелетом](https://github.com/flightphp/skeleton), який за замовчуванням використовує Twig: той самий макет [інструменти AI](/learn/ai) також чітко відображається на панелі Tracy.

Це панель

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

І кожна панель відображає дуже корисну інформацію про вашу програму!

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Натисніть [тут](https://github.com/flightphp/tracy-extensions), щоб переглянути код.

## Встановлення

Виконайте `composer require flightphp/tracy-extensions --dev` і вперед!

Twig **не** є жорсткою залежністю пакета. Встановіть `twig/twig` тільки якщо вам потрібна панель Twig (скелет вже робить це для представлень).

## Конфігурація

Для початку роботи вам потрібно зробити дуже мало конфігурацій. Вам потрібно буде ініціювати налагоджувач Tracy перед використанням цього [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// bootstrap code
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Вам може знадобитися вказати ваше середовище за допомогою Debugger::enable(Debugger::DEVELOPMENT)

// якщо ви використовуєте з'єднання з базою даних у вашій програмі, існує 
// необхідна обгортка PDO для використання ТІЛЬКИ В РОЗРОБЦІ (не у продакшені будь ласка!)
// Вона має ті самі параметри, що й звичайне з'єднання PDO
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// або якщо ви приєднуєте це до фреймворку Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// тепер кожного разу, коли ви робите запит, буде фіксуватися час, запит та параметри

// Це з'єднує точки
if(Debugger::$showBar === true) {
	// Це повинно бути false, інакше Tracy не зможе рендерити :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// більше коду

Flight::start();
```

## Додаткова конфігурація

### Дані сесії

Якщо у вас є власний обробник сесій (наприклад, ghostff/session), ви можете передати будь-який масив даних сесії Tracy, і він автоматично виведе його для вас. Ви передаєте його за допомогою ключа `session_data` у другому параметрі конструктора `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// або використовуйте flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Це повинно бути false, інакше Tracy не зможе рендерити :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// маршрути та інші речі...

Flight::start();
```

### Панель Twig (необов'язково)

Якщо ваша програма використовує [Twig](/awesome-plugins/twig) (включаючи офіційний скелет), ви можете показати метрики шаблонів на панелі Tracy. Створіть Twig `Profile`, приєднайте `ProfilerExtension` до вашого середовища, потім передайте цей профіль у завантажувач під ключем **`twig_profile`**. Приєднуйте профілювання тільки у розробці.

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

// Необов'язково: експонуйте хелпери дампу Tracy у шаблонах
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

// Зіставлення Flight::render() з Twig (приклад)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**Що показує панель**

- Загальний час рендерингу Twig та пам'ять
- Кількість викликів шаблонів / блоків / макросів
- Кожен шаблон, який рендерився, з його власним часом та пам'яттю

Вкладка Twig **прихована**, коли для запиту не рендерилися шаблони, або коли ви опускаєте `twig_profile` (або не маєте Twig встановленого) — інші панелі Flight продовжують працювати.

У `services.php` у стилі скелету, створюйте той самий `$profile` / `ProfilerExtension`, коли налагодження увімкнено, передайте `twig_profile` у `TracyExtensionLoader`, і продовжуйте використовувати ваше спільне середовище Twig для `$app->render()`.

### Latte

_Для цього розділу потрібен PHP 8.1+._

Якщо у вашому проекті встановлено Latte, Tracy має нативну інтеграцію з Latte для аналізу ваших шаблонів. Ви просто реєструєте розширення з вашим екземпляром Latte (це власний міст Tracy для Latte, а не панель Twig, описана вище).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// інші конфігурації...

	// додаємо розширення тільки якщо Tracy Debug Bar увімкнено
	if(Debugger::$showBar === true) {
		// це місце, де ви додаєте панель Latte до Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Дивіться також

- [Tracy](/awesome-plugins/tracy) - Базове налаштування Tracy для Flight
- [Twig](/awesome-plugins/twig) - Шаблонізація, яку використовує скелет та панель Twig
- [Templates](/learn/templates) - Як Flight зіставляє `render` з Twig/Latte
- [Installation](/install) - Скелет включає tracy-extensions у dev