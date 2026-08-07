# Créer un blog simple avec Flight PHP

Ce guide vous apprend à créer un blog de base en utilisant le framework PHP Flight. Vous configurerez un projet, définirez des routes, gérerez des articles avec du JSON et les afficherez avec le moteur de templates Latte, tout en montrant la simplicité et la flexibilité de Flight. Au final, vous disposerez d'un blog fonctionnel avec une page d'accueil, des pages individuelles pour les articles et un formulaire de création.

## Prérequis
- **PHP 7.4+** : Installé sur votre système.
- **Composer** : Pour la gestion des dépendances.
- **Éditeur de texte** : N'importe quel éditeur comme VS Code ou PHPStorm.
- Connaissances de base en PHP et développement web.

## Étape 1 : Configurer votre projet

Commencez par créer un nouveau répertoire de projet et installez Flight via Composer.

1. **Créez un répertoire** :
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Installez Flight** :
   ```bash
   composer require flightphp/core
   ```

3. **Créez un répertoire public** :
   Flight utilise un point d'entrée unique (`index.php`). Créez un dossier `public/` pour celui-ci :
   ```bash
   mkdir public
   ```

4. **`index.php` de base** :
   Créez `public/index.php` avec une simple route « hello world » :
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Exécutez le serveur intégré** :
   Testez votre configuration avec le serveur de développement PHP :
   ```bash
   php -S localhost:8000 -t public/
   ```
   Visitez `http://localhost:8000` pour voir « Hello, Flight ! ».

## Étape 2 : Organiser la structure de votre projet

Pour une configuration propre, structurez votre projet comme ceci :

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

- `app/config/` : Fichiers de configuration (par exemple, événements, routes).
- `app/views/` : Templates pour afficher les pages.
- `data/` : Fichier JSON pour stocker les articles du blog.
- `public/` : Racine web avec `index.php`.

## Étape 3 : Installer et configurer Latte

Latte est un moteur de templates léger qui s'intègre bien avec Flight.

1. **Installez Latte** :
   ```bash
   composer require latte/latte
   ```

2. **Configurez Latte dans Flight** :
   Mettez à jour `public/index.php` pour enregistrer Latte comme moteur de vue :
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

3. **Créez un modèle de mise en page : dans `app/views/layout.latte`** :
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

4. **Créez un modèle d'accueil** :
   Dans `app/views/home.latte` :
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
   Redémarrez le serveur si vous l'avez quitté et visitez `http://localhost:8000` pour voir la page rendue.

5. **Créez un fichier de données** :

   Utilisez un fichier JSON pour simuler une base de données par simplicité.

   Dans `data/posts.json` :
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Étape 4 : Définir les routes

Séparez vos routes dans un fichier de configuration pour une meilleure organisation.

1. **Créez `routes.php`** :
   Dans `app/config/routes.php` :
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

2. **Mettez à jour `index.php`** :
   Incluez le fichier de routes :
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

## Étape 5 : Stocker et récupérer les articles du blog

Ajoutez les méthodes pour charger et enregistrer les articles.

1. **Ajoutez une méthode Posts** :
   Dans `index.php`, ajoutez une méthode pour charger les articles :
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Mettez à jour les routes** :
   Modifiez `app/config/routes.php` pour utiliser les articles :
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

## Étape 6 : Créer des modèles

Mettez à jour vos modèles pour afficher les articles.

1. **Page d'article (`app/views/post.latte`)** :
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Étape 7 : Ajouter la création d'article

Gérez la soumission du formulaire pour ajouter de nouveaux articles.

1. **Créez le formulaire (`app/views/create.latte`)** :
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

2. **Ajoutez la route POST** :
   Dans `app/config/routes.php` :
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

3. **Testez-le** :
   - Visitez `http://localhost:8000/create`.
   - Soumettez un nouvel article (par exemple, « Second Post » avec du contenu).
   - Vérifiez la page d'accueil pour le voir listé.

## Étape 8 : Améliorer avec la gestion des erreurs

Remplacez la méthode `notFound` pour une meilleure expérience 404.

Dans `index.php` :
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Créez `app/views/404.latte` :
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Prochaines étapes
- **Ajoutez du style** : Utilisez du CSS dans vos modèles pour un meilleur rendu.
- **Base de données** : Remplacez `posts.json` par une base de données comme SQLite en utilisant [SimplePdo](/learn/simple-pdo).
- **Validation** : Ajoutez des vérifications pour les slugs en double ou les entrées vides.
- **Middleware** : Implémentez l'authentification pour la création d'articles.

## Conclusion

Vous avez créé un blog simple avec Flight PHP ! Ce guide démontre les fonctionnalités de base comme le routage, les templates avec Latte et la gestion des soumissions de formulaires, tout en restant léger. Explorez la documentation de Flight pour des fonctionnalités plus avancées afin d'aller plus loin avec votre blog !