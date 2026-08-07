# Twig

[Twig](https://twig.symfony.com/) é um mecanismo de template flexível, rápido e seguro para PHP. É a linguagem de modelagem usada pelo Symfony e muitos outros projetos, o que significa que ferramentas de codificação com IA e a maioria dos desenvolvedores PHP já conhecem bem sua sintaxe. O Twig compila templates em PHP otimizado, escapa automaticamente a saída por padrão (ótimo para proteção contra XSS) e é fácil de estender com filtros, funções e extensões.

## Instalação

Instale com o composer.

```bash
composer require twig/twig
```

## Configuração Básica

Existem algumas opções básicas de configuração para começar. Você pode ler mais sobre elas na [Documentação do Twig](https://twig.symfony.com/doc/3.x/).

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->map('render', function(string $template, array $data): void {
	$loader = new \Twig\Loader\FilesystemLoader(Flight::get('flight.views.path'));
	$twig = new \Twig\Environment($loader, [
		// Onde o Twig armazena seus templates compilados
		'cache' => __DIR__ . '/../cache/twig',
		// Recompila templates quando a fonte muda (útil no desenvolvimento)
		'auto_reload' => true,
	]);

	echo $twig->render($template, $data);
});
```

### Registrando o Twig como a Classe de Visualização

Se preferir reutilizar um único ambiente Twig (recomendado para produção), registre-o e aponte `render` para ele:

```php
require 'vendor/autoload.php';

$app = Flight::app();

$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'auto_reload' => true,
	],
]);

$app->map('render', function(string $template, array $data): void {
	echo Flight::view()->render($template, $data);
});
```

## Exemplo Simples de Layout

Aqui está um exemplo simples de um arquivo de layout. Este é o arquivo que será usado para envolver todas as suas outras visualizações.

```html
{# app/views/layout.twig #}
<!doctype html>
<html lang="en">
	<head>
		<title>{% if title %}{{ title }} - {% endif %}My App</title>
		<link rel="stylesheet" href="style.css">
	</head>
	<body>
		<header>
			<nav>
				{# seus elementos de navegação aqui #}
			</nav>
		</header>
		<div id="content">
			{# Esta é a mágica aqui #}
			{% block content %}{% endblock %}
		</div>
		<div id="footer">
			&copy; Copyright
		</div>
	</body>
</html>
```

E agora temos seu arquivo que será renderizado dentro daquele bloco de conteúdo:

```html
{# app/views/home.twig #}
{# Isso diz ao Twig que este arquivo está "dentro" do arquivo layout.twig #}
{% extends 'layout.twig' %}

{# Este é o conteúdo que será renderizado dentro do layout no bloco de conteúdo #}
{% block content %}
	<h1>Página Inicial</h1>
	<p>Bem-vindo ao meu aplicativo!</p>
{% endblock %}
```

Então, quando você for renderizar isso dentro de sua função ou controlador, você faria algo assim:

```php
// rota simples
Flight::route('/', function () {
	Flight::render('home.twig', [
		'title' => 'Página Inicial'
	]);
});

// ou se você estiver usando um controlador
Flight::route('/', [HomeController::class, 'index']);

// HomeController.php
class HomeController
{
	public function index()
	{
		Flight::render('home.twig', [
			'title' => 'Página Inicial'
		]);
	}
}
```

Consulte a [Documentação do Twig](https://twig.symfony.com/doc/3.x/) para mais informações sobre como usar o Twig em todo o seu potencial!

## Depuração

O Twig vem com uma [Extensão de Depuração](https://twig.symfony.com/doc/3.x/functions/dump.html) que adiciona uma função `dump()` que você pode usar dentro dos templates. Ative-a apenas no desenvolvimento:

```php
$app->register('view', \Twig\Environment::class, [
	new \Twig\Loader\FilesystemLoader($app->get('flight.views.path')),
	[
		'cache' => __DIR__ . '/../cache/twig',
		'debug' => true, // necessário para a função dump()
		'auto_reload' => true,
	],
], function (\Twig\Environment $twig): void {
	$twig->addExtension(new \Twig\Extension\DebugExtension());
});
```

Então em um template:

```html
{{ dump(user) }}
```

Você também pode combinar o Twig com [Tracy](/awesome-plugins/tracy) para depuração em nível de PHP. Para métricas em nível de template (tempo de renderização, memória, quais templates/blocos foram executados), use o **painel Twig** opcional em [flightphp/tracy-extensions](/awesome-plugins/tracy-extensions): passe um `Twig\Profiler\Profile` como `twig_profile` para `TracyExtensionLoader`. O `TwigTracyExtension` opcional expõe `{{ dump() }}` / `{{ bdump() }}` / `{{ dumpe() }}` em templates quando o Tracy está ativo.

## Nota de Segurança

O Twig escapa automaticamente a saída por padrão, o que ajuda a proteger contra ataques XSS. Prefira `{{ variable }}` para texto. Use o filtro `|raw` apenas quando confiar intencionalmente no conteúdo HTML (por exemplo, markdown sanitizado que você já processou no lado do servidor).