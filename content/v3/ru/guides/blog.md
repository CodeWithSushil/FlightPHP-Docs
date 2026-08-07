# Создание простого блога с Flight PHP

Это руководство проведет вас через создание базового блога с использованием PHP-фреймворка Flight. Вы настроите проект, определите маршруты, будете управлять постами с помощью JSON и отображать их с помощью шаблонизатора Latte — все это демонстрирует простоту и гибкость Flight. В итоге у вас будет работающий блог с главной страницей, отдельными страницами постов и формой создания.

## Предварительные требования
- **PHP 7.4+**: Установлен в вашей системе.
- **Composer**: Для управления зависимостями.
- **Текстовый редактор**: Любой редактор, например VS Code или PHPStorm.
- Базовые знания PHP и веб-разработки.

## Шаг 1: Настройка проекта

Начните с создания нового каталога проекта и установки Flight через Composer.

1. **Создайте каталог**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Установите Flight**:
   ```bash
   composer require flightphp/core
   ```

3. **Создайте общедоступный каталог**:
   Flight использует единую точку входа (`index.php`). Создайте папку `public/` для него:
   ```bash
   mkdir public
   ```

4. **Базовый `index.php`**:
   Создайте `public/index.php` с простым маршрутом «hello world»:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **Запустите встроенный сервер**:
   Проверьте настройку с помощью встроенного сервера разработки PHP:
   ```bash
   php -S localhost:8000 -t public/
   ```
   Перейдите по адресу `http://localhost:8000`, чтобы увидеть «Hello, Flight!».

## Шаг 2: Организация структуры проекта

Для аккуратной настройки структурируйте проект следующим образом:

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

- `app/config/`: Файлы конфигурации (например, события, маршруты).
- `app/views/`: Шаблоны для отображения страниц.
- `data/`: JSON-файл для хранения записей блога.
- `public/`: Корень веб-сайта с `index.php`.

## Шаг 3: Установка и настройка Latte

Latte — это легкий шаблонизатор, который хорошо интегрируется с Flight.

1. **Установите Latte**:
   ```bash
   composer require latte/latte
   ```

2. **Настройте Latte во Flight**:
   Обновите `public/index.php`, чтобы зарегистрировать Latte в качестве шаблонизатора представлений:
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

3. **Создайте шаблон макета:
   В `app/views/layout.latte`**:
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

4. **Создайте домашний шаблон**:
   В `app/views/home.latte`:
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
   Перезапустите сервер, если вы его остановили, и перейдите по адресу `http://localhost:8000`, чтобы увидеть отрендеренную страницу.

5. **Создайте файл данных**:
   Используйте JSON-файл для имитации базы данных для простоты.

   В `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## Шаг 4: Определение маршрутов

Вынесите ваши маршруты в отдельный файл конфигурации для лучшей организации.

1. **Создайте `routes.php`**:
   В `app/config/routes.php`:
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

2. **Обновите `index.php`**:
   Подключите файл маршрутов:
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

## Шаг 5: Сохранение и получение записей блога

Добавьте методы для загрузки и сохранения записей.

1. **Добавьте метод Posts**:
   В `index.php` добавьте метод для загрузки записей:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **Обновите маршруты**:
   Измените `app/config/routes.php`, чтобы использовать записи:
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

## Шаг 6: Создание шаблонов

Обновите шаблоны для отображения записей.

1. **Страница записи (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## Шаг 7: Добавление создания записей

Обработайте отправку формы для добавления новых записей.

1. **Создайте форму (`app/views/create.latte`)**:
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

2. **Добавьте POST-маршрут**:
   В `app/config/routes.php`:
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

3. **Проверьте**:
   - Перейдите по адресу `http://localhost:8000/create`.
   - Отправьте новую запись (например, «Second Post» с некоторым содержимым).
   - Проверьте главную страницу, чтобы увидеть её в списке.

## Шаг 8: Улучшение с помощью обработки ошибок

Переопределите метод `notFound` для лучшего отображения ошибки 404.

В `index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

Создайте `app/views/404.latte`:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## Следующие шаги
- **Добавьте стили**: Используйте CSS в шаблонах для лучшего внешнего вида.
- **База данных**: Замените `posts.json` на базу данных, например SQLite, с помощью [SimplePdo](/learn/simple-pdo).
- **Валидация**: Добавьте проверки на дублирующиеся слаги или пустые поля.
- **Middleware**: Реализуйте аутентификацию для создания записей.

## Заключение

Вы создали простой блог с помощью Flight PHP! Это руководство демонстрирует основные возможности, такие как маршрутизация, шаблонизация с Latte и обработка отправки форм — всё это остается легковесным. Изучите документацию Flight, чтобы узнать о более продвинутых функциях и развить ваш блог дальше!