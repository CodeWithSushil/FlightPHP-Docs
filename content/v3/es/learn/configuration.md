# Configuración

## Descripción general 

Flight proporciona una forma sencilla de configurar varios aspectos del framework para adaptarse a las necesidades de tu aplicación. Algunos están establecidos por defecto, pero puedes sobrescribirlos según sea necesario. También puedes establecer tus propias variables para usarlas en toda tu aplicación.

## Entendimiento

Puedes personalizar ciertos comportamientos de Flight estableciendo valores de configuración
a través del método `set`.

```php
Flight::set('flight.log_errors', true);
```

En el archivo `app/config/config.php`, puedes ver todas las variables de configuración predeterminadas disponibles para ti.

## Uso básico

### Opciones de configuración de Flight

La siguiente es una lista de todas las configuraciones de configuración disponibles:

- **flight.base_url** `?string` - Sobrescribe la url base de la solicitud si Flight se ejecuta en un subdirectorio. (predeterminado: null)
- **flight.case_sensitive** `bool` - Coincidencia sensible a mayúsculas y minúsculas para URLs. (predeterminado: false)
- **flight.handle_errors** `bool` - Permitir que Flight maneje todos los errores internamente. (predeterminado: true)
  - Si quieres que Flight maneje los errores en lugar del comportamiento predeterminado de PHP, esto debe ser true.
  - Si tienes [Tracy](/awesome-plugins/tracy) instalado, quieres establecer esto en false para que Tracy pueda manejar los errores.
  - Si tienes el complemento [APM](/awesome-plugins/apm) instalado, quieres establecer esto en true para que el APM pueda registrar los errores.
- **flight.log_errors** `bool` - Registrar errores en el archivo de registro de errores del servidor web. (predeterminado: false)
  - Si tienes [Tracy](/awesome-plugins/tracy) instalado, Tracy registrará los errores según las configuraciones de Tracy, no esta configuración.
- **flight.debug** `bool` - Mostrar información detallada del error (mensaje de excepción, código y seguimiento de pila) en el navegador cuando ocurre un error. (predeterminado: false)
  - **Nunca habilites esto en producción** — filtra detalles internos de la aplicación. Úsalo solo para desarrollo local o staging.
  - Cuando es `false`, se muestra un `500 Internal Server Error` genérico en su lugar. Combínalo con `flight.log_errors` para capturar errores del lado del servidor.
- **flight.allow_method_override** `bool` - Permitir que el método HTTP se sobrescriba a través del encabezado de solicitud `X-HTTP-Method-Override` o un campo `_method` en el cuerpo POST. (predeterminado: true)
  - **Se recomienda establecer esto en `false`** para aplicaciones que no necesitan suplantación de métodos basada en formularios HTML, ya que evita que los clientes falsifiquen solicitudes `DELETE` o `PUT` a través de un formulario POST estándar.
  - Consulta [Seguridad](/learn/security#flight-configuration-hardening) para más detalles.
- **flight.views.path** `string` - Directorio que contiene archivos de plantilla de vista. (predeterminado: ./views)
- **flight.views.extension** `string` - Extensión de archivo de plantilla de vista. (predeterminado: .php)
- **flight.content_length** `bool` - Establecer el encabezado `Content-Length`. (predeterminado: true)
  - Si estás usando [Tracy](/awesome-plugins/tracy), esto debe establecerse en false para que Tracy pueda renderizar correctamente.
- **flight.v2.output_buffering** `bool` - Usar el almacenamiento en búfer de salida heredado. Consulta [migrando a v3](migrating-to-v3). (predeterminado: false)

### Configuración del cargador

Además, hay otra configuración para el cargador. Esto te permitirá 
cargar automáticamente clases con `_` en el nombre de la clase.

```php
// Habilitar la carga de clases con guiones bajos
// Predeterminado en true
Loader::$v2ClassLoading = false;
```

### Variables

Flight te permite guardar variables para que puedan usarse en cualquier lugar de tu aplicación.

```php
// Guarda tu variable
Flight::set('id', 123);

// En otro lugar de tu aplicación
$id = Flight::get('id');
```
Para ver si una variable ha sido establecida puedes hacer:

```php
if (Flight::has('id')) {
  // Haz algo
}
```

Puedes borrar una variable haciendo:

```php
// Borra la variable id
Flight::clear('id');

// Borra todas las variables
Flight::clear();
```

> **Nota:** Solo porque puedas establecer una variable no significa que debas hacerlo. Usa esta característica con moderación. La razón es que cualquier cosa almacenada aquí se convierte en una variable global. Las variables globales son malas porque pueden cambiarse desde cualquier lugar de tu aplicación, lo que dificulta rastrear errores. Además, esto puede complicar cosas como [pruebas unitarias](/guides/unit-testing).

### Errores y excepciones

Todos los errores y excepciones son capturados por Flight y pasados al método `error`.
si `flight.handle_errors` está establecido en true.

El comportamiento predeterminado es enviar una respuesta genérica `HTTP 500 Internal Server Error`
con alguna información de error.

Puedes [sobrescribir](/learn/extending) este comportamiento para tus propias necesidades:

```php
Flight::map('error', function (Throwable $error) {
  // Manejar error
  echo $error->getTraceAsString();
});
```

Por defecto, los errores no se registran en el servidor web. Puedes habilitar esto cambiando la configuración:

```php
Flight::set('flight.log_errors', true);
```

#### 404 No encontrado

Cuando no se puede encontrar una URL, Flight llama al método `notFound`. El comportamiento
predeterminado es enviar una respuesta `HTTP 404 Not Found` con un mensaje simple.

Puedes [sobrescribir](/learn/extending) este comportamiento para tus propias necesidades:

```php
Flight::map('notFound', function () {
  // Manejar no encontrado
});
```

## Ver también
- [Extender Flight](/learn/extending) - Cómo extender y personalizar la funcionalidad principal de Flight.
- [Pruebas unitarias](/guides/unit-testing) - Cómo escribir pruebas unitarias para tu aplicación Flight.
- [Tracy](/awesome-plugins/tracy) - Un complemento para manejo avanzado de errores y depuración.
- [Extensiones de Tracy](/awesome-plugins/tracy_extensions) - Extensiones para integrar Tracy con Flight.
- [APM](/awesome-plugins/apm) - Un complemento para monitoreo de rendimiento de aplicaciones y seguimiento de errores.

## Solución de problemas
- Si tienes problemas para encontrar todos los valores de tu configuración, puedes hacer `var_dump(Flight::get());`

## Registro de cambios
- v3.18.1 - Se agregaron las opciones de configuración `flight.debug` y `flight.allow_method_override`.
- v3.5.0 - Se agregó la configuración para `flight.v2.output_buffering` para admitir el comportamiento de almacenamiento en búfer de salida heredado.
- v2.0 - Configuraciones principales agregadas.