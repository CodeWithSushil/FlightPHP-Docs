# Flight PHP Framework

Flight é um framework rápido, simples e extensível para PHP—construído para desenvolvedores que querem realizar tarefas rapidamente, sem complicações. Seja você construindo um aplicativo web clássico, uma API ultrarrápida, ou trabalhando com assistentes de codificação de IA, o baixo consumo de recursos e o design direto do Flight o tornam uma escolha perfeita. O Flight foi projetado para ser leve, mas também pode atender aos requisitos de arquitetura empresarial.

## Por que Escolher o Flight?

- **Amigável para Iniciantes:** O Flight é um ótimo ponto de partida para novos desenvolvedores PHP. Sua estrutura clara e sintaxe simples ajudam você a aprender desenvolvimento web sem se perder em código repetitivo.
- **Amado pelos Profissionais:** Desenvolvedores experientes adoram o Flight pela sua flexibilidade e controle. Você pode escalar de um pequeno protótipo para um aplicativo completo sem precisar trocar de frameworks.
- **Compatibilidade Retroativa:** Valorizamos seu tempo. O Flight v3 é uma evolução do v2, mantendo quase toda a mesma API. Acreditamos em evolução, não em revolução—sem mais "quebrar tudo" toda vez que uma versão principal é lançada.
- **Zero Dependências:** O núcleo do Flight é completamente livre de dependências—sem polyfills, sem pacotes externos, nem mesmo interfaces PSR. Isso significa menos vetores de ataque, um footprint menor e sem alterações inesperadas vindas de dependências de terceiros. Plugins opcionais podem incluir dependências, mas o núcleo sempre permanecerá leve e seguro.
- **Amigável para IA:** A pequena superfície de API do Flight e o [esqueleto oficial](https://github.com/flightphp/skeleton) (um layout, `AGENTS.md`, injeção de construtor) facilitam para ferramentas de codificação de IA permanecerem no padrão. Mesma base de código, independentemente de você digitar cada linha ou trabalhar com um agente. [Saiba mais sobre usar IA com Flight](/learn/ai).

## Visão Geral em Vídeo

<div class="flight-block-video">
  <div class="row">
    <div class="col-12 col-md-6 position-relative video-wrapper">
      <iframe class="video-bg" width="100vw" height="315" src="https://www.youtube.com/embed/VCztp1QLC2c?si=W3fSWEKmoCIlC7Z5" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
    </div>
    <div class="col-12 col-md-6 fs-5 text-center mt-5 pt-5">
      <span class="flight-title-video">Simples o suficiente, certo?</span>
      <br>
      <a href="https://docs.flightphp.com/learn">Saiba mais</a> sobre o Flight na documentação!
    </div>
  </div>
</div>

## Início Rápido

Para fazer uma instalação básica rápida, instale-o com o Composer:

```bash
composer require flightphp/core
```

Ou você pode baixar um zip do repositório [aqui](https://github.com/flightphp/core). Então você terá um arquivo `index.php` básico como o seguinte:

```php
<?php

// se instalado com composer
require 'vendor/autoload.php';
// ou se instalado manualmente por arquivo zip
// require 'flight/Flight.php';

Flight::route('/', function() {
  echo 'hello world!';
});

Flight::route('/json', function() {
  Flight::json([
	'hello' => 'world'
  ]);
});

Flight::start();
```

Isso é tudo! Você tem uma aplicação Flight básica. Agora você pode executar este arquivo com `php -S localhost:8000` e visitar `http://localhost:8000` no seu navegador para ver o resultado.

Exemplos curtos como `Flight::` são ótimos para aprendizado e micro aplicativos. Para um layout de projeto completo que humanos e ferramentas de IA compartilham, use o esqueleto abaixo.

## Aplicativo Esqueleto/Boilerplate

Existe um inicializador oficial para ajudar você a começar qualquer novo projeto Flight. Ele configura estrutura, configuração, scripts do Composer e instruções amigáveis para IA desde o início.

Confira [flightphp/skeleton](https://github.com/flightphp/skeleton) para um projeto pronto para uso, ou visite a página de [exemplos](examples) para inspiração. Quer os detalhes do fluxo de trabalho com IA? [Explore IA e experiência do desenvolvedor](/learn/ai).

O que você obtém (nível alto):

- **Namespaces `App\`** com pastas em PascalCase (`app/Controller/`, `app/Middleware/`, `app/Model/`, …)—a **capitalização** da pasta deve corresponder ao namespace (veja [Autoloading](/learn/autoloading))
- Injeção de **Dice + `Engine`** para que os controladores permaneçam testáveis (prefira `$this->app` ao invés de `Flight::` no código do aplicativo)
- Visualizações **Twig**, **SimplePdo** + exemplo de ActiveRecord, **migrate** do Runway
- **`AGENTS.md`** na raiz (mais cópias com escopo) e **`SECURITY.md`** para assistentes e política de segurança

## Instalando o Aplicativo Esqueleto

Simples o suficiente!

```bash
# Criar o novo projeto
composer create-project flightphp/skeleton my-project/
# Entrar no diretório do novo projeto
cd my-project/
# Iniciar o servidor de desenvolvimento local para começar imediatamente!
composer start
```

Ele cria a estrutura do projeto, copia `config_sample.php` → `config.php` (e `.env.example` → `.env` quando presente), e você está pronto para começar. Dados de amostra opcionais:

```bash
php runway migrate
# então visite /posts e /api/posts
```

## Alto Desempenho

O Flight é um dos frameworks PHP mais rápidos disponíveis. Seu núcleo leve significa menos sobrecarga e mais velocidade—perfeito tanto para aplicativos tradicionais quanto para fluxos de trabalho modernos assistidos por IA. Você pode ver todos os benchmarks em [TechEmpower](https://www.techempower.com/benchmarks/#section=data-r18&hw=ph&test=frameworks)

Veja o benchmark abaixo com alguns outros frameworks PHP populares.

| Framework | Requisições/sec Texto Simples | Requisições/sec JSON |
| --------- | ------------ | ------------ |
| Flight      | 190,421    | 182,491 |
| Yii         | 145,749    | 131,434 |
| Fat-Free    | 139,238    | 133,952 |
| Slim        | 89,588     | 87,348  |
| Phalcon     | 95,911     | 87,675  |
| Symfony     | 65,053     | 63,237  |
| Lumen       | 40,572     | 39,700  |
| Laravel     | 26,657     | 26,901  |
| CodeIgniter | 20,628     | 19,901  |


## Flight e IA

Curioso sobre como o Flight funciona com LLMs de codificação? [Descubra](/learn/ai) como `AGENTS.md`, comandos `ai:*` do Runway e o layout do esqueleto mantêm os assistentes no caminho certo.

## Estabilidade e Compatibilidade Retroativa

Valorizamos seu tempo. Já vimos frameworks que se reinventam completamente a cada poucos anos, deixando desenvolvedores com código quebrado e migrações caras. O Flight é diferente. O Flight v3 foi projetado como uma evolução do v2, o que significa que a API que você conhece e ama não foi removida. Na verdade, a maioria dos projetos v2 funcionará sem nenhuma alteração no v3.

Estamos comprometidos em manter o Flight estável para que você possa focar na construção do seu aplicativo, não na correção do seu framework. O esqueleto pode ser opinativo para projetos *novos*; as APIs principais permanecem familiares para todos os outros.

# Comunidade

Estamos no Matrix Chat

[![Matrix](https://img.shields.io/matrix/flight-php-framework%3Amatrix.org?server_fqdn=matrix.org&style=social&logo=matrix)](https://matrix.to/#/#flight-php-framework:matrix.org)

E no Discord

[![](https://dcbadge.limes.pink/api/server/https://discord.gg/Ysr4zqHfbX)](https://discord.gg/Ysr4zqHfbX)

# Contribuindo

Existem duas maneiras de você contribuir com o Flight:

1. Contribua com o framework principal visitando o [repositório principal](https://github.com/flightphp/core).
2. Ajude a melhorar a documentação! Este site de documentação está hospedado no [Github](https://github.com/flightphp/docs). Se você encontrar um erro ou quiser melhorar algo, sinta-se à vontade para enviar um pull request. Adoramos atualizações e novas ideias—especialmente relacionadas a IA e novas tecnologias!

# Requisitos

O Flight requer PHP 7.4 ou superior.

**Nota:** O PHP 7.4 é suportado porque no momento atual da escrita (2024) o PHP 7.4 é a versão padrão para algumas distribuições Linux LTS. Forçar uma migração para PHP >8 causaria muitos problemas para esses usuários. O framework também suporta PHP >8.

# Licença

O Flight é lançado sob a licença [MIT](https://github.com/flightphp/core/blob/master/LICENSE).