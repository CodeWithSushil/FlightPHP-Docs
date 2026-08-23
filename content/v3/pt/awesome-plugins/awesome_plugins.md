# Plugins Incríveis

Flight é incrivelmente extensível. Existem vários plugins que podem ser usados para adicionar funcionalidades à sua aplicação Flight. Alguns são oficialmente suportados pela Equipe Flight e outros são bibliotecas micro/lite para ajudar você a começar.

## Ferramentas de IA

Flight pode ficar ainda mais legal com plugins alimentados por IA.

- [Flight MCP](/awesome-plugins/mcp) - Um plugin para integrar MCP (Model Control Protocol) com Flight, permitindo funcionalidades perfeitas alimentadas por IA. Focado principalmente nas páginas de documentação, ajuda a reduzir custos de tokens fornecendo as informações mais atualizadas sobre seus projetos Flight.

## Documentação de API

A documentação de API é crucial para qualquer API. Ela ajuda os desenvolvedores a entenderem como interagir com sua API e o que esperar em retorno. Existem algumas ferramentas disponíveis para ajudar você a gerar documentação de API para seus Projetos Flight.

- [FlightPHP OpenAPI Generator](https://dev.to/danielsc/define-generate-and-implement-an-api-first-approach-with-openapi-generator-and-flightphp-1fb3) - Post de blog escrito por Daniel Schreiber sobre como usar a Especificação OpenAPI com FlightPHP para construir sua API usando uma abordagem API-first.
- [SwaggerUI](https://github.com/zircote/swagger-php) - Swagger UI é uma ótima ferramenta para ajudar você a gerar documentação de API para seus projetos Flight. É muito fácil de usar e pode ser personalizado para atender suas necessidades. Esta é a biblioteca PHP para ajudar você a gerar a documentação Swagger.

## Monitoramento de Performance de Aplicação (APM)

O Monitoramento de Performance de Aplicação (APM) é crucial para qualquer aplicação. Ele ajuda você a entender como sua aplicação está performando e onde estão os gargalos. Existem várias ferramentas APM que podem ser usadas com Flight.
- <span class="badge bg-primary">official</span> [flightphp/apm](/awesome-plugins/apm) - Flight APM é uma biblioteca APM simples que pode ser usada para monitorar suas aplicações Flight. Pode ser usada para monitorar a performance da sua aplicação e ajudar você a identificar gargalos.

## Assíncrono

Flight já é um framework rápido, mas colocar um motor turbo nele torna tudo mais divertido (e desafiador)!

- [flightphp/async](/awesome-plugins/async) - Biblioteca Async oficial do Flight. Esta biblioteca é uma forma simples de adicionar processamento assíncrono à sua aplicação. Ela usa Swoole/Openswoole por baixo dos panos para fornecer uma forma simples e eficaz de executar tarefas de forma assíncrona.

## Autorização/Permissões

Autorização e Permissões são cruciais para qualquer aplicação que requer controles para quem pode acessar o quê.

- <span class="badge bg-primary">official</span> [flightphp/permissions](/awesome-plugins/permissions) - Biblioteca de Permissões oficial do Flight. Esta biblioteca é uma forma simples de adicionar permissões de usuário e aplicação à sua aplicação. 

## Autenticação

A autenticação é essencial para aplicações que precisam verificar a identidade do usuário e proteger endpoints de API.

- [firebase/php-jwt](/awesome-plugins/jwt) - Biblioteca JSON Web Token (JWT) para PHP. Uma forma simples e segura de implementar autenticação baseada em token em suas aplicações Flight. Perfeita para autenticação de API sem estado, proteger rotas com middleware e implementar fluxos de autorização estilo OAuth.

## Cache

O cache é uma ótima forma de acelerar sua aplicação. Existem várias bibliotecas de cache que podem ser usadas com Flight.

- <span class="badge bg-primary">official</span> [flightphp/cache](/awesome-plugins/php-file-cache) - Classe de cache em arquivo PHP leve, simples e independente

## CLI

Aplicações CLI são uma ótima forma de interagir com sua aplicação. Você pode usá-las para gerar controllers, exibir todas as rotas e muito mais.

- <span class="badge bg-primary">official</span> [flightphp/runway](/awesome-plugins/runway) - Runway é uma aplicação CLI que ajuda você a gerenciar suas aplicações Flight.

## Cookies

Cookies são uma ótima forma de armazenar pequenos pedaços de dados no lado do cliente. Eles podem ser usados para armazenar preferências do usuário, configurações da aplicação e mais.

- [overclokk/cookie](/awesome-plugins/php-cookie) - PHP Cookie é uma biblioteca PHP que fornece uma forma simples e eficaz de gerenciar cookies.

## Depuração

A depuração é crucial quando você está desenvolvendo em seu ambiente local. Existem alguns plugins que podem elevar sua experiência de depuração.

- [tracy/tracy](/awesome-plugins/tracy) - Este é um manipulador de erros completo que pode ser usado com Flight. Tem vários painéis que podem ajudar você a depurar sua aplicação. Também é muito fácil de estender e adicionar seus próprios painéis.
- <span class="badge bg-primary">official</span> [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions) - Usado com o manipulador de erros [Tracy](/awesome-plugins/tracy), este plugin adiciona alguns painéis extras para ajudar na depuração especificamente para projetos Flight.

## Bancos de Dados

Bancos de dados são o núcleo da maioria das aplicações. É assim que você armazena e recupera dados. Algumas bibliotecas de banco de dados são simplesmente wrappers para escrever consultas e algumas são ORMs completos.

- <span class="badge bg-primary">official</span> [flightphp/core SimplePdo](/learn/simple-pdo) - Helper PDO oficial do Flight que faz parte do núcleo. Este é um wrapper moderno com métodos auxiliares convenientes como `insert()`, `update()`, `delete()` e `transaction()` para simplificar operações de banco de dados. Todos os resultados são retornados como Collections para acesso flexível em array/objeto. Não é um ORM, apenas uma forma melhor de trabalhar com PDO.
- <span class="badge bg-warning">deprecated</span> [flightphp/core PdoWrapper](/learn/pdo-wrapper) - Wrapper PDO oficial do Flight que faz parte do núcleo (obsoleto a partir da v3.18.0). Use SimplePdo em vez disso.
- <span class="badge bg-primary">official</span> [flightphp/active-record](/awesome-plugins/active-record) - ORM/Mapper ActiveRecord oficial do Flight. Uma ótima biblioteca pequena para recuperar e armazenar dados facilmente em seu banco de dados.
- [byjg/php-migration](/awesome-plugins/migrations) - Plugin para acompanhar todas as mudanças de banco de dados do seu projeto.
- [knifelemon/easy-query](/awesome-plugins/easy-query) - Construtor de consultas SQL fluente e leve que gera SQL e parâmetros para prepared statements. Funciona bem com [SimplePdo](/learn/simple-pdo).

## Criptografia

A criptografia é crucial para qualquer aplicação que armazena dados sensíveis. Criptografar e descriptografar os dados não é muito difícil, mas armazenar corretamente a chave de criptografia [pode](https://stackoverflow.com/questions/6767839/where-should-i-store-an-encryption-key-for-php#:~:text=Write%20a%20php%20config%20file%20and%20store%20it,folder%20is%20not%20accessible%20to%20the%20end%20user.) [ser](https://www.reddit.com/r/PHP/comments/luqsn/the_encryption_key_where_do_you_store_it/) [difícil](https://security.stackexchange.com/questions/48047/location-to-store-an-encryption-key). A coisa mais importante é nunca armazenar sua chave de criptografia em um diretório público ou commitá-la em seu repositório de código.

- [defuse/php-encryption](/awesome-plugins/php-encryption) - Esta é uma biblioteca que pode ser usada para criptografar e descriptografar dados. Começar é bastante simples para começar a criptografar e descriptografar dados.

## E-mail

Enviar e-mail é uma necessidade central para a maioria das aplicações web - mensagens de boas-vindas, redefinição de senha, notificações. Essas bibliotecas tornam isso simples sem perder a confiabilidade da entrega.

- [ryanstubbs/flightmail](/awesome-plugins/flightmail) - FlightMail envolve o Symfony Mailer com uma API fluente e amigável ao Flight. Envie por SMTP ou qualquer provedor importante via strings DSN simples, roteie provedores diferentes por mensagem e renderize corpos com templates Twig ou Latte. Este é um plugin não oficial para Flight e não é mantido pela equipe do Flight.

## Fila de Jobs

Filas de jobs são realmente úteis para processar tarefas de forma assíncrona. Isso pode ser enviar emails, processar imagens ou qualquer coisa que não precise ser feita em tempo real.

- [n0nag0n/simple-job-queue](/awesome-plugins/simple-job-queue) - Simple Job Queue é uma biblioteca que pode ser usada para processar jobs de forma assíncrona. Pode ser usada com beanstalkd, MySQL/MariaDB, SQLite e PostgreSQL.

## Sessão

Sessões não são realmente úteis para APIs, mas para construir uma aplicação web, sessões podem ser cruciais para manter estado e informações de login.

- <span class="badge bg-primary">official</span> [flightphp/session](/awesome-plugins/session) - Biblioteca de Sessão oficial do Flight. Esta é uma biblioteca de sessão simples que pode ser usada para armazenar e recuperar dados de sessão. Ela usa o gerenciamento de sessão integrado do PHP.
- [Ghostff/Session](/awesome-plugins/ghost-session) - Gerenciador de Sessão PHP (não-bloqueante, flash, segment, criptografia de sessão). Usa PHP open_ssl para criptografia/descriptografia opcional de dados de sessão.

## Templates

Templates são essenciais para qualquer aplicação web com interface. Existem vários motores de template que podem ser usados com Flight.

- <span class="badge bg-warning">deprecated</span> [flightphp/core View](/learn#views) - Este é um motor de template muito básico que faz parte do núcleo. Não é recomendado usá-lo se você tiver mais de algumas páginas no seu projeto.
- [latte/latte](/awesome-plugins/latte) - Latte é um motor de template completo que é muito fácil de usar e parece mais próximo da sintaxe PHP que Twig ou Smarty. Também é muito fácil de estender e adicionar seus próprios filtros e funções.
- [twig/twig](/awesome-plugins/twig) - Twig é um motor de template flexível, rápido e seguro (o mesmo usado pelo Symfony). Ferramentas de IA e muitos desenvolvedores PHP o conhecem bem, ele escapa automaticamente a saída por padrão e tem um enorme ecossistema de extensões.
- [knifelemon/comment-template](/awesome-plugins/comment-template) - CommentTemplate é um poderoso motor de template PHP com compilação de assets, herança de templates e processamento de variáveis. Possui minificação automática de CSS/JS, cache, codificação Base64 e integração opcional com o framework Flight PHP.

## Integração com WordPress

Quer usar Flight no seu projeto WordPress? Tem um plugin prático para isso!

- [n0nag0n/wordpress-integration-for-flight-framework](/awesome-plugins/n0nag0n_wordpress) - Este plugin WordPress permite que você execute Flight junto com o WordPress. É perfeito para adicionar APIs personalizadas, microsserviços ou até mesmo aplicações completas ao seu site WordPress usando o framework Flight. Super útil se você quer o melhor dos dois mundos!

## Contribuindo

Tem um plugin que gostaria de compartilhar? Envie um pull request para adicioná-lo à lista!