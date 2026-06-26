# flightphp/cache

Легкий, простой и самостоятельный PHP класс файлового кэширования, ответвленный от [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Преимущества**
- Легкий, самостоятельный и простой
- Весь код в одном файле — никаких бесполезных драйверов.
- Безопасный — каждый сгенерированный файл кэша имеет PHP-заголовок с die, что делает прямой доступ невозможным, даже если кто-то знает путь и ваш сервер неправильно настроен
- Хорошо документирован и протестирован
- Корректно обрабатывает параллелизм через flock
- Поддерживает PHP 7.4+
- Бесплатен под лицензией MIT

Этот сайт документации использует эту библиотеку для кэширования каждой из страниц!

Нажмите [here](https://github.com/flightphp/cache), чтобы посмотреть код.

## Установка

Установите через composer:

```bash
composer require flightphp/cache
```

## Использование

Использование довольно простое. Это сохраняет файл кэша в каталоге кэша.

```php
use flight\Cache;

$app = Flight::app();

// Вы передаете в конструктор каталог, в котором будет храниться кэш
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Это гарантирует, что кэш будет использоваться только в режиме production
	// ENVIRONMENT — это константа, которая устанавливается в вашем bootstrap-файле или в другом месте вашего приложения
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Получение значения кэша

Вы используете метод `get()`, чтобы получить кэшированное значение. Если вам нужен удобный метод, который обновит кэш, если он истек, вы можете использовать `refreshIfExpired()`.

```php

// Получить экземпляр кэша
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // return data to be cached
}, 10); // 10 seconds

// or
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 seconds
}
```

### Сохранение значения кэша

Вы используете метод `set()`, чтобы сохранить значение в кэше.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 seconds
```

### Удаление значения кэша

Вы используете метод `delete()`, чтобы удалить значение в кэше.

```php
Flight::cache()->delete('simple-cache-test');
```

### Проверка существования значения кэша

Вы используете метод `exists()`, чтобы проверить, существует ли значение в кэше.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// do something
}
```

### Очистка кэша
Вы используете метод `flush()`, чтобы очистить весь кэш.

```php
Flight::cache()->flush();
```

### Получение метаданных кэша

Если вы хотите получить временные метки и другие метаданные о записи кэша, убедитесь, что вы передаете `true` в качестве правильного параметра.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // return data to be cached
}, 10, true); // true = return with metadata
// or
$data = $cache->get("simple-cache-meta-test", true); // true = return with metadata

/*
Example cached item retrieved with metadata:
{
    "time":1511667506, <-- save unix timestamp
    "expire":10,       <-- expire time in seconds
    "data":"04:38:26", <-- unserialized data
    "permanent":false
}

Using metadata, we can, for example, calculate when item was saved or when it expires
We can also access the data itself with the "data" key
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // get unix timestamp when data expires and subtract current timestamp from it
$cacheddate = $data["data"]; // we access the data itself with the "data" key

echo "Latest cache save: $cacheddate, expires in $expiresin seconds";
```

## Исходный код

Посетите [https://github.com/flightphp/cache](https://github.com/flightphp/cache), чтобы посмотреть код.