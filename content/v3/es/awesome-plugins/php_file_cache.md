# flightphp/cache

Ligera, simple e independiente clase de caché en archivo PHP bifurcada de [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Ventajas** 
- Ligera, independiente y simple
- Todo el código en un solo archivo - sin controladores innecesarios.
- Segura - cada archivo de caché generado tiene un encabezado php con die, haciendo imposible el acceso directo incluso si alguien conoce la ruta y tu servidor no está configurado correctamente
- Bien documentada y probada
- Maneja la concurrencia correctamente mediante flock
- Compatible con PHP 7.4+
- Gratuita bajo licencia MIT

¡Este sitio de documentación utiliza esta biblioteca para almacenar en caché cada una de las páginas!

Haz clic [aquí](https://github.com/flightphp/cache) para ver el código.

## Instalación

Instalar mediante composer:

```bash
composer require flightphp/cache
```

## Uso

El uso es bastante sencillo. Esto guarda un archivo de caché en el directorio de caché.

```php
use flight\Cache;

$app = Flight::app();

// Pasas el directorio donde se almacenará la caché al constructor
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Esto asegura que la caché solo se utilice cuando esté en modo de producción
	// ENVIRONMENT es una constante que se establece en tu archivo bootstrap o en otro lugar de tu aplicación
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Obtener un Valor de Caché

Utilizas el método `get()` para obtener un valor en caché. Si quieres un método de conveniencia que actualice la caché si ha expirado, puedes usar `refreshIfExpired()`.

```php

// Obtener instancia de caché
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // devolver datos a almacenar en caché
}, 10); // 10 segundos

// o
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 segundos
}
```

### Almacenar un Valor de Caché

Utilizas el método `set()` para almacenar un valor en la caché.

```php
Flight::cache()->set('simple-cache-test', 'my cached data', 10); // 10 segundos
```

### Borrar un Valor de Caché

Utilizas el método `delete()` para borrar un valor en la caché.

```php
Flight::cache()->delete('simple-cache-test');
```

### Verificar si Existe un Valor de Caché

Utilizas el método `exists()` para verificar si un valor existe en la caché.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// hacer algo
}
```

### Limpiar la Caché
Utilizas el método `flush()` para limpiar toda la caché.

```php
Flight::cache()->flush();
```

### Extraer metadatos con caché

Si quieres extraer marcas de tiempo y otros metadatos sobre una entrada de caché, asegúrate de pasar `true` como el parámetro correcto.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "Refreshing data!" . PHP_EOL;
    return date("H:i:s"); // devolver datos a almacenar en caché
}, 10, true); // true = devolver con metadatos
// o
$data = $cache->get("simple-cache-meta-test", true); // true = devolver con metadatos

/*
Ejemplo de elemento en caché recuperado con metadatos:
{
    "time":1511667506, <-- marca de tiempo unix de guardado
    "expire":10,       <-- tiempo de expiración en segundos
    "data":"04:38:26", <-- datos deserializados
    "permanent":false
}

Usando metadatos, podemos, por ejemplo, calcular cuándo se guardó el elemento o cuándo expira
También podemos acceder a los datos en sí con la clave "data"
*/

$expiresin = ($data["time"] + $data["expire"]) - time(); // obtener marca de tiempo unix cuando los datos expiran y restarla de la marca de tiempo actual
$cacheddate = $data["data"]; // accedemos a los datos en sí con la clave "data"

echo "Último guardado en caché: $cacheddate, expira en $expiresin segundos";
```

## Código Fuente

Visita [https://github.com/flightphp/cache](https://github.com/flightphp/cache) para ver el código.