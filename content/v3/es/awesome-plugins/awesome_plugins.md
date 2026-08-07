# Plugins Impresionantes

Flight es increíblemente extensible. Hay una serie de plugins que se pueden utilizar para agregar funcionalidad a tu aplicación Flight. Algunos son oficialmente soportados por el Equipo de Flight y otros son bibliotecas micro/lite para ayudarte a comenzar.

## Herramientas de IA

Flight puede ser aún más genial con plugins impulsados por IA.

- [Flight MCP](/awesome-plugins/mcp) - Un plugin para integrar MCP (Model Control Protocol) con Flight, permitiendo una funcionalidad sin problemas impulsada por IA. Principalmente enfocado en las páginas de documentación, ayuda a mantener bajos los costos de tokens proporcionando la información más actualizada sobre tus proyectos Flight.

## Documentación de API

La documentación de API es crucial para cualquier API. Ayuda a los desarrolladores a entender cómo interactuar con tu API y qué esperar a cambio. Hay un par de herramientas disponibles para ayudarte a generar documentación de API para tus Proyectos Flight.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Publicación de blog escrita por Daniel Schreiber sobre cómo usar la Especificación OpenAPI con FlightPHP para construir tu API utilizando un enfoque API primero.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI es una gran herramienta para ayudarte a generar documentación de API para tus proyectos Flight. Es muy fácil de usar y se puede personalizar para satisfacer tus necesidades. Esta es la biblioteca PHP para ayudarte a generar la documentación Swagger.

## Monitoreo de Rendimiento de Aplicaciones (APM)

El Monitoreo de Rendimiento de Aplicaciones (APM) es crucial para cualquier aplicación. Te ayuda a entender cómo está funcionando tu aplicación y dónde están los cuellos de botella. Hay una serie de herramientas APM que se pueden usar con Flight.
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM es una biblioteca APM simple que se puede usar para monitorear tus aplicaciones Flight. Se puede usar para monitorear el rendimiento de tu aplicación y ayudarte a identificar cuellos de botella.

## Asíncrono

Flight ya es un framework rápido, ¡pero agregarle un motor turbo lo hace todo más divertido (y desafiante)!

- [flightphp/async](/awesome-plugins/async) - Biblioteca Asíncrona oficial de Flight. Esta biblioteca es una forma simple de agregar procesamiento asíncrono a tu aplicación. Utiliza Swoole/Openswoole bajo el capó para proporcionar una forma simple y efectiva de ejecutar tareas de forma asíncrona.

## Autorización/Permisos

La autorización y los permisos son cruciales para cualquier aplicación que requiera controles para establecer quién puede acceder a qué.

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - Biblioteca de Permisos oficial de Flight. Esta biblioteca es una forma simple de agregar permisos a nivel de usuario y aplicación a tu aplicación.

## Autenticación

La autenticación es esencial para las aplicaciones que necesitan verificar la identidad del usuario y asegurar los endpoints de API.

- [firebase/php-jwt](/awesome-plugins/jwt) - Biblioteca de JSON Web Token (JWT) para PHP. Una forma simple y segura de implementar autenticación basada en tokens en tus aplicaciones Flight. Perfecta para autenticación de API sin estado, proteger rutas con middleware e implementar flujos de autorización de estilo OAuth.

## Caché

El caché es una gran manera de acelerar tu aplicación. Hay una serie de bibliotecas de caché que se pueden usar con Flight.

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Clase de caché en archivo PHP ligera, simple e independiente

## CLI

Las aplicaciones CLI son una gran manera de interactuar con tu aplicación. Puedes usarlas para generar controladores, mostrar todas las rutas, y más.

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway es una aplicación CLI que te ayuda a gestionar tus aplicaciones Flight.

## Cookies

Las cookies son una gran manera de almacenar pequeños bits de datos en el lado del cliente. Se pueden usar para almacenar preferencias de usuario, configuraciones de aplicación, y más.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie es una biblioteca PHP que proporciona una forma simple y efectiva de gestionar cookies.

## Depuración

La depuración es crucial cuando estás desarrollando en tu entorno local. Hay algunos plugins que pueden elevar tu experiencia de depuración.

- [tracy/tracy](/awesome-plugins/tracy) - Este es un manejador de errores con todas las funciones que se puede usar con Flight. Tiene una serie de paneles que pueden ayudarte a depurar tu aplicación. También es muy fácil de extender y agregar tus propios paneles.
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Usado con el manejador de errores [Tracy](/awesome-plugins/tracy), este plugin agrega algunos paneles adicionales para ayudar con la depuración específicamente para proyectos Flight.

## Bases de Datos

Las bases de datos son el núcleo de la mayoría de las aplicaciones. Así es como almacenas y recuperas datos. Algunas bibliotecas de bases de datos son simplemente envoltorios para escribir consultas y algunas son ORMs completos.

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Ayudante PDO oficial de Flight que forma parte del núcleo. Este es un envoltorio moderno con métodos de ayuda convenientes como `insert()`, `update()`, `delete()`, y `transaction()` para simplificar las operaciones de base de datos. Todos los resultados se devuelven como Colecciones para acceso flexible de array/objeto. No es un ORM, solo una mejor manera de trabajar con PDO.
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Envoltorio PDO oficial de Flight que forma parte del núcleo (obsoleto desde v3.18.0). Usa SimplePdo en su lugar.
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - ORM/Mapeador ActiveRecord oficial de Flight. Una gran biblioteca pequeña para recuperar y almacenar fácilmente datos en tu base de datos.
- [byjg/php-migration](/awesome-plugins/migrations) - Plugin para hacer seguimiento de todos los cambios de base de datos para tu proyecto.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Constructor de consultas SQL fluido y ligero que genera SQL y parámetros para sentencias preparadas. Funciona muy bien con [SimplePdo](/learn/simple-pdo).

## Encriptación

La encriptación es crucial para cualquier aplicación que almacene datos sensibles. Encriptar y desencriptar los datos no es terriblemente difícil, pero almacenar correctamente la clave de encriptación [puede](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [ser](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [difícil](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). Lo más importante es nunca almacenar tu clave de encriptación en un directorio público o comprometerla en tu repositorio de código.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - Esta es una biblioteca que se puede usar para encriptar y desencriptar datos. Ponerse en marcha es bastante simple para comenzar a encriptar y desencriptar datos.

## Cola de Trabajos

Las colas de trabajos son realmente útiles para procesar tareas de forma asíncrona. Esto puede ser enviar correos electrónicos, procesar imágenes, o cualquier cosa que no necesite hacerse en tiempo real.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue es una biblioteca que se puede usar para procesar trabajos de forma asíncrona. Se puede usar con beanstalkd, MySQL/MariaDB, SQLite, y PostgreSQL.

## Sesión

Las sesiones no son realmente útiles para APIs pero para construir una aplicación web, las sesiones pueden ser cruciales para mantener el estado y la información de inicio de sesión.

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - Biblioteca de Sesiones oficial de Flight. Esta es una biblioteca de sesiones simple que se puede usar para almacenar y recuperar datos de sesión. Utiliza el manejo de sesiones incorporado de PHP.
- [Ghostff/Session](/awesome-plugins/ghost-session) - Administrador de Sesiones PHP (sin bloqueo, flash, segmento, encriptación de sesión). Utiliza PHP open_ssl para encriptación/desencriptación opcional de datos de sesión.

## Plantillas

Las plantillas son el núcleo de cualquier aplicación web con una interfaz de usuario. Hay una serie de motores de plantillas que se pueden usar con Flight.

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - Este es un motor de plantillas muy básico que forma parte del núcleo. No se recomienda usarlo si tienes más de un par de páginas en tu proyecto.
- [latte/latte](/awesome-plugins/latte) - Latte es un motor de plantillas completo que es muy fácil de usar y se siente más cercano a una sintaxis PHP que Twig o Smarty. También es muy fácil de extender y agregar tus propios filtros y funciones.
- [twig/twig](/awesome-plugins/twig) - Twig es un motor de plantillas flexible, rápido y seguro (el mismo que usa Symfony). Las herramientas de IA y muchos desarrolladores PHP lo conocen bien, escapa automáticamente la salida por defecto, y tiene un enorme ecosistema de extensiones.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate es un potente motor de plantillas PHP con compilación de assets, herencia de plantillas y procesamiento de variables. Cuenta con minificación automática de CSS/JS, caché, codificación Base64 e integración opcional con el framework Flight PHP.

## Integración con WordPress

¿Quieres usar Flight en tu proyecto WordPress? ¡Hay un plugin útil para eso!

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Este plugin de WordPress te permite ejecutar Flight junto con WordPress. Es perfecto para agregar APIs personalizadas, microservicios, o incluso aplicaciones completas a tu sitio WordPress usando el framework Flight. ¡Súper útil si quieres lo mejor de ambos mundos!

## Contribuyendo

¿Tienes un plugin que te gustaría compartir? ¡Envía una solicitud de extracción para agregarlo a la lista!