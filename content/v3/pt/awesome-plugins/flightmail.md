# FlightMail

> **Plugin de terceiros** - mantido por [Ryan Stubbs](https://ryanstubbs.co.uk) ([ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail), licença MIT). Não faz parte do núcleo do Flight - por favor reporte problemas no [repositório GitHub](https://github.com/ryanstubbs/flightmail/issues).

[ryanstubbs/flightmail](https://github.com/ryanstubbs/flightmail) deixa você enviar e-mail da sua app Flight sem as dores de cabeça. Ele envolve o **Symfony Mailer** - a biblioteca de e-mail mais consolidada do PHP - e faz parecer parte do Flight. Uma linha para instalar, um encadeamento fluente para enviar:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Você conseguiu!')
    ->text('Seu primeiro e-mail está a caminho.')
    ->send();
```

## Recursos

- **Qualquer provedor, uma linha para cada.** SMTP, Postmark, Sendgrid, Mailgun, Amazon SES, Brevo e companhia funcionam todos através de strings DSN simples.
- **Use vários provedores ao mesmo tempo.** E-mail transacional pelo Postmark, newsletters pelo seu próprio SMTP - escolha por mensagem.
- **Templates se você quiser.** Renderize os corpos com Twig ou Latte. Não quer templates? Só passe strings e não instale nada extra.
- **Polimento na hora do envio.** Inlining de CSS opcional e partes de texto puro automáticas derivadas do seu HTML, impulsionadas por bibliotecas que você só instala se for usar.
- **Entediante da melhor forma.** Conexões preguiçosas, erros claros em vez de e-mail engolido em silêncio, e tudo é substituível se você precisar de algo personalizado.

## Requisitos

| O quê          | Versão                                       |
| -------------- | -------------------------------------------- |
| PHP            | 8.2 ou mais recente                          |
| Flight PHP     | core ^3.15                                   |
| Symfony Mailer | ^7.2 ou ^8.0 (instalado automaticamente)     |

## Instalação

```bash
composer require ryanstubbs/flightmail
```

É só isso para enviar e-mails de texto puro e HTML. A renderização de templates é opcional - adicione um motor só se for usar:

```bash
composer require twig/twig      # para templates .twig
composer require latte/latte    # para templates .latte
```

Mais duas bibliotecas opcionais impulsionam as melhorias na hora do envio cobertas [abaixo](#estilizando-html-e-gerando-partes-de-texto):

```bash
composer require pelago/emogrifier         # para CSS inline ("inline_css")
composer require league/html-to-markdown   # para partes de texto Markdown ("text_from_html")
```

Todas elas podem ser instaladas lado a lado; o FlightMail escolhe a certa com base no que você configurar.

## Seu primeiro e-mail

Adicione isto ao seu bootstrap (o mesmo lugar onde você define as rotas):

```php
<?php
require 'vendor/autoload.php';

use ryanstubbs\FlightMail\MailPlugin;

// Diga ao FlightMail de onde e por onde enviar o e-mail.
MailPlugin::install([
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],
    'from' => 'no-reply@example.com',
]);

Flight::route('/signup', function () {
    Flight::mail()->compose()
        ->to('new-user@example.com')
        ->subject('Bem-vindo a bordo!')
        ->html('<h1>Bem-vindo!</h1><p>Estamos felizes que você esteja aqui.</p>')
        ->send();
});

Flight::start();
```

Usando o [esqueleto Flight PHP](https://github.com/flightphp/skeleton)? Registre em `app/config/services.php` com o estilo de instância em vez disso:

```php
use ryanstubbs\FlightMail\MailPlugin;

MailPlugin::register($app, [
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'from' => 'no-reply@example.com',
]);
```

Os dois estilos expõem o mesmo mailer: `Flight::mail()` e `$app->mail()` são intercambiáveis.

> **Testando localmente?** Se o seu projeto roda no [DDEV](https://ddev.com), aponte o DSN para `smtp://127.0.0.1:1025` e leia cada e-mail capturado no Mailpit em `http://<project>.ddev.site:8025`. Nada sai da sua máquina.

## Enviando e-mail

### Strings simples (nenhum motor de template necessário)

`->text()` e `->html()` recebem strings brutas e não precisam de mais nada instalado:

```php
Flight::mail()->compose()
    ->to('ops@example.com')
    ->subject('Backup concluído')
    ->text('O backup noturno foi concluído em 42 minutos.')
    ->send();

Flight::mail()->compose()
    ->to('billing@example.com')
    ->subject('Fatura #123')
    ->html('<h1>Fatura #123</h1><p>Total devido: $42.00</p>')
    ->send();
```

### Templates Twig

```php
// welcome.html.twig contém: Olá {{ name }}, obrigado por se cadastrar!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bem-vindo!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])
    ->send();
```

### Templates Latte

Mesma ideia, extensão `.latte`:

```php
// welcome.latte contém: Olá {$name}, obrigado por se cadastrar!
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bem-vindo!')
    ->template('welcome.latte', ['name' => 'Ryan'])
    ->send();
```

### HTML + texto puro juntos

Boa prática para entregabilidade - dê aos clientes de e-mail as duas versões:

```php
Flight::mail()->compose()
    ->to('someone@example.com')
    ->subject('Bem-vindo!')
    ->template('welcome.html.twig', ['name' => 'Ryan'])     // versão rica
    ->textTemplate('welcome.txt.twig', ['name' => 'Ryan'])  // versão de fallback
    ->send();
```

Algumas coisas que vale a pena saber sobre templates:

- Eles são renderizados de forma **preguiçosa**, no momento do envio - compose agora, renderize depois.
- O motor é escolhido pela extensão: `.twig` → Twig, `.latte` → Latte, qualquer outra coisa → o padrão configurado (opção `renderer`).
- Um corpo explícito com `->html()` ou `->text()` sempre ganha de um template, então você pode definir um template padrão e sobrescrevê-lo por mensagem.

## Estilizando HTML e gerando partes de texto

Duas melhorias opcionais na hora do envio, ambas desligadas por padrão e ambas impulsionadas por bibliotecas que você só instala se quiser:

| Recurso                      | Instalar                   | Chave de configuração |
| ---------------------------- | -------------------------- | --------------------- |
| CSS inline                   | `pelago/emogrifier`        | `inline_css`          |
| Parte de texto a partir do HTML | `league/html-to-markdown` | `text_from_html`    |

### Inserir CSS inline no seu e-mail HTML

O Gmail e a maioria dos clientes de webmail removem blocos `<style>` - atributos `style=""` inline são o único estilo que eles honram de forma confiável. Escrevê-los na mão é miserável; deixe o [Emogrifier](https://github.com/MyIntervals/emogrifier) fazer isso na hora do envio:

```bash
composer require pelago/emogrifier
```

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'inline_css' => true,
]);
```

Com isso ligado, cada corpo HTML recebe o CSS inline logo antes do envio - tenha vindo de um template ou de `->html()`. Uma mensagem como `<style>p { color: red; }</style><p>Olá</p>` sai como `<p style="color: red;">Olá</p>`.

Para injetar estilos compartilhados em todo e-mail (cores da marca, resets) sem repeti-los em cada template, passe as regras diretamente ou aponte para um arquivo de stylesheet:

```php
'inline_css' => ['css_file' => __DIR__ . '/mail-styles/base.css'],
// ou
'inline_css' => ['css' => '.button { background: #0a84ff; color: #fff; }'],
```

Controle por mensagem:

```php
$message->inlineCss();          // forçar inlining para esta mensagem
$message->withoutInlineCss();   // pular mesmo quando habilitado globalmente
```

### Gerar a parte de texto a partir do seu HTML

A boa prática é enviar uma versão HTML e uma de texto puro juntas, mas escrever as duas é tedioso. O FlightMail pode derivar a parte de texto do HTML final automaticamente - a conversão básica não precisa de nenhuma dependência extra, já que o conversor vem com o Symfony Mime:

```php
MailPlugin::install([
    'dsns' => ['default' => 'smtp://user:pass@localhost:1025'],
    'text_from_html' => true,       // Markdown quando possível, puro caso contrário
]);
```

Modos:

- `true` ou `'auto'` - saída Markdown se `league/html-to-markdown` estiver instalado, senão uma simples remoção de tags.
- `'markdown'` - forçar Markdown (`composer require league/html-to-markdown`; títulos viram `==`, links `[text](url)`, negrito `**bold**`).
- `'plain'` - sempre remover tags; funciona com zero pacotes extras.

A geração roda depois da renderização e do CSS inline, e só quando a mensagem tem um corpo HTML mas nenhum corpo de texto - um `->text()` ou `->textTemplate()` explícito sempre ganha. As sobrescritas por mensagem espelham o inlining:

```php
$message->textFromHtml('plain');    // forçar remoção de tags nesta
$message->withoutTextFromHtml();    // e-mail só HTML
```

Ative um modo cuja biblioteca não esteja instalada e você recebe um erro claro nomeando o `composer require` exato a rodar - nunca degradação silenciosa.

## Escolhendo um provedor

Provedores se conectam através de strings DSN. Instale o pacote ponte, cole o DSN em `dsns`, pronto.

| Provedor             | Instalar                                     | Exemplo de DSN                               |
| -------------------- | -------------------------------------------- | -------------------------------------------- |
| SMTP                 | embutido                                     | `smtp://user:pass@host:587`                  |
| Sendmail             | embutido                                     | `sendmail://default`                         |
| Dev/null (descartar e-mail) | embutido                              | `null://null`                                |
| Postmark             | `composer require symfony/postmark-mailer`   | `postmark+api://KEY@api.postmarkapp.com`     |
| Sendgrid             | `composer require symfony/sendgrid-mailer`   | `sendgrid+api://KEY@default`                 |
| Mailgun              | `composer require symfony/mailgun-mailer`    | `mailgun+https://KEY:DOMAIN@api.mailgun.net` |
| Amazon SES           | `composer require symfony/amazon-mailer`     | `ses+https://KEY:SECRET@default`             |
| Brevo                | `composer require symfony/brevo-mailer`      | `brevo+api://KEY@default`                    |
| MailerSend           | `composer require symfony/mailersend-mailer` | `mailersend+api://KEY@default`               |

A lista completa vive na [documentação do Symfony Mailer](https://symfony.com/doc/current/mailer.html) - qualquer coisa documentada lá funciona aqui sem mudanças.

### Vários provedores ao mesmo tempo

Nomeie cada transporte, depois escolha por mensagem:

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
// Sem chamada a ->transport() = primeira chave em "dsns" ("transactional" aqui).
Flight::mail()->compose()->to('...')->text('recibo')->send();

// Escolha outra rota explicitamente.
Flight::mail()->compose()->to('...')->text('newsletter')->transport('bulk')->send();
```

## Referência de configuração

Tudo é opcional, exceto `dsns`.

```php
MailPlugin::install([
    // OBRIGATÓRIO - nome do transporte => Symfony DSN.
    // A primeira entrada é usada quando uma mensagem não nomeia uma.
    'dsns' => [
        'default' => 'smtp://user:pass@localhost:1025',
    ],

    // Transporte usado quando uma mensagem não tem um ->transport() explícito e
    // você não quer a primeira chave. Deve existir em "dsns".
    'default_transport' => 'default',

    // Remetente global. String, Symfony Address, ou ['email' => 'Name'].
    // Aplicado somente quando uma mensagem não define o próprio ->from().
    'from' => ['no-reply@example.com' => 'Meu App'],

    // Motor de template padrão: 'twig', 'latte', ou um nome personalizado.
    // Consultado somente para templates cuja extensão não é um renderer registrado.
    'renderer' => 'twig',

    // Onde os templates vivem, buscados em ordem; mais um dir de cache opcional.
    'templates' => [
        'paths' => [__DIR__ . '/mail-templates'],
        'cache' => __DIR__ . '/cache/mail',
    ],

    // Opções extras passadas direto para Twig\Environment.
    'twig' => ['options' => ['strict_variables' => true]],

    // Ajuste o motor Latte no boot: fn(Latte\Engine $engine): void.
    'latte' => ['setup' => static fn (Latte\Engine $e) => $e->addExtension(new MyExtension())],

    // Melhorias do corpo na hora do envio (veja "Estilizando HTML e gerando partes de texto").
    'inline_css' => true,           // ou ['css' => '...', 'css_file' => '...']
    'text_from_html' => true,       // ou 'plain' / 'markdown'

    // Esquemas DSN personalizados, renderers personalizados, hooks pré-envio (veja abaixo).
    'transport_factories' => [],
    'renderers' => [],
    'hooks' => [],

    // Infraestrutura opcional entregue a cada transporte.
    'event_dispatcher' => $dispatcher,  // Symfony MessageEvents
    'logger' => $psr3Logger,
]);
```

## Indo além

Tudo abaixo é opcional. Os padrões cobrem a maioria das apps.

### Adicionar um esquema DSN personalizado

Implemente o `TransportFactoryInterface` do Symfony e registre-o - então o seu próprio esquema funciona exatamente como um embutido:

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
        // ... construa um transporte que fale com a sua operadora
    }
}

$plugin = MailPlugin::install(['dsns' => ['carrier' => 'mycarrier://key']]);
$plugin->addTransportFactory(new MyCarrierFactory());
```

### Adicionar um renderizador de templates personalizado

Qualquer coisa que transforme um nome de template mais params em uma string serve:

```php
use ryanstubbs\FlightMail\MailPlugin;
use ryanstubbs\FlightMail\Render\RendererInterface;

$plugin = MailPlugin::install($config);

$plugin->addRenderer('markdown', fn (array $config): RendererInterface =>
    new MarkdownMailRenderer($config['templates']['paths'] ?? [])
);
```

```php
// Templates terminando em .markdown agora o usam automaticamente:
Flight::mail()->compose()->to('...')->template('welcome.markdown', ['name' => 'Ryan'])->send();
```

### Executar algo logo antes de enviar

Hooks recebem a mensagem pronta - depois da renderização, depois dos padrões, logo antes do fio:

```php
$plugin->addHook(function (ryanstubbs\FlightMail\Message $message): void {
    $message->getHeaders()->addTextHeader('X-Mailer', 'MyApp/1.0');
});
```

### Eventos e logging

Passe um event dispatcher do Symfony e/ou um logger PSR-3 e cada transporte vai usá-los:

```php
$plugin->eventDispatcher($dispatcher); // recebe MessageEvent antes de cada envio
$plugin->logger($logger);              // logs no nível do transporte
```

## Folha de consulta da API

```php
// Configuração
MailPlugin::install($config)             // registrar na app Flight global
MailPlugin::register($app, $config)      // registrar em um Engine específico
$mailer = Flight::mail();                // a instância Mailer compartilhada

// Construindo mensagens
$mailer->compose(): Message
$message->to(...)->from(...)->subject(...)   // métodos padrão do Symfony Mime
$message->text(string)                       // corpo em string de texto puro
$message->html(string)                       // corpo em string HTML
$message->template($name, $params)           // corpo HTML a partir de um template
$message->htmlTemplate($name, $params)       // alias de template()
$message->textTemplate($name, $params)       // corpo de texto a partir de um template
$message->inlineCss() / ->withoutInlineCss() // CSS inline por mensagem
$message->textFromHtml($mode)                // parte de texto auto: true/'auto'/'plain'/'markdown'/false
$message->withoutTextFromHtml()              // e-mail só HTML
$message->transport($name)                   // rotear via um DSN nomeado
$message->send(): ?SentMessage               // renderizar + enviar

// No próprio mailer
$mailer->send($message): ?SentMessage        // alternativa explícita a $message->send()
$mailer->render($template, $params): string  // renderizar sem enviar
$mailer->addHook(callable): static           // fn(Message $message): void
$mailer->transports(): TransportManager      // get() / has() / names()
$mailer->renderers(): RendererFactory        // create() / has() / add()
```

Como `Message` estende `Symfony\Component\Mime\Email`, todo método Symfony que você já conhece - `attach()`, `embed()`, `priority()`, `replyTo()` - funciona de imediato.

## Solução de Problemas

**"No mail DSNs configured"**
Você chamou `Flight::mail()` antes de registrar o plugin, ou o array de config não incluía `dsns`. Este erro é deliberado - o FlightMail se recusa a adivinhar para onde seu e-mail deve ir em vez de descartá-lo em silêncio.

**"Unknown mail template renderer ..."**
Você usou um template cujo motor não está instalado. Corrija com `composer require twig/twig` ou `composer require latte/latte`, ou registre um renderer personalizado nomeado conforme a extensão.

**"Unknown mail transport ..."**
Um `->transport('name')` (ou `default_transport`) não corresponde a nenhuma chave em `dsns`. Confira a grafia - o erro lista os nomes configurados.

**O e-mail não está chegando**
Aponte `dsns` para `null://null` para confirmar que o resto do seu código funciona, depois volte para o DSN real. No DDEV, use `smtp://127.0.0.1:1025` e inspecione as mensagens no Mailpit na porta 8025.

---

Para relatos de bugs, pull requests e o código-fonte completo, visite o [repositório GitHub](https://github.com/ryanstubbs/flightmail).
