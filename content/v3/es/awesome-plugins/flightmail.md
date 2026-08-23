# FlightMail

> **Plugin de terceros** - mantenido por [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), licencia MIT). No forma parte del núcleo de Flight - por favor reporta los problemas en [su repositorio de GitHub](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) te permite enviar correo desde tu app de Flight sin los dolores de cabeza. Envuelve **Symfony Mailer** - la librería de correo más consolidada de PHP - y lo hace sentir como parte de Flight. Una línea para instalar, una cadena fluida para enviar:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('¡Lo lograste!')
    ->text('Tu primer correo está en camino.')
    ->send();
```

## Características

- **Cualquier proveedor, una línea cada uno.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo y compañía funcionan todos a través de cadenas DSN simples.
- **Usa varios proveedores a la vez.** Correo transaccional por Postmark, boletines por tu propio SMTP - elige por mensaje.
- **Plantillas si las quieres.** Renderiza los cuerpos con Twig o Latte. ¿No quieres plantillas? Solo pasa cadenas e instala nada extra.
- **Pulido al enviar.** Inlining de CSS opcional y partes de texto plano automáticas derivadas de tu HTML, impulsadas por librerías que solo instalas si las usas.
- **Aburrido, de la mejor manera.** Conexiones perezosas, errores claros en lugar de correo tragado en silencio, y todo es intercambiable si necesitas algo personalizado.

## Requisitos

| Qué            | Versión                                      |
| -------------- | -------------------------------------------- |
| PHP            | 8.2 o más reciente                           |
| Flight PHP     | core ^3.15                                   |
| Symfony Mailer | ^7.2 o ^8.0 (instalado automáticamente)      |

## Instalación

```bash
composer require ryanstubbs/flightmail
```

Eso es todo para enviar correos de texto plano y HTML. El renderizado de plantillas es opcional - agrega un motor solo si lo vas a usar:

```bash
composer require twig/twig      # para plantillas .twig
composer require latte/latte    # para plantillas .latte
```

Dos librerías opcionales más impulsan las mejoras al enviar cubiertas [más abajo](#estilos-html-y-generacion-de-partes-de-texto):

```bash
composer require pelago/emogrifier         # para CSS en línea ("inline_css")
composer require league/html-to-markdown   # para partes de texto Markdown ("text_from_html")
```

Todas estas se pueden instalar juntas; FlightMail elige la correcta según lo que configures.

## Tu primer correo

Agrega esto a tu bootstrap (el mismo lugar donde defines las rutas):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Dile a FlightMail desde dónde y a través de qué enviar el correo.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('¡Bienvenido a bordo!')
        ->html('<h1>¡Bienvenido!</h1><p>Nos alegra que estés aquí.</p>')
        ->send();
});

Flight::start();
```

¿Usas el [esqueleto de Flight PHP](https://github.com/flightphp/skeleton)? Regístralo en `app/config/services.php` con el estilo de instancia en su lugar:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Ambos estilos exponen el mismo mailer: `Flight::mail()` y `$app->mail()` son intercambiables.

> **¿Probando en local?** Si tu proyecto corre en [DDEV](https://ddev.com), apunta el DSN a `smtp://127.0.0.1:1025` y lee cada correo capturado en Mailpit en `http://<project>.ddev.site:8025`. Nada sale de tu máquina.

## Enviar correo

### Cadenas simples (no se necesita motor de plantillas)

`->text()` y `->html()` toman cadenas crudas y no necesitan nada más instalado:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Respaldo terminado')
    ->text('El respaldo nocturno se completó en 42 minutos.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Factura #123')
    ->html('<h1>Factura #123</h1><p>Total a pagar: $42.00</p>')
    ->send();
```

### Plantillas Twig

```php
// welcome.html.twig contiene: ¡Hola {{ name }}, gracias por registrarte!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('¡Bienvenido!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Plantillas Latte

Misma idea, extensión `.latte`:

```php
// welcome.latte contiene: ¡Hola {$name}, gracias por registrarte!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('¡Bienvenido!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + texto plano juntos

Buena práctica para la entregabilidad - dale a los clientes de correo ambas versiones:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('¡Bienvenido!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // versión enriquecida
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // versión de respaldo
    ->send();
```

Algunas cosas que vale la pena saber sobre las plantillas:

- Se renderizan de forma **perezosa**, en el momento del envío - compones ahora, renderizas después.
- El motor se elige por la extensión: `.twig` → Twig, `.latte` → Latte, cualquier otra cosa → tu valor por defecto configurado (opción `renderer`).
- Un cuerpo explícito con `->html()` o `->text()` siempre gana sobre una plantilla, así que puedes definir una plantilla por defecto y sobreescribirla por mensaje.

## Estilos HTML y generación de partes de texto

Dos mejoras opcionales al enviar, ambas desactivadas por defecto y ambas impulsadas por librerías que solo instalas si las quieres:

| Función                   | Instalar                   | Clave de configuración |
| ------------------------- | -------------------------- | ---------------------- |
| CSS en línea              | `pelago/emogrifier`        | `inline_css`           |
| Parte de texto desde HTML | `league/html-to-markdown`  | `text_from_html`       |

### Insertar CSS en línea en tu correo HTML

Gmail y la mayoría de los clientes de webmail eliminan los bloques `<style>` - los atributos `style=""` en línea son el único estilo que honran de forma fiable. Escribirlos a mano es miserable; deja que [Emogrifier](https://github.com/MyIntervals/emogrifier) lo haga al enviar:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Con eso activado, cada cuerpo HTML recibe su CSS en línea justo antes de enviar - ya sea que viniera de una plantilla o de `->html()`. Un mensaje como `<style>p { color: red; }</style><p>Hola</p>` sale como `<p style="color: red;">Hola</p>`.

Para inyectar estilos compartidos en cada correo (colores de marca, resets) sin repetirlos en cada plantilla, pasa las reglas directamente o apunta a un archivo de hoja de estilos:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// o
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Control por mensaje:

```php
$message->inlineCss();          // forzar inlining para este mensaje
$message->withoutInlineCss();   // saltarlo incluso cuando está habilitado globalmente
```

### Generar la parte de texto a partir de tu HTML

La buena práctica es enviar una versión HTML y una de texto plano juntas, pero escribir ambas es tedioso. FlightMail puede derivar la parte de texto del HTML final automáticamente - la conversión básica no necesita ninguna dependencia extra, ya que el convertidor viene con Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown cuando sea posible, plano si no
]);
```

Modos:

- `true` o `'auto'` - salida Markdown si `league/html-to-markdown` está instalado, de lo contrario un simple recorte de etiquetas.
- `'markdown'` - forzar Markdown (`composer require league/html-to-markdown`; los encabezados se vuelven `==`, los enlaces `[text](url)`, negrita `**bold**`).
- `'plain'` - siempre recortar etiquetas; funciona con cero paquetes extra.

La generación corre después del renderizado y del CSS en línea, y solo cuando el mensaje tiene un cuerpo HTML pero no un cuerpo de texto - un `->text()` o `->textTemplate()` explícito siempre gana. Las anulaciones por mensaje reflejan el inlining:

```php
$message->textFromHtml('plain');    // forzar recorte de etiquetas para este
$message->withoutTextFromHtml();    // correo solo HTML
```

Habilita un modo cuya librería no esté instalada y obtienes un error claro que nombra el `composer require` exacto a ejecutar - nunca una degradación silenciosa.

## Elegir un proveedor

Los proveedores se conectan a través de cadenas DSN. Instala el paquete puente, pega el DSN en `dsns`, listo.

| Proveedor            | Instalar                                     | Ejemplo de DSN                               |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | integrado                                    | `smtp://user:pass@host:587`                  |
| Sendmail             | integrado                                    | `sendmail://default`                         |
| Dev/null (descartar correo) | integrado                             | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

La lista completa vive en la [documentación de Symfony Mailer](https://symfony.com/doc/current/mailer.html) - cualquier cosa documentada ahí funciona aquí sin cambios.

### Varios proveedores a la vez

Nombra cada transporte, luego elige por mensaje:

```php
MailPlugin::install([
    'dsns' => [
        'transactional' => 'postmark+api://KEY@api.postmarkapp.com',
        'bulk'          => 'smtp://user:pass@bulk.example.com:587',
    ],
    'from' => 'no-reply@example.com',
]);
```

```php
// Sin llamada a ->transport() = primera clave en "dsns" ("transactional" aquí).
Flight::mail()->compose()->to('...')->text('recibo')->send();

// Elige otra ruta de forma explícita.
Flight::mail()->compose()->to('...')->text('boletín')->transport('bulk')->send();
```

## Referencia de configuración

Todo es opcional excepto `dsns`.

```php
MailPlugin::install([
    // OBLIGATORIO - nombre de transporte => Symfony DSN.
    // La primera entrada se usa cuando un mensaje no nombra una.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transporte usado cuando un mensaje no tiene un ->transport() explícito y
    // no quieres la primera clave. Debe existir en "dsns".
    'default_transport' => 'default',

    // Remitente global. String, Symfony Address, o ['email' => 'Name'].
    // Se aplica solo cuando un mensaje no define su propio ->from().
    'from' => ['no-reply@example.com' => 'Mi App'],

    // Motor de plantillas por defecto: 'twig', 'latte', o un nombre personalizado.
    // Solo se consulta para plantillas cuya extensión no es un renderer registrado.
    'renderer' => 'twig',

    // Dónde viven las plantillas, buscadas en orden; más un dir de caché opcional.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Opciones extra pasadas directo a Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Ajusta el motor Latte al arrancar: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Mejoras del cuerpo al enviar (ver "Estilos HTML y generación de partes de texto").
    'inline_css' => true,           // o ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // o 'plain' / 'markdown'

    // Esquemas DSN personalizados, renderers personalizados, hooks pre-envío (ver más abajo).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Fontanería opcional entregada a cada transporte.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Ir más allá

Todo lo de abajo es opcional. Los valores por defecto cubren la mayoría de las apps.

### Agregar un esquema DSN personalizado

Implementa `TransportFactoryInterface` de Symfony y regístralo - entonces tu propio esquema funciona exactamente como uno integrado:

```php
use ryanstubbs\FlightMail\MailPlugin;
use Symfony\Component\Mailer\Transport\Dsn;
use Symfony\Component\Mailer\Transport\TransportFactoryInterface;
use Symfony\Component\Mailer\Transport\TransportInterface;

class MyCarrierFactory implements TransportFactoryInterface
{
    public function supports(Dsn $dsn): bool
    {
        return $dsn->getScheme() === 'mycarrier';
    }

    public function create(Dsn $dsn): TransportInterface
    {
        // ... construye un transporte que hable con tu operador
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Agregar un renderizador de plantillas personalizado

Cualquier cosa que convierta un nombre de plantilla más params en una cadena califica:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Las plantillas que terminan en .markdown ahora lo usan automáticamente:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Ejecutar algo justo antes de enviar

Los hooks reciben el mensaje terminado - después del renderizado, después de los valores por defecto, justo antes del cable:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Eventos y registro

Entrega un event dispatcher de Symfony y/o un logger PSR-3 y cada transporte los usará:

```php
$plugin->eventDispatcher($dispatcher); // recibe MessageEvent antes de cada envío
$plugin->logger($logger);              // logs a nivel de transporte
```

## Hoja de referencia de la API

```php
// Configuración
MailPlugin::install($config)             // registrar en la app global de Flight
MailPlugin::register($app, $config)      // registrar en un Engine específico
$mailer = Flight::mail();                // la instancia Mailer compartida

// Construir mensajes
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // métodos estándar de Symfony Mime
$message->text(string)                       // cuerpo de cadena de texto plano
$message->html(string)                       // cuerpo de cadena HTML
$message->template($name, $params)           // cuerpo HTML desde una plantilla
$message->htmlTemplate($name, $params)       // alias de template()
$message->textTemplate($name, $params)       // cuerpo de texto desde una plantilla
$message->inlineCss() / ->withoutInlineCss() // CSS en línea por mensaje
$message->textFromHtml($mode)                // parte de texto auto: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // correo solo HTML
$message->transport($name)                   // enrutar vía un DSN con nombre
$message->send(): ?SentMessage               // renderizar + enviar

// En el mailer mismo
$mailer->send($message): ?SentMessage        // alternativa explícita a $message->send()
$mailer->render($template, $params): string  // renderizar sin enviar
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Como `Message` extiende `Symfony\Component\Mime\Email`, cada método de Symfony que ya conoces - `attach()`, `embed()`, `priority()`, `replyTo()` - funciona de inmediato.

## Solución de problemas

**"No mail DSNs configured"**
Llamaste a `Flight::mail()` antes de registrar el plugin, o el array de config no incluía `dsns`. Este error es deliberado - FlightMail se niega a adivinar a dónde debe ir tu correo en lugar de soltarlo en silencio.

**"Unknown mail template renderer ..."**
Usaste una plantilla cuyo motor no está instalado. Arréglalo con `composer require twig/twig` o `composer require latte/latte`, o registra un renderer personalizado nombrado según la extensión.

**"Unknown mail transport ..."**
Un `->transport('name')` (o `default_transport`) no coincide con ninguna clave en `dsns`. Revisa la ortografía - el error lista los nombres configurados.

**El correo no está llegando**
Apunta `dsns` a `null://null` para confirmar que el resto de tu código funciona, luego vuelve al DSN real. En DDEV, usa `smtp://127.0.0.1:1025` e inspecciona los mensajes en Mailpit en el puerto 8025.

---

Para reportes de errores, pull requests y el código fuente completo, visita el [repositorio de GitHub](https://github.com/ryanstubbs/flightmail).
