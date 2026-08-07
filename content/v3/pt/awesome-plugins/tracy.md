# Tracy

Tracy é um manipulador de erros incrível que pode ser usado com Flight. Ele possui vários painéis que podem ajudar você a depurar sua aplicação. Também é muito fácil estender e adicionar seus próprios painéis. A equipe do Flight criou alguns painéis específicos para projetos Flight com o plugin [flightphp/tracy-extensions](https://github.com/flightphp/tracy-extensions) (variáveis do Flight, consultas de banco de dados, requisição, sessão e um painel opcional de **Twig** quando você passa um perfil de profiler—veja [Extensões do Tracy](/awesome-plugins/tracy-extensions)).

## Instalação

Instale com o composer. E você realmente vai querer instalar isso sem a versão de desenvolvimento, pois o Tracy vem com um componente de tratamento de erros para produção.

```bash
composer require tracy/tracy
```

## Configuração Básica

Existem algumas opções básicas de configuração para começar. Você pode ler mais sobre elas na [Documentação do Tracy](https://tracy.nette.org/en/configuring).

```php

require 'vendor/autoload.php';

use Tracy\Debugger;

// Habilita o Tracy
Debugger::enable();
// Debugger::enable(Debugger::DEVELOPMENT) // às vezes você precisa ser explícito (também Debugger::PRODUCTION)
// Debugger::enable('23.75.345.200'); // você também pode fornecer um array de endereços IP

// Aqui é onde os erros e exceções serão registrados. Certifique-se de que este diretório existe e é gravável.
Debugger::$logDirectory = __DIR__ . '/../log/';
Debugger::$strictMode = true; // exibe todos os erros
// Debugger::$strictMode = E_ALL & ~E_DEPRECATED & ~E_USER_DEPRECATED; // todos os erros exceto avisos de descontinuação
if (Debugger::$showBar) {
    $app->set('flight.content_length', false); // se a barra do Debugger estiver visível, então o content-length não pode ser definido pelo Flight

	// Isso é específico para a Extensão Tracy para Flight se você a incluiu
	// caso contrário, comente isso.
	new TracyExtensionLoader($app);
}
```

## Dicas Úteis

Quando você está depurando seu código, existem algumas funções muito úteis para exibir dados para você.

- `bdump($var)` - Isso irá despejar a variável na Barra do Tracy em um painel separado.
- `dumpe($var)` - Isso irá despejar a variável e então morrer imediatamente.