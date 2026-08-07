# Tracy

Tracy — это потрясающий обработчик ошибок, который можно использовать с Flight. Он имеет ряд панелей, которые могут помочь вам отладить ваше приложение. Его также очень легко расширять и добавлять собственные панели. Команда Flight создала несколько панелей специально для проектов на Flight с помощью плагина [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) (переменные Flight, запросы к БД, запросы, сессии и необязательная панель **Twig**, когда вы передаете профиль профилировщика — см. [Расширения Tracy](/awesome-plugins/tracy-extensions)).

## Установка

Установите с помощью composer. И вам действительно стоит установить это без dev-версии, так как Tracy поставляется с компонентом обработки ошибок для production.

```bash
composer require tracy/tracy
```

## Базовая конфигурация

Есть некоторые базовые параметры конфигурации для начала работы. Вы можете узнать больше о них в [Документации Tracy](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Включить Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // иногда вам нужно явно указать (также Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // вы также можете предоставить массив IP-адресов

// Здесь будут логироваться ошибки и исключения. Убедитесь, что эта директория существует и доступна для записи.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // отображать все ошибки
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // все ошибки, кроме устаревших уведомлений
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // если панель Debugger видима, то content-length не может быть установлен Flight

	// Это специфично для расширения Tracy для Flight, если вы его подключили
	// иначе закомментируйте это.
	new TracyExtensionLoader($app);
}
```

## Полезные советы

При отладке вашего кода есть несколько очень полезных функций для вывода данных.

- `bdump($var)` - Это выведет переменную в панель Tracy Bar в отдельной панели.
- `dumpe($var)` - Это выведет переменную, а затем немедленно завершит выполнение.