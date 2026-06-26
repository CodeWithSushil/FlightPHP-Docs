# Segurança

## Visão Geral

A segurança é algo muito importante quando se trata de aplicações web. Você quer garantir que sua aplicação seja segura e que os dados dos seus usuários estejam 
seguros. O Flight fornece uma série de recursos para ajudá-lo a proteger suas aplicações web.

## Entendendo

Existem várias ameaças de segurança comuns das quais você deve estar ciente ao construir aplicações web. Algumas das ameaças mais comuns
incluem:
- Cross Site Request Forgery (CSRF)
- Cross Site Scripting (XSS)
- SQL Injection
- Cross Origin Resource Sharing (CORS)

Os [Templates](/learn/templates) ajudam com XSS ao escapar a saída por padrão, para que você não precise se lembrar de fazer isso. As [Sessions](/awesome-plugins/session) podem ajudar com CSRF armazenando um token CSRF na sessão do usuário, conforme descrito abaixo. Usar instruções preparadas com PDO pode ajudar a prevenir ataques de injeção de SQL (ou usar métodos úteis na classe [PdoWrapper](/learn/pdo-wrapper)). CORS pode ser tratado com um hook simples antes de `Flight::start()` ser chamado.

Todos esses métodos funcionam em conjunto para ajudar a manter suas aplicações web seguras. Deve sempre estar em primeiro plano na sua mente aprender e entender as melhores práticas de segurança.

## Uso Básico

### Cabeçalhos

Os cabeçalhos HTTP são uma das maneiras mais fáceis de proteger suas aplicações web. Você pode usar cabeçalhos para prevenir clickjacking, XSS e outros ataques. 
Existem várias maneiras de adicionar esses cabeçalhos à sua aplicação.

Dois ótimos sites para verificar a segurança dos seus cabeçalhos são [securityheaders.com](https://securityheaders.com/) e 
[observatory.mozilla.org](https://observatory.mozilla.org/). Depois de configurar o código abaixo, você pode facilmente verificar se seus cabeçalhos estão funcionando com esses dois sites.

#### Adicionar Manualmente

Você pode adicionar manualmente esses cabeçalhos usando o método `header` no objeto `Flight\Response`.
```php
// Define o cabeçalho X-Frame-Options para prevenir clickjacking
Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');

// Define o cabeçalho Content-Security-Policy para prevenir XSS
// Nota: este cabeçalho pode ficar muito complexo, então você vai querer
//  consultar exemplos na internet para sua aplicação
Flight::response()->header("Content-Security-Policy", "default-src 'self'");

// Define o cabeçalho X-XSS-Protection para prevenir XSS
Flight::response()->header('X-XSS-Protection', '1; mode=block');

// Define o cabeçalho X-Content-Type-Options para prevenir MIME sniffing
Flight::response()->header('X-Content-Type-Options', 'nosniff');

// Define o cabeçalho Referrer-Policy para controlar quanto de informação de referência é enviada
Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');

// Define o cabeçalho Strict-Transport-Security para forçar HTTPS
Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');

// Define o cabeçalho Permissions-Policy para controlar quais recursos e APIs podem ser usados
Flight::response()->header('Permissions-Policy', 'geolocation=()');
```

Estes podem ser adicionados no topo dos seus arquivos `routes.php` ou `index.php`.

#### Adicionar como um Filtro

Você também pode adicioná-los em um filtro/hook como o seguinte: 

```php
// Adiciona os cabeçalhos em um filtro
Flight::before('start', function() {
	Flight::response()->header('X-Frame-Options', 'SAMEORIGIN');
	Flight::response()->header("Content-Security-Policy", "default-src 'self'");
	Flight::response()->header('X-XSS-Protection', '1; mode=block');
	Flight::response()->header('X-Content-Type-Options', 'nosniff');
	Flight::response()->header('Referrer-Policy', 'no-referrer-when-downgrade');
	Flight::response()->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
	Flight::response()->header('Permissions-Policy', 'geolocation=()');
});
```

#### Adicionar como um Middleware

Você também pode adicioná-los como uma classe middleware que fornece a maior flexibilidade para quais rotas aplicar isso. Em geral, esses cabeçalhos devem ser aplicados a todas as respostas HTML e API.

```php
// app/middlewares/SecurityHeadersMiddleware.php

namespace app\middlewares;

use flight\Engine;

class SecurityHeadersMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		$response = $this->app->response();
		$response->header('X-Frame-Options', 'SAMEORIGIN');
		$response->header("Content-Security-Policy", "default-src 'self'");
		$response->header('X-XSS-Protection', '1; mode=block');
		$response->header('X-Content-Type-Options', 'nosniff');
		$response->header('Referrer-Policy', 'no-referrer-when-downgrade');
		$response->header('Strict-Transport-Security', 'max-age=31536000; includeSubDomains; preload');
		$response->header('Permissions-Policy', 'geolocation=()');
	}
}

// index.php ou onde quer que você tenha suas rotas
// FYI, este grupo de string vazia atua como um middleware global para
// todas as rotas. Claro que você poderia fazer a mesma coisa e apenas adicionar
// isso apenas a rotas específicas.
Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// mais rotas
}, [ SecurityHeadersMiddleware::class ]);
```

### Cross Site Request Forgery (CSRF)

Cross Site Request Forgery (CSRF) é um tipo de ataque onde um site malicioso pode fazer o navegador de um usuário enviar uma solicitação para o seu site. 
Isso pode ser usado para realizar ações no seu site sem o conhecimento do usuário. O Flight não fornece um mecanismo de proteção CSRF integrado, 
mas você pode facilmente implementar o seu próprio usando middleware.

#### Configuração

Primeiro você precisa gerar um token CSRF e armazená-lo na sessão do usuário. Você pode então usar este token nos seus formulários e verificá-lo quando 
o formulário for enviado. Vamos usar o plugin [flightphp/session](/awesome-plugins/session) para gerenciar sessões.

```php
// Gera um token CSRF e armazena-o na sessão do usuário
// (assumindo que você criou um objeto de sessão e o anexou ao Flight)
// veja a documentação da sessão para mais informações
Flight::register('session', flight\Session::class);

// Você só precisa gerar um único token por sessão (para que funcione 
// em várias abas e solicitações para o mesmo usuário)
if(Flight::session()->get('csrf_token') === null) {
	Flight::session()->set('csrf_token', bin2hex(random_bytes(32)) );
}
```

##### Usando o Template PHP Flight Padrão

```html
<!-- Use o token CSRF no seu formulário -->
<form method="post">
	<input type="hidden" name="csrf_token" value="<?= Flight::session()->get('csrf_token') ?>">
	<!-- outros campos do formulário -->
</form>
```

##### Usando Latte

Você também pode definir uma função personalizada para exibir o token CSRF nos seus templates Latte.

```php

Flight::map('render', function(string $template, array $data, ?string $block): void {
	$latte = new Latte\Engine;

	// outras configurações...

	// Define uma função personalizada para exibir o token CSRF
	$latte->addFunction('csrf', function() {
		$csrfToken = Flight::session()->get('csrf_token');
		return new \Latte\Runtime\Html('<input type="hidden" name="csrf_token" value="' . $csrfToken . '">');
	});

	$latte->render($finalPath, $data, $block);
});
```

E agora nos seus templates Latte você pode usar a função `csrf()` para exibir o token CSRF.

```html
<form method="post">
	{csrf()}
	<!-- outros campos do formulário -->
</form>
```

#### Verificar o Token CSRF

Você pode verificar o token CSRF usando vários métodos.

##### Middleware

```php
// app/middlewares/CsrfMiddleware.php

namespace app\middleware;

use flight\Engine;

class CsrfMiddleware
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function before(array $params): void
	{
		if($this->app->request()->method == 'POST') {
			$token = $this->app->request()->data->csrf_token;
			if($token !== $this->app->session()->get('csrf_token')) {
				$this->app->halt(403, 'Invalid CSRF token');
			}
		}
	}
}

// index.php ou onde quer que você tenha suas rotas
use app\middlewares\CsrfMiddleware;

Flight::group('', function(Router $router) {
	$router->get('/users', [ 'UserController', 'getUsers' ]);
	// mais rotas
}, [ CsrfMiddleware::class ]);
```

##### Filtros de Evento

```php
// Este middleware verifica se a solicitação é uma solicitação POST e, se for, verifica se o token CSRF é válido
Flight::before('start', function() {
	if(Flight::request()->method == 'POST') {

		// captura o token csrf dos valores do formulário
		$token = Flight::request()->data->csrf_token;
		if($token !== Flight::session()->get('csrf_token')) {
			Flight::halt(403, 'Invalid CSRF token');
			// ou para uma resposta JSON
			Flight::jsonHalt(['error' => 'Invalid CSRF token'], 403);
		}
	}
});
```

### Cross Site Scripting (XSS)

Cross Site Scripting (XSS) é um tipo de ataque onde uma entrada de formulário maliciosa pode injetar código no seu site. A maioria dessas oportunidades vem 
de valores de formulário que seus usuários finais preencherão. Você **nunca** deve confiar na saída dos seus usuários! Sempre assuma que todos eles são os 
melhores hackers do mundo. Eles podem injetar JavaScript ou HTML maliciosos na sua página. Este código pode ser usado para roubar informações dos seus 
usuários ou realizar ações no seu site. Usando a classe de visualização do Flight ou outro mecanismo de template como [Latte](/awesome-plugins/latte), você pode facilmente escapar a saída para prevenir ataques XSS.

```php
// Vamos assumir que o usuário é inteligente e tenta usar isso como seu nome
$name = '<script>alert("XSS")</script>';

// Isso vai escapar a saída
Flight::view()->set('name', $name);
// Isso vai exibir: &lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;

// Se você usar algo como Latte registrado como sua classe de visualização, ele também vai escapar automaticamente isso.
Flight::view()->render('template', ['name' => $name]);
```

### SQL Injection

SQL Injection é um tipo de ataque onde um usuário malicioso pode injetar código SQL no seu banco de dados. Isso pode ser usado para roubar informações 
do seu banco de dados ou realizar ações no seu banco de dados. Novamente, você **nunca** deve confiar na entrada dos seus usuários! Sempre assuma que eles estão 
procurando sangue. Você pode usar instruções preparadas nos seus objetos `PDO` para prevenir SQL injection.

```php
// Assumindo que você tem Flight::db() registrado como seu objeto PDO
$statement = Flight::db()->prepare('SELECT * FROM users WHERE username = :username');
$statement->execute([':username' => $username]);
$users = $statement->fetchAll();

// Se você usar a classe PdoWrapper, isso pode ser facilmente feito em uma linha
$users = Flight::db()->fetchAll('SELECT * FROM users WHERE username = :username', [ 'username' => $username ]);

// Você pode fazer a mesma coisa com um objeto PDO com placeholders ?
$statement = Flight::db()->fetchAll('SELECT * FROM users WHERE username = ?', [ $username ]);
```

#### Exemplo Inseguro

Abaixo está o motivo de usarmos instruções preparadas SQL para proteger de exemplos inocentes como o abaixo:

```php
// o usuário final preenche um formulário web.
// para o valor do formulário, o hacker coloca algo como isto:
$username = "' OR 1=1; -- ";

$sql = "SELECT * FROM users WHERE username = '$username' LIMIT 5";
$users = Flight::db()->fetchAll($sql);
// Depois que a consulta é construída, ela fica assim
// SELECT * FROM users WHERE username = '' OR 1=1; -- LIMIT 5

// Parece estranho, mas é uma consulta válida que vai funcionar. Na verdade,
// é um ataque de injeção SQL muito comum que vai retornar todos os usuários.

var_dump($users); // isso vai despejar todos os usuários no banco de dados, não apenas o único nome de usuário
```

### Validação de Callback JSONP

Se você usa o método `Flight::jsonp()` do Flight, esteja ciente de que o Flight valida o nome do parâmetro de callback JSONP contra uma regex estrita de lista de permitidos (`/^[A-Za-z_$][\w$.]{0,127}$/`). Qualquer nome de callback que não corresponda a este padrão fará com que o Flight lance uma exceção, prevenindo a injeção de JavaScript arbitrário através de um valor de callback malicioso.

Esta validação está integrada e não requer configuração adicional, mas vale a pena saber sobre isso ao depurar erros inesperados de endpoints JSONP.

### CORS

Cross-Origin Resource Sharing (CORS) é um mecanismo que permite que muitos recursos (ex: fontes, JavaScript, etc.) em uma página web sejam 
solicitados de outro domínio fora do domínio de onde o recurso se originou. O Flight não tem funcionalidade integrada, 
mas isso pode ser facilmente tratado com um hook para executar antes do método `Flight::start()` ser chamado.

```php
// app/utils/CorsUtil.php

namespace app\utils;

class CorsUtil
{
	public function set(array $params): void
	{
		$request = Flight::request();
		$response = Flight::response();
		if ($request->getVar('HTTP_ORIGIN') !== '') {
			$this->allowOrigins();
			$response->header('Access-Control-Allow-Credentials', 'true');
			$response->header('Access-Control-Max-Age', '86400');
		}

		if ($request->method === 'OPTIONS') {
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_METHOD') !== '') {
				$response->header(
					'Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, PATCH, OPTIONS, HEAD'
				);
			}
			if ($request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS') !== '') {
				$response->header(
					"Access-Control-Allow-Headers",
					$request->getVar('HTTP_ACCESS_CONTROL_REQUEST_HEADERS')
				);
			}

			$response->status(200);
			$response->send();
			exit;
		}
	}

	private function allowOrigins(): void
	{
		// personalize seus hosts permitidos aqui.
		$allowed = [
			'capacitor://localhost',
			'ionic://localhost',
			'http://localhost',
			'http://localhost:4200',
			'http://localhost:8080',
			'http://localhost:8100',
		];

		$request = Flight::request();

		if (in_array($request->getVar('HTTP_ORIGIN'), $allowed, true) === true) {
			$response = Flight::response();
			$response->header("Access-Control-Allow-Origin", $request->getVar('HTTP_ORIGIN'));
		}
	}
}

// index.php ou onde quer que você tenha suas rotas
$CorsUtil = new CorsUtil();

// Isso precisa ser executado antes de start ser executado.
Flight::before('start', [ $CorsUtil, 'setupCors' ]);
```

### Endurecimento da Configuração do Flight

O Flight expõe várias configurações do engine que têm implicações de segurança diretas. Configurar essas corretamente é uma das maneiras mais fáceis de endurecer sua aplicação.

#### `flight.allow_method_override`

Por padrão, o Flight permite que clientes substituam o método HTTP de uma solicitação usando o cabeçalho `X-HTTP-Method-Override` ou um campo `_method` no corpo de um POST. Embora isso seja útil para formulários HTML que só podem enviar `GET`/`POST`, pode ser perigoso se você não estiver esperando por isso — um atacante poderia falsificar solicitações `DELETE` ou `PUT` através de um formulário regular.

Se sua aplicação não depende desse comportamento (ex: você está construindo uma API consumida por clientes modernos ou frontends JavaScript que podem enviar qualquer verbo HTTP), você deve desabilitá-lo:

```php
// No seu index.php ou arquivo de bootstrap, antes de Flight::start()
Flight::set('flight.allow_method_override', false);
```

O valor padrão é `true` para compatibilidade retroativa, mas **definir como `false` é fortemente recomendado** para qualquer aplicação que não precise explicitamente do recurso de substituição.

#### `flight.debug`

O Flight tem uma configuração `flight.debug` que controla se informações detalhadas de erro (mensagem de exceção, código e trace completo da pilha) são renderizadas no navegador quando uma exceção não tratada ocorre. O padrão é `false`, o que significa que apenas uma mensagem genérica `500 Internal Server Error` é mostrada — nenhum detalhe interno é vazado para o cliente.

Nunca habilite isso em um servidor de produção. Use apenas localmente ou em um ambiente de staging:

```php
// Seguro apenas para desenvolvimento local — NUNCA em produção
Flight::set('flight.debug', true);
```

Quando `flight.debug` é `false` (o padrão), você ainda pode capturar erros habilitando `flight.log_errors`:

```php
// Registra erros no lado do servidor sem expô-los ao cliente
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

#### Configuração de produção recomendada

```php
// index.php ou app/config/config.php
Flight::set('flight.allow_method_override', false);
Flight::set('flight.debug', false);
Flight::set('flight.log_errors', true);
```

### Tratamento de Erros
Oculte detalhes de erro sensíveis em produção para evitar vazamento de informações para atacantes. Em produção, registre erros em vez de exibi-los com `display_errors` definido como `0`.

```php
// No seu bootstrap.php ou index.php

// adicione isso ao seu app/config/config.php
$environment = ENVIRONMENT;
if ($environment === 'production') {
    ini_set('display_errors', 0); // Desabilita a exibição de erros
    ini_set('log_errors', 1);     // Registra erros em vez disso
    ini_set('error_log', '/path/to/error.log');
}

// Nas suas rotas ou controladores
// Use Flight::halt() para respostas de erro controladas
Flight::halt(403, 'Access denied');
```

### Sanitização de Entrada
Nunca confie na entrada do usuário. Sanitize-a usando [filter_var](https://www.php.net/manual/en/function.filter-var.php) antes de processar para prevenir que dados maliciosos se infiltrem.

```php

// Vamos assumir uma solicitação $_POST com $_POST['input'] e $_POST['email']

// Sanitiza uma entrada de string
$clean_input = filter_var(Flight::request()->data->input, FILTER_SANITIZE_STRING);
// Sanitiza um email
$clean_email = filter_var(Flight::request()->data->email, FILTER_SANITIZE_EMAIL);
```

### Hash de Senhas
Armazene senhas com segurança e verifique-as com segurança usando funções integradas do PHP como [password_hash](https://www.php.net/manual/en/function.password-hash.php) e [password_verify](https://www.php.net/manual/en/function.password-verify.php). Senhas nunca devem ser armazenadas em texto simples, nem devem ser criptografadas com métodos reversíveis. O hashing garante que mesmo se seu banco de dados for comprometido, as senhas reais permaneçam protegidas.

```php
$password = Flight::request()->data->password;
// Faz hash de uma senha ao armazenar (ex: durante o registro)
$hashed_password = password_hash($password, PASSWORD_DEFAULT);

// Verifica uma senha (ex: durante o login)
if (password_verify($password, $stored_hash)) {
    // Senha corresponde
}
```

### Limitação de Taxa
Proteja contra ataques de força bruta ou ataques de negação de serviço limitando as taxas de solicitação com um cache.

```php
// Assumindo que você tem flightphp/cache instalado e registrado
// Usando flightphp/cache em um filtro
Flight::before('start', function() {
    $cache = Flight::cache();
    $ip = Flight::request()->ip;
    $key = "rate_limit_{$ip}";
    $attempts = (int) $cache->retrieve($key);
    
    if ($attempts >= 10) {
        Flight::halt(429, 'Too many requests');
    }
    
    $cache->set($key, $attempts + 1, 60); // Redefine após 60 segundos
});
```

## Veja Também
- [Sessions](/awesome-plugins/session) - Como gerenciar sessões de usuário com segurança.
- [Templates](/learn/templates) - Usando templates para escapar automaticamente a saída e prevenir XSS.
- [PDO Wrapper](/learn/pdo-wrapper) - Interações simplificadas com banco de dados usando instruções preparadas.
- [Middleware](/learn/middleware) - Como usar middleware para simplificar o processo de adicionar cabeçalhos de segurança.
- [Responses](/learn/responses) - Como personalizar respostas HTTP com cabeçalhos seguros.
- [Requests](/learn/requests) - Como lidar e sanitizar entrada do usuário.
- [filter_var](https://www.php.net/manual/en/function.filter-var.php) - Função PHP para sanitização de entrada.
- [password_hash](https://www.php.net/manual/en/function.password-hash.php) - Função PHP para hash seguro de senhas.
- [password_verify](https://www.php.net/manual/en/function.password-verify.php) - Função PHP para verificar senhas com hash.

## Solução de Problemas
- Consulte a seção "Veja Também" acima para informações de solução de problemas relacionadas a questões com componentes do Flight Framework.

## Changelog
- v3.18.1 - Adicionada seção de Endurecimento da Configuração do Flight cobrindo `flight.allow_method_override`, `flight.debug` e validação de callback JSONP.
- v3.1.0 - Adicionadas seções sobre CORS, Tratamento de Erros, Sanitização de Entrada, Hash de Senhas e Limitação de Taxa.
- v2.0 - Adicionado escape para visualizações padrão para prevenir XSS.