# Instrucciones de Instalación

Hay algunos requisitos previos básicos antes de poder instalar Flight. Concretamente, necesitarás:

1. [Instalar PHP en tu sistema](#installing-php)
2. [Instalar Composer](https://getcomposer.org) para la mejor experiencia de desarrollo.

## Instalación Básica

Si estás usando [Composer](https://getcomposer.org), puedes ejecutar el siguiente comando:

```bash
composer require flightphp/core
```

Esto solo colocará los archivos principales de Flight en tu sistema. Necesitarás definir la estructura del proyecto, [layout](/learn/templates), [dependencias](/learn/dependency-injection-container), [configuraciones](/learn/configuration), [autoloading](/learn/autoloading), etc. Este método asegura que no se instalen otras dependencias además de Flight.

También puedes [descargar los archivos](https://github.com/flightphp/core/archive/master.zip) directamente y extraerlos a tu directorio web.

La instalación básica es perfecta para aprender, micro APIs y experimentos de copiar y pegar. Para una estructura de aplicación completa que los humanos *y* [herramientas de codificación de IA](/learn/ai) puedan seguir de la misma manera, usa el esqueleto recomendado a continuación.

## Instalación Recomendada

Es muy recomendable comenzar con la aplicación [flightphp/skeleton](https://github.com/flightphp/skeleton) para cualquier proyecto nuevo. La instalación es muy sencilla.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# base de datos de muestra opcional + demo de publicaciones
php runway migrate
```

Ese paso configura la estructura del proyecto, el autoloading PSR-4 de Composer, la configuración y herramientas como [Tracy](/awesome-plugins/tracy), [Tracy Extensions](/awesome-plugins/tracy-extensions) y [Runway](/awesome-plugins/runway). También incluye el **`AGENTS.md`** raíz (y copias específicas bajo `app/`) para que los asistentes de IA compartan una misma estructura contigo—ver [IA y experiencia de desarrollador](/learn/ai).

### Lo que te ofrece el esqueleto

```text
project-root/
├── AGENTS.md              # fuente de verdad para IA / agentes
├── SECURITY.md            # expectativas de seguridad
├── .env.example           # secretos / overlays de despliegue (copiado a .env)
├── public/index.php       # solo entrada web
├── app/
│   ├── config/            # bootstrap, rutas, servicios, config_sample.php
│   ├── Controller/        # App\Controller\*  (¡carpeta PascalCase!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # comandos CLI de Runway
│   ├── views/             # plantillas Twig (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # migraciones SQL (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Los namespaces siguen el caso de la carpeta.** Composer mapea `"App\\": "app/"`, por lo tanto:

| Ruta en disco | Namespace |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

En Linux, `app/controller/` **no** es lo mismo que `app/Controller/`. El autoloading distingue entre mayúsculas y minúsculas—coincide con las carpetas PascalCase del esqueleto. Detalles: [Autoloading](/learn/autoloading).

**Configuración por defecto (nuevos proyectos):** vistas Twig, SimplePdo + ActiveRecord, Dice con inyección de `Engine` (prefiere no usar `Flight::` dentro de las clases de la aplicación), SQLite opcional después de `php runway migrate`.

`create-project` normalmente copia `app/config/config_sample.php` → `config.php` y `.env.example` → `.env` cuando están presentes. Las rutas viven en `app/config/routes.php`; los servicios y la DI viven en `app/config/services.php`.

> **Docs ↔ esqueleto:** Estos documentos enseñan las **APIs** de Flight (a menudo con ejemplos cortos de `Flight::`). El esqueleto fija la **forma de la aplicación**. Al agregar código bajo `app/`, sigue el árbol del esqueleto; usa los docs para nombres de métodos, opciones y plugins.

## Configura tu Servidor Web

### Servidor de Desarrollo PHP Integrado

Esta es, con mucho, la forma más simple de poner todo en marcha. Puedes usar el servidor integrado para ejecutar tu aplicación e incluso usar SQLite como base de datos (siempre que sqlite3 esté instalado en tu sistema) y ¡no necesitar mucho más! Solo ejecuta el siguiente comando una vez que PHP esté instalado:

```bash
php -S localhost:8000
# o con la aplicación esqueleto
composer start
```

Luego abre tu navegador y ve a `http://localhost:8000`.

Si quieres que la raíz de documentos de tu proyecto sea un directorio diferente (Ej: tu proyecto está en `~/myproject`, pero tu raíz de documentos es `~/myproject/public/`), puedes ejecutar el siguiente comando una vez que estés en el directorio `~/myproject`:

```bash
php -S localhost:8000 -t public/
# con la aplicación esqueleto, esto ya está configurado
composer start
```

Luego abre tu navegador y ve a `http://localhost:8000`.

### Apache

Asegúrate de que Apache ya esté instalado en tu sistema. Si no, busca en Google cómo instalar Apache en tu sistema.

Para Apache, edita tu archivo `.htaccess` con lo siguiente:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Nota**: Si necesitas usar Flight en un subdirectorio, agrega la línea
> `RewriteBase /subdir/` justo después de `RewriteEngine On`.

> **Nota**: Si quieres proteger todos los archivos del servidor, como una base de datos o un archivo env.
> Pon esto en tu archivo `.htaccess`:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Asegúrate de que Nginx ya esté instalado en tu sistema. Si no, busca en Google cómo instalar Nginx en tu sistema.

Para Nginx, agrega lo siguiente a tu declaración de servidor:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Crea tu archivo `index.php`

Si estás haciendo una instalación básica, necesitarás algo de código para comenzar.

```php
<?php

// Si estás usando Composer, requiere el autoloader.
require 'vendor/autoload.php';
// si no estás usando Composer, carga el framework directamente
// require 'flight/Flight.php';

// Luego define una ruta y asigna una función para manejar la solicitud.
Flight::route('/', function () {
  echo 'hello world!';
});

// Finalmente, inicia el framework.
Flight::start();
```

Con la aplicación esqueleto, la entrada pública solo arranca la aplicación. Las rutas se registran en `app/config/routes.php` (típicamente `[App\Controller\…::class, 'method']` para que Dice pueda inyectar dependencias). Los servicios, Twig, SimplePdo y el contenedor se conectan en `app/config/services.php`. Esa estructura es intencional para que las herramientas de IA y los humanos editen los mismos lugares cada vez.

## Instalando PHP

Si ya tienes `php` instalado en tu sistema, continúa y omite estas instrucciones y ve a [la sección de descarga](#download-the-files)

### **macOS**

#### **Instalando PHP usando Homebrew**

1. **Instala Homebrew** (si aún no está instalado):
   - Abre Terminal y ejecuta:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Instala PHP**:
   - Instala la última versión:
     ```bash
     brew install php
     ```
   - Para instalar una versión específica, por ejemplo, PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Cambia entre versiones de PHP**:
   - Desvincula la versión actual y vincula la versión deseada:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Verifica la versión instalada:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **Instalando PHP manualmente**

1. **Descarga PHP**:
   - Visita [PHP para Windows](https://windows.php.net/download/) y descarga la última versión o una versión específica (por ejemplo, 7.4, 8.0) como un archivo zip no seguro para subprocesos.

2. **Extrae PHP**:
   - Extrae el archivo zip descargado a `C:\php`.

3. **Agrega PHP al PATH del sistema**:
   - Ve a **Propiedades del sistema** > **Variables de entorno**.
   - Bajo **Variables del sistema**, busca **Path** y haz clic en **Editar**.
   - Agrega la ruta `C:\php` (o donde hayas extraído PHP).
   - Haz clic en **Aceptar** para cerrar todas las ventanas.

4. **Configura PHP**:
   - Copia `php.ini-development` a `php.ini`.
   - Edita `php.ini` para configurar PHP según sea necesario (por ejemplo, estableciendo `extension_dir`, habilitando extensiones).

5. **Verifica la instalación de PHP**:
   - Abre el Símbolo del sistema y ejecuta:
     ```cmd
     php -v
     ```

#### **Instalando Múltiples Versiones de PHP**

1. **Repite los pasos anteriores** para cada versión, colocando cada una en un directorio separado (por ejemplo, `C:\php7`, `C:\php8`).

2. **Cambia entre versiones** ajustando la variable PATH del sistema para que apunte al directorio de la versión deseada.

### **Ubuntu (20.04, 22.04, etc.)**

#### **Instalando PHP usando apt**

1. **Actualiza las listas de paquetes**:
   - Abre Terminal y ejecuta:
     ```bash
     sudo apt update
     ```

2. **Instala PHP**:
   - Instala la última versión de PHP:
     ```bash
     sudo apt install php
     ```
   - Para instalar una versión específica, por ejemplo, PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Instala módulos adicionales** (opcional):
   - Por ejemplo, para instalar soporte para MySQL:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Cambia entre versiones de PHP**:
   - Usa `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Verifica la versión instalada**:
   - Ejecuta:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **Instalando PHP usando yum/dnf**

1. **Habilita el repositorio EPEL**:
   - Abre Terminal y ejecuta:
     ```bash
     sudo dnf install epel-release
     ```

2. **Instala el repositorio de Remi**:
   - Ejecuta:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Instala PHP**:
   - Para instalar la versión predeterminada:
     ```bash
     sudo dnf install php
     ```
   - Para instalar una versión específica, por ejemplo, PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Cambia entre versiones de PHP**:
   - Usa el comando de módulo `dnf`:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Verifica la versión instalada**:
   - Ejecuta:
     ```bash
     php -v
     ```

### **Notas Generales**

- Para entornos de desarrollo, es importante configurar los ajustes de PHP según los requisitos de tu proyecto.
- Al cambiar las versiones de PHP, asegúrate de que todas las extensiones de PHP relevantes estén instaladas para la versión específica que planeas usar.
- Reinicia tu servidor web (Apache, Nginx, etc.) después de cambiar las versiones de PHP o actualizar configuraciones para aplicar los cambios.