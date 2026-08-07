# Extensões do Painel Tracy para Flight

Este é um conjunto de extensões para tornar o trabalho com o Flight um pouco mais rico.

- **Flight** - Analise todas as variáveis do Flight.
- **Database** - Analise todas as consultas que foram executadas na página (se você iniciar corretamente a conexão com o banco de dados)
- **Request** - Analise todas as variáveis `$_SERVER` e examine todos os payloads globais (`$_GET`, `$_POST`, `$_FILES`)
- **Session** - Analise todas as variáveis `$_SESSION` se as sessões estiverem ativas.
- **Twig** *(opcional)* - Analise o tempo de renderização do template Twig, memória e quais templates/blocks/macros foram executados (requer `twig/twig` e uma configuração `twig_profile`)

Isso é especialmente útil com o [esqueleto oficial](https://github.com/flightphp/skeleton), que usa Twig por padrão: o mesmo layout [ferramentas de IA](/learn/ai) também aparece claramente na barra do Tracy.

Este é o Painel

![Barra do Flight](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-tracy-bar.png)

E cada painel exibe informações muito úteis sobre sua aplicação!

![Dados do Flight](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-var-data.png)
![Banco de Dados do Flight](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-db.png)
![Requisição do Flight](https://raw.githubusercontent.com/flightphp/tracy-extensions/master/flight-request.png)

Clique [aqui](https://github.com/flightphp/tracy-extensions) para ver o código.

## Instalação

Execute `composer require flightphp/tracy-extensions --dev` e pronto!

O Twig **não** é uma dependência obrigatória do pacote. Instale `twig/twig` apenas se quiser o painel Twig (o esqueleto já faz isso para as views).

## Configuração

Há muito pouca configuração necessária para começar. Você precisará iniciar o debugger Tracy antes de usar isso [https://tracy.nette.org/en/guide](https://tracy.nette.org/en/guide):

```php
<?php

use Tracy\Debugger;
use flight\debug\tracy\TracyExtensionLoader;

// código bootstrap
require __DIR__ . '/vendor/autoload.php';

Debugger::enable();
// Você pode precisar especificar seu ambiente com Debugger::enable(Debugger::DEVELOPMENT)

// se você usa conexões de banco de dados em sua aplicação, há um 
// wrapper PDO obrigatório para usar APENAS EM DESENVOLVIMENTO (não em produção, por favor!)
// Ele tem os mesmos parâmetros de uma conexão PDO regular
$pdo = new PdoQueryCapture('sqlite:test.db', 'user', 'pass');
// ou se você anexar isso ao framework Flight
Flight::register('db', PdoQueryCapture::class, ['sqlite:test.db', 'user', 'pass']);
// agora sempre que você fizer uma consulta, ele capturará o tempo, a consulta e os parâmetros

// Isso conecta os pontos
if(Debugger::$showBar === true) {
	// Isso precisa ser false ou o Tracy não consegue renderizar de verdade :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app());
}

// mais código

Flight::start();
```

## Configuração Adicional

### Dados de Sessão

Se você tiver um manipulador de sessão personalizado (como ghostff/session), pode passar qualquer array de dados de sessão para o Tracy e ele automaticamente os exibirá para você. Você passa isso com a chave `session_data` no segundo parâmetro do construtor `TracyExtensionLoader`.

```php

use Ghostff\Session\Session;
// ou use flight\Session;

require 'vendor/autoload.php';

$app = Flight::app();

$app->register('session', Session::class);

if(Debugger::$showBar === true) {
	// Isso precisa ser false ou o Tracy não consegue renderizar de verdade :(
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), [ 'session_data' => Flight::session()->getAll() ]);
}

// rotas e outras coisas...

Flight::start();
```

### Painel Twig (opcional)

Se sua aplicação usa [Twig](/awesome-plugins/twig) (incluindo o esqueleto oficial), você pode mostrar métricas de template na barra do Tracy. Crie um `Profile` do Twig, anexe o `ProfilerExtension` ao seu ambiente, então passe esse perfil para o loader sob a chave **`twig_profile`**. Anexe a profilagem apenas em desenvolvimento.

```php
<?php

use flight\debug\tracy\TracyExtensionLoader;
use flight\debug\tracy\TwigTracyExtension;
use Tracy\Debugger;
use Twig\Environment;
use Twig\Extension\ProfilerExtension;
use Twig\Loader\FilesystemLoader;
use Twig\Profiler\Profile;

$loader = new FilesystemLoader(__DIR__ . '/views');
$twig = new Environment($loader, [
	'debug' => true,
	'cache' => false,
]);

// Opcional: exponha os helpers dump do Tracy nos templates
// {{ dump(var) }}, {{ bdump(var) }}, {{ dumpe(var) }}
$twig->addExtension(new TwigTracyExtension());

$tracyConfig = [];
if (Debugger::$showBar === true) {
	$profile = new Profile();
	$twig->addExtension(new ProfilerExtension($profile));
	$tracyConfig['twig_profile'] = $profile;
}

if (Debugger::$showBar === true) {
	Flight::set('flight.content_length', false);
	new TracyExtensionLoader(Flight::app(), $tracyConfig);
}

// Mapeie Flight::render() para Twig (exemplo)
Flight::map('render', function (string $template, array $data = []) use ($twig) {
	if (substr($template, -5) !== '.twig') {
		$template .= '.twig';
	}
	echo $twig->render($template, $data);
});
```

**O que o painel mostra**

- Tempo total de renderização e memória do Twig
- Contagens de chamadas de template / block / macro
- Cada template que foi renderizado, com seu próprio tempo e memória

A aba Twig fica **oculta** quando nenhum template foi renderizado para a requisição, ou quando você omite `twig_profile` (ou não tem o Twig instalado) - outros painéis do Flight continuam funcionando.

Em um `services.php` no estilo do esqueleto, construa o mesmo `$profile` / `ProfilerExtension` quando o debug estiver ativo, passe `twig_profile` para o `TracyExtensionLoader`, e continue usando seu ambiente Twig compartilhado para `$app->render()`.

### Latte

_PHP 8.1+ é necessário para esta seção._

Se você tem o Latte instalado em seu projeto, o Tracy tem uma integração nativa com o Latte para analisar seus templates. Você simplesmente registra a extensão com sua instância do Latte (esta é a própria ponte Tracy do Latte, não o painel Twig acima).

```php

require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function($template, $data, $block = null) {
	$latte = new Latte\Engine;

	// outras configurações...

	// adicione a extensão apenas se a Barra de Debug do Tracy estiver habilitada
	if(Debugger::$showBar === true) {
		// aqui é onde você adiciona o Painel Latte ao Tracy
		$latte->addExtension(new Latte\Bridges\Tracy\TracyExtension);
	}

	$latte->render($template, $data, $block);
});
```

## Veja Também

- [Tracy](/awesome-plugins/tracy) - Configuração base do Tracy para o Flight
- [Twig](/awesome-plugins/twig) - Templating usado pelo esqueleto e pelo painel Twig
- [Templates](/learn/templates) - Como o Flight mapeia `render` para Twig/Latte
- [Installation](/install) - O esqueleto inclui tracy-extensions em dev