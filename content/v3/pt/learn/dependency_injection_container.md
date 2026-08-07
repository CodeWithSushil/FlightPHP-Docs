# Container de Injeção de Dependência

## Visão Geral

O Container de Injeção de Dependência (DIC) é um recurso poderoso que permite gerenciar
as dependências da sua aplicação. É também uma das maiores razões pelas quais o Flight funciona bem com [ferramentas de codificação por IA](/learn/ai) e testes de unidade: os controladores recebem o que precisam no construtor em vez de acessar variáveis globais.

## Entendendo

A Injeção de Dependência (DI) é um conceito-chave em frameworks PHP modernos e é
usada para gerenciar a instanciação e a configuração de objetos. Alguns exemplos de
bibliotecas de DIC são: [flightphp/container](https://github.com/flightphp/container), [Dice](https://r.je/dice), [Pimple](https://pimple.symfony.com/),
[PHP-DI](http://php-di.org/) e [league/container](https://container.thephpleague.com/).

Um DIC é uma forma elegante de criar e gerenciar suas classes em um
local centralizado. Isso é útil quando você precisa passar o mesmo objeto para
várias classes (controladores, middleware, comandos e assim por diante).

O [flightphp/skeleton](https://github.com/flightphp/skeleton) oficial conecta o **Dice** em `app/config/services.php`, substitui a instância compartilhada de `flight\Engine` e resolve alvos de rota como `[App\Controller\HomeController::class, 'index']`. Prefira esse padrão para novos projetos, para que humanos e agentes editem os mesmos lugares.

## Uso Básico

A forma antiga de fazer as coisas pode ser assim:
```php

require 'vendor/autoload.php';

// classe para gerenciar usuários do banco de dados
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// no seu arquivo routes.php

$db = new PDO('mysql:host=localhost;dbname=test', 'user', 'pass');

$UserController = new UserController($db);
Flight::route('/user/@id', [ $UserController, 'view' ]);
// outras rotas do UserController...

Flight::start();
```

Você pode ver no código acima que estamos criando um novo objeto `PDO` e passando-o
para a classe `UserController`. Isso é bom para uma aplicação pequena, mas conforme sua
aplicação cresce, você descobrirá que está criando ou repassando o mesmo objeto `PDO`
em vários lugares. É aí que um DIC se torna útil.

Aqui está o mesmo exemplo usando um DIC (com Dice):
```php

require 'vendor/autoload.php';

// mesma classe acima. Nada mudou
class UserController {

	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function view(int $id) {
		$stmt = $this->pdo->prepare('SELECT * FROM users WHERE id = :id');
		$stmt->execute(['id' => $id]);

		print_r($stmt->fetch());
	}
}

// cria um novo container
$container = new \Dice\Dice;

// adiciona uma regra para dizer ao container como criar um objeto PDO
// não se esqueça de reatribuí-lo a si mesmo como abaixo!
$container = $container->addRule('PDO', [
	// shared significa que o mesmo objeto será retornado toda vez
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// Isso registra o manipulador do container para que o Flight saiba usá-lo.
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

// agora podemos usar o container para criar nosso UserController
Flight::route('/user/@id', [ UserController::class, 'view' ]);

Flight::start();
```

Aposto que você pode estar pensando que foi adicionado muito código extra ao exemplo.
A mágica acontece quando você tem outro controlador que precisa do objeto `PDO`.

```php

// Se todos os seus controladores tiverem um construtor que precise de um objeto PDO
// cada uma das rotas abaixo terá ele injetado automaticamente!!!
Flight::route('/company/@id', [ CompanyController::class, 'view' ]);
Flight::route('/organization/@id', [ OrganizationController::class, 'view' ]);
Flight::route('/category/@id', [ CategoryController::class, 'view' ]);
Flight::route('/settings', [ SettingsController::class, 'view' ]);
```

O benefício adicional de utilizar um DIC é que os testes de unidade se tornam muito mais fáceis. Você pode criar um objeto mock e passá-lo para sua classe. Isso é um enorme benefício quando você está escrevendo testes para sua aplicação — e quando um assistente de IA gera um controlador, a injeção no construtor fornece um padrão claro e consistente a seguir ([guia de testes de unidade](/guides/unit-testing)).

### Criando um manipulador de DIC centralizado

Você pode criar um manipulador de DIC centralizado em seu arquivo de serviços [estendendo](/learn/extending) sua aplicação. Aqui está um exemplo:

```php
// services.php

// cria um novo container
$container = new \Dice\Dice;
// não se esqueça de reatribuí-lo a si mesmo como abaixo!
$container = $container->addRule('PDO', [
	// shared significa que o mesmo objeto será retornado toda vez
	'shared' => true,
	'constructParams' => ['mysql:host=localhost;dbname=test', 'user', 'pass' ]
]);

// agora podemos criar um método mapeável para criar qualquer objeto. 
Flight::map('make', function($class, $params = []) use ($container) {
	return $container->create($class, $params);
});

// Isso registra o manipulador do container para que o Flight saiba usá-lo para controladores/middleware
Flight::registerContainerHandler(function($class, $params) {
	return Flight::make($class, $params);
});


// digamos que temos a seguinte classe de exemplo que recebe um objeto PDO no construtor
class EmailCron {
	protected PDO $pdo;

	public function __construct(PDO $pdo) {
		$this->pdo = $pdo;
	}

	public function send() {
		// código que envia um e-mail
	}
}

// E finalmente você pode criar objetos usando injeção de dependência
$emailCron = Flight::make(EmailCron::class);
$emailCron->send();
```

### `flightphp/container`

O Flight tem um plugin que fornece um container simples compatível com PSR-11 que você pode usar para lidar com sua injeção de dependência. Aqui está um exemplo rápido de como usá-lo:

```php

// index.php por exemplo
require 'vendor/autoload.php';

use flight\Container;

$container = new Container;

$container->set(PDO::class, fn(): PDO => new PDO('sqlite::memory:'));

Flight::registerContainerHandler([$container, 'get']);

class TestController {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function index() {
    var_dump($this->pdo);
	// vai exibir isso corretamente!
  }
}

Flight::route('GET /', [TestController::class, 'index']);

Flight::start();
```

#### Uso Avançado do flightphp/container

Você também pode resolver dependências de forma recursiva. Aqui está um exemplo:

```php
<?php

require 'vendor/autoload.php';

use flight\Container;

class User {}

interface UserRepository {
  function find(int $id): ?User;
}

class PdoUserRepository implements UserRepository {
  private PDO $pdo;

  function __construct(PDO $pdo) {
    $this->pdo = $pdo;
  }

  function find(int $id): ?User {
    // Implementação ...
    return null;
  }
}

$container = new Container;

$container->set(PDO::class, static fn(): PDO => new PDO('sqlite::memory:'));
$container->set(UserRepository::class, PdoUserRepository::class);

$userRepository = $container->get(UserRepository::class);
var_dump($userRepository);

/*
object(PdoUserRepository)#4 (1) {
  ["pdo":"PdoUserRepository":private]=>
  object(PDO)#3 (0) {
  }
}
 */
```

### DICE

Você também pode criar seu próprio manipulador de DIC. Isso é útil se você tiver um container personalizado que queira usar e que não seja PSR-11 (Dice). Consulte a seção [uso básico](#basic-usage) para saber como fazer isso.

Além disso, existem alguns padrões úteis que facilitarão sua vida ao usar o Flight.

#### Instância do Engine (necessária para injeção de `$app`)

Se você usar type-hint `flight\Engine` em controladores ou middleware, **o Dice não deve construir um novo Engine**. Substitua pela mesma instância do bootstrap. É isso que o skeleton oficial faz, e é o padrão que o `AGENTS.md` espera para controladores gerados por IA:

```php
// Em algum lugar do seu bootstrap / services.php
use flight\Engine;
use flight\database\SimplePdo;

$app = Flight::app(); // ou $engine = Flight::app();

$container = new \Dice\Dice;
$container = $container->addRule('*', [
	'substitutions' => [
		// Crítico: reutilize o Engine do bootstrap — não deixe o Dice fazer `new Engine()`
		Engine::class => $app,
		// Prefira SimplePdo para código novo
		// SimplePdo::class => $db,
		// Config::class => $config,
		// \Twig\Environment::class => $twig,
	]
]);

$app->registerContainerHandler(function ($class, $params) use ($container) {
	return $container->create($class, $params);
});

// Auxiliar opcional para código fora de rotas
$app->map('make', function ($class, $params = []) use ($container) {
	return $container->create($class, $params);
});
```

```php
// app/Controller/MyController.php  (layout do skeleton — a capitalização da pasta corresponde ao namespace)
namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// Sem fachada Flight:: na camada da aplicação — mais fácil de testar e mais claro para ferramentas de IA
		$this->app->render('welcome', ['message' => 'Hello']);
	}
}
```

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/', [MyController::class, 'index']);
```

Se você pular a substituição do `Engine`, o Dice pode construir um segundo Engine e seu controlador não compartilhará rotas, configuração ou o `render` do Twig mapeado a partir do bootstrap.

#### Adicionando outros serviços compartilhados (SimplePdo, Config, Twig)

```php
use flight\database\SimplePdo;
use flight\Engine;

// Depois que você criar $db, $config, $twig em services.php:
$substitutions = [
	Engine::class => $app,
	SimplePdo::class => $db,
	// App\Utils\Config::class => $config,
	// \Twig\Environment::class => $twig,
];

$container = $container->addRule('*', [
	'substitutions' => $substitutions,
]);
```

Então os controladores podem receber `SimplePdo $db` (ou seu tipo de configuração) no construtor e nunca chamar `Flight::db()`. Isso está de acordo com as orientações de [testes de unidade](/guides/unit-testing) e com o estilo padrão do skeleton.

#### Adicionando outras classes

Se você tiver outras classes que queira adicionar ao container, com Dice é fácil, pois elas serão resolvidas automaticamente pelo container. Aqui está um exemplo:

```php

$container = new \Dice\Dice;
// Se você não precisar injetar nenhuma dependência em suas classes
// você não precisa definir nada!
Flight::registerContainerHandler(function($class, $params) use ($container) {
	return $container->create($class, $params);
});

class MyCustomClass {
	public function parseThing() {
		return 'thing';
	}
}

class UserController {

	protected MyCustomClass $MyCustomClass;

	public function __construct(MyCustomClass $MyCustomClass) {
		$this->MyCustomClass = $MyCustomClass;
	}

	public function index() {
		echo $this->MyCustomClass->parseThing();
	}
}

Flight::route('/user', 'UserController->index');
```

### PSR-11

O Flight também pode usar qualquer container compatível com PSR-11. Isso significa que você pode usar qualquer container que implemente a interface PSR-11. Aqui está um exemplo usando o container PSR-11 da League:

```php

require 'vendor/autoload.php';

use flight\database\SimplePdo;

// mesma ideia do UserController acima, usando type-hint de SimplePdo em vez de PDO puro

$container = new \League\Container\Container();
$container->add(UserController::class)->addArgument(SimplePdo::class);
$container->add(SimplePdo::class)
	->addArgument('mysql:host=localhost;dbname=test')
	->addArgument('user')
	->addArgument('pass');
Flight::registerContainerHandler($container);

Flight::route('/user', [ 'UserController', 'view' ]);

Flight::start();
```

Isso pode ser um pouco mais verboso do que o exemplo anterior com Dice, mas ainda assim cumpre o objetivo com os mesmos benefícios!

## Veja Também
- [Instalação](/install) - Layout do skeleton e onde `services.php` está localizado.
- [Autoloading](/learn/autoloading) - Namespaces `App\` e **capitalização** da pasta.
- [Estendendo o Flight](/learn/extending) - Aprenda como adicionar injeção de dependência às suas próprias classes estendendo o framework.
- [Configuração](/learn/configuration) - Aprenda como configurar o Flight para sua aplicação.
- [Roteamento](/learn/routing) - Aprenda como definir rotas para sua aplicação e como a injeção de dependência funciona com controladores.
- [Middleware](/learn/middleware) - Aprenda como criar middleware para sua aplicação e como a injeção de dependência funciona com middleware.
- [Testes de Unidade](/guides/unit-testing) - Por que a injeção no construtor é melhor do que globais `Flight::`.
- [IA e Experiência do Desenvolvedor](/learn/ai) - Um padrão de DI para humanos e agentes.
- [SimplePdo](/learn/simple-pdo) - Auxiliar de banco de dados preferido para injeção.

## Solução de Problemas
- Se você estiver tendo problemas com seu container, certifique-se de que está passando os nomes de classe corretos para ele.
- Controladores que usam type-hint `Engine` mas recebem uma aplicação "em branco": adicione a **substituição do Engine** (veja acima). O Dice não deve fazer `new` em um segundo Engine.
- Classe não encontrada para `App\Controller\…`: verifique a capitalização da pasta em `app/Controller/` — veja [Autoloading](/learn/autoloading).
- O manipulador deve **retornar** o objeto criado a partir de `registerContainerHandler` (não chame `Flight::make()` sem `return`).

## Histórico de Alterações
- Documentação – Documentar skeleton Dice + substituições de Engine, SimplePdo e layout `App\Controller` para projetos amigáveis a IA.
- v3.7.0 - Adicionada a capacidade de registrar um manipulador de DIC no Flight.