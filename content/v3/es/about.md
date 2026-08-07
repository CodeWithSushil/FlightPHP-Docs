# Flight PHP Framework

Flight es un framework rápido, simple y extensible para PHP—diseñado para desarrolladores que quieren hacer las cosas rápidamente, sin complicaciones. Ya sea que estés construyendo una aplicación web clásica, una API ultrarrápida, o trabajando con asistentes de codificación IA, el bajo consumo de recursos y el diseño sencillo de Flight lo convierten en una opción perfecta. Flight está pensado para ser ligero, pero también puede manejar los requisitos de arquitecturas empresariales.

## ¿Por qué elegir Flight?

- **Fácil para principiantes:** Flight es un excelente punto de partida para nuevos desarrolladores PHP. Su estructura clara y sintaxis simple te ayudan a aprender desarrollo web sin perderte entre código repetitivo.
- **Amado por profesionales:** Los desarrolladores experimentados aprecian Flight por su flexibilidad y control. Puedes escalar desde un pequeño prototipo hasta una aplicación completa sin necesidad de cambiar de framework.
- **Compatible con versiones anteriores:** Valoramos tu tiempo. Flight v3 es una mejora de v2, manteniendo casi toda la misma API. Creemos en la evolución, no en la revolución—no más "romper el mundo" cada vez que sale una versión mayor.
- **Sin dependencias:** El núcleo de Flight está completamente libre de dependencias—sin polyfills, sin paquetes externos, ni siquiera interfaces PSR. Esto significa menos vectores de ataque, un menor consumo de recursos y sin cambios inesperados que rompan la compatibilidad provenientes de dependencias externas. Los plugins opcionales pueden incluir dependencias, pero el núcleo siempre permanecerá ligero y seguro.
- **Amigable con IA:** La pequeña superficie de API de Flight y el [esqueleto oficial](https://github.com/flightphp/skeleton) (un diseño, `AGENTS.md`, inyección por constructor) facilitan que las herramientas de codificación IA se mantengan dentro del patrón. Mismo código base ya sea que escribas cada línea o trabajes con un agente. [Aprende más sobre usar IA con Flight](/learn/ai).

## Resumen en Video

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">Bastante simple, ¿verdad?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">Aprende más</a> sobre Flight en la documentación!
    </div>
  </div>
</div>

## Inicio Rápido

Para hacer una instalación básica rápida, instálalo con Composer:

```bash
composer require flightphp/core
```

O puedes descargar un zip del repositorio [aquí](https://github.com/flightphp/core). Luego tendrás un archivo `index.php` básico como el siguiente:

```php
<?php

// si se instaló con composer
require 'vendor/autoload.php';
// o si se instaló manualmente por archivo zip
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

¡Eso es todo! Tienes una aplicación Flight básica. Ahora puedes ejecutar este archivo con `php -S localhost:8000` y visitar `http://localhost:8000` en tu navegador para ver la salida.

Ejemplos cortos de `Flight::` como este son geniales para aprender y aplicaciones micro. Para un diseño de proyecto completo que humanos y herramientas IA compartan, usa el esqueleto de abajo.

## Aplicación Esqueleto/Plantilla

Hay un iniciador oficial para ayudarte a comenzar cualquier nuevo proyecto Flight. Configura la estructura, configuración, scripts de Composer e instrucciones amigables para IA desde el primer momento.

Consulta [flightphp/skeleton](https://github.com/flightphp/skeleton) para un proyecto listo para usar, o visita la página de [ejemplos](examples) para inspiración. ¿Quieres los detalles del flujo de trabajo de IA? [Explora IA y experiencia de desarrollador](/learn/ai).

Lo que obtienes (nivel alto):

- **Espacios de nombres `App\`** con carpetas en PascalCase (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—la **capitalización** de las carpetas debe coincidir con el espacio de nombres (ver [Autocarga](/learn/autoloading))
- **Inyección de Dice + `Engine`** para que los controladores permanezcan testeables (prefiere `$this->app` sobre `Flight::` en el código de la aplicación)
- Vistas **Twig**, muestra de **SimplePdo** + ActiveRecord, **migrate** de Runway
- **`AGENTS.md`** en la raíz (más copias por alcance) y **`SECURITY.md`** para asistentes y política de seguridad

## Instalando la Aplicación Esqueleto

¡Bastante simple!

```bash
# Crear el nuevo proyecto
composer create-project flightphp/skeleton my-project/
# Entrar al directorio del nuevo proyecto
cd my-project/
# ¡Levantar el servidor de desarrollo local para comenzar de inmediato!
composer start
```

Crea la estructura del proyecto, copia `config_sample.php` → `config.php` (y `.env.example` → `.env` cuando esté presente), y estás listo para comenzar. Datos de muestra opcionales:

```bash
php runway migrate
# luego visita /posts y /api/posts
```

## Alto Rendimiento

Flight es uno de los frameworks PHP más rápidos que existen. Su núcleo ligero significa menos sobrecarga y más velocidad—perfecto tanto para aplicaciones tradicionales como para flujos de trabajo modernos asistidos por IA. Puedes ver todos los benchmarks en [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Mira el benchmark a continuación con algunos otros frameworks PHP populares.

| Framework | Reqs/seg Texto plano | Reqs/seg JSON |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight e IA

¿Curioso sobre cómo Flight se combina con LLMs de codificación? [Descubre](/learn/ai) cómo `AGENTS.md`, los comandos `ai:*` de Runway, y el diseño del esqueleto mantienen a los asistentes en el camino correcto.

## Estabilidad y Compatibilidad con Versiones Anteriores

Valoramos tu tiempo. Todos hemos visto frameworks que se reinventan completamente cada par de años, dejando a los desarrolladores con código roto y migraciones costosas. Flight es diferente. Flight v3 fue diseñado como una mejora de v2, lo que significa que la API que conoces y amas no ha sido eliminada. De hecho, la mayoría de los proyectos de v2 funcionarán sin ningún cambio en v3. 

Estamos comprometidos a mantener Flight estable para que puedas enfocarte en construir tu aplicación, no en arreglar tu framework. El esqueleto puede ser opinado para proyectos *nuevos*; las APIs del núcleo permanecen familiares para todos los demás.

# Comunidad

Estamos en Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

Y Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Contribuir

Hay dos formas en las que puedes contribuir a Flight:

1. Contribuir al framework principal visitando el [repositorio principal](https://github.com/flightphp/core).
2. ¡Ayuda a mejorar la documentación! Este sitio web de documentación está alojado en [Github](https://github.com/flightphp/docs). Si encuentras un error o quieres mejorar algo, no dudes en enviar una solicitud de extracción. ¡Nos encantan las actualizaciones y nuevas ideas—especialmente alrededor de IA y nuevas tecnologías!

# Requisitos

Flight requiere PHP 7.4 o superior.

**Nota:** PHP 7.4 es compatible porque en el momento actual de escritura (2024) PHP 7.4 es la versión predeterminada para algunas distribuciones Linux LTS. Forzar un cambio a PHP >8 causaría muchos problemas para esos usuarios. El framework también soporta PHP >8.

# Licencia

Flight se publica bajo la licencia [MIT](https://github.com/flightphp/core/blob/master/LICENSE).