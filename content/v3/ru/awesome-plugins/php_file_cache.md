# flightphp/cache

Легкий, простой и автономный PHP-класс для кэширования в файлах, форкнутый из [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Преимущества**
- Легкий, автономный и простой
- Весь код в одном файле - без лишних драйверов.
- Безопасный - каждый созданный файл кэша имеет PHP-заголовок с die, что делает прямой доступ невозможным, даже если кто-то знает путь и ваш сервер настроен неправильно
- Хорошо документирован и протестирован
- Корректно обрабатывает параллелизм через flock
- Поддерживает PHP 7.4+
- Бесплатно под лицензией MIT

Этот сайт документации использует эту библиотеку для кэширования каждой страницы!

Нажмите [здесь](https://github.com/flightphp/cache) для просмотра кода.

## Установка

Установите через composer:

```bash
composer require flightphp/cache
```

## Использование

Использование довольно простое. Это сохраняет файл кэша в директории кэша.

```php
use flight\Cache;

$app = Flight::app();

// Вы передаете директорию, в которой будет храниться кэш, в конструктор
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Это гарантирует, что кэш используется только в режиме production
	// ENVIRONMENT - это константа, которая устанавливается в вашем bootstrap файле или в другом месте вашего приложения
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Получение значения кэша

Вы используете метод `get()` для получения кэшированного значения. Если вам нужен удобный метод, который обновит кэш, если он истек, вы можете использовать `refreshIfExpired()`.

```php

// Получение экземпляра кэша
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

Вы используете метод `set()` для сохранения значения в кэше.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 seconds
```

### Удаление значения кэша

Вы используете метод `delete()` для удаления значения в кэше.

```php
Flight::cache()->delete('simple-cache-test');
```

### Проверка существования значения кэша

Вы используете метод `exists()` для проверки существования значения в кэше.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// do something
}
```

### Очистка кэша
Вы используете метод `flush()` для очистки всего кэша.

```php
Flight::cache()->flush();
```

### Извлечение метаданных с кэшем

Если вы хотите извлечь временные метки и другие метаданные о записи кэша, убедитесь, что вы передаете `true` в качестве правильного параметра.

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

Посетите [https://github.com/flightphp/cache](https://github.com/flightphp/cache) для просмотра кода.