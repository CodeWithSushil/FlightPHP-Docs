# Carregamento automático (Autoloading)

## Visão geral

O carregamento automático (autoloading) é um conceito em PHP onde você especifica um diretório ou diretórios para carregar classes. Isso é muito mais benéfico do que usar `require` ou `include` para carregar classes. Também é um requisito para usar pacotes Composer.

Acertar o autoloading também é importante para o [desenvolvimento assistido por IA](/learn/ai): os agentes colocam arquivos onde o namespace aponta. Se as **maiúsculas/minúsculas** das pastas e as do namespace divergirem, erros de classe não encontrada aparecem no Linux, mesmo quando as coisas "funcionavam" em um disco Mac sem distinção de maiúsculas/minúsculas.

## Entendendo

Por padrão, qualquer classe `Flight` é carregada automaticamente para você graças ao Composer. Para as classes de **sua** aplicação, você tem duas abordagens comuns:

1. **Composer PSR-4** (o que o [skeleton oficial](https://github.com/flightphp/skeleton) usa): mapeie um prefixo de namespace para um diretório em `composer.json` e depois execute `composer dump-autoload`.
2. **`Flight::path()`**: aponte o carregador do Flight para diretórios (útil para aplicações simples ou quando você não está usando Composer para o código da aplicação).

Usar um autoloader simplifica muito seu código. Em vez de uma parede de `include` / `require` no topo de cada arquivo, as classes são carregadas quando você as usa pela primeira vez.

### Sensibilidade a maiúsculas/minúsculas (leia isto duas vezes)

**Os namespaces devem corresponder à estrutura de diretórios *e* às maiúsculas/minúsculas desses diretórios.**

| Funciona | Quebra no Linux |
|-------|-----------------|
| `App\Controller\HomeController` → `app/Controller/HomeController.php` | `App\Controller\…` com a pasta `app/controllers/` |
| `app\controllers\MyController` → `app/controllers/MyController.php` | Misturar `App\` com `controllers` minúsculo |

Namespaces em PHP não diferenciam maiúsculas de minúsculas em alguns contextos, mas **o Composer e o sistema de arquivos diferenciam**. O skeleton oficial é padronizado em:

- Composer: `"App\\": "app/"`
- Pastas: **`Controller`**, **`Middleware`**, **`Model`**, **`Utils`** (PascalCase), não `controllers` / `middlewares`

Documentações antigas e exemplos da comunidade às vezes usavam `app\controllers` minúsculo. Isso ainda funciona se suas pastas forem minúsculas—mas **os novos projetos skeleton usam `App\` + pastas PascalCase**. Escolha uma convenção por projeto e siga-a para que humanos e ferramentas de IA não inventem uma segunda estrutura.

## Skeleton (recomendado para novos projetos)

Após `composer create-project flightphp/skeleton`, o código da aplicação é carregado automaticamente via Composer—nenhum `Flight::path()` é necessário para classes `App\`:

```json
{
  "autoload": {
    "psr-4": {
      "App\\": "app/"
    }
  }
}
```

```php
// app/Controller/HomeController.php
namespace App\Controller;

use flight\Engine;

class HomeController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		$this->app->render('welcome', ['message' => 'Hello!']);
	}
}
```

```php
// app/config/routes.php — Dice resolve App\Controller\… via o container
$router->get('/', [HomeController::class, 'index']);
```

Veja [Instalação](/install) para a árvore completa e [IA & experiência de desenvolvimento](/learn/ai) para saber como o `AGENTS.md` documenta essa estrutura para assistentes de codificação.

## Uso básico (`Flight::path()`)

Vamos supor que temos uma árvore de diretórios como a seguinte:

```text
# Exemplo de caminho
/home/user/project/my-flight-project/
├── app
│   ├── cache
│   ├── config
│   ├── controllers - contém os controllers deste projeto
│   ├── translations
│   ├── UTILS - contém classes apenas para esta aplicação (todo em maiúsculas de propósito para um exemplo mais adiante)
│   └── views
└── public
    └── css
	└── js
	└── index.php
```

Você deve ter notado que isso é semelhante a uma árvore típica de aplicação (o próprio site da documentação usa uma estrutura organizada). O `controllers` minúsculo aqui é uma *escolha* válida—apenas não é o padrão atual do skeleton.

Você pode especificar cada diretório para carregar desta forma:

```php

/**
 * public/index.php
 */

// Adiciona um caminho ao autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');


/**
 * app/controllers/MyController.php
 */

// nenhum namespace é necessário

// Todas as classes autoloaded são recomendadas em Pascal Case (cada palavra capitalizada, sem espaços)
class MyController {

	public function index() {
		// faz alguma coisa
	}
}
```

## Namespaces com `Flight::path()`

Se você tiver namespaces, na verdade fica muito fácil implementar isso. Você deve usar o método `Flight::path()` para especificar o diretório raiz (não o document root nem a pasta `public/`) da sua aplicação.

```php

/**
 * public/index.php
 */

// Adiciona um caminho ao autoloader
Flight::path(__DIR__.'/../');
```

Agora é assim que seu controller pode ficar. Veja o exemplo abaixo, mas preste atenção nos comentários para informações importantes.

```php
/**
 * app/controllers/MyController.php
 */

// namespaces são obrigatórios
// namespaces são iguais à estrutura de diretórios
// namespaces devem seguir as mesmas maiúsculas/minúsculas da estrutura de diretórios
// namespaces e diretórios não podem conter underscores (a menos que Loader::setV2ClassLoading(false) esteja definido)
namespace app\controllers;

// Todas as classes autoloaded são recomendadas em Pascal Case (cada palavra capitalizada, sem espaços)
// A partir da 3.7.2, você pode usar Pascal_Snake_Case para os nomes das suas classes executando Loader::setV2ClassLoading(false);
class MyController {

	public function index() {
		// faz alguma coisa
	}
}
```

E se você quisesse autocarregar uma classe no diretório utils, faria basicamente o mesmo:

```php

/**
 * app/UTILS/ArrayHelperUtil.php
 */

// o namespace deve corresponder à estrutura de diretórios e às maiúsculas/minúsculas (observe que o diretório UTILS está todo em maiúsculas
//     como na árvore de arquivos acima)
namespace app\UTILS;

class ArrayHelperUtil {

	public function changeArrayCase(array $array) {
		// faz alguma coisa
	}
}
```

### Namespace no estilo skeleton (mesmas regras, maiúsculas/minúsculas diferentes)

```php
/**
 * app/Controller/MyController.php
 */
namespace App\Controller;

class MyController {
	// ...
}
```

A regra não mudou—apenas a escolha de maiúsculas/minúsculas de pasta/namespace do skeleton. **Qualquer que seja a capitalização das suas pastas, sua linha `namespace` deve corresponder.**

## Underscores em nomes de classes

A partir da 3.7.2, você pode usar Pascal_Snake_Case para os nomes das suas classes executando `Loader::setV2ClassLoading(false);`. 
Isso permitirá que você use underscores nos nomes das suas classes. 
Não é recomendado, mas está disponível para quem precisar.

```php
use flight\core\Loader;

/**
 * public/index.php
 */

// Adiciona um caminho ao autoloader
Flight::path(__DIR__.'/../app/controllers/');
Flight::path(__DIR__.'/../app/utils/');
Loader::setV2ClassLoading(false);

/**
 * app/controllers/My_Controller.php
 */

// nenhum namespace é necessário

class My_Controller {

	public function index() {
		// faz alguma coisa
	}
}
```

## Veja também
- [Instalação](/install) - Árvore do skeleton e padrões `App\` para novos projetos.
- [Roteamento](/learn/routing) - Como mapear rotas para controllers e renderizar views.
- [Injeção de dependência](/learn/dependency-injection-container) - Como os controllers obtêm `Engine` e serviços.
- [IA & experiência de desenvolvimento](/learn/ai) - Mantenha os agentes alinhados com sua estrutura via `AGENTS.md`.
- [Por que um framework?](/learn/why-frameworks) - Entendendo os benefícios de usar um framework como o Flight.

## Solução de problemas
- Se você não conseguir descobrir por que suas classes com namespace não estão sendo encontradas, lembre-se: com `Flight::path()`, aponte para a **raiz do projeto** (ou a base correta para seu namespace), não apenas uma pasta aninhada que você esqueceu de espelhar no namespace.
- Com o Composer PSR-4, execute `composer dump-autoload` após alterar os mapeamentos em `composer.json`.
- Em CI Linux ou produção, uma capitalização errada de pasta é uma falha muito comum de "funciona na minha máquina".

### Classe não encontrada (autoloading não funcionando)

Pode haver um ou dois motivos para isso não acontecer. Abaixo estão alguns exemplos.

#### Nome de arquivo incorreto
O mais comum é que o nome da classe não corresponda ao nome do arquivo.

Se você tem uma classe chamada `MyClass`, então o arquivo deve ser nomeado `MyClass.php`. Se você tem uma classe chamada `MyClass` e o arquivo é nomeado `myclass.php`, 
o autoloader não conseguirá encontrá-la.

#### Namespace ou capitalização de pasta incorretos
Se você estiver usando namespaces, o namespace deve corresponder à estrutura de diretórios **incluindo maiúsculas/minúsculas**.

```php
// ...código...

// se o seu MyController está em app/Controller (skeleton) e com namespace App\Controller
// isto não funcionará:
Flight::route('/hello', 'MyController->hello');

// Estilo skeleton:
use App\Controller\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);

// Layout antigo em minúsculas (somente se suas pastas forem realmente app/controllers):
use app\controllers\MyController;
Flight::route('/hello', [ MyController::class, 'hello' ]);
// ou totalmente qualificado:
Flight::route('/hello', [ 'App\Controller\MyController', 'hello' ]);
```

#### `path()` não definido (código de aplicação sem Composer)

Se você confiar em `Flight::path()` em vez do Composer para as classes da aplicação, defina o caminho antes das rotas que referenciam essas classes (geralmente no início do bootstrap / `public/index.php`):

```php
// Adiciona um caminho ao autoloader (raiz do projeto para aplicações com namespace)
Flight::path(__DIR__.'/../');
```

O skeleton oficial usa principalmente **Composer PSR-4** para `App\`, então você normalmente não precisará de `Flight::path()` para controllers e models lá.

## Changelog
- Documentação – Documentar o skeleton `App\` + pastas PascalCase e as armadilhas de sensibilidade a maiúsculas/minúsculas para humanos e ferramentas de IA.
- v3.7.2 - Você pode usar Pascal_Snake_Case para os nomes das suas classes executando `Loader::setV2ClassLoading(false);`
- v2.0 - Funcionalidade de autoload adicionada.