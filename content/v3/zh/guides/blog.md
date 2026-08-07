# 使用 Flight PHP 构建一个简单的博客

本指南将带您使用 Flight PHP 框架创建一个基础博客。您将设置项目、定义路由、使用 JSON 管理文章，并使用 Latte 模板引擎渲染它们——所有这些都展示了 Flight 的简洁性和灵活性。最后，您将拥有一个功能齐全的博客，包含首页、单篇文章页面和一个创建表单。

## 先决条件
- **PHP 7.4+**：系统上已安装。
- **Composer**：用于依赖管理。
- **文本编辑器**：任何编辑器，如 VS Code 或 PHPStorm。
- 具备 PHP 和 Web 开发的基础知识。

## 第一步：设置项目

首先创建一个新项目目录，并通过 Composer 安装 Flight。

1. **创建目录**：
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **安装 Flight**：
   ```bash
   composer require flightphp/core
   ```

3. **创建 public 目录**：
   Flight 使用单一入口点（`index.php`）。为其创建一个 `public/` 文件夹：
   ```bash
   mkdir public
   ```

4. **基本 `index.php`**：
   创建一个简单的 “hello world” 路由的 `public/index.php`：
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **运行内置服务器**：
   使用 PHP 开发服务器测试您的设置：
   ```bash
   php -S localhost:8000 -t public/
   ```
   访问 `http://localhost:8000` 即可看到 “Hello, Flight!”。

## 第二步：组织项目结构

为了清晰设置，请按如下结构组织项目：

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

- `app/config/`：配置文件（如事件、路由）。
- `app/views/`：渲染页面的模板。
- `data/`：用于存储博客文章的 JSON 文件。
- `public/`：Web 根目录，包含 `index.php`。

## 第三步：安装并配置 Latte

Latte 是一个轻量级模板引擎，与 Flight 集成得很好。

1. **安装 Latte**：
   ```bash
   composer require latte/latte
   ```

2. **在 Flight 中配置 Latte**：
   更新 `public/index.php`，将 Latte 注册为视图引擎：
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

3. **创建布局模板：**  
在 `app/views/layout.latte` 中：
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

4. **创建首页模板**：
   在 `app/views/home.latte` 中：
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
   如果您退出了服务器，请重启并访问 `http://localhost:8000` 查看渲染后的页面。

5. **创建数据文件**：

   为简单起见，使用 JSON 文件模拟数据库。

   在 `data/posts.json` 中：
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## 第四步：定义路由

为了更好的组织，将路由分离到配置文件中。

1. **创建 `routes.php`**：
   在 `app/config/routes.php` 中：
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

2. **更新 `index.php`**：
   引入路由文件：
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

## 第五步：存储和检索博客文章

添加加载和保存文章的方法。

1. **添加文章方法**：
   在 `index.php` 中，添加一个加载文章的方法：
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **更新路由**：
   修改 `app/config/routes.php` 以使用文章：
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

## 第六步：创建模板

更新模板以显示文章。

1. **文章页面（`app/views/post.latte`）**：
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## 第七步：添加文章创建功能

处理表单提交以添加新文章。

1. **创建表单（`app/views/create.latte`）**：
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

2. **添加 POST 路由**：
   在 `app/config/routes.php` 中：
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

3. **测试**：
   - 访问 `http://localhost:8000/create`。
   - 提交一篇新文章（例如，带有一些内容的 “Second Post”）。
   - 检查首页是否显示该文章。

## 第八步：通过错误处理增强体验

重写 `notFound` 方法以获得更好的 404 体验。

在 `index.php` 中：
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

创建 `app/views/404.latte`：
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## 下一步
- **添加样式**：在模板中使用 CSS 以获得更好的外观。
- **数据库**：使用 [SimplePdo](/learn/simple-pdo) 等数据库替换 `posts.json`。
- **验证**：添加对重复 slug 或空输入的检查。
- **中间件**：为文章创建实现身份验证。

## 结论

您已经使用 Flight PHP 构建了一个简单的博客！本指南演示了路由、使用 Latte 进行模板渲染以及处理表单提交等核心功能——同时保持了轻量级。探索 Flight 的文档，了解更高级的功能，让您的博客更进一步！