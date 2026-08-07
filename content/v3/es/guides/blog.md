# Cómo crear un blog simple con Flight PHP

Esta guía te guía a través de la creación de un blog básico usando el framework PHP Flight. Configurarás un proyecto, definirás rutas, gestionarás publicaciones con JSON y las renderizarás con el motor de plantillas Latte, todo mostrando la simplicidad y flexibilidad de Flight. Al final, tendrás un blog funcional con una página de inicio, páginas individuales de publicación y un formulario de creación.

## Requisitos previos
- **PHP 7.4+**: Instalado en tu sistema.
- **Composer**: Para la gestión de dependencias.
- **Editor de texto**: Cualquier editor como VS Code o PHPStorm.
- Conocimientos básicos de PHP y desarrollo web.

## Paso 1: Configura tu proyecto

Comienza creando un nuevo directorio de proyecto e instalando Flight mediante Composer.

1. **Crea un directorio**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Instala Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Crea un directorio público**:
   Flight usa un único punto de entrada (`index.php`). Crea una carpeta `public/` para ello:
   ```bash
   mkdir public
   ```

4. **`index.php` básico**:
   Crea `public/index.php` con una ruta simple de “hola mundo”:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Ejecuta el servidor integrado**:
   Prueba tu configuración con el servidor de desarrollo de PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Visita `http://localhost:8000` para ver “Hello, Flight!”.

## Paso 2: Organiza la estructura de tu proyecto

Para una configuración limpia, estructura tu proyecto así:

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

- `app/config/`: Archivos de configuración (por ejemplo, eventos, rutas).
- `app/views/`: Plantillas para renderizar páginas.
- `data/`: Archivo JSON para almacenar publicaciones del blog.
- `public/`: Raíz web con `index.php`.

## Paso 3: Instala y configura Latte

Latte es un motor de plantillas ligero que se integra bien con Flight.

1. **Instala Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Configura Latte en Flight**:
   Actualiza `public/index.php` para registrar Latte como el motor de vistas:
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

3. **Crea una plantilla de diseño: 
En `app/views/layout.latte`**:
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

4. **Crea una plantilla de inicio**:
   En `app/views/home.latte`:
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
   Reinicia el servidor si lo has cerrado y visita `http://localhost:8000` para ver la página renderizada.

5. **Crea un archivo de datos**:

   Usa un archivo JSON para simular una base de datos por simplicidad.

   En `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Paso 4: Define las rutas

Separa tus rutas en un archivo de configuración para una mejor organización.

1. **Crea `routes.php`**:
   En `app/config/routes.php`:
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

2. **Actualiza `index.php`**:
   Incluye el archivo de rutas:
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

## Paso 5: Almacena y recupera publicaciones del blog

Agrega los métodos para cargar y guardar publicaciones.

1. **Agrega un método de publicaciones**:
   En `index.php`, agrega un método para cargar publicaciones:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Actualiza las rutas**:
   Modifica `app/config/routes.php` para usar las publicaciones:
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

## Paso 6: Crea plantillas

Actualiza tus plantillas para mostrar las publicaciones.

1. **Página de publicación (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Paso 7: Agrega creación de publicaciones

Maneja el envío del formulario para agregar nuevas publicaciones.

1. **Crea el formulario (`app/views/create.latte`)**:
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

2. **Agrega la ruta POST**:
   En `app/config/routes.php`:
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

3. **Pruébalo**:
   - Visita `http://localhost:8000/create`.
   - Envía una nueva publicación (por ejemplo, “Second Post” con algo de contenido).
   - Revisa la página de inicio para verla listada.

## Paso 8: Mejora con manejo de errores

Sobrescribe el método `notFound` para una mejor experiencia de error 404.

En `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Crea `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Próximos pasos
- **Agrega estilos**: Usa CSS en tus plantillas para una mejor apariencia.
- **Base de datos**: Reemplaza `posts.json` con una base de datos como SQLite usando [SimplePdo](/learn/simple-pdo).
- **Validación**: Agrega comprobaciones para slugs duplicados o entradas vacías.
- **Middleware**: Implementa autenticación para la creación de publicaciones.

## Conclusión

¡Has creado un blog simple con Flight PHP! Esta guía demuestra características principales como el enrutamiento, las plantillas con Latte y el manejo de envíos de formularios, todo mientras se mantiene ligero. Explora la documentación de Flight para obtener más funciones avanzadas y llevar tu blog más lejos.