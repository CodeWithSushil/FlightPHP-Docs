# Colecciones

## Resumen

La clase `Collection` en Flight es una utilidad práctica para gestionar conjuntos de datos. Te permite acceder y manipular datos usando tanto notación de array como de objeto, haciendo tu código más limpio y flexible.

## Entendiendo

Un `Collection` es básicamente un envoltorio alrededor de un array, pero con poderes adicionales. Puedes usarlo como un array, recorrerlo, contar sus elementos e incluso acceder a ellos como si fueran propiedades de objeto. Esto es especialmente útil cuando deseas pasar datos estructurados en tu aplicación, o cuando quieres que tu código sea un poco más legible.

Las colecciones implementan varias interfaces de PHP:
- `ArrayAccess` (para que puedas usar sintaxis de array)
- `Iterator` (para que puedas iterar con `foreach`)
- `Countable` (para que puedas usar `count()`)
- `JsonSerializable` (para que puedas convertir fácilmente a JSON)

## Uso Básico

### Creando una Colección

Puedes crear una colección simplemente pasando un array a su constructor:

```php
use flight\util\Collection;

$data = [
  'name' => 'Flight',
  'version' => 3,
  'features' => ['routing', 'views', 'extending']
];

$collection = new Collection($data);
```

### Accediendo a los Elementos

Puedes acceder a los elementos usando la notación de array o de objeto:

```php
// Notación de array
echo $collection['name']; // Salida: FlightPHP

// Notación de objeto
echo $collection->version; // Salida: 3
```

Si intentas acceder a una clave que no existe, obtendrás `null` en lugar de un error.

### Estableciendo Elementos

Puedes establecer elementos usando cualquiera de las dos notaciones también:

```php
// Notación de array
$collection['author'] = 'Mike Cao';

// Notación de objeto
$collection->license = 'MIT';
```

### Comprobando y Eliminando Elementos

Comprueba si un elemento existe:

```php
if (isset($collection['name'])) {
  // Haz algo
}

if (isset($collection->version)) {
  // Haz algo
}
```

Elimina un elemento:

```php
unset($collection['author']);
unset($collection->license);
```

### Iterando Sobre una Colección

Las colecciones son iterables, por lo que puedes usarlas en un bucle `foreach`:

```php
foreach ($collection as $key => $value) {
  echo "$key: $value\n";
}
```

### Contando Elementos

Puedes contar el número de elementos en una colección:

```php
echo count($collection); // Salida: 4
```

### Obteniendo Todas las Claves o Datos

Obtén todas las claves:

```php
$keys = $collection->keys(); // ['name', 'version', 'features', 'license']
```

Obtén todos los datos como un array:

```php
$data = $collection->getData();
```

### Limpiando la Colección

Elimina todos los elementos:

```php
$collection->clear();
```

### Serialización JSON

Las colecciones pueden convertirse fácilmente a JSON:

```php
echo json_encode($collection);
// Salida: {"name":"FlightPHP","version":3,"features":["routing","views","extending"],"license":"MIT"}
```

## Uso Avanzado

Puedes reemplazar el array de datos interno por completo si es necesario:

```php
$collection->setData(['foo' => 'bar']);
```

Las colecciones son especialmente útiles cuando deseas pasar datos estructurados entre componentes, o cuando quieres proporcionar una interfaz más orientada a objetos para los datos de array.

## Ver También

- [Solicitudes](/learn/requests) - Aprende cómo manejar solicitudes HTTP y cómo las colecciones pueden usarse para gestionar datos de solicitudes.
- [SimplePdo](/learn/simple-pdo) - Ayudante de base de datos que devuelve filas de consultas como colecciones.

## Solución de Problemas

- Si intentas acceder a una clave que no existe, obtendrás `null` en lugar de un error.
- Recuerda que las colecciones no son recursivas: los arrays anidados no se convierten automáticamente en colecciones.
- Si necesitas restablecer la colección, usa `$collection->clear()` o `$collection->setData([])`.

## Historial de Cambios

- v3.0 - Mejora de los type hints y soporte para PHP 8+.
- v1.0 - Lanzamiento inicial de la clase Collection.