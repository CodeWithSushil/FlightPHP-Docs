# Erstellen eines einfachen Blogs mit Flight PHP

In diesem Leitfaden erstellen Sie einen einfachen Blog mit dem Flight-PHP-Framework. Sie richten ein Projekt ein, definieren Routen, verwalten Beiträge mit JSON und rendern sie mit der Latte-Template-Engine – alles zeigt die Einfachheit und Flexibilität von Flight. Am Ende haben Sie einen funktionsfähigen Blog mit einer Startseite, einzelnen Beitragsseiten und einem Formular zum Erstellen.

## Voraussetzungen
- **PHP 7.4+**: Auf Ihrem System installiert.
- **Composer**: Für die Verwaltung von Abhängigkeiten.
- **Texteditor**: Ein beliebiger Editor wie VS Code oder PHPStorm.
- Grundkenntnisse in PHP und Webentwicklung.

## Schritt 1: Projekt einrichten

Beginnen Sie mit der Erstellung eines neuen Projektverzeichnisses und der Installation von Flight über Composer.

1. **Verzeichnis erstellen**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Flight installieren**:
   ```bash
   composer require flightphp/core
   ```

3. **Ein öffentliches Verzeichnis erstellen**:
   Flight verwendet einen einzigen Einstiegspunkt (`index.php`). Erstellen Sie einen `public/`-Ordner dafür:
   ```bash
   mkdir public
   ```

4. **Basis-`index.php`**:
   Erstellen Sie `public/index.php` mit einer einfachen „Hallo-Welt“-Route:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Den eingebauten Server ausführen**:
   Testen Sie Ihre Einrichtung mit dem Entwicklungsserver von PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Besuchen Sie `http://localhost:8000`, um „Hello, Flight!“ zu sehen.

## Schritt 2: Projektstruktur organisieren

Für eine saubere Einrichtung strukturieren Sie Ihr Projekt wie folgt:

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

- `app/config/`: Konfigurationsdateien (z. B. Ereignisse, Routen).
- `app/views/`: Vorlagen zum Rendern von Seiten.
- `data/`: JSON-Datei zum Speichern von Blogbeiträgen.
- `public/`: Web-Root mit `index.php`.

## Schritt 3: Latte installieren und konfigurieren

Latte ist eine leichtgewichtige Template-Engine, die sich gut in Flight integrieren lässt.

1. **Latte installieren**:
   ```bash
   composer require latte/latte
   ```

2. **Latte in Flight konfigurieren**:
   Aktualisieren Sie `public/index.php`, um Latte als View-Engine zu registrieren:
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

3. **Ein Layout-Template erstellen: 
In `app/views/layout.latte`**:
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

4. **Eine Home-Vorlage erstellen**:
   In `app/views/home.latte`:
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
   Starten Sie den Server neu, falls Sie ihn beendet haben, und besuchen Sie `http://localhost:8000`, um die gerenderte Seite zu sehen.

5. **Eine Datendatei erstellen**:
   Verwenden Sie eine JSON-Datei, um eine Datenbank der Einfachheit halber zu simulieren.
   In `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Schritt 4: Routen definieren

Lagern Sie Ihre Routen für eine bessere Organisation in eine Konfigurationsdatei aus.

1. **`routes.php` erstellen**:
   In `app/config/routes.php`:
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

2. **`index.php` aktualisieren**:
   Binden Sie die Routendatei ein:
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

## Schritt 5: Blogbeiträge speichern und abrufen

Fügen Sie die Methoden zum Laden und Speichern von Beiträgen hinzu.

1. **Eine Posts-Methode hinzufügen**:
   Fügen Sie in `index.php` eine Methode zum Laden von Beiträgen hinzu:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Routen aktualisieren**:
   Ändern Sie `app/config/routes.php`, um die Beiträge zu verwenden:
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

## Schritt 6: Vorlagen erstellen

Aktualisieren Sie Ihre Vorlagen, um Beiträge anzuzeigen.

1. **Beitragsseite (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Schritt 7: Beitragserstellung hinzufügen

Behandeln Sie die Formularübermittlung, um neue Beiträge hinzuzufügen.

1. **Formular erstellen (`app/views/create.latte`)**:
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

2. **POST-Route hinzufügen**:
   In `app/config/routes.php`:
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

3. **Testen Sie es**:
   - Besuchen Sie `http://localhost:8000/create`.
   - Senden Sie einen neuen Beitrag (z. B. „Second Post“ mit etwas Inhalt).
   - Prüfen Sie die Startseite, um den Beitrag aufgelistet zu sehen.

## Schritt 8: Mit Fehlerbehandlung verbessern

Überschreiben Sie die `notFound`-Methode für eine bessere 404-Erfahrung.

In `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Erstellen Sie `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Nächste Schritte
- **Styling hinzufügen**: Verwenden Sie CSS in Ihren Vorlagen für ein besseres Aussehen.
- **Datenbank**: Ersetzen Sie `posts.json` durch eine Datenbank wie SQLite mithilfe von [SimplePdo](/learn/simple-pdo).
- **Validierung**: Fügen Sie Prüfungen für doppelte Slugs oder leere Eingaben hinzu.
- **Middleware**: Implementieren Sie eine Authentifizierung für das Erstellen von Beiträgen.

## Fazit

Sie haben einen einfachen Blog mit Flight PHP erstellt! Dieser Leitfaden zeigt Kernfunktionen wie Routing, Templating mit Latte und die Verarbeitung von Formularübermittlungen – und bleibt dabei leichtgewichtig. Entdecken Sie die Dokumentation von Flight für weitere fortgeschrittene Funktionen, um Ihren Blog noch weiter zu bringen!