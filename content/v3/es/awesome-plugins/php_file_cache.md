# flightphp/cache

Clase ligera, simple e independiente de caché en archivo PHP bifurcada de [Wruczek/PHP-File-Cache](https://github.com/Wruczek/PHP-File-Cache)

**Ventajas** 
- Ligera, independiente y simple
- Todo el código en un archivo - sin controladores innecesarios.
- Segura - cada archivo de caché generado tiene una cabecera php con die, haciendo imposible el acceso directo incluso si alguien conoce la ruta y tu servidor no está configurado correctamente
- Bien documentada y probada
- Maneja la concurrencia correctamente mediante flock
- Compatible con PHP 7.4+
- Gratuita bajo licencia MIT

¡Este sitio de documentación está usando esta librería para cachear cada una de las páginas!

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

// Pasas el directorio donde se almacenará la caché en el constructor
$app->register('cache', Cache::class, [ __DIR__ . '/../cache/' ], function(Cache $cache) {

	// Esto asegura que la caché solo se use en modo producción
	// ENVIRONMENT es una constante que se establece en tu archivo bootstrap o en otro lugar de tu aplicación
	$cache->setDevMode(ENVIRONMENT === 'development');
});
```

### Obtener un Valor de Caché

Usas el método `get()` para obtener un valor en caché. Si quieres un método de conveniencia que actualice la caché si está expirada, puedes usar `refreshIfExpired()`.

```php

// Obtener instancia de caché
$cache = Flight::cache();
$data = $cache->refreshIfExpired('simple-cache-test', function () {
    return date("H:i:s"); // devolver datos para almacenar en caché
}, 10); // 10 segundos

// o
$data = $cache->get('simple-cache-test');
if(empty($data)) {
	$data = date("H:i:s");
	$cache->set('simple-cache-test', $data, 10); // 10 segundos
}
```

### Almacenar un Valor de Caché

Usas el método `set()` para almacenar un valor en la caché.

```php
Flight::cache()->set('simple-cache-test', 'mis datos en caché', 10); // 10 segundos
```

### Borrar un Valor de Caché

Usas el método `delete()` para borrar un valor en la caché.

```php
Flight::cache()->delete('simple-cache-test');
```

### Comprobar si Existe un Valor de Caché

Usas el método `exists()` para comprobar si un valor existe en la caché.

```php
if(Flight::cache()->exists('simple-cache-test')) {
	// hacer algo
}
```

### Limpiar la Caché
Usas el método `flush()` para limpiar toda la caché.

```php
Flight::cache()->flush();
```

### Extraer metadatos con caché

Si quieres extraer marcas de tiempo y otros metadatos sobre una entrada de caché, asegúrate de pasar `true` como parámetro correcto.

```php
$data = $cache->refreshIfExpired("simple-cache-meta-test", function () {
    echo "¡Actualizando datos!" . PHP_EOL;
    return date("H:i:s"); // devolver datos para almacenar en caché
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

$expiresin = ($data["time"] + $data["expire"]) - time(); // obtener marca de tiempo unix cuando los datos expiren y restar la marca de tiempo actual
$cacheddate = $data["data"]; // accedemos a los datos en sí con la clave "data"

echo "Último guardado de caché: $cacheddate, expira en $expiresin segundos";
```

## Código Fuente

Visita [https://github.com/flightphp/cache](https://github.com/flightphp/cache) para ver el código.