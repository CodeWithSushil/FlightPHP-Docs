# Twig

[Twig](https://twig.symfony.com/) 是一个灵活、快速且安全的 PHP 模板引擎。它是 Symfony 和许多其他项目使用的模板语言，这意味着 AI 编码工具和大多数 PHP 开发者已经非常熟悉其语法。Twig 将模板编译为优化的 PHP，默认自动转义输出（非常适合防止 XSS 攻击），并且可以通过过滤器、函数和扩展轻松扩展。

## 安装

使用 composer 安装。

```bash
composer require twig/twig
```

## 基本配置

有一些基本配置选项可以开始使用。您可以在 [Twig 文档](https://twig.symfony.com/doc/3.x/) 中阅读更多相关内容。

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Twig 存储编译后模板的位置
		'cache' => __DIR__ . '/../cache/twig',
		// 源文件更改时重新编译模板（在开发环境中很方便）
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### 将 Twig 注册为视图类

如果您希望重用单个 Twig 环境（生产环境推荐），请注册它并将 `render` 指向它：

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

## 简单布局示例

这是一个简单布局文件的示例。这个文件将用于包装您所有的其他视图。

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
				{# 您的导航元素在这里 #}
			</nav>
		</header>
		<div id="content">
			{# 魔法就在这里 #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

现在我们有了将在内容块中渲染的文件：

```html
{# app/views/home.twig #}
{# 这告诉 Twig 这个文件是"位于"layout.twig 文件中 #}
{% extends 'layout.twig' %}

{# 这是在布局的内容块中渲染的内容 #}
{% block content %}
	<h1>主页</h1>
	<p>欢迎来到我的应用！</p>
{% endblock %}
```

然后当您在函数或控制器中渲染它时，您可以这样做：

```php
// 简单路由
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => '主页'
	]);
});

// 或者如果您使用控制器
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => '主页'
		]);
	}
}
```

请参阅 [Twig 文档](https://twig.symfony.com/doc/3.x/) 了解如何充分利用 Twig 的更多信息！

## 调试

Twig 附带一个 [调试扩展](https://twig.symfony.com/doc/3.x/functions/dump.html)，它添加了一个 `dump()` 函数，您可以在模板中使用它。仅在开发环境中启用它：

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // dump() 函数必需
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

然后在模板中：

```html
{{ dump(user) }}
```

您还可以将 Twig 与 [Tracy](/awesome-plugins/tracy) 结合使用进行 PHP 级调试。对于模板级指标（渲染时间、内存、哪些模板/块运行），请使用 [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) 中的可选 **Twig 面板**：将 `Twig\Profiler\Profile` 作为 `twig_profile` 传递给 `TracyExtensionLoader`。可选的 `TwigTracyExtension` 在 Tracy 开启时在模板中公开 `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}`。

## 安全注意事项

Twig 默认自动转义输出，这有助于防止 XSS 攻击。对于文本优先使用 `{{ variable }}`。仅在您有意信任 HTML 内容时使用 `|raw` 过滤器（例如，您已在服务端处理的已清理的 markdown）。