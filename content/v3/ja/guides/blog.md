# Flight PHPでシンプルなブログを構築する

このガイドでは、Flight PHPフレームワークを使って基本的なブログを作成する手順を説明します。プロジェクトのセットアップ、ルートの定義、JSONでの投稿管理、Latteテンプレートエンジンでのレンダリングを行い、Flightのシンプルさと柔軟性を示します。最後には、ホームページ、個別投稿ページ、作成フォームを備えた機能的なブログが完成します。

## 前提条件
- **PHP 7.4+**: システムにインストールされていること。
- **Composer**: 依存関係の管理用。
- **テキストエディタ**: VS CodeやPHPStormなどの任意のエディタ。
- PHPとWeb開発の基本的な知識。

## ステップ 1: プロジェクトのセットアップ

まず、新しいプロジェクトディレクトリを作成し、Composerを使ってFlightをインストールします。

1. **ディレクトリを作成**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Flightをインストール**:
   ```bash
   composer require flightphp/core
   ```

3. **publicディレクトリを作成**:
   Flightは単一のエントリポイント（`index.php`）を使用します。そのための`public/`フォルダを作成します:
   ```bash
   mkdir public
   ```

4. **基本的な`index.php`**:
   `public/index.php`にシンプルな「Hello World」ルートを作成します:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **ビルトインサーバーを実行**:
   PHPの開発サーバーでセットアップをテストします:
   ```bash
   php -S localhost:8000 -t public/
   ```
   `http://localhost:8000`にアクセスして「Hello, Flight!」が表示されることを確認します。

## ステップ 2: プロジェクト構造を整理する

クリーンなセットアップのために、プロジェクトを次のように構成します:

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

- `app/config/`: 設定ファイル（例: イベント、ルート）。
- `app/views/`: ページをレンダリングするためのテンプレート。
- `data/`: ブログ投稿を保存するJSONファイル。
- `public/`: `index.php`を含むWebルート。

## ステップ 3: Latteのインストールと設定

Latteは、Flightとよく統合する軽量なテンプレートエンジンです。

1. **Latteをインストール**:
   ```bash
   composer require latte/latte
   ```

2. **FlightでLatteを設定**:
   `public/index.php`を更新して、Latteをビューエンジンとして登録します:
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

3. **レイアウトテンプレートを作成: 
`app/views/layout.latte`**:
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

4. **ホームテンプレートを作成**:
   `app/views/home.latte`:
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
   サーバーを終了した場合は再起動し、`http://localhost:8000`にアクセスしてレンダリングされたページを確認します。

5. **データファイルを作成**:

   簡単にするために、JSONファイルでデータベースをシミュレートします。

   `data/posts.json`:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## ステップ 4: ルートを定義する

整理しやすくするために、ルートを設定ファイルに分離します。

1. **`routes.php`を作成**:
   `app/config/routes.php`:
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

2. **`index.php`を更新**:
   ルートファイルを読み込みます:
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

## ステップ 5: ブログ投稿の保存と取得

投稿を読み込んで保存するメソッドを追加します。

1. **投稿メソッドを追加**:
   `index.php`で、投稿を読み込むメソッドを追加します:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **ルートを更新**:
   `app/config/routes.php`を変更して投稿を使用します:
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

## ステップ 6: テンプレートを作成する

投稿を表示するようにテンプレートを更新します。

1. **投稿ページ（`app/views/post.latte`）**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## ステップ 7: 投稿作成機能を追加する

フォーム送信を処理して新しい投稿を追加します。

1. **フォームを作成（`app/views/create.latte`）**:
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

2. **POSTルートを追加**:
   `app/config/routes.php`:
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

3. **テストする**:
   - `http://localhost:8000/create`にアクセスします。
   - 新しい投稿を送信します（例: 「Second Post」とコンテンツ）。
   - ホームページにそれが一覧表示されることを確認します。

## ステップ 8: エラーハンドリングを強化する

`notFound`メソッドをオーバーライドして、より良い404エラーページを提供します。

`index.php`:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

`app/views/404.latte`を作成:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## 次のステップ
- **スタイリングを追加**: テンプレートにCSSを使用して見た目を良くします。
- **データベース**: [SimplePdo](/learn/simple-pdo)を使用して`posts.json`をSQLiteなどのデータベースに置き換えます。
- **バリデーション**: 重複したスラッグや空の入力をチェックする機能を追加します。
- **ミドルウェア**: 投稿作成のための認証を実装します。

## 結論

Flight PHPでシンプルなブログを構築できました！このガイドでは、ルーティング、Latteを使ったテンプレート、フォーム送信の処理など、Flightのコア機能を軽量なまま保ちながら紹介しました。Flightのドキュメントを参照して、さらに高度な機能を使い、ブログをさらに発展させましょう！