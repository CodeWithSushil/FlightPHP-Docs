# HTML 视图与模板

## 概述

Flight 默认提供了一些基本的 HTML 模板功能。模板化是一种非常有效的方式，可以让你将应用逻辑与表现层分离。专用引擎（Twig、Latte 等）也为 [AI 编码工具](/learn/ai) 提供了一种熟悉且受限的语法，因此它们不太可能将业务逻辑直接倾倒到你的 HTML 中。

## 理解

在构建应用时，你可能会有一些想要返回给最终用户的 HTML。PHP 本身是一种模板语言，但它_非常_容易将数据库调用、API 调用等业务逻辑直接塞入 HTML 文件，从而使测试和解耦变得非常困难。通过将数据推入模板并让模板自行渲染，解耦和单元测试代码就会变得容易得多。如果你使用模板，你会感谢我们的！

## 基本用法

Flight 允许你通过映射 `render`（或注册视图类）来替换默认的视图引擎。向下滚动查看 Twig、Latte、Smarty、Blade 等。

> **骨架默认：** 官方 [flightphp/skeleton](https://github.com/flightphp/skeleton) 在 `app/views/`（`*.twig`）下**仅使用 Twig**。控制器调用 `$this->app->render('welcome', $data)`（扩展名可选）。这是针对新项目的应用选择——并非 Flight 核心的要求。Latte 和其他引擎仍然完全受支持。

### Twig

<span class="badge bg-info">骨架默认</span>

[Twig](https://twig.symfony.com/) 是一个灵活、快速且安全的模板引擎，被 Symfony 和许多其他 PHP 项目所使用。AI 编码工具往往特别熟悉 Twig，而且它默认会自动转义输出，这有助于防范 XSS。

#### 安装

```bash
composer require twig/twig
```

（当你执行 `composer create-project flightphp/skeleton` 时已包含。）

#### 基本配置

覆盖 `render` 方法以使用 Twig 替代默认的 PHP 渲染器：

```php
// 覆盖 render 方法以使用 Twig 替代默认的 PHP 渲染器
Flight::map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Twig 存储其编译后模板的位置
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	]);

	// 允许 "welcome" 或 "welcome.twig"
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}

	echo $twig->render($template, $data);
});
```

在骨架中，这段接线位于 `app/config/services.php`（共享 Twig 环境、缓存路径、类似 `base_url` / CSP nonce 的全局变量）。建议注入 `Engine` 并从控制器调用 `$app->render()`，以便代码保持 [AI 友好和测试友好](/learn/ai)。

#### 在 Flight 中使用 Twig

既然你现在可以使用 Twig 渲染，你可以这样做：

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

当你在浏览器中访问 `/Bob` 时，输出将是：

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

#### 延伸阅读

更多关于在布局中使用 Twig 的完整示例，请参阅本文档的 [awesome 插件](/awesome-plugins/twig) 部分。有关 Tracy 调试栏上的渲染时间指标，请参阅 [Tracy Extensions 中的 Twig 面板](/awesome-plugins/tracy-extensions#twig-panel-optional)。

你可以通过阅读 [官方文档](https://twig.symfony.com/doc/3.x/) 来了解 Twig 的全部功能。

### Latte

<span class="badge bg-secondary">很好的替代方案</span>

[Latte](https://latte.nette.org/) 是一个功能完备的引擎，采用类似 PHP 的语法。对于 Flight 应用来说，它仍然是一个极好的选择；骨架只是统一使用 Twig 作为一个共享默认值（尤其是在 AI 工具生成模板时很有帮助）。

#### 安装

```bash
composer require latte/latte
```

#### 基本配置

主要思想是覆盖 `render` 方法，以使用 Latte 替代默认的 PHP 渲染器。

```php
// 覆盖 render 方法以使用 latte 替代默认的 PHP 渲染器
Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// latte 专门存储其缓存的位置
	$latte->setTempDirectory(__DIR__ . '/../cache/');
	
	$finalPath = Flight::get('flight.views.path') . $template;

	$latte->render($finalPath, $data, $block);
});
```

#### 在 Flight 中使用 Latte

既然你现在可以使用 Latte 渲染，你可以这样做：

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

当你在浏览器中访问 `/Bob` 时，输出将是：

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

#### 延伸阅读

更多关于在布局中使用 Latte 的复杂示例，请参阅本文档的 [awesome 插件](/awesome-plugins/latte) 部分。

你可以通过阅读 [官方文档](https://latte.nette.org/en/) 来了解 Latte 的全部功能，包括翻译和多语言能力。

### 内置视图引擎

<span class="badge bg-warning">已弃用</span>

> **注意：** 这仍然是默认功能，且从技术上讲仍然可以使用。

要显示一个视图模板，请调用 `render` 方法并传入模板文件的名称以及可选的模板数据：

```php
Flight::render('hello.php', ['name' => 'Bob']);
```

你传入的模板数据会自动注入到模板中，并且可以像局部变量一样被引用。模板文件就是简单的 PHP 文件。如果 `hello.php` 模板文件的内容是：

```php
Hello, <?= $name ?>!
```

输出将是：

```text
Hello, Bob!
```

你还可以使用 set 方法手动设置视图变量：

```php
Flight::view()->set('name', 'Bob');
```

变量 `name` 现在可以在你的所有视图中使用。因此你可以直接这样做：

```php
Flight::render('hello');
```

请注意，在 render 方法中指定模板名称时，你可以省略 `.php` 扩展名。

默认情况下，Flight 会在 `views` 目录中查找模板文件。你可以通过设置以下配置来为模板设置替代路径：

```php
Flight::set('flight.views.path', '/path/to/views');
```

#### 布局

网站通常有一个单独的布局模板文件，其中的内容可以互换。要渲染用于布局的内容，你可以向 `render` 方法传递一个可选参数。

```php
Flight::render('header', ['heading' => 'Hello'], 'headerContent');
Flight::render('body', ['body' => 'World'], 'bodyContent');
```

之后，你的视图将拥有名为 `headerContent` 和 `bodyContent` 的已保存变量。然后你可以通过以下方式渲染布局：

```php
Flight::render('layout', ['title' => 'Home Page']);
```

如果模板文件如下所示：

`header.php`：

```php
<h1><?= $heading ?></h1>
```

`body.php`：

```php
<div><?= $body ?></div>
```

`layout.php`：

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

输出将是：
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

以下是如何在视图中使用 [Smarty](http://www.smarty.net/) 模板引擎：

```php
// 加载 Smarty 库
require './Smarty/libs/Smarty.class.php';

// 将 Smarty 注册为视图类
// 同时传入一个回调函数，在加载时配置 Smarty
Flight::register('view', Smarty::class, [], function (Smarty $smarty) {
  $smarty->setTemplateDir('./templates/');
  $smarty->setCompileDir('./templates_c/');
  $smarty->setConfigDir('./config/');
  $smarty->setCacheDir('./cache/');
});

// 分配模板数据
Flight::view()->assign('name', 'Bob');

// 显示模板
Flight::view()->display('hello.tpl');
```

为了完整性，你还应该覆盖 Flight 的默认 render 方法：

```php
Flight::map('render', function(string $template, array $data): void {
  Flight::view()->assign($data);
  Flight::view()->display($template);
});
```

### Blade

以下是如何在视图中使用 [Blade](https://laravel.com/docs/8.x/blade) 模板引擎：

首先，你需要通过 Composer 安装 BladeOne 库：

```bash
composer require eftec/bladeone
```

然后，你可以在 Flight 中将 BladeOne 配置为视图类：

```php
<?php
// 加载 BladeOne 库
use eftec\bladeone\BladeOne;

// 将 BladeOne 注册为视图类
// 同时传入一个回调函数，在加载时配置 BladeOne
Flight::register('view', BladeOne::class, [], function (BladeOne $blade) {
  $views = __DIR__ . '/../views';
  $cache = __DIR__ . '/../cache';

  $blade->setPath($views);
  $blade->setCompiledPath($cache);
});

// 分配模板数据
Flight::view()->share('name', 'Bob');

// 显示模板
echo Flight::view()->run('hello', []);
```

为了完整性，你还应该覆盖 Flight 的默认 render 方法：

```php
<?php
Flight::map('render', function(string $template, array $data): void {
  echo Flight::view()->run($template, $data);
});
```

在此示例中，hello.blade.php 模板文件可能如下所示：

```php
<?php
Hello, {{ $name }}!
```

输出将是：

```
Hello, Bob!
```

## 另请参阅
- [安装](/install) - 新项目的骨架布局（`app/views/*.twig`）。
- [扩展](/learn/extending) - 如何覆盖 `render` 方法以使用不同的模板引擎。
- [路由](/learn/routing) - 如何将路由映射到控制器并渲染视图。
- [响应](/learn/responses) - 如何自定义 HTTP 响应。
- [安全](/learn/security) - 自动转义与 XSS。
- [AI 与开发者体验](/learn/ai) - 为什么默认一种视图引擎有助于编码代理。
- [为什么选择框架？](/learn/why-frameworks) - 模板如何融入整体。

## 故障排除
- 如果你的中间件中有重定向，但你的应用似乎没有重定向，请确保在中间件中添加 `exit;` 语句。
- 如果 Twig 找不到模板，请检查 `flight.views.path`，并确认该路径下存在具有预期扩展名的文件（骨架：`app/views/`）。

## 变更日志
- 文档 – 将 Twig 记录为官方骨架默认；Latte 仍然是一流的替代方案。
- v2.0 - 初始发布。