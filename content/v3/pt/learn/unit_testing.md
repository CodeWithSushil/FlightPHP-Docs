# Testes Unitários

## Visão Geral

Os testes unitários no Flight ajudam a garantir que sua aplicação se comporte conforme o esperado, detectar bugs cedo e tornar a base de código mais fácil de manter. O Flight foi projetado para funcionar bem com o [PHPUnit](https://phpunit.de/), o framework de testes PHP mais popular.

## Compreensão

Os testes unitários verificam o comportamento de pequenas partes da sua aplicação (como controladores ou serviços) de forma isolada. No Flight, isso significa testar como suas rotas, controladores e lógica respondem a diferentes entradas—sem depender de estado global ou serviços externos reais.

Princípios-chave:
- **Teste o comportamento, não a implementação:** Concentre-se no que seu código faz, não em como faz.
- **Evite estado global:** Use injeção de dependência em vez de `Flight::set()` ou `Flight::get()`.
- **Simule serviços externos:** Substitua coisas como bancos de dados ou serviços de e-mail por dublês de teste.
- **Mantenha os testes rápidos e focados:** Os testes unitários não devem acessar bancos de dados ou APIs reais.

## Uso Básico

### Configurando o PHPUnit

1. Instale o PHPUnit com o Composer:
   ```bash
   composer require --dev phpunit/phpunit
   ```
2. Crie um diretório `tests` na raiz do seu projeto.
3. Adicione um script de teste ao seu `composer.json`:
   ```json
   "scripts": {
       "test": "phpunit --configuration phpunit.xml"
   }
   ```
4. Crie um arquivo `phpunit.xml`:
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

Agora você pode executar seus testes com `composer test`.

### Testando um Manipulador de Rota Simples

Suponha que você tenha uma rota que valide um e-mail:

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
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        return $this->app->json(['status' => 'success', 'message' => 'Valid email']);
    }
}
```

Um teste simples para este controlador:

```php
use PHPUnit\Framework\TestCase;
use flight\Engine;

class UserControllerTest extends TestCase {
    public function testValidEmailReturnsSuccess() {
        $app = new Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('success', $output['status']);
        $this->assertEquals('Valid email', $output['message']);
    }

    public function testInvalidEmailReturnsError() {
        $app = new Engine();
        $app->request()->data->email = 'invalid-email';
        $controller = new UserController($app);
        $controller->register();
        $response = $app->response()->getBody();
        $output = json_decode($response, true);
        $this->assertEquals('error', $output['status']);
        $this->assertEquals('Invalid email', $output['message']);
    }
}
```

**Dicas:**
- Simule dados POST usando `$app->request()->data`.
- Evite usar estáticos `Flight::` em seus testes—use a instância `$app`.

### Usando Injeção de Dependência para Controladores Testáveis

Injete dependências (como o banco de dados ou serviço de e-mail) em seus controladores para facilitar a simulação (mock) nos testes:

```php
use flight\database\SimplePdo;

class UserController {
    protected $app;
    protected $db;
    protected $mailer;
    public function __construct($app, $db, $mailer) {
        $this->app = $app;
        $this->db = $db;
        $this->mailer = $mailer;
    }
    public function register() {
        $email = $this->app->request()->data->email;
        if (!filter_var($email, FILTER_VALIDATE_EMAIL)) {
            return $this->app->json(['status' => 'error', 'message' => 'Invalid email']);
        }
        $this->db->runQuery('INSERT INTO users (email) VALUES (?)', [$email]);
        $this->mailer->sendWelcome($email);
        return $this->app->json(['status' => 'success', 'message' => 'User registered']);
    }
}
```

E um teste com mocks (simulações):

```php
use PHPUnit\Framework\TestCase;

class UserControllerDICTest extends TestCase {
    public function testValidEmailSavesAndSendsEmail() {
        $mockDb = $this->createMock(flight\database\SimplePdo::class);
        $mockDb->method('runQuery')->willReturn(true);
        $mockMailer = new class {
            public $sentEmail = null;
            public function sendWelcome($email) { $this->sentEmail = $email; return true; }
        };
        $app = new flight\Engine();
        $app->request()->data->email = 'test@example.com';
        $controller = new UserController($app, $mockDb, $mockMailer);
        $controller->register();
        $response = $app->response()->getBody();
        $result = json_decode($response, true);
        $this->assertEquals('success', $result['status']);
        $this->assertEquals('User registered', $result['message']);
        $this->assertEquals('test@example.com', $mockMailer->sentEmail);
    }
}
```

## Uso Avançado

- **Simulação (Mocking):** Use os mocks integrados do PHPUnit ou classes anônimas para substituir dependências.
- **Testando controladores diretamente:** Instancie os controladores com um novo `Engine` e simule as dependências.
- **Evite excesso de mocks:** Deixe a lógica real executar quando possível; simule apenas serviços externos.

## Veja Também

- [Guia de Testes Unitários](/guides/unit-testing) - Um guia completo sobre boas práticas de testes unitários.
- [Contêiner de Injeção de Dependência](/learn/dependency-injection-container) - Como usar DICs para gerenciar dependências e melhorar a testabilidade.
- [Estendendo](/learn/extending) - Como adicionar seus próprios helpers ou sobrescrever classes principais.
- [SimplePdo](/learn/simple-pdo) - Simplifica as interações com o banco de dados e é mais fácil de simular em testes.
- [Requisições](/learn/requests) - Lidando com requisições HTTP no Flight.
- [Respostas](/learn/responses) - Enviando respostas aos usuários.
- [Testes Unitários e Princípios SOLID](/learn/unit-testing-and-solid-principles) - Aprenda como os princípios SOLID podem melhorar seus testes unitários.

## Solução de Problemas

- Evite usar estado global (`Flight::set()`, `$_SESSION`, etc.) em seu código e testes.
- Se seus testes estão lentos, você pode estar escrevendo testes de integração—simule serviços externos para manter os testes unitários rápidos.
- Se a configuração do teste for complexa, considere refatorar seu código para usar injeção de dependência.

## Registro de Alterações

- v3.15.0 - Adicionados exemplos para injeção de dependência e simulação (mocking).