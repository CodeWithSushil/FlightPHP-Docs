# Aprende sobre Flight

Flight es un framework para PHP rápido, simple y extensible. Es bastante versátil y puede usarse para construir cualquier tipo de aplicación web.
Está creado pensando en la simplicidad y está escrito de una manera fácil de entender y usar, tanto por humanos como por [asistentes de codificación con IA](/learn/ai).

> **Nota:** Verás ejemplos que usan `Flight::` como variable estática y otros que usan el objeto Engine `$app->`. Ambos funcionan de manera intercambiable. `$app` y `$this->app` en un controlador/middleware es el enfoque recomendado por el equipo de Flight (y lo que el esqueleto oficial y `AGENTS.md` estandarizan para proyectos nuevos).

## Componentes principales

### [Enrutamiento](/learn/routing)

Aprende cómo gestionar las rutas de tu aplicación web. Esto también incluye agrupación de rutas, parámetros de ruta y middleware.

### [Middleware](/learn/middleware)

Aprende cómo usar middleware para filtrar solicitudes y respuestas en tu aplicación.

### [Autocarga](/learn/autoloading)

Aprende cómo autocargar tus propias clases. La **capitalización** de las carpetas debe coincidir con tus espacios de nombres; el esqueleto usa `App\` y carpetas en PascalCase como `app/Controller/`.

### [Peticiones](/learn/requests)

Aprende cómo manejar solicitudes y respuestas en tu aplicación.

### [Respuestas](/learn/responses)

Aprende cómo enviar respuestas a tus usuarios.

### [Plantillas HTML](/learn/templates)

Aprende cómo renderizar HTML con Twig (predeterminado del esqueleto), Latte u otros motores, no solo las vistas PHP integradas.

### [Seguridad](/learn/security)

Aprende cómo proteger tu aplicación de amenazas de seguridad comunes.

### [Configuración](/learn/configuration)

Aprende cómo configurar el framework para tu aplicación.

### [Administrador de eventos](/learn/events)

Aprende cómo usar el sistema de eventos para agregar eventos personalizados a tu aplicación.

### [Extender Flight](/learn/extending)

Aprende cómo extender el framework agregando tus propios métodos y clases.

### [Hooks de métodos y filtrado](/learn/filtering)

Aprende cómo agregar hooks de eventos a tus métodos y a los métodos internos del framework.

### [Contenedor de inyección de dependencias (DIC)](/learn/dependency-injection-container)

Aprende cómo usar contenedores de inyección de dependencias (DIC) para gestionar las dependencias de tu aplicación.

## Clases de utilidad

### [Collections](/learn/collections)

Las colecciones se usan para almacenar datos y permitir acceder a ellos como un array o como un objeto para facilitar su uso.

### [JSON Wrapper](/learn/json)

Tiene algunas funciones simples para que la codificación y decodificación de tu JSON sea consistente.

### [SimplePdo](/learn/simple-pdo)

PDO a veces puede causar más dolores de cabeza de lo necesario. SimplePdo es una clase auxiliar moderna de PDO con métodos convenientes como `insert()`, `update()`, `delete()` y `transaction()` para facilitar las operaciones de base de datos.

### [PdoWrapper](/learn/pdo-wrapper) (Obsoleto)

El wrapper original de PDO está obsoleto a partir de la versión v3.18.0. Por favor, usa [SimplePdo](/learn/simple-pdo) en su lugar.

### [Manejador de archivos subidos](/learn/uploaded-file)

Una clase simple para ayudar a gestionar archivos subidos y moverlos a una ubicación permanente.

## Conceptos importantes

### [¿Por qué un framework?](/learn/why-frameworks)

Aquí hay un artículo corto sobre por qué deberías usar un framework. Es buena idea entender los beneficios de usar un framework antes de empezar a usar uno.

Además, un excelente tutorial ha sido creado por [@lubiana](https://git.php.fail/lubiana). Aunque no entra en gran detalle sobre Flight específicamente,
esta guía te ayudará a entender algunos de los conceptos principales que rodean a un framework y por qué son beneficiosos de usar.
Puedes encontrar el tutorial [aquí](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight comparado con otros frameworks](/learn/flight-vs-another-framework)

Si estás migrando desde otro framework como Laravel, Slim, Fat-Free o Symfony a Flight, esta página te ayudará a entender las diferencias entre ambos.

## Otros temas

### [Pruebas unitarias](/learn/unit-testing)

Sigue esta guía para aprender cómo probar unitariamente tu código de Flight para que sea sólido como una roca.

### [IA y experiencia de desarrollo](/learn/ai)

Flight está diseñado para combinarse con LLMs de codificación: `AGENTS.md`, comandos `ai:*` de Runway y un diseño de esqueleto claro para que los agentes mantengan el patrón.

### [Migración de v2 a v3](/learn/migrating-to-v3)

La compatibilidad hacia atrás se ha mantenido en su mayor parte, pero hay algunos cambios que debes tener en cuenta al migrar de v2 a v3.