# Vienkārša emuāra izveide ar Flight PHP

Šis ceļvedis palīdzēs jums izveidot vienkāršu emuāru, izmantojot Flight PHP ietvaru. Jūs izveidosiet projektu, definēsiet maršrutus, pārvaldīsiet ierakstus ar JSON un atveidosiet tos ar Latte šablonu dzinēju—vienlaikus parādot Flight vienkāršību un elastīgumu. Līdz beigām jums būs funkcionējošs emuārs ar sākumlapu, atsevišķu ierakstu lapām un izveides formu.

## Priekšnosacījumi
- **PHP 7.4+**: Instalēts jūsu sistēmā.
- **Composer**: Atkarību pārvaldībai.
- **Teksta redaktors**: Jebkurš redaktors, piemēram, VS Code vai PHPStorm.
- Pamatzināšanas par PHP un tīmekļa izstrādi.

## 1. solis: Projekta iestatīšana

Sāciet, izveidojot jaunu projekta direktoriju un instalējot Flight, izmantojot Composer.

1. **Izveidojiet direktoriju**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Instalējiet Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Izveidojiet publisko direktoriju**:
   Flight izmanto vienu ieejas punktu (`index.php`). Izveidojiet tam `public/` mapi:
   ```bash
   mkdir public
   ```

4. **Vienkāršs `index.php`**:
   Izveidojiet `public/index.php` ar vienkāršu “hello world” maršrutu:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Palaidiet iebūvēto serveri**:
   Pārbaudiet savu iestatījumu ar PHP izstrādes serveri:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Apmeklējiet `http://localhost:8000`, lai redzētu “Hello, Flight!”.

## 2. solis: Projekta struktūras organizēšana

Lai iegūtu tīru iestatījumu, strukturējiet projektu šādi:

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

- `app/config/`: Konfigurācijas faili (piemēram, notikumi, maršruti).
- `app/views/`: Veidnes lapu atveidošanai.
- `data/`: JSON fails emuāra ierakstu glabāšanai.
- `public/`: Tīmekļa sakne ar `index.php`.

## 3. solis: Latte instalēšana un konfigurēšana

Latte ir viegls šablonu dzinējs, kas labi integrējas ar Flight.

1. **Instalējiet Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Konfigurējiet Latte Flight**:
   Atjauniniet `public/index.php`, lai reģistrētu Latte kā skatu dzinēju:
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

3. **Izveidojiet izkārtojuma veidni**:
   Failā `app/views/layout.latte`:
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

4. **Izveidojiet sākumlapas veidni**:
   Failā `app/views/home.latte`:
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
   Restartējiet serveri, ja esat no tā izgājis, un apmeklējiet `http://localhost:8000`, lai redzētu atveidoto lapu.

5. **Izveidojiet datu failu**:
   Izmantojiet JSON failu, lai vienkāršības labad simulētu datubāzi.
   Failā `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## 4. solis: Maršrutu definēšana

Atdaliet savus maršrutus konfigurācijas failā, lai nodrošinātu labāku organizāciju.

1. **Izveidojiet `routes.php`**:
   Failā `app/config/routes.php`:
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

2. **Atjauniniet `index.php`**:
   Iekļaujiet maršrutu failu:
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

## 5. solis: Emuāra ierakstu glabāšana un ielāde

Pievienojiet metodes ierakstu ielādei un saglabāšanai.

1. **Pievienojiet ierakstu metodi**:
   Failā `index.php` pievienojiet metodi ierakstu ielādei:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Atjauniniet maršrutus**:
   Modificējiet `app/config/routes.php`, lai izmantotu ierakstus:
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

## 6. solis: Veidņu izveide

Atjauniniet savas veidnes, lai attēlotu ierakstus.

1. **Ieraksta lapa (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## 7. solis: Ierakstu izveides pievienošana

Apstrādājiet veidlapas iesniegšanu, lai pievienotu jaunus ierakstus.

1. **Izveidojiet veidlapu (`app/views/create.latte`)**:
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

2. **Pievienojiet POST maršrutu**:
   Failā `app/config/routes.php`:
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

3. **Izmēģiniet to**:
   - Apmeklējiet `http://localhost:8000/create`.
   - Iesniedziet jaunu ierakstu (piemēram, “Otrais ieraksts” ar kādu saturu).
   - Pārbaudiet sākumlapu, lai redzētu to sarakstā.

## 8. solis: Kļūdu apstrādes uzlabošana

Pārdefinējiet `notFound` metodi, lai nodrošinātu labāku 404 pieredzi.

Failā `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Izveidojiet `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Turpmākās darbības
- **Pievienojiet stilus**: Izmantojiet CSS savās veidnēs, lai iegūtu labāku izskatu.
- **Datubāze**: Aizstājiet `posts.json` ar datubāzi, piemēram, SQLite, izmantojot [SimplePdo](/learn/simple-pdo).
- **Validācija**: Pievienojiet pārbaudes dublikātu slug vai tukšu ievades lauku noteikšanai.
- **Starpniekprogrammatūra**: Ieviesiet autentifikāciju ierakstu izveidei.

## Secinājums

Jūs esat izveidojuši vienkāršu emuāru ar Flight PHP! Šis ceļvedis demonstrē galvenās funkcijas, piemēram, maršrutēšanu, šablonu izmantošanu ar Latte un veidlapu iesniegumu apstrādi—vienlaikus saglabājot vieglumu. Izpētiet Flight dokumentāciju, lai uzzinātu par papildu funkcijām, kas palīdzēs attīstīt jūsu emuāru tālāk!