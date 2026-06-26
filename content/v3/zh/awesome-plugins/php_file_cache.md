# flightphp/cache

轻量、简单且独立的 PHP 文件缓存类，fork 自 [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**优点** 
- 轻量、独立且简单
- 所有代码都在一个文件中 - 无需无意义的驱动程序。
- 安全 - 每个生成的缓存文件都有一个带有 die 的 php 头，即使有人知道路径且服务器配置不正确，也无法直接访问
- 文档完善且经过测试
- 通过 flock 正确处理并发
- 支持 PHP 7.4+
- 基于 MIT 许可证免费使用

本文档站点正在使用此库来缓存每个页面！

点击 [此处](https://github.com/flightphp/cache) 查看代码。

## 安装

通过 composer 安装：

```bash
composer require flightphp/cache
```

## 用法

用法相当简单。这会在缓存目录中保存一个缓存文件。

```php
use flight\Cache;

$app = Flight::app();

// 你将缓存存储的目录传入构造函数
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// 这确保缓存仅在生产模式下使用
	// ENVIRONMENT 是一个在引导文件中或应用其他地方设置的常量
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### 获取缓存值

你可以使用 `get()` 方法获取缓存值。如果你想要一个在缓存过期时自动刷新的便捷方法，可以使用 `refreshIfExpired()`。

```php

// 获取缓存实例
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // 返回要缓存的数据
}, 10); // 10 秒

// 或者
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 秒
}
```

### 存储缓存值

你可以使用 `set()` 方法在缓存中存储一个值。

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 秒
```

### 删除缓存值

你可以使用 `delete()` 方法删除缓存中的一个值。

```php
Flight::cache()->delete('simple-cache-test');
```

### 检查缓存值是否存在

你可以使用 `exists()` 方法检查缓存中是否存在某个值。

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// 执行某些操作
}
```

### 清除缓存
你可以使用 `flush()` 方法清除整个缓存。

```php
Flight::cache()->flush();
```

### 使用元数据提取缓存

如果你想提取时间戳和其他关于缓存条目的元数据，请确保将 `true` 作为正确的参数传递。

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // 返回要缓存的数据
}, 10, true); // true = 返回带元数据
// 或者
$data = $cache->get("simple-cache-meta-test", true); // true = 返回带元数据

/*
示例带元数据的缓存项：
{
    "time":1511667506, <-- 保存的 unix 时间戳
    "expire":10,       <-- 过期时间（秒）
    "data":"04:38:26", <-- 反序列化后的数据
    "permanent":false
}

使用元数据，我们可以，例如，计算项目何时保存或何时过期
我们也可以通过 "data" 键访问数据本身
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // 获取数据过期的 unix 时间戳并减去当前时间戳
$cacheddate = $data["data"]; // 我们通过 "data" 键访问数据本身

echo "最新缓存保存: $cacheddate, $expiresin 秒后过期";
```

## 源代码

访问 [https://github.com/flightphp/cache](https://github.com/flightphp/cache) 查看代码。