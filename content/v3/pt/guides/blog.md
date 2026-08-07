# Construindo um Blog Simples com Flight PHP

Este guia mostra como criar um blog básico usando o framework Flight PHP. Você configurará um projeto, definirá rotas, gerenciará posts com JSON e os renderizará com o mecanismo de templates Latte — tudo demonstrando a simplicidade e flexibilidade do Flight. Ao final, você terá um blog funcional com uma página inicial, páginas individuais de posts e um formulário de criação.

## Pré-requisitos
- **PHP 7.4+**: Instalado no seu sistema.
- **Composer**: Para gerenciamento de dependências.
- **Editor de Texto**: Qualquer editor como VS Code ou PHPStorm.
- Conhecimento básico de PHP e desenvolvimento web.

## Passo 1: Configure Seu Projeto

Comece criando um novo diretório de projeto e instalando o Flight via Composer.

1. **Crie um Diretório**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Instale o Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Crie um Diretório Público**:
   O Flight usa um único ponto de entrada (`index.php`). Crie uma pasta `public/` para ele:
   ```bash
   mkdir public
   ```

4. **`index.php` Básico**:
   Crie `public/index.php` com uma rota simples de “hello world”:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Execute o Servidor Embutido**:
   Teste sua configuração com o servidor de desenvolvimento do PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Visite `http://localhost:8000` para ver “Hello, Flight!”.

## Passo 2: Organize a Estrutura do Seu Projeto

Para uma configuração organizada, estruture seu projeto assim:

```text
flight-blog/
├── app/
│   ├── config/
│   └── views/
├── data/
├── public/
│   └── index.php
├── vendor/
└── composer.json
```

- `app/config/`: Arquivos de configuração (ex.: eventos, rotas).
- `app/views/`: Templates para renderização de páginas.
- `data/`: Arquivo JSON para armazenar posts do blog.
- `public/`: Raiz web com `index.php`.

## Passo 3: Instale e Configure o Latte

Latte é um mecanismo de templates leve que se integra bem ao Flight.

1. **Instale o Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Configure o Latte no Flight**:
   Atualize `public/index.php` para registrar o Latte como mecanismo de visualização:
   ```php
   <?php
   require '../vendor/autoload.php';

   use Latte\Engine;

   Flight::register('view', Engine::class, [], function ($latte) {
       $latte->setTempDirectory(__DIR__ . '/../cache/');
       $latte->setLoader(new \Latte\Loaders\FileLoader(__DIR__ . '/../app/views/'));
   });

   Flight::route('/', function () {
       Flight::view()->render('home.latte', ['title' => 'My Blog']);
   });

   Flight::start();
   ```

3. **Crie um Template de Layout:
Em `app/views/layout.latte`**:
```html
<!DOCTYPE html>
<html>
<head>
    <title>{$title}</title>
</head>
<body>
    <header>
        <h1>My Blog</h1>
        <nav>
            <a href="/">Home</a> | 
            <a href="/create">Create a Post</a>
        </nav>
    </header>
    <main>
        {block content}{/block}
    </main>
    <footer>
        <p>&copy; {date('Y')} Flight Blog</p>
    </footer>
</body>
</html>
```

4. **Crie um Template Inicial**:
   Em `app/views/home.latte`:
   ```html
  {extends 'layout.latte'}

	{block content}
		<h2>{$title}</h2>
		<ul>
		{foreach $posts as $post}
			<li><a href="/post/{$post['slug']}">{$post['title']}</a></li>
		{/foreach}
		</ul>
	{/block}
   ```
   Reinicie o servidor se você o tiver encerrado e visite `http://localhost:8000` para ver a página renderizada.

5. **Crie um Arquivo de Dados**:

   Use um arquivo JSON para simular um banco de dados por simplicidade.

   Em `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Passo 4: Defina Rotas

Separe suas rotas em um arquivo de configuração para melhor organização.

1. **Crie `routes.php`**:
   Em `app/config/routes.php`:
   ```php
   <?php
   Flight::route('/', function () {
       Flight::view()->render('home.latte', ['title' => 'My Blog']);
   });

   Flight::route('/post/@slug', function ($slug) {
       Flight::view()->render('post.latte', ['title' => 'Post: ' . $slug, 'slug' => $slug]);
   });

   Flight::route('GET /create', function () {
       Flight::view()->render('create.latte', ['title' => 'Create a Post']);
   });
   ```

2. **Atualize `index.php`**:
   Inclua o arquivo de rotas:
   ```php
   <?php
   require '../vendor/autoload.php';

   use Latte\Engine;

   Flight::register('view', Engine::class, [], function ($latte) {
       $latte->setTempDirectory(__DIR__ . '/../cache/');
       $latte->setLoader(new \Latte\Loaders\FileLoader(__DIR__ . '/../app/views/'));
   });

   require '../app/config/routes.php';

   Flight::start();
   ```

## Passo 5: Armazene e Recupere Posts do Blog

Adicione os métodos para carregar e salvar posts.

1. **Adicione um Método de Posts**:
   Em `index.php`, adicione um método para carregar posts:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Atualize Rotas**:
   Modifique `app/config/routes.php` para usar os posts:
   ```php
   <?php
   Flight::route('/', function () {
       $posts = Flight::posts();
       Flight::view()->render('home.latte', [
           'title' => 'My Blog',
           'posts' => $posts
       ]);
   });

   Flight::route('/post/@slug', function ($slug) {
       $posts = Flight::posts();
       $post = array_filter($posts, fn($p) => $p['slug'] === $slug);
       $post = reset($post) ?: null;
       if (!$post) {
           Flight::notFound();
           return;
       }
       Flight::view()->render('post.latte', [
           'title' => $post['title'],
           'post' => $post
       ]);
   });

   Flight::route('GET /create', function () {
       Flight::view()->render('create.latte', ['title' => 'Create a Post']);
   });
   ```

## Passo 6: Crie Templates

Atualize seus templates para exibir posts.

1. **Página do Post (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Passo 7: Adicione Criação de Posts

Lide com o envio do formulário para adicionar novos posts.

1. **Crie o Formulário (`app/views/create.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$title}</h2>
		<form method="POST" action="/create">
			<div class="form-group">
				<label for="title">Title:</label>
				<input type="text" name="title" id="title" required>
			</div>
			<div class="form-group">
				<label for="content">Content:</label>
				<textarea name="content" id="content" required></textarea>
			</div>
			<button type="submit">Save Post</button>
		</form>
	{/block}
   ```

2. **Adicione a Rota POST**:
   Em `app/config/routes.php`:
   ```php
   Flight::route('POST /create', function () {
       $request = Flight::request();
       $title = $request->data['title'];
       $content = $request->data['content'];
       $slug = strtolower(str_replace(' ', '-', $title));

       $posts = Flight::posts();
       $posts[] = ['slug' => $slug, 'title' => $title, 'content' => $content];
       file_put_contents(__DIR__ . '/../../data/posts.json', json_encode($posts, JSON_PRETTY_PRINT));

       Flight::redirect('/');
   });
   ```

3. **Teste**:
   - Visite `http://localhost:8000/create`.
   - Envie um novo post (ex.: “Segundo Post” com algum conteúdo).
   - Verifique a página inicial para vê-lo listado.

## Passo 8: Melhore com Tratamento de Erros

Sobrescreva o método `notFound` para uma melhor experiência de 404.

Em `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Crie `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Próximos Passos
- **Adicione Estilos**: Use CSS em seus templates para uma melhor aparência.
- **Banco de Dados**: Substitua `posts.json` por um banco de dados como SQLite usando [SimplePdo](/learn/simple-pdo).
- **Validação**: Adicione verificações para slugs duplicados ou entradas vazias.
- **Middleware**: Implemente autenticação para a criação de posts.

## Conclusão

Você construiu um blog simples com Flight PHP! Este guia demonstra recursos principais como roteamento, templates com Latte e manipulação de envios de formulários — tudo enquanto mantém as coisas leves. Explore a documentação do Flight para recursos mais avançados e leve seu blog adiante!