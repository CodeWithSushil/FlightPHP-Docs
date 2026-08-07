# Runway

Runway é uma aplicação CLI que ajuda você a gerenciar suas aplicações Flight. Ele pode gerar controllers, exibir todas as rotas, executar assistentes de configuração de IA, migrations (no skeleton) e mais. É baseado na excelente biblioteca [adhocore/php-cli](https://github.com/adhocore/php-cli).

Clique [aqui](https://github.com/flightphp/runway) para ver o código.

Os comandos de scaffolding estão intencionalmente alinhados com o [skeleton oficial](https://github.com/flightphp/skeleton) para que [ferramentas de codificação de IA](/learn/ai) e humanos obtenham os mesmos caminhos, namespaces e estilo de injeção de construtor toda vez.

## Instalação

Instale com composer.

```bash
composer require flightphp/runway
```

O skeleton já depende do Runway; use `php runway` a partir da raiz do projeto.

## Configuração Básica

Na primeira vez que você executar o Runway, ele tentará encontrar uma configuração `runway` em `app/config/config.php` através da chave `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// opcional; o skeleton também usa index_root para a entrada pública
		'index_root' => 'public/index.php',
    ],
];
```

> **NOTA** - A partir da **v1.2.0**, `.runway-config.json` está obsoleto em favor de `app/config/config.php`. Migre com `php runway config:migrate` ao atualizar projetos antigos. O skeleton ainda pode escrever um pequeno `.runway-config.json` em create-project para compatibilidade; prefira a chave `runway` em `config.php` a partir de agora.

### Detecção da Raiz do Projeto

O Runway é inteligente o suficiente para detectar a raiz do seu projeto, mesmo que você o execute a partir de um subdiretório. Ele procura indicadores como `composer.json`, `.git` ou `app/config/config.php` para determinar onde está a raiz do projeto. Isso significa que você pode executar comandos do Runway de qualquer lugar do seu projeto!

## Uso

O Runway tem vários comandos que você pode usar para gerenciar sua aplicação Flight. Existem duas maneiras fáceis de usar o Runway.

1. Se você estiver usando o projeto skeleton, pode executar `php runway [comando]` a partir da raiz do seu projeto.
1. Se você estiver usando o Runway como um pacote instalado via composer, pode executar `vendor/bin/runway [comando]` a partir da raiz do seu projeto.

### Lista de Comandos

Você pode ver uma lista de todos os comandos disponíveis executando o comando `php runway`.

```bash
php runway
```

Confie apenas nos comandos que realmente aparecem nessa lista para sua instalação (comandos principais do Runway vs comandos específicos do projeto como o `migrate` do skeleton).

### Ajuda do Comando

Para qualquer comando, você pode passar a flag `--help` para obter mais informações sobre como usar o comando.

```bash
php runway routes --help
php runway make:controller --help
```

Aqui estão alguns exemplos:

### Gerar um Controller

`make:controller` cria um scaffold de controller que corresponde ao layout do skeleton oficial:

| | |
|--|--|
| **Caminho** | `app/Controller/{Nome}.php` |
| **Namespace** | `App\Controller` |
| **Estilo** | Injeção de construtor de `flight\Engine` (sem `Flight::` no corpo da classe) |

```bash
php runway make:controller MeuController
# → app/Controller/MeuController.php
#   namespace App\Controller;
```

Exemplo do formato que você deve esperar (simplificado):

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MeuController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// ex. $this->app->render('…', […]);
	}
}
```

Registre-o com um callable de classe para que o Dice possa construir o controller:

```php
// app/config/routes.php
use App\Controller\MeuController;

$router->get('/meu', [MeuController::class, 'index']);
```

**Por que esse layout?** A **capitalização** da pasta deve corresponder ao namespace (`Controller` não `controllers`) para o Composer PSR-4 no Linux—veja [Autoloading](/learn/autoloading). O mesmo caminho é o que os arquivos `AGENTS.md` raiz e com escopo dizem para as ferramentas de IA usarem, então controllers gerados e escritos manualmente permanecem idênticos.

> Documentações antigas e projetos comunitários às vezes usavam `app/controllers/` e `app\controllers`. Isso permanece válido se *sua* árvore ainda usar pastas em minúsculas. **Novos projetos skeleton e a saída atual do `make:controller` usam `app/Controller/` + `App\Controller`.**

### Gerar um Modelo Active Record

Primeiro certifique-se de que você instalou o plugin [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record usuarios
```

No skeleton oficial, os modelos vivem em **`app/Model/`** com namespace **`App\Model`**, e a conexão com o banco de dados é **[SimplePdo](/learn/simple-pdo)** (injete-o ou passe-o para o construtor do ActiveRecord). Nomes de arquivos/namespaces gerados seguem os padrões atuais do Runway e sua configuração `runway`—prefira alinhar novos modelos com `App\Model` para que correspondam ao [autoloading](/learn/autoloading) e `AGENTS.md`.

Exemplo de um modelo consistente com a demonstração de posts do skeleton:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $titulo
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Se um gerador antigo ainda emitir `app/records` / `app\records`, você pode manter essa convenção em aplicações legadas ou mover arquivos para `app/Model/` e atualizar o namespace para corresponder à capitalização da pasta.

### Migrations (skeleton)

O skeleton oficial inclui um comando do projeto (descoberto de `app/commands/`) como:

```bash
php runway migrate
```

As migrations são arquivos SQL em `migrations/` (por exemplo `YYYYMMDDHHMMSS_descricao.sql` para SQLite e `…_descricao.mysql.sql` para MySQL), selecionados a partir da sua configuração/env do driver de banco de dados. Flags e comportamentos exatos são definidos por esse comando do projeto—execute `php runway migrate --help` na sua aplicação.

### Auxiliares de IA

O Runway expõe comandos orientados para IA usados com [IA e experiência do desenvolvedor](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Estes armazenam credenciais LLM e geram instruções do projeto (principalmente **`AGENTS.md`**). No skeleton, trate `AGENTS.md` (e cópias com escopo em `app/`) mais **`SECURITY.md`** como a fonte da verdade para agentes.

### Exibir Todas as Rotas

Isso exibirá todas as rotas que estão atualmente registradas com o Flight.

```bash
php runway routes
```

Se você quiser visualizar apenas rotas específicas, pode passar uma flag para filtrar as rotas.

```bash
# Exibir apenas rotas GET
php runway routes --get

# Exibir apenas rotas POST
php runway routes --post

# etc.
```

## Adicionando Comandos Personalizados ao Runway

Se você estiver criando um pacote para o Flight, ou quiser adicionar seus próprios comandos personalizados ao seu projeto, pode fazer isso criando um diretório `src/commands/`, `flight/commands/`, `app/commands/` ou `commands/` para seu projeto/pacote. Se você precisar de mais personalização, veja a seção abaixo sobre Configuração.

No skeleton, os comandos do projeto vivem em **`app/commands/`** com namespace **`App\Command`**. O Runway os descobre por caminho; mantenha essa pasta em sincronia com o classmap/PSR-4 do Composer como seu projeto já faz.

Para criar um comando, você simplesmente estende a classe `AbstractBaseCommand` e implementa pelo menos um método `__construct` e um método `execute`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ComandoExemplo extends AbstractBaseCommand
{
	/**
     * Construtor
     *
     * @param array<string,mixed> $config Config de app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:exemplo', 'Cria um exemplo para a documentação', $config);
        $this->argument('<gif-engraçado>', 'O nome do gif engraçado');
    }

	/**
     * Executa a função
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Criando exemplo...');

		// Faça algo aqui

		$io->ok('Exemplo criado!');
	}
}
```

Veja a [Documentação adhocore/php-cli](https://github.com/adhocore/php-cli) para mais informações sobre como construir seus próprios comandos personalizados em sua aplicação Flight!

## Gerenciamento de Configuração

Como a configuração foi movida para `app/config/config.php` a partir da `v1.2.0`, existem alguns comandos auxiliares para gerenciar a configuração.

> **Dica do skeleton:** Mantenha `config.php` como valores PHP **literais**. Segredos pertencem ao `.env`. Evite expressões `$_ENV[...]` dentro de `config.php`—`config:set` reescreve esse arquivo como dados estáticos e pode incorporar segredos no arquivo. Veja [Configuração](/learn/configuration).

### Migrar Configuração Antiga

Se você tiver um arquivo `.runway-config.json` antigo, pode facilmente migrá-lo para `app/config/config.php` com o seguinte comando:

```bash
php runway config:migrate
```

### Definir Valor de Configuração

Você pode definir um valor de configuração usando o comando `config:set`. Isso é útil se você quiser atualizar um valor de configuração sem abrir o arquivo.

```bash
php runway config:set app_root "app/"
```

### Obter Valor de Configuração

Você pode obter um valor de configuração usando o comando `config:get`.

```bash
php runway config:get app_root
```

## Todas as Configurações do Runway

Se você precisar personalizar a configuração para o Runway, pode definir esses valores em `app/config/config.php`. Abaixo estão algumas configurações adicionais que você pode definir:

```php
<?php
// app/config/config.php
return [
    // ... outros valores de configuração ...

    'runway' => [
        // Este é o local do diretório da sua aplicação
        'app_root' => 'app/',

        // Este é o diretório onde seu arquivo index raiz está localizado
        'index_root' => 'public/',

        // Estes são os caminhos para as raízes de outros projetos
        'root_paths' => [
            '/home/usuario/projeto-diferente',
            '/var/www/outro-projeto'
        ],

        // Caminhos base provavelmente não precisam ser configurados, mas estão aqui se você quiser
        'base_paths' => [
            '/includes/libs/vendor', // se você tiver um caminho muito único para seu diretório vendor ou algo assim
        ],

        // Caminhos finais são locais dentro de um projeto para procurar os arquivos de comando
        'final_paths' => [
            'src/caminho-diferente/commands',
            'app/module/admin/commands',
        ],

        // Se você quiser apenas adicionar o caminho completo, vá em frente (absoluto ou relativo à raiz do projeto)
        'paths' => [
            '/home/usuario/projeto-diferente/src/caminho-diferente/commands',
            '/var/www/outro-projeto/app/module/admin/commands',
            'app/meus-comandos-unicos'
        ]
    ]
];
```

### Acessando a Configuração

Se você precisar acessar os valores de configuração efetivamente, pode acessá-los através do método `__construct` ou do método `app()`. Também é importante notar que se você tiver um arquivo `app/config/services.php`, esses serviços também estarão disponíveis para seu comando.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Acessar configuração
    $app_root = $this->config['runway']['app_root'];
    
    // Acessar serviços como talvez uma conexão de banco de dados
    $database = $this->config['database']
    
    // ...
}
```

## Wrappers Auxiliares de IA

O Runway tem alguns wrappers auxiliares que facilitam para a IA gerar comandos. Você pode usar `addOption` e `addArgument` de uma forma que parece similar ao Symfony Console. Isso é útil se você estiver usando ferramentas de IA para gerar seus comandos.

```php
public function __construct(array $config)
{
    parent::__construct('make:exemplo', 'Cria um exemplo para a documentação', $config);
    
    // O argumento mode é anulável e padrão para completamente opcional
    $this->addOption('nome', 'O nome do exemplo', null);
}
```

## Veja Também

- [Instalação](/install) - Árvore do skeleton e padrões create-project
- [Autoloading](/learn/autoloading) - `App\` e capitalização de pastas
- [Injeção de Dependência](/learn/dependency-injection-container) - Injeção Dice + Engine para controllers gerados
- [IA e Experiência do Desenvolvedor](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Modelos usados com `make:record` / skeleton `App\Model`
- [SimplePdo](/learn/simple-pdo) - Conexão DB usada por migrations e modelos do skeleton