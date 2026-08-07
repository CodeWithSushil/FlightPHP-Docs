# HTML-представления и шаблоны

## Обзор

Flight по умолчанию предоставляет базовую функциональность HTML-шаблонизации. Шаблонизация — это очень эффективный способ отделить логику приложения от уровня представления. Выделенный движок (Twig, Latte и т.д.) также даёт [инструментам ИИ для написания кода](/learn/ai) знакомый ограниченный синтаксис, поэтому они с меньшей вероятностью будут встраивать бизнес-логику прямо в ваш HTML.

## Понимание

При создании приложения у вас, скорее всего, будет HTML, который нужно отдавать конечному пользователю. PHP сам по себе является языком шаблонов, но _очень_ легко добавить бизнес-логику, например вызовы базы данных, API и т.д., прямо в HTML-файл, что делает тестирование и разделение кода очень сложным процессом. Передавая данные в шаблон и позволяя шаблону отображать себя, становится гораздо проще разделять и юнит-тестировать код. Вы скажете нам спасибо, если будете использовать шаблоны!

## Базовое использование

Flight позволяет заменить стандартный движок представлений, просто сопоставив `render` (или зарегистрировав класс представления). Прокрутите вниз, чтобы узнать о Twig, Latte, Smarty, Blade и других.

> **Параметр скелета по умолчанию:** официальный [flightphp/skeleton](https://github.com/flightphp/skeleton) использует **только Twig** в каталоге `app/views/` (`*.twig`). Контроллеры вызывают `$this->app->render('welcome', $data)` (расширение необязательно). Это выбор приложения для новых проектов, а не требование ядра Flight. Latte и другие движки полностью поддерживаются.

### Twig

<span class="badge bg-info">по умолчанию в скелете</span>

[Twig](https://twig.symfony.com/) — это гибкий, быстрый и безопасный шаблонизатор, используемый в Symfony и многих других PHP-проектах. Инструменты ИИ для написания кода особенно хорошо знают Twig, и он по умолчанию автоматически экранирует вывод, что помогает защититься от XSS.

#### Установка

```bash
composer require twig/twig
```

(Уже включён, если вы выполнили `composer create-project flightphp/skeleton`.)

#### Базовая конфигурация

Переопределите метод `render`, чтобы использовать Twig вместо стандартного PHP-рендерера:

```php
// переопределяем метод render, чтобы использовать Twig вместо стандартного PHP-рендерера
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Где Twig хранит скомпилированные шаблоны
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Разрешаем "welcome" или "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

В скелете эта настройка находится в `app/config/services.php` (общее окружение Twig, путь к кэшу, глобальные переменные, такие как `base_url` / nonce для CSP). Предпочтительнее внедрять `Engine` и вызывать `$app->render()` из контроллеров, чтобы код оставался [удобным для ИИ и тестирования](/learn/ai).

#### Использование Twig во Flight

Теперь, когда вы можете рендерить с помощью Twig, можно сделать, например, следующее:

```html
{# app/views/home.twig #}
<html>
  <head>
	<title>{% if title %}{{ title }} - {% endif %}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {{ name }}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.twig', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Когда вы откроете `/Bob` в браузере, результат будет следующим:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Дополнительное чтение

Более полный пример использования Twig с макетами приведён в разделе [awesome plugins](/awesome-plugins/twig) этой документации. О метриках времени рендеринга на панели Tracy см. [панель Twig в Tracy Extensions](/awesome-plugins/tracy-extensions#twig-panel-optional).

Вы можете узнать больше о полных возможностях Twig, прочитав [официальную документацию](https://twig.symfony.com/doc/3.x/).

### Latte

<span class="badge bg-secondary">отличная альтернатива</span>

[Latte](https://latte.nette.org/) — это многофункциональный движок с синтаксисом, похожим на PHP. Он по-прежнему является отличным выбором для приложений на Flight; скелет просто стандартизирует использование Twig как единого варианта по умолчанию (особенно полезно, когда ИИ генерирует шаблоны).

#### Установка

```bash
composer require latte/latte
```

#### Базовая конфигурация

Основная идея — переопределить метод `render`, чтобы использовать Latte вместо стандартного PHP-рендерера.

```php
// переопределяем метод render, чтобы использовать Latte вместо стандартного PHP-рендерера
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// Где Latte хранит свой кэш
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Использование Latte во Flight

Теперь, когда вы можете рендерить с помощью Latte, можно сделать, например, следующее:

```html
<!-- app/views/home.latte -->
<html>
  <head>
	<title>{$title ? $title . ' - '}My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, {$name}!</h1>
  </body>
</html>
```

```php
// routes.php
Flight::route('/@name', function ($name) {
	Flight::render('home.latte', [
		'title' => 'Home Page',
		'name' => $name
	]);
});
```

Когда вы откроете `/Bob` в браузере, результат будет следующим:

```html
<html>
  <head>
	<title>Home Page - My App</title>
	<link rel="stylesheet" href="style.css">
  </head>
  <body>
	<h1>Hello, Bob!</h1>
  </body>
</html>
```

#### Дополнительное чтение

Более сложный пример использования Latte с макетами приведён в разделе [awesome plugins](/awesome-plugins/latte) этой документации.

Вы можете узнать больше о полных возможностях Latte, включая возможности перевода и локализации, прочитав [официальную документацию](https://latte.nette.org/en/).

### Встроенный движок представлений

<span class="badge bg-warning">устарело</span>

> **Примечание:** Это всё ещё стандартная функциональность и технически продолжает работать.

Чтобы отобразить шаблон представления, вызовите метод `render` с именем файла шаблона и необязательными данными шаблона:

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Передаваемые данные шаблона автоматически внедряются в шаблон и могут использоваться как локальная переменная. Файлы шаблонов — это просто PHP-файлы. Если содержимое файла шаблона `hello.php` выглядит так:

```php
Hello, <?= $name ?>!
```

Результат будет следующим:

```text
Hello, Bob!
```

Вы также можете вручную устанавливать переменные представления с помощью метода `set`:

```php
Flight::view()->set('name', 'Bob');
```

Переменная `name` теперь доступна во всех ваших представлениях. Поэтому вы можете просто сделать:

```php
Flight::render('hello');
```

Обратите внимание, что при указании имени шаблона в методе `render` можно опустить расширение `.php`.

По умолчанию Flight ищет каталог `views` для файлов шаблонов. Вы можете указать альтернативный путь для своих шаблонов, задав следующую конфигурацию:

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### Макеты

Для веб-сайтов обычно используется один файл макета с изменяемым содержимым. Чтобы отобразить контент для использования в макете, вы можете передать необязательный параметр в метод `render`.

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

После этого в представлении будут сохранены переменные `headerContent` и `bodyContent`. Затем вы можете отобразить макет следующим образом:

```php
Flight::render('layout', ['title' => 'Home Page']);
```

Если файлы шаблонов выглядят так:

`header.php`:

```php
<h1><?= $heading ?></h1>
```

`body.php`:

```php
<div><?= $body ?></div>
```

`layout.php`:

```php
<html>
  <head>
    <title><?= $title ?></title>
  </head>
  <body>
    <?= $headerContent ?>
    <?= $bodyContent ?>
  </body>
</html>
```

Результат будет следующим:
```html
<html>
  <head>
    <title>Home Page</title>
  </head>
  <body>
    <h1>Hello</h1>
    <div>World</div>
  </body>
</html>
```

### Smarty

Вот как можно использовать шаблонизатор [Smarty](http://www.smarty.net/) для ваших представлений:

```php
// Загружаем библиотеку Smarty
require './Smarty/libs/Smarty.class.php';

// Регистрируем Smarty как класс представления
// Также передаём callback-функцию для настройки Smarty при загрузке
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Передаём данные шаблона
Flight::view()->assign('name', 'Bob');

// Отображаем шаблон
Flight::view()->display('hello.tpl');
```

Для полноты картины вам также следует переопределить стандартный метод render во Flight:

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

Вот как можно использовать шаблонизатор [Blade](https://laravel.com/docs/8.x/blade) для ваших представлений:

Сначала нужно установить библиотеку BladeOne через Composer:

```bash
composer require eftec/bladeone
```

Затем вы можете настроить BladeOne как класс представления во Flight:

```php
<?php
// Загружаем библиотеку BladeOne
use eftec\bladeone\BladeOne;

// Регистрируем BladeOne как класс представления
// Также передаём callback-функцию для настройки BladeOne при загрузке
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Передаём данные шаблона
Flight::view()->share('name', 'Bob');

// Отображаем шаблон
echo Flight::view()->run('hello', []);
```

Для полноты картины вам также следует переопределить стандартный метод render во Flight:

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

В этом примере файл шаблона `hello.blade.php` может выглядеть так:

```php
<?php
Hello, {{ $name }}!
```

Результат будет следующим:

```
Hello, Bob!
```

## Смотрите также
- [Установка](/install) — Макет скелета (`app/views/*.twig`) для новых проектов.
- [Расширение](/learn/extending) — Как переопределить метод `render` для использования другого шаблонизатора.
- [Маршрутизация](/learn/routing) — Как сопоставлять маршруты с контроллерами и отображать представления.
- [Ответы](/learn/responses) — Как настраивать HTTP-ответы.
- [Безопасность](/learn/security) — Автоматическое экранирование и XSS.
- [ИИ и опыт разработчика](/learn/ai) — Почему один шаблонизатор по умолчанию помогает агентам кода.
- [Зачем нужен фреймворк?](/learn/why-frameworks) — Как шаблоны вписываются в общую картину.

## Устранение неполадок
- Если в вашем middleware есть редирект, но ваше приложение, похоже, не перенаправляет, убедитесь, что вы добавили оператор `exit;` в ваш middleware.
- Если Twig не может найти шаблон, проверьте `flight.views.path` и что файл существует по этому пути с ожидаемым расширением (скелет: `app/views/`).

## Журнал изменений
- Документация — Twig описан как официальный шаблонизатор по умолчанию для скелета; Latte остаётся полноценной альтернативой.
- v2.0 — Первоначальный выпуск.