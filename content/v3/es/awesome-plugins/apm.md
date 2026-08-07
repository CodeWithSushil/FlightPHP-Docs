# Documentación de FlightPHP APM

Bienvenido a FlightPHP APM—tu coach personal de rendimiento de aplicaciones. Esta guía es tu hoja de ruta para configurar, usar y dominar el Monitoreo de Rendimiento de Aplicaciones (APM) con FlightPHP. Ya sea que estés buscando solicitudes lentas o quieras entusiasmarte con los gráficos de latencia, te tenemos cubierto. ¡Hagamos tu aplicación más rápida, a tus usuarios más felices y tus sesiones de depuración más fáciles!

Ver una [demo](https://flightphp-docs-apm.sky-9.com/apm/dashboard) del panel para el sitio de documentación de Flight.

![FlightPHP APM](/images/apm.png)

## Por qué importa el APM

Imagina esto: tu aplicación es un restaurante ocupado. Sin una forma de rastrear cuánto tiempo toman los pedidos o dónde se atasca la cocina, estás adivinando por qué los clientes se van enfadados. El APM es tu sous-chef—vigila cada paso, desde las solicitudes entrantes hasta las consultas de base de datos, y marca cualquier cosa que te esté ralentizando. Las páginas lentas pierden usuarios (¡los estudios dicen que el 53% rebota si un sitio tarda más de 3 segundos en cargar!), y el APM te ayuda a detectar esos problemas *antes* de que duela. Es tranquilidad proactiva—menos momentos de "¿por qué está roto esto?" y más victorias de "¡mira qué fluido funciona esto!"

## Instalación

Comienza con Composer:

```bash
composer require flightphp/apm
```

Necesitarás:
- **PHP 7.4+**: Nos mantiene compatibles con las distribuciones LTS de Linux mientras soportamos PHP moderno.
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**: El framework ligero que estamos potenciando.

## Bases de datos soportadas

FlightPHP APM actualmente soporta las siguientes bases de datos para almacenar métricas:

- **SQLite3**: Simple, basada en archivos, y excelente para desarrollo local o aplicaciones pequeñas. Opción predeterminada en la mayoría de configuraciones.
- **MySQL/MariaDB**: Ideal para proyectos más grandes o entornos de producción donde necesitas almacenamiento robusto y escalable.

Puedes elegir tu tipo de base de datos durante el paso de configuración (ver abajo). Asegúrate de que tu entorno PHP tenga las extensiones necesarias instaladas (ej. `pdo_sqlite` o `pdo_mysql`).

## Comenzando

Aquí está tu guía paso a paso para la grandeza del APM:

### 1. Registrar el APM

Coloca esto en tu archivo `index.php` o `services.php` para comenzar el seguimiento:

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// Si estás agregando una conexión de base de datos
// Prefiere SimplePdo (o PdoQueryCapture de Tracy Extensions en desarrollo).
// Habilita el seguimiento de consultas APM a través del array de opciones (5to argumento).
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // requerido para capturar consultas para el APM
]);
$Apm->addPdoConnection($pdo);
```

**¿Qué está pasando aquí?**
- `LoggerFactory::create()` toma tu configuración (más sobre eso pronto) y configura un registrador—SQLite por defecto.
- `Apm` es la estrella—escucha los eventos de Flight (solicitudes, rutas, errores, etc.) y recolecta métricas.
- `bindEventsToFlightInstance($app)` lo vincula todo a tu aplicación Flight.

**Consejo Pro: Muestreo**
Si tu aplicación está ocupada, registrar *cada* solicitud podría sobrecargar las cosas. Usa una tasa de muestreo (0.0 a 1.0):

```php
$Apm = new Apm($ApmLogger, 0.1); // Registra el 10% de las solicitudes
```

Esto mantiene el rendimiento ágil mientras te da datos sólidos.

### 2. Configurarlo

Ejecuta esto para crear tu `.runway-config.json`:

```bash
php vendor/bin/runway apm:init
```

**¿Qué hace esto?**
- Lanza un asistente preguntando de dónde vienen las métricas crudas (fuente) y dónde va la data procesada (destino).
- El predeterminado es SQLite—ej. `sqlite:/tmp/apm_metrics.sqlite` para fuente, otro para destino.
- Terminarás con una configuración como:
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> Este proceso también preguntará si quieres ejecutar las migraciones para esta configuración. Si lo estás configurando por primera vez, la respuesta es sí.

**¿Por qué dos ubicaciones?**
Las métricas crudas se acumulan rápidamente (piensa en registros sin filtrar). El worker las procesa en un destino estructurado para el panel. ¡Mantiene las cosas ordenadas!

### 3. Procesar métricas con el Worker

El worker convierte las métricas crudas en datos listos para el panel. Ejecútalo una vez:

```bash
php vendor/bin/runway apm:worker
```

**¿Qué está haciendo?**
- Lee de tu fuente (ej. `apm_metrics.sqlite`).
- Procesa hasta 100 métricas (tamaño de lote predeterminado) a tu destino.
- Se detiene cuando termina o si no quedan métricas.

**Mantenerlo Ejecutándose**
Para aplicaciones en vivo, querrás procesamiento continuo. Aquí están tus opciones:

- **Modo Daemon**:
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  Se ejecuta para siempre, procesando métricas a medida que llegan. Excelente para desarrollo o configuraciones pequeñas.

- **Crontab**:
  Agrega esto a tu crontab (`crontab -e`):
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  Se ejecuta cada minuto—perfecto para producción.

- **Tmux/Screen**:
  Inicia una sesión desmontable:
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B, luego D para desmontar; `tmux attach -t apm-worker` para reconectar
  ```
  Lo mantiene ejecutándose incluso si cierras sesión.

- **Personalizaciones**:
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`: Procesa 50 métricas a la vez.
  - `--max_messages 1000`: Se detiene después de 1000 métricas.
  - `--timeout 300`: Sale después de 5 minutos.

**¿Por qué molestarse?**
Sin el worker, tu panel está vacío. Es el puente entre los registros crudos y los insights accionables.

### 4. Lanzar el Panel

Ve los signos vitales de tu aplicación:

```bash
php vendor/bin/runway apm:dashboard
```

**¿Qué es esto?**
- Inicia un servidor PHP en `http://localhost:8001/apm/dashboard`.
- Muestra registros de solicitudes, rutas lentas, tasas de error, y más.

**Personalizarlo**:
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`: Accesible desde cualquier IP (útil para visualización remota).
- `--port 8080`: Usa un puerto diferente si 8001 está ocupado.
- `--php-path`: Apunta a PHP si no está en tu PATH.

¡Visita la URL en tu navegador y explora!

#### Modo Producción

Para producción, puede que necesites probar algunas técnicas para hacer que el panel funcione ya que probablemente hay firewalls y otras medidas de seguridad en su lugar. Aquí algunas opciones:

- **Usar un Proxy Inverso**: Configura Nginx o Apache para reenviar solicitudes al panel.
- **Túnel SSH**: Si puedes hacer SSH al servidor, usa `ssh -L 8080:localhost:8001
tuusuario@tuservidor` para tunelizar el panel a tu máquina local.
- **VPN**: Si tu servidor está detrás de una VPN, conéctate a ella y accede al panel directamente.
- **Configurar Firewall**: Abre el puerto 8001 para tu IP o la red del servidor. (o cualquier puerto que configures).
- **Configurar Apache/Nginx**: Si tienes un servidor web frente a tu aplicación, puedes configurarlo a un dominio o subdominio. Si haces esto, establecerás el document root a `/ruta/a/tu/proyecto/vendor/flightphp/apm/dashboard`

#### ¿Quieres un panel diferente?

¡Puedes construir tu propio panel si quieres! ¡Mira el directorio vendor/flightphp/apm/src/apm/presenter para ideas sobre cómo presentar los datos para tu propio panel!

## Características del Panel

El panel es tu HQ del APM—aquí está lo que verás:

- **Registro de Solicitudes**: Cada solicitud con marca de tiempo, URL, código de respuesta, y tiempo total. Haz clic en "Detalles" para middleware, consultas y errores.
- **Solicitudes Más Lentas**: Las 5 solicitudes principales que consumen tiempo (ej. "/api/heavy" en 2.5s).
- **Rutas Más Lentas**: Las 5 rutas principales por tiempo promedio—excelente para detectar patrones.
- **Tasa de Error**: Porcentaje de solicitudes fallidas (ej. 2.3% 500s).
- **Percentiles de Latencia**: Tiempos de respuesta del percentil 95 (p95) y 99 (p99)—conoce tus peores escenarios.
- **Gráfico de Códigos de Respuesta**: Visualiza 200s, 404s, 500s a lo largo del tiempo.
- **Consultas/Middleware Largos**: Las 5 llamadas de base de datos y capas de middleware más lentas.
- **Aciertos/Fallos de Caché**: Con qué frecuencia tu caché salva el día.

**Extras**:
- Filtrar por "Última Hora," "Último Día," o "Última Semana."
- Alternar modo oscuro para esas sesiones nocturnas.

**Ejemplo**:
Una solicitud a `/users` podría mostrar:
- Tiempo Total: 150ms
- Middleware: `AuthMiddleware->handle` (50ms)
- Consulta: `SELECT * FROM users` (80ms)
- Caché: Acierto en `user_list` (5ms)

## Agregando Eventos Personalizados

Rastrea cualquier cosa—como una llamada API o proceso de pago:

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**¿Dónde aparece?**
En los detalles de solicitud del panel bajo "Eventos Personalizados"—expandible con formato JSON bonito.

**Caso de Uso**:
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
¡Ahora verás si esa API está arrastrando tu aplicación!

## Monitoreo de Base de Datos

Rastrea consultas PDO así:

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // requerido para capturar consultas para el APM
]);
$Apm->addPdoConnection($pdo);
```

**Lo que obtienes**:
- Texto de consulta (ej. `SELECT * FROM users WHERE id = ?`)
- Tiempo de ejecución (ej. 0.015s)
- Conteo de filas (ej. 42)

**Atención**:
- **Opcional**: Omite esto si no necesitas seguimiento de BD.
- **SimplePdo (preferido)**: Usa `SimplePdo` con `trackApmQueries => true`. El obsoleto `PdoWrapper` aún funciona (5to argumento del constructor `true`). PDO core crudo aún no está conectado—¡mantente atento!
- **Advertencia de Rendimiento**: Registrar cada consulta en un sitio pesado en BD puede ralentizar las cosas. Usa muestreo (`$Apm = new Apm($ApmLogger, 0.1)`) para aligerar la carga.

**Salida de Ejemplo**:
- Consulta: `SELECT name FROM products WHERE price > 100`
- Tiempo: 0.023s
- Filas: 15

## Opciones del Worker

Ajusta el worker a tu gusto:

- `--timeout 300`: Se detiene después de 5 minutos—bueno para pruebas.
- `--max_messages 500`: Limita a 500 métricas—lo mantiene finito.
- `--batch_size 200`: Procesa 200 a la vez—equilibra velocidad y memoria.
- `--daemon`: Se ejecuta sin parar—ideal para monitoreo en vivo.

**Ejemplo**:
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
Se ejecuta por una hora, procesando 100 métricas a la vez.

## ID de Solicitud en la App

Cada solicitud tiene un ID de solicitud único para seguimiento. Puedes usar este ID en tu aplicación para correlacionar registros y métricas. Por ejemplo, puedes agregar el ID de solicitud a una página de error:

```php
Flight::map('error', function($message) {
	// Obtén el ID de solicitud del encabezado de respuesta X-Flight-Request-Id
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// Adicionalmente podrías obtenerlo de la variable de Flight
	// Este método no funcionará bien en swoole u otras plataformas async.
	// $requestId = Flight::get('apm.request_id');
	
	echo "Error: $message (Request ID: $requestId)";
});
```

## Actualización

Si estás actualizando a una versión más nueva del APM, hay una posibilidad de que haya migraciones de base de datos que necesiten ejecutarse. Puedes hacer esto ejecutando el siguiente comando:

```bash
php vendor/bin/runway apm:migrate
```
Esto ejecutará cualquier migración que sea necesaria para actualizar el esquema de la base de datos a la última versión.

**Nota:** Si tu base de datos APM es grande en tamaño, estas migraciones pueden tomar algo de tiempo para ejecutarse. Puede que quieras ejecutar este comando durante horas de bajo tráfico.

### Actualizando de 0.4.3 -> 0.5.0

Si estás actualizando de 0.4.3 a 0.5.0, necesitarás ejecutar el siguiente comando:

```bash
php vendor/bin/runway apm:config-migrate
```

Esto migrará tu configuración del formato antiguo usando el archivo `.runway-config.json` al nuevo formato que almacena los pares clave/valor en el archivo `config.php`.

## Purgando Datos Antiguos

Para mantener tu base de datos ordenada, puedes purgar datos antiguos. Esto es especialmente útil si estás ejecutando una aplicación ocupada y quieres mantener el tamaño de la base de datos manejable.
Puedes hacer esto ejecutando el siguiente comando:

```bash
php vendor/bin/runway apm:purge
```
Esto eliminará todos los datos más antiguos de 30 días de la base de datos. Puedes ajustar el número de días pasando un valor diferente a la opción `--days`:

```bash
php vendor/bin/runway apm:purge --days 7
```
Esto eliminará todos los datos más antiguos de 7 días de la base de datos.

## Solución de Problemas

¿Atascado? Prueba esto:

- **¿Sin datos en el panel?**
  - ¿Está ejecutándose el worker? Verifica `ps aux | grep apm:worker`.
  - ¿Coinciden las rutas de configuración? Verifica que los DSN de `.runway-config.json` apunten a archivos reales.
  - Ejecuta `php vendor/bin/runway apm:worker` manualmente para procesar métricas pendientes.

- **¿Errores del Worker?**
  - Echa un vistazo a tus archivos SQLite (ej. `sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`).
  - Revisa los registros de PHP para trazas de pila.

- **¿El panel no inicia?**
  - ¿Puerto 8001 en uso? Usa `--port 8080`.
  - ¿PHP no encontrado? Usa `--php-path /usr/bin/php`.
  - ¿Firewall bloqueando? Abre el puerto o usa `--host localhost`.

- **¿Demasiado lento?**
  - Baja la tasa de muestreo: `$Apm = new Apm($ApmLogger, 0.05)` (5%).
  - Reduce el tamaño de lote: `--batch_size 20`.

- **¿No rastreando excepciones/errores?**
  - Si tienes [Tracy](https://tracy.nette.org/) habilitado para tu proyecto, sobrescribirá el manejo de errores de Flight. Necesitarás deshabilitar Tracy y luego asegurarte de que `Flight::set('flight.handle_errors', true);` esté configurado.

- **¿No rastreando consultas de base de datos?**
  - Prefiere `SimplePdo` con `['trackApmQueries' => true]` como el 5to argumento del constructor (array de opciones).
  - Si aún usas el obsoleto `PdoWrapper`, pasa `true` como el 5to argumento.
  - Llama a `$Apm->addPdoConnection($pdo)` después de crear la conexión.