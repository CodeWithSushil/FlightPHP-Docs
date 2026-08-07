# Testes Unitários no Flight PHP com PHPUnit

Este guia introduz testes unitários no Flight PHP usando o [PHPUnit](https://phpunit.de/), voltado para iniciantes que desejam entender *por que* os testes unitários são importantes e como aplicá-los na prática. Vamos nos concentrar em testar o *comportamento* — garantindo que sua aplicação faça o que você espera, como enviar um e-mail ou salvar um registro — em vez de cálculos triviais. Começaremos com um [manipulador de rota](/learn/routing) simples e avançaremos para um [controller](/learn/routing) mais complexo, incorporando [injeção de dependência](/learn/dependency-injection-container) (DI) e simulação (mock) de serviços de terceiros.

## Por que Testes Unitários?

Testes unitários garantem que seu código se comporte como esperado, detectando bugs antes que cheguem à produção. Eles são especialmente valiosos no Flight, onde o roteamento leve e a flexibilidade podem levar a interações complexas. Para desenvolvedores solo ou equipes, os testes unitários atuam como uma rede de segurança, documentando o comportamento esperado e prevenindo regressões quando você revisita o código mais tarde. Eles também melhoram o design: código difícil de testar frequentemente sinaliza classes excessivamente complexas ou fortemente acopladas.

Diferente de exemplos simplistas (por exemplo, testar `x * y = z`), vamos nos concentrar em comportamentos do mundo real, como validar entrada, salvar dados ou acionar ações como e-mails. Nosso objetivo é tornar os testes acessíveis e significativos.

## Princípios Gerais Orientadores

1. **Teste o Comportamento, Não a Implementação**: Concentre-se nos resultados (por exemplo, "e-mail enviado" ou "registro salvo") em vez de detalhes internos. Isso torna os testes robustos contra refatorações.
2. **Pare de usar `Flight::`**: Os métodos estáticos do Flight são terrivelmente convenientes, mas dificultam os testes. Você deve se acostumar a usar a variável `$app` de `$app = Flight::app();`. `$app` tem todos os mesmos métodos que `Flight::` tem. Você ainda poderá usar `$app->route()` ou `$this->app->json()` no seu controller, etc. Você também deve usar o roteador real do Flight com `$router = $app->router()` e então pode usar `$router->get()`, `$router->post()`, `$router->group()` etc. Veja [Roteamento](/learn/routing).
3. **Mantenha os Testes Rápidos**: Testes rápidos incentivam a execução frequente. Evite operações lentas, como chamadas de banco de dados, em testes unitários. Se você tem um teste lento, é um sinal de que está escrevendo um teste de integração, não um teste unitário. Testes de integração são quando você realmente envolve bancos de dados reais, chamadas HTTP reais, envio de e-mail real, etc. Eles têm seu lugar, mas são lentos e podem ser instáveis, ou seja, às vezes falham por um motivo desconhecido.
4. **Use Nomes Descritivos**: Os nomes dos testes devem descrever claramente o comportamento que está sendo testado. Isso melhora a legibilidade e a manutenibilidade.
5. **Evite Globais como uma Praga**: Minimize o uso de `$app->set()` e `$app->get()`, pois eles agem como estado global, exigindo mocks em todos os testes. Prefira DI ou um contêiner de DI (veja [Contêiner de Injeção de Dependência](/learn/dependency-injection-container)). Até mesmo usar o método `$app->map()` é tecnicamente um "global" e deve ser evitado em favor de DI. Use uma biblioteca de sessão como [flightphp/session](https://github.com/flightphp/session) para que você possa simular o objeto de sessão em seus testes. **Não** chame [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php) diretamente no seu código, pois isso está injetando uma variável global no seu código, dificultando o teste.
6. **Use Injeção de Dependência**: Injete dependências (por exemplo, [`PDO`](https://www.php.net/manual/en/class.pdo.php), mailers) nos controllers para isolar a lógica e simplificar os mocks. Se você tem uma classe com muitas dependências, considere refatorá-la em classes menores que cada uma tenha uma única responsabilidade seguindo os [princípios SOLID](https://en.wikipedia.org/wiki/SOLID).
7. **Simule Serviços de Terceiros**: Simule bancos de dados, clientes HTTP (cURL) ou serviços de e-mail para evitar chamadas externas. Teste uma ou duas camadas de profundidade, mas deixe sua lógica principal ser executada. Por exemplo, se seu aplicativo envia uma mensagem de texto, você **NÃO** quer realmente enviar uma mensagem de texto toda vez que executa seus testes, porque esses custos vão aumentar (e será mais lento). Em vez disso, simule o serviço de mensagem de texto e apenas verifique se seu código chamou o serviço de mensagem de texto com os parâmetros corretos.
8. **Busque Alta Cobertura, Não Perfeição**: 100% de cobertura de linha é bom, mas isso não significa que tudo no seu código está testado da maneira que deveria (pesquise sobre [cobertura de branch/path no PHPUnit](https://localheinz.com/articles/2023/03/22/collecting-line-branch-and-path-coverage-with-phpunit/)). Priorize comportamentos críticos (por exemplo, registro de usuário, respostas de API e captura de respostas com falha).
9. **Use Controllers para Rotas**: Em suas definições de rota, use controllers em vez de closures. O `flight\Engine $app` é injetado em cada controller via construtor por padrão. Em testes, use `$app = new Flight\Engine()` para instanciar o Flight dentro de um teste, injete-o no seu controller e chame os métodos diretamente (por exemplo, `$controller->register()`). Veja [Estendendo o Flight](/learn/extending) e [Roteamento](/learn/routing).
10. **Escolha um estilo de mock e mantenha-o**: O PHPUnit suporta vários estilos de mock (por exemplo, prophecy, mocks embutidos), ou você pode usar classes anônimas que têm seus próprios benefícios, como autocompletar código, quebrar se você alterar a definição do método, etc. Apenas seja consistente em seus testes. Veja [Objetos Mock do PHPUnit](https://docs.phpunit.de/en/12.3/test-doubles.html#test-doubles).
11. **Use visibilidade `protected` para métodos/propriedades que você deseja testar em subclasses**: Isso permite que você os substitua em subclasses de teste sem torná-los públicos, isso é especialmente útil para mocks de classes anônimas.

## Configurando o PHPUnit

Primeiro, configure o [PHPUnit](https://phpunit.de/) no seu projeto Flight PHP usando o Composer para facilitar os testes. Veja o [Guia de Introdução ao PHPUnit](https://phpunit.readthedocs.io/en/12.3/installation.html) para mais detalhes.

1. No diretório do seu projeto, execute:
   ```bash
   composer require --dev phpunit/phpunit
   ```
   Isso instala o PHPUnit mais recente como uma dependência de desenvolvimento.

2. Crie um diretório `tests` na raiz do seu projeto para os arquivos de teste.

3. Adicione um script de teste ao `composer.json` para conveniência:
   ```json
   // outro conteúdo do composer.json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```

4. Crie um arquivo `phpunit.xml` na raiz:
   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <phpunit bootstrap="vendor/autoload.php">
       <testsuites>
           <testsuite name="Flight Tests">
               <directory>tests</directory>
           </testsuite>
       </testsuites>
   </phpunit>
   ```

Agora, quando seus testes estiverem prontos, você pode executar `composer test` para executar os testes.

## Testando um Manipulador de Rota Simples

Vamos começar com uma [rota](/learn/routing) básica que valida a entrada de e-mail de um usuário. Vamos testar seu comportamento: retornar uma mensagem de sucesso para e-mails válidos e um erro para e-mails inválidos. Para validação de e-mail, usamos [`filter_var`](https://www.php.net/manual/en/function.filter-var.php).

```php
// index.php
$app->route('POST /register', [ UserController::class, 'register' ]);

// UserController.php
class UserController {
	protected $app;

	public function __construct(flight\Engine $app) {
		$this->app = $app;
	}

	public function register() {
		$email = $this->app->request()->data->email;
		$responseArray = [];
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			$responseArray = ['status' => 'error', 'message' => 'E-mail inválido'];
		} else {
			$responseArray = ['status' => 'success', 'message' => 'E-mail válido'];
		}

		$this->app->json($responseArray);
	}
}
```

Para testar isso, crie um arquivo de teste. Veja [Testes Unitários e Princípios SOLID](/learn/unit-testing-and-solid-principles) para mais sobre como estruturar testes:

```php
// tests/UserControllerTest.php
use PHPUnit\Framework\TestCase;
use Flight;
use flight\Engine;

class UserControllerTest extends TestCase {

    public function testValidEmailReturnsSuccess() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'test@example.com'; // Simula dados POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
        $response = $app->response()->getBody();
		$output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('E-mail válido', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
		$app = new Engine();
		$request = $app->request();
		$request->data->email = 'invalid-email'; // Simula dados POST
		$UserController = new UserController($app);
		$UserController->register($request->data->email);
		$response = $app->response()->getBody();
		$output = json_decode($response, true);
		$this->assertEquals('error', $output['status']);
		$this->assertEquals('E-mail inválido', $output['message']);
	}
}
```

**Pontos-chave**:
- Simulamos dados POST usando a classe de requisição. Não use globais como `$_POST`, `$_GET`, etc., pois isso torna o teste mais complicado (você sempre precisa redefinir esses valores ou outros testes podem falhar).
- Todos os controllers, por padrão, terão a instância de `flight\Engine` injetada neles, mesmo sem um contêiner DIC configurado. Isso torna muito mais fácil testar controllers diretamente.
- Não há nenhum uso de `Flight::`, tornando o código mais fácil de testar.
- Os testes verificam o comportamento: status e mensagem corretos para e-mails válidos/inválidos.

Execute `composer test` para verificar se a rota se comporta como esperado. Para mais sobre [requisições](/learn/requests) e [respostas](/learn/responses) no Flight, consulte a documentação relevante.

## Usando Injeção de Dependência para Controllers Testáveis

Para cenários mais complexos, use [injeção de dependência](/learn/dependency-injection-container) (DI) para tornar os controllers testáveis. Evite os globais do Flight (por exemplo, `Flight::set()`, `Flight::map()`, `Flight::register()`) pois eles agem como estado global, exigindo mocks para todos os testes. Em vez disso, use o contêiner DI do Flight, [DICE](https://github.com/Level-2/Dice), [PHP-DI](https://php-di.org/) ou DI manual.

Vamos usar [`flight\database\SimplePdo`](/learn/simple-pdo) em vez de PDO cru. Este helper é muito mais fácil de simular e testar unitariamente (e é preferido em relação ao `PdoWrapper` obsoleto).

Aqui está um controller que salva um usuário no banco de dados e envia um e-mail de boas-vindas:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
			// adicionar o return aqui ajuda no teste unitário a interromper a execução
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'E-mail inválido']);
		}

		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);

		return $this->app->json(['status' => 'success', 'message' => 'Usuário registrado']);
    }
}
```

**Pontos-chave**:
- O controller depende de uma instância de [`SimplePdo`](/learn/simple-pdo) e de um `MailerInterface` (um serviço de e-mail de terceiros fictício).
- As dependências são injetadas via construtor, evitando globais.

### Testando o Controller com Mocks

Agora, vamos testar o comportamento do `UserController`: validar e-mails, salvar no banco de dados e enviar e-mails. Vamos simular o banco de dados e o mailer para isolar o controller.

```php
// tests/UserControllerDICTest.php
use flight\database\SimplePdo;
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {

		// Às vezes, misturar estilos de mock é necessário
		// Aqui usamos o mock embutido do PHPUnit para PDOStatement
		$statementMock = $this->createMock(PDOStatement::class);
		$statementMock->method('execute')->willReturn(true);
		// Usando uma classe anônima para simular SimplePdo
        $mockDb = new class($statementMock) extends SimplePdo {
			protected $statementMock;
			public function __construct($statementMock) {
				$this->statementMock = $statementMock;
			}

			// Quando simulamos dessa forma, não estamos realmente fazendo uma chamada de banco de dados.
			// Podemos configurar ainda mais isso para alterar o mock do PDOStatement para simular falhas, etc.
            public function runQuery(string $sql, array $params = []): PDOStatement {
                return $this->statementMock;
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                $this->sentEmail = $email;
                return true;	
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('Usuário registrado', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }

    public function testInvalidEmailSkipsSaveAndEmail() {
		 $mockDb = new class() extends SimplePdo {
			// Um construtor vazio evita o construtor pai
			public function __construct() {}
            public function runQuery(string $sql, array $params = []): PDOStatement {
                throw new Exception('Não deveria ser chamado');
            }
        };
        $mockMailer = new class implements MailerInterface {
            public $sentEmail = null;
            public function sendWelcome($email): bool {
                throw new Exception('Não deveria ser chamado');
            }
        };
		$app = new Engine();
		$app->request()->data->email = 'invalid-email';

		// Precisamos mapear jsonHalt para evitar sair
		$app->map('jsonHalt', function($data) use ($app) {
			$app->json($data, 400);
		});
        $controller = new UserControllerDIC($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('error', $result['status']);
        $this->assertEquals('E-mail inválido', $result['message']);
    }
}
```

**Pontos-chave**:
- Simulamos `SimplePdo` e `MailerInterface` para evitar chamadas reais de banco de dados ou e-mail.
- Os testes verificam o comportamento: e-mails válidos acionam inserções no banco de dados e envios de e-mail; e-mails inválidos pulam ambos.
- Simule dependências de terceiros (por exemplo, `SimplePdo`, `MailerInterface`), permitindo que a lógica do controller seja executada.

### Simulando demais

Tome cuidado para não simular demais o seu código. Deixe-me dar um exemplo abaixo sobre por que isso pode ser ruim usando nosso `UserController`. Vamos transformar essa verificação em um método chamado `isEmailValid` (usando `filter_var`) e as outras novas adições em um método separado chamado `registerUser`.

```php
use flight\database\SimplePdo;
use flight\Engine;

// UserControllerDICV2.php
class UserControllerDICV2 {
	protected $app;
    protected $db;
    protected $mailer;

    public function __construct(Engine $app, SimplePdo $db, MailerInterface $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }

    public function register() {
		$email = $this->app->request()->data->email;
		if (!$this->isEmailValid($email)) {
			// adicionar o return aqui ajuda no teste unitário a interromper a execução
			return $this->app->jsonHalt(['status' => 'error', 'message' => 'E-mail inválido']);
		}

		$this->registerUser($email);

		$this->app->json(['status' => 'success', 'message' => 'Usuário registrado']);
    }

	protected function isEmailValid($email) {
		return filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
	}

	protected function registerUser($email) {
		$this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
		$this->mailer->sendWelcome($email);
	}
}
```

E agora o teste unitário excessivamente simulado que na verdade não testa nada:

```php
use PHPUnit\Framework\TestCase;

class UserControllerTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
		$app = new Engine();
		$app->request()->data->email = 'test@example.com';
		// estamos pulando a injeção de dependência extra aqui porque é "fácil"
        $controller = new class($app) extends UserControllerDICV2 {
			protected $app;
			// Evita as dependências no construtor
			public function __construct($app) {
				$this->app = $app;
			}

			// Vamos apenas forçar isso para ser válido.
			protected function isEmailValid($email) {
				return true; // Sempre retorna true, evitando a validação real
			}

			// Evita as chamadas reais de banco de dados e mailer
			protected function registerUser($email) {
				return false;
			}
		};
        $controller->register();
		$response = $app->response()->getBody();
		$result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('Usuário registrado', $result['message']);
    }
}
```

Viva, temos testes unitários e eles estão passando! Mas espere, e se eu realmente alterar o funcionamento interno de `isEmailValid` ou `registerUser`? Meus testes ainda passarão porque eu simulei toda a funcionalidade. Deixe-me mostrar o que quero dizer.

```php
// UserControllerDICV2.php
class UserControllerDICV2 {

	// ... outros métodos ...

	protected function isEmailValid($email) {
		// Lógica alterada
		$validEmail = filter_var($email, FILTER_VALIDATE_EMAIL) !== false;
		// Agora deve ter apenas um domínio específico
		$validDomain = strpos($email, '@example.com') !== false; 
		return $validEmail && $validDomain;
	}
}
```

Se eu executar meus testes unitários acima, eles ainda passarão! Mas porque eu não estava testando o comportamento (realmente deixando parte do código ser executado), eu potencialmente codifiquei um bug prestes a acontecer em produção. O teste deve ser modificado para levar em conta o novo comportamento, e também o oposto de quando o comportamento não é o que esperamos.

## Exemplo Completo

Você pode encontrar um exemplo completo de um projeto Flight PHP com testes unitários no GitHub: [n0nag0n/flight-unit-tests-guide](https://github.com/n0nag0n/flight-unit-tests-guide).
Para um entendimento mais profundo, veja [Testes Unitários e Princípios SOLID](/learn/unit-testing-and-solid-principles).

## Armadilhas Comuns

- **Excesso de Mock**: Não simule todas as dependências; deixe alguma lógica (por exemplo, validação do controller) ser executada para testar o comportamento real. Veja [Testes Unitários e Princípios SOLID](/learn/unit-testing-and-solid-principles).
- **Estado Global**: Usar variáveis globais do PHP (por exemplo, [`$_SESSION`](https://www.php.net/manual/en/reserved.variables.session.php), [`$_COOKIE`](https://www.php.net/manual/en/reserved.variables.cookie.php)) intensamente torna os testes frágeis. O mesmo vale para `Flight::`. Refatore para passar dependências explicitamente.
- **Configuração Complexa**: Se a configuração do teste for trabalhosa, sua classe pode ter dependências ou responsabilidades demais, violando os [princípios SOLID](/learn/unit-testing-and-solid-principles).

## Escalando com Testes Unitários

Testes unitários brilham em projetos maiores ou ao revisitar código após meses. Eles documentam o comportamento e detectam regressões, economizando você de ter que reaprender sua aplicação. Para desenvolvedores solo, teste caminhos críticos (por exemplo, registro de usuário, processamento de pagamentos). Para equipes, os testes garantem um comportamento consistente entre as contribuições. Veja [Por que Frameworks?](/learn/why-frameworks) para mais sobre os benefícios de usar frameworks e testes.

Contribua com suas próprias dicas de teste para o repositório de documentação do Flight PHP!

_Escrito por [n0nag0n](https://github.com/n0nag0n) 2025_