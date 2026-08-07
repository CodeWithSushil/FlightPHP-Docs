# Aprenda Sobre o Flight

O Flight é um framework rápido, simples e extensível para PHP. É bastante versátil e pode ser usado para construir qualquer tipo de aplicação web.
Ele é construído tendo a simplicidade em mente e é escrito de uma forma fácil de entender e usar — por humanos e por [assistentes de codificação por IA](/learn/ai).

> **Nota:** Você verá exemplos que usam `Flight::` como uma variável estática e alguns que usam o objeto Engine `$app->`. Ambos funcionam de forma intercambiável. `$app` e `$this->app` em um controlador/middleware são a abordagem recomendada pela equipe do Flight (e o que o esqueleto oficial + `AGENTS.md` padronizam para novos projetos).

## Componentes Principais

### [Roteamento](/learn/routing)

Aprenda como gerenciar rotas para sua aplicação web. Isso também inclui agrupamento de rotas, parâmetros de rota e middleware.

### [Middleware](/learn/middleware)

Aprenda como usar middleware para filtrar requisições e respostas na sua aplicação.

### [Carregamento Automático](/learn/autoloading)

Aprenda como carregar suas próprias classes automaticamente. A **capitalização** das pastas deve corresponder aos seus namespaces — o esqueleto usa `App\` e pastas PascalCase como `app/Controller/`.

### [Requisições](/learn/requests)

Aprenda como lidar com requisições e respostas na sua aplicação.

### [Respostas](/learn/responses)

Aprenda como enviar respostas aos seus usuários.

### [Templates HTML](/learn/templates)

Aprenda como renderizar HTML com Twig (padrão do esqueleto), Latte ou outros motores — não apenas as views PHP integradas.

### [Segurança](/learn/security)

Aprenda como proteger sua aplicação contra ameaças de segurança comuns.

### [Configuração](/learn/configuration)

Aprenda como configurar o framework para sua aplicação.

### [Gerenciador de Eventos](/learn/events)

Aprenda como usar o sistema de eventos para adicionar eventos personalizados à sua aplicação.

### [Estendendo o Flight](/learn/extending)

Aprenda como estender o framework adicionando seus próprios métodos e classes.

### [Ganchos de Método e Filtragem](/learn/filtering)

Aprenda como adicionar ganchos de evento aos seus métodos e aos métodos internos do framework.

### [Container de Injeção de Dependência (DIC)](/learn/dependency-injection-container)

Aprenda como usar containers de injeção de dependência (DIC) para gerenciar as dependências da sua aplicação.

## Classes Utilitárias

### [Coleções](/learn/collections)

As coleções são usadas para armazenar dados e serem acessíveis como um array ou como um objeto para facilitar o uso.

### [Wrapper JSON](/learn/json)

Este tem algumas funções simples para tornar a codificação e decodificação do seu JSON consistente.

### [SimplePdo](/learn/simple-pdo)

O PDO às vezes pode adicionar mais dor de cabeça do que o necessário. O SimplePdo é uma classe auxiliar moderna de PDO com métodos convenientes como `insert()`, `update()`, `delete()` e `transaction()` para tornar as operações de banco de dados muito mais fáceis.

### [PdoWrapper](/learn/pdo-wrapper) (Obsoleto)

O wrapper PDO original está obsoleto a partir da v3.18.0. Use [SimplePdo](/learn/simple-pdo) em seu lugar.

### [Manipulador de Arquivo Enviado](/learn/uploaded-file)

Uma classe simples para ajudar a gerenciar arquivos enviados e movê-los para um local permanente.

## Conceitos Importantes

### [Por que um Framework?](/learn/why-frameworks)

Aqui está um pequeno artigo sobre por que você deve usar um framework. É uma boa ideia entender os benefícios de usar um framework antes de começar a usar um.

Além disso, um excelente tutorial foi criado por [@lubiana](https://git.php.fail/lubiana). Embora não entre em grandes detalhes especificamente sobre o Flight,
este guia ajudará você a entender alguns dos principais conceitos em torno de um framework e por que eles são benéficos de usar.
Você pode encontrar o tutorial [aqui](https://git.php.fail/lubiana/no-framework-tutorial/src/branch/master/README.md).

### [Flight Comparado a Outros Frameworks](/learn/flight-vs-another-framework)

Se você está migrando de outro framework como Laravel, Slim, Fat-Free ou Symfony para o Flight, esta página ajudará você a entender as diferenças entre os dois.

## Outros Tópicos

### [Testes Unitários](/learn/unit-testing)

Siga este guia para aprender como testar unitariamente seu código Flight para que ele seja extremamente sólido.

### [IA e Experiência do Desenvolvedor](/learn/ai)

O Flight é construído para trabalhar com LLMs de codificação: `AGENTS.md`, comandos Runway `ai:*` e um layout de esqueleto claro para que os agentes mantenham o padrão.

### [Migrando da v2 para v3](/learn/migrating-to-v3)

A compatibilidade com versões anteriores foi, na maior parte, mantida, mas há algumas alterações das quais você deve estar ciente ao migrar da v2 para a v3.