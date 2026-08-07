# HTML-представлення та шаблони

## Огляд

Flight надає базову функціональність HTML-шаблонізації за замовчуванням. Шаблонізація — це дуже ефективний спосіб відокремити логіку вашого застосунку від рівня представлення. Виділений рушій (Twig, Latte тощо) також дає [інструментам ШІ для кодування](/learn/ai) знайомий, обмежений синтаксис, тож вони з меншою ймовірністю вкидатимуть бізнес-логіку у ваш HTML.

## Розуміння

Коли ви створюєте застосунок, у вас, найімовірніше, буде HTML, який ви захочете повертати кінцевому користувачеві. Сам по собі PHP є мовою шаблонів, але дуже легко вплутати бізнес-логіку, як-от виклики бази даних, виклики API тощо, у ваш HTML-файл і зробити тестування та розділення дуже складним процесом. Передаючи дані в шаблон і дозволяючи шаблону відтворювати себе, набагато легше розділити та модульно тестувати ваш код. Ви будете нам вдячні, якщо користуватиметеся шаблонами!

## Базове використання

Flight дозволяє замінити типовий рушій представлень, просто зіставивши `render` (або зареєструвавши клас представлення). Прокрутіть униз, щоб побачити Twig, Latte, Smarty, Blade тощо.

> **Типово для скелетона:** Офіційний [flightphp/skeleton](https://github.com/flightphp/skeleton) використовує **лише Twig** у `app/views/` (`*.twig`). Контролери викликають `$this->app->render('welcome', $data)` (розширення необов'язкове). Це вибір застосунку для нових проєктів, а не вимога ядра Flight. Latte та інші рушії залишаються повністю підтримуваними.

### Twig

<span class="badge bg-info">типово для скелетона</span>

[Twig](https://twig.symfony.com/) — це гнучкий, швидкий і безпечний шаблонний рушій, який використовується Symfony та багатьма іншими PHP-проєктами. Інструменти ШІ для кодування, як правило, особливо добре знають Twig, і він за замовчуванням автоматично екранує вивід, що допомагає захистити від XSS.

#### Встановлення

```bash
composer require twig/twig
```

(Уже включено, коли ви виконуєте `composer create-project flightphp/skeleton`.)

#### Базова конфігурація

Перевизначте метод `render`, щоб використовувати Twig замість типового PHP-рендерера:

```php
// перевизначаємо метод render, щоб використовувати Twig замість типового PHP-рендерера
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// де Twig зберігає свої скомпільовані шаблони
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// Дозволяємо "welcome" або "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

У скелетоні ця прив'язка знаходиться у `app/config/services.php` (спільне середовище Twig, шлях до кешу, глобальні змінні, як-от `base_url` / nonce для CSP). Надавайте перевагу впровадженню `Engine` і викликайте `$app->render()` з контролерів, щоб код залишався [дружнім до ШІ та тестування](/learn/ai).

#### Використання Twig у Flight

Тепер, коли ви можете рендерити за допомогою Twig, ви можете зробити, наприклад, таке:

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

Коли ви відвідаєте `/Bob` у своєму браузері, результат буде таким:

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

#### Подальше читання

Більш повний приклад використання Twig із макетами наведено в розділі [чудові плагіни](/awesome-plugins/twig) цієї документації. Щодо метрик часу рендерингу на панелі Tracy, дивіться [панель Twig у розширеннях Tracy](/awesome-plugins/tracy-extensions#twig-panel-optional).

Ви можете дізнатися більше про повні можливості Twig, прочитавши [офіційну документацію](https://twig.symfony.com/doc/3.x/).

### Latte

<span class="badge bg-secondary">чудова альтернатива</span>

[Latte](https://latte.nette.org/) — це повнофункціональний рушій із синтаксисом, схожим на PHP. Для застосунків на Flight це все ще чудовий вибір; скелетон просто стандартизує Twig як єдиний типовий рушій (особливо корисно, коли інструменти ШІ генерують шаблони).

#### Встановлення

```bash
composer require latte/latte
```

#### Базова конфігурація

Основна ідея полягає в тому, що ви перевизначаєте метод `render`, щоб використовувати Latte замість типового PHP-рендерера.

```php
// перевизначаємо метод render, щоб використовувати Latte замість типового PHP-рендерера
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// де Latte зокрема зберігає свій кеш
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### Використання Latte у Flight

Тепер, коли ви можете рендерити за допомогою Latte, ви можете зробити, наприклад, таке:

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

Коли ви відвідаєте `/Bob` у своєму браузері, результат буде таким:

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

#### Подальше читання

Більш складний приклад використання Latte з макетами наведено в розділі [чудові плагіни](/awesome-plugins/latte) цієї документації.

Ви можете дізнатися більше про повні можливості Latte, зокрема переклад і мовні можливості, прочитавши [офіційну документацію](https://latte.nette.org/en/).

### Вбудований рушій представлень

<span class="badge bg-warning">застаріло</span>

> **Примітка:** Це все ще типова функціональність і технічно все ще працює.

Щоб відобразити шаблон представлення, викличте метод `render` із назвою файлу шаблону та необов'язковими даними шаблону:

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

Дані шаблону, які ви передаєте, автоматично впроваджуються в шаблон і можуть використовуватися як локальна змінна. Файли шаблонів — це просто PHP-файли. Якщо вміст файлу шаблону `hello.php` такий:

```php
Hello, <?= $name ?>!
```

Результат буде таким:

```text
Hello, Bob!
```

Ви також можете вручну встановити змінні представлення за допомогою методу `set`:

```php
Flight::view()->set('name', 'Bob');
```

Змінна `name` тепер доступна у всіх ваших представленнях. Тож ви можете просто зробити:

```php
Flight::render('hello');
```

Зверніть увагу, що під час вказання назви шаблону в методі `render` ви можете опустити розширення `.php`.

За замовчуванням Flight шукає каталог `views` для файлів шаблонів. Ви можете встановити альтернативний шлях для ваших шаблонів, задавши таку конфігурацію:

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### Макети

Для вебсайтів зазвичай використовують один файл макета зі змінним вмістом. Щоб відрендерити вміст, який буде використано в макеті, ви можете передати необов'язковий параметр методу `render`.

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

Ваше представлення тоді матиме збережені змінні `headerContent` та `bodyContent`. Після цього ви можете відрендерити макет так:

```php
Flight::render('layout', ['title' => 'Home Page']);
```

Якщо файли шаблонів виглядають так:

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

Результат буде таким:
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

Ось як ви можете використовувати рушій шаблонів [Smarty](http://www.smarty.net/) для ваших представлень:

```php
// Завантажуємо бібліотеку Smarty
require './Smarty/libs/Smarty.class.php';

// Реєструємо Smarty як клас представлення
// Також передаємо функцію зворотного виклику для налаштування Smarty під час завантаження
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// Призначаємо дані шаблону
Flight::view()->assign('name', 'Bob');

// Відображаємо шаблон
Flight::view()->display('hello.tpl');
```

Для повноти варто також перевизначити типовий метод `render` у Flight:

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

Ось як ви можете використовувати рушій шаблонів [Blade](https://laravel.com/docs/8.x/blade) для ваших представлень:

Спершу потрібно встановити бібліотеку BladeOne через Composer:

```bash
composer require eftec/bladeone
```

Потім ви можете налаштувати BladeOne як клас представлення у Flight:

```php
<?php
// Завантажуємо бібліотеку BladeOne
use eftec\bladeone\BladeOne;

// Реєструємо BladeOne як клас представлення
// Також передаємо функцію зворотного виклику для налаштування BladeOne під час завантаження
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// Призначаємо дані шаблону
Flight::view()->share('name', 'Bob');

// Відображаємо шаблон
echo Flight::view()->run('hello', []);
```

Для повноти варто також перевизначити типовий метод `render` у Flight:

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

У цьому прикладі файл шаблону `hello.blade.php` може виглядати так:

```php
<?php
Hello, {{ $name }}!
```

Результат буде таким:

```
Hello, Bob!
```

## Дивіться також

- [Встановлення](/install) — Розташування скелетона (`app/views/*.twig`) для нових проєктів.
- [Розширення](/learn/extending) — Як перевизначити метод `render`, щоб використовувати інший рушій шаблонів.
- [Маршрутизація](/learn/routing) — Як зіставляти маршрути з контролерами та рендерити представлення.
- [Відповіді](/learn/responses) — Як налаштовувати HTTP-відповіді.
- [Безпека](/learn/security) — Автоматичне екранування та XSS.
- [ШІ та досвід розробника](/learn/ai) — Чому один типовий рушій представлень допомагає агентам з кодування.
- [Чому фреймворк?](/learn/why-frameworks) — Як шаблони вписуються в загальну картину.

## Усунення проблем

- Якщо у вашому проміжному програмному забезпеченні є перенаправлення, але ваш застосунок, схоже, не перенаправляє, переконайтеся, що ви додали оператор `exit;` у своє проміжне програмне забезпечення.
- Якщо Twig не може знайти шаблон, перевірте `flight.views.path` і чи існує файл за цим шляхом із очікуваним розширенням (скелетон: `app/views/`).

## Журнал змін

- Документація — Twig задокументовано як офіційний типовий рушій скелетона; Latte залишається повноцінною альтернативою.
- v2.0 — Початковий випуск.