# Twig

[Twig](https://twig.symfony.com/) — это гибкий, быстрый и безопасный шаблонизатор для PHP. Это язык шаблонов, используемый Symfony и многими другими проектами, что означает, что инструменты ИИ-кодирования и большинство PHP-разработчиков уже хорошо знакомы с его синтаксисом. Twig компилирует шаблоны в оптимизированный PHP, автоматически экранирует вывод по умолчанию (отлично для защиты от XSS) и легко расширяется фильтрами, функциями и расширениями.

## Установка

Установите с помощью composer.

```bash
composer require twig/twig
```

## Базовая настройка

Существуют некоторые базовые параметры настройки, чтобы начать работу. Вы можете узнать больше о них в [документации Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Где Twig хранит скомпилированные шаблоны
		'cache' => __DIR__ . '/../cache/twig',
		// Перекомпилировать шаблоны при изменении исходного кода (удобно при разработке)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Регистрация Twig как класса представления

Если вы предпочитаете повторно использовать одну среду Twig (рекомендуется для продакшена), зарегистрируйте её и укажите `render` на неё:

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

## Пример простого макета

Вот простой пример файла макета. Это файл, который будет использоваться для обёртки всех ваших других представлений.

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
				{# ваши элементы навигации здесь #}
			</nav>
		</header>
		<div id="content">
			{# Вот здесь магия #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

А теперь у нас есть ваш файл, который будет рендериться внутри этого блока content:

```html
{# app/views/home.twig #}
{# Это говорит Twig, что этот файл "внутри" файла layout.twig #}
{% extends 'layout.twig' %}

{# Это содержимое, которое будет рендериться внутри макета в блоке content #}
{% block content %}
	<h1>Home Page</h1>
	<p>Welcome to my app!</p>
{% endblock %}
```

Затем, когда вы собираетесь рендерить это внутри вашей функции или контроллера, вы должны сделать что-то вроде этого:

```php
// простой маршрут
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Home Page'
	]);
});

// или если вы используете контроллер
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

Смотрите [документацию Twig](https://twig.symfony.com/doc/3.x/) для получения дополнительной информации о том, как использовать Twig на полную мощность!

## Отладка

Twig поставляется с [расширением отладки](https://twig.symfony.com/doc/3.x/functions/dump.html), которое добавляет функцию `dump()`, которую вы можете использовать внутри шаблонов. Включайте его только при разработке:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // требуется для функции dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Затем в шаблоне:

```html
{{ dump(user) }}
```

Вы также можете объединить Twig с [Tracy](/awesome-plugins/tracy) для отладки на уровне PHP. Для метрик на уровне шаблона (время рендеринга, память, какие шаблоны/блоки выполнялись), используйте опциональную **панель Twig** в [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): передайте `Twig\Profiler\Profile` как `twig_profile` в `TracyExtensionLoader`. Опциональное `TwigTracyExtension` предоставляет `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` в шаблонах, когда Tracy включён.

## Примечание по безопасности

Twig автоматически экранирует вывод по умолчанию, что помогает защититься от XSS-атак. Предпочтительно использовать `{{ variable }}` для текста. Используйте фильтр `|raw` только тогда, когда вы намеренно доверяете HTML-содержимому (например, очищенному markdown, который вы уже обработали на стороне сервера).