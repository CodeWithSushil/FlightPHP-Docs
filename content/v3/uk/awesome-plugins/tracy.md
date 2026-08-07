# Tracy

Tracy — це чудовий обробник помилок, який можна використовувати з Flight. Він має ряд панелей, які допоможуть вам налагодити ваш додаток. Його також дуже легко розширити та додати власні панелі. Команда Flight створила кілька панелей спеціально для проектів Flight з плагіном [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) (Flight vars, DB queries, request, session та необов'язкова панель **Twig**, коли ви передаєте профіль профайлера — див. [Tracy Extensions](/awesome-plugins/tracy-extensions)).

## Встановлення

Встановіть за допомогою composer. І ви дійсно захочете встановити це без dev-версії, оскільки Tracy постачається з компонентом обробки помилок для продакшену.

```bash
composer require tracy/tracy
```

## Базова конфігурація

Є деякі базові опції конфігурації для початку роботи. Ви можете дізнатися більше про них у [Документації Tracy](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Увімкнути Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // іноді вам потрібно бути явним (також Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // ви також можете надати масив IP-адрес

// Тут будуть записуватися помилки та винятки. Переконайтеся, що ця директорія існує та доступна для запису.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // відображати всі помилки
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // всі помилки, крім застарілих сповіщень
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // якщо панель Debugger видима, то content-length не може бути встановлений Flight

	// Це специфічно для розширення Tracy для Flight, якщо ви його включили
	// в іншому випадку закоментуйте це.
	new TracyExtensionLoader($app);
}
```

## Корисні поради

Коли ви налагоджуєте свій код, є деякі дуже корисні функції для виведення даних для вас.

- `bdump($var)` - Це виведе змінну в панель Tracy Bar в окремій панелі.
- `dumpe($var)` - Це виведе змінну і потім негайно завершиться.