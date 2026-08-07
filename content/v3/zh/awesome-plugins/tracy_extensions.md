# Tracy Flight 面板扩展

这是一组扩展，用于让与 Flight 的工作更加丰富。

- **Flight** - 分析所有 Flight 变量。
- **Database** - 分析页面上运行的所有查询（如果您正确启动数据库连接）
- **Request** - 分析所有 `$_SERVER` 变量并检查所有全局负载（`$_GET`、`$_POST`、`$_FILES`）
- **Session** - 如果会话处于活动状态，分析所有 `$_SESSION` 变量。
- **Twig** *(可选)* - 分析 Twig 模板渲染时间、内存，以及哪些模板/块/宏运行（需要 `twig/twig` 和 `twig_profile` 配置）

这对于[官方骨架](https://github.com/flightphp/skeleton)特别方便，它默认使用 Twig：相同的布局 [AI 工具](/learn/ai) 遵循也清晰地显示在 Tracy 栏上。

这是面板

![Flight Bar](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

每个面板都会显示有关您的应用程序的非常有用的信息！

![Flight Data](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Flight Database](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Flight Request](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

点击 [这里](https://github.com/flightphp/tracy-extensions) 查看代码。

## 安装

运行 `composer require flightphp/tracy-extensions --dev`，您就可以开始了！

Twig **不是** 该包的硬依赖项。仅当您需要 Twig 面板时才安装 `twig/twig`（骨架已经为视图安装了）。

## 配置

要开始使用，您需要做的配置非常少。您需要在使用此功能之前启动 Tracy 调试器 [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide)：

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// 引导代码
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// 您可能需要使用 Debugger::enable(Debugger::DEVELOPMENT) 指定您的环境

// 如果您在应用程序中使用数据库连接，这里有一个
// 仅在开发中使用（请不要在生产中使用！）的必需 PDO 包装器
// 它具有与常规 PDO 连接相同的参数
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// 或者如果您将此附加到 Flight 框架
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// 现在每当您进行查询时，它都会捕获时间、查询和参数

// 这将连接各个部分
if(Debugger::$showBar === true) {
	// 这需要为 false，否则 Tracy 无法实际渲染 :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// 更多代码

Flight::start();
```

## 其他配置

### 会话数据

如果您有自定义会话处理程序（例如 ghostff/session），您可以将任何会话数据数组传递给 Tracy，它将自动为您输出。您在 `TracyExtensionLoader` 构造函数的第二个参数中使用 `session_data` 键传递它。

```php

use Ghostff\Session\Session;
// 或使用 flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// 这需要为 false，否则 Tracy 无法实际渲染 :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// 路由和其他内容...

Flight::start();
```

### Twig 面板（可选）

如果您的应用程序使用 [Twig](/awesome-plugins/twig)（包括官方骨架），您可以在 Tracy 栏上显示模板指标。创建 Twig `Profile`，将 `ProfilerExtension` 附加到您的环境中，然后将该配置文件传递给加载器下的 **`twig_profile`** 键。仅在开发中附加分析。

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

// 可选：在模板中公开 Tracy dump 助手
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

// 将 Flight::render() 映射到 Twig（示例）
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**面板显示的内容**

- 总 Twig 渲染时间和内存
- 模板 / 块 / 宏调用计数
- 每个渲染的模板，及其自己的时间和内存

当请求没有渲染模板时，或者当您省略 `twig_profile`（或没有安装 Twig）时，Twig 选项卡是**隐藏的**——其他 Flight 面板继续工作。

在骨架风格的 `services.php` 中，当调试开启时构建相同的 `$profile` / `ProfilerExtension`，将 `twig_profile` 传递给 `TracyExtensionLoader`，并继续使用共享的 Twig 环境进行 `$app->render()`。

### Latte

_此部分需要 PHP 8.1+。_

如果您的项目中安装了 Latte，Tracy 具有与 Latte 的原生集成来分析您的模板。您只需用 Latte 实例注册扩展（这是 Latte 自己的 Tracy 桥接，而不是上面的 Twig 面板）。

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// 其他配置...

	// 仅当 Tracy 调试栏启用时才添加扩展
	if(Debugger::$showBar === true) {
		// 这是您将 Latte 面板添加到 Tracy 的地方
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## 另请参阅

- [Tracy](/awesome-plugins/tracy) - Flight 的基础 Tracy 设置
- [Twig](/awesome-plugins/twig) - 骨架和 Twig 面板使用的模板引擎
- [Templates](/learn/templates) - Flight 如何将 `render` 映射到 Twig/Latte
- [Installation](/install) - 骨架在开发中包含 tracy-extensions