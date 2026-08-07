# Twig

[Twig](https://twig.symfony.com/) — це гнучкий, швидкий і безпечний шаблонізатор для PHP. Це мова шаблонів, яку використовує Symfony та багато інших проєктів, а це означає, що інструменти ШІ для кодування та більшість PHP-розробників вже добре знайомі з її синтаксисом. Twig компілює шаблони в оптимізований PHP, автоматично екранує вивід за замовчуванням (відмінно для захисту від XSS) і легко розширюється за допомогою фільтрів, функцій та розширень.

## Встановлення

Встановіть за допомогою composer.

```bash
composer require twig/twig
```

## Базове налаштування

Є кілька базових опцій налаштування для початку роботи. Ви можете дізнатися більше про них у [Документації Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Де Twig зберігає скомпільовані шаблони
		'cache' => __DIR__ . '/../cache/twig',
		// Перекомпілювати шаблони при зміні вихідного коду (зручно під час розробки)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Реєстрація Twig як класу View

Якщо ви хочете повторно використовувати одне середовище Twig (рекомендовано для production), зареєструйте його і направте `render` на нього:

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	],
]);

$app->map('render', function(string $template, array $data): void {
	echo Flight::view()->render($template, $data);
});
```

## Приклад простого макета

Ось простий приклад файлу макета. Це файл, який буде використовуватися для обгортання всіх ваших інших представлень.

```html
{# app/views/layout.twig #}
<!doctype html>
<html lang="en">
	<head>
		<title>{% if title %}{{ title }} - {% endif %}My App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				{# ваші елементи навігації тут #}
			</nav>
		</header>
		<div id="content">
			{# Це і є та сама магія #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

А тепер у нас є ваш файл, який буде рендеритися всередині блоку content:

```html
{# app/views/home.twig #}
{# Це повідомляє Twig, що цей файл "всередині" файлу layout.twig #}
{% extends 'layout.twig' %}

{# Це вміст, який буде рендеритися всередині макета в блоці content #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Потім, коли ви будете рендерити це у вашій функції чи контролері, ви зробите щось на кшталт цього:

```php
// простий маршрут
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// або якщо ви використовуєте контролер
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Home Page'
		]);
	}
}
```

Перегляньте [Документацію Twig](https://twig.symfony.com/doc/3.x/) для отримання додаткової інформації про те, як використовувати Twig на повну потужність!

## Налагодження

Twig поставляється з [Розширенням для налагодження](https://twig.symfony.com/doc/3.x/functions/dump.html), яке додає функцію `dump()`, яку ви можете використовувати всередині шаблонів. Увімкніть його лише під час розробки:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // обов'язково для функції dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Потім у шаблоні:

```html
{{ dump(user) }}
```

Ви також можете поєднувати Twig з [Tracy](/awesome-plugins/tracy) для налагодження на рівні PHP. Для метрик на рівні шаблону (час рендерингу, пам'ять, які шаблони/блоки виконувалися), використовуйте опціональну **панель Twig** у [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): передайте `Twig\Profiler\Profile` як `twig_profile` до `TracyExtensionLoader`. Опціональне `TwigTracyExtension` надає `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` у шаблонах, коли Tracy увімкнено.

## Застереження щодо безпеки

Twig автоматично екранує вивід за замовчуванням, що допомагає захистити від XSS-атак. Віддавайте перевагу `{{ variable }}` для тексту. Використовуйте фільтр `|raw` лише тоді, коли ви свідомо довіряєте HTML-вмісту (наприклад, очищеному markdown, який ви вже обробили на стороні сервера).