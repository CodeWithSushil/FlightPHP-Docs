# Створення простого блогу за допомогою Flight PHP

Цей посібник проведе вас через створення базового блогу з використанням PHP-фреймворку Flight. Ви налаштуєте проєкт, визначите маршрути, керуватимете публікаціями за допомогою JSON та відображатимете їх за допомогою шаблонізатора Latte — все це демонструє простоту та гнучкість Flight. Наприкінці у вас буде функціональний блог із головною сторінкою, сторінками окремих публікацій та формою створення.

## Передумови
- **PHP 7.4+**: встановлений у вашій системі.
- **Composer**: для керування залежностями.
- **Текстовий редактор**: будь-який редактор, як-от VS Code або PHPStorm.
- Базові знання PHP та веб-розробки.

## Крок 1: Налаштування вашого проєкту

Почніть зі створення нового каталогу проєкту та встановлення Flight через Composer.

1. **Створіть каталог**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Встановіть Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Створіть публічний каталог**:
   Flight використовує єдину точку входу (`index.php`). Створіть папку `public/` для неї:
   ```bash
   mkdir public
   ```

4. **Базовий `index.php`**:
   Створіть `public/index.php` із простим маршрутом "hello world":
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Запустіть вбудований сервер**:
   Перевірте ваше налаштування за допомогою сервера розробки PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Відвідайте `http://localhost:8000`, щоб побачити "Hello, Flight!".

## Крок 2: Організація структури проєкту

Для чистого налаштування структуруйте свій проєкт так:

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

- `app/config/`: файли конфігурації (наприклад, події, маршрути).
- `app/views/`: шаблони для відображення сторінок.
- `data/`: файл JSON для зберігання публікацій блогу.
- `public/`: корінь веб-сайту з `index.php`.

## Крок 3: Встановлення та налаштування Latte

Latte — це легкий шаблонізатор, який добре інтегрується з Flight.

1. **Встановіть Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Налаштуйте Latte у Flight**:
   Оновіть `public/index.php`, щоб зареєструвати Latte як механізм перегляду:
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

3. **Створіть шаблон макета**:
   У `app/views/layout.latte`:
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

4. **Створіть головний шаблон**:
   У `app/views/home.latte`:
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
   Перезапустіть сервер, якщо ви вийшли з нього, і відвідайте `http://localhost:8000`, щоб побачити відрендерену сторінку.

5. **Створіть файл даних**:

   Використовуйте файл JSON для імітації бази даних задля простоти.

   У `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Крок 4: Визначення маршрутів

Виділіть свої маршрути в окремий файл конфігурації для кращої організації.

1. **Створіть `routes.php`**:
   У `app/config/routes.php`:
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

2. **Оновіть `index.php`**:
   Підключіть файл маршрутів:
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

## Крок 5: Зберігання та отримання публікацій блогу

Додайте методи для завантаження та збереження публікацій.

1. **Додайте метод для публікацій**:
   У `index.php` додайте метод для завантаження публікацій:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Оновіть маршрути**:
   Змініть `app/config/routes.php`, щоб використовувати публікації:
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

## Крок 6: Створення шаблонів

Оновіть шаблони для відображення публікацій.

1. **Сторінка публікації (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Крок 7: Додавання створення публікацій

Обробка надсилання форми для додавання нових публікацій.

1. **Форма (`app/views/create.latte`)**:
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

2. **Додайте POST-маршрут**:
   У `app/config/routes.php`:
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

3. **Перевірте**:
   - Відвідайте `http://localhost:8000/create`.
   - Надішліть нову публікацію (наприклад, "Second Post" із якимось вмістом).
   - Перевірте головну сторінку, щоб побачити її у списку.

## Крок 8: Покращення обробки помилок

Перевизначте метод `notFound` для кращого досвіду з помилкою 404.

У `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Створіть `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Наступні кроки
- **Додайте стилі**: Використовуйте CSS у своїх шаблонах для кращого вигляду.
- **База даних**: Замініть `posts.json` на базу даних, як-от SQLite, використовуючи [SimplePdo](/learn/simple-pdo).
- **Валідація**: Додайте перевірки на дублікати slug або порожні поля.
- **Проміжне програмне забезпечення**: Впровадьте автентифікацію для створення публікацій.

## Висновок

Ви створили простий блог за допомогою Flight PHP! Цей посібник демонструє основні функції, як-от маршрутизацію, шаблонізацію за допомогою Latte та обробку надсилання форм — усе це залишається легким. Вивчайте документацію Flight, щоб дізнатися про більш розширені функції та розвинути свій блог далі!