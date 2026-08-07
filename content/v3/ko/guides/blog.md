# Flight PHP로 간단한 블로그 만들기

이 가이드는 Flight PHP 프레임워크를 사용하여 기본 블로그를 만드는 과정을 안내합니다. 프로젝트를 설정하고, 라우트를 정의하고, JSON으로 게시물을 관리하고, Latte 템플릿 엔진으로 렌더링하는 방법을 살펴봅니다. 이 모든 과정을 통해 Flight의 단순함과 유연성을 확인할 수 있습니다. 마지막에는 홈페이지, 개별 게시물 페이지, 작성 양식을 갖춘 작동하는 블로그를 만들 수 있습니다.

## 사전 요구 사항
- **PHP 7.4+**: 시스템에 설치되어 있어야 합니다.
- **Composer**: 의존성 관리를 위해 필요합니다.
- **텍스트 편집기**: VS Code나 PHPStorm 같은 편집기.
- PHP와 웹 개발에 대한 기본 지식.

## 1단계: 프로젝트 설정

먼저 새 프로젝트 디렉터리를 만들고 Composer를 통해 Flight를 설치합니다.

1. **디렉터리 만들기**:
   ```bash
   mkdir flight-blog
   cd flight-blog
   ```

2. **Flight 설치**:
   ```bash
   composer require flightphp/core
   ```

3. **public 디렉터리 만들기**:
   Flight는 단일 진입점(`index.php`)을 사용합니다. 이를 위해 `public/` 폴더를 만듭니다:
   ```bash
   mkdir public
   ```

4. **기본 `index.php`**:
   간단한 "hello world" 라우트를 가진 `public/index.php`를 만듭니다:
   ```php
   <?php
   require '../vendor/autoload.php';

   Flight::route('/', function () {
       echo 'Hello, Flight!';
   });

   Flight::start();
   ```

5. **내장 서버 실행**:
   PHP 개발 서버로 설정을 테스트합니다:
   ```bash
   php -S localhost:8000 -t public/
   ```
   `http://localhost:8000`에 접속하면 "Hello, Flight!"가 표시됩니다.

## 2단계: 프로젝트 구조 정리

깔끔한 구성을 위해 프로젝트를 다음과 같이 구성합니다:

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

- `app/config/`: 설정 파일 (예: 이벤트, 라우트).
- `app/views/`: 페이지 렌더링을 위한 템플릿.
- `data/`: 블로그 게시물을 저장하는 JSON 파일.
- `public/`: `index.php`가 있는 웹 루트.

## 3단계: Latte 설치 및 구성

Latte는 Flight와 잘 통합되는 가벼운 템플릿 엔진입니다.

1. **Latte 설치**:
   ```bash
   composer require latte/latte
   ```

2. **Flight에서 Latte 구성**:
   `public/index.php`를 업데이트하여 Latte를 뷰 엔진으로 등록합니다:
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

3. **레이아웃 템플릿 만들기:
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

4. **홈 템플릿 만들기**:
   `app/views/home.latte`에 다음을 작성합니다:
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
   서버에서 나왔다면 다시 시작하고 `http://localhost:8000`에 접속하여 렌더링된 페이지를 확인합니다.

5. **데이터 파일 만들기**:

   간단히 하기 위해 JSON 파일을 데이터베이스처럼 사용합니다.

   `data/posts.json`에 다음을 작성합니다:
   ```json
   [
       {
           "slug": "first-post",
           "title": "My First Post",
           "content": "This is my very first blog post with Flight PHP!"
       }
   ]
   ```

## 4단계: 라우트 정의

더 나은 구성을 위해 라우트를 설정 파일로 분리합니다.

1. **`routes.php` 만들기**:
   `app/config/routes.php`에 다음을 작성합니다:
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

2. **`index.php` 업데이트**:
   라우트 파일을 포함합니다:
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

## 5단계: 블로그 게시물 저장 및 불러오기

게시물을 로드하고 저장하는 메서드를 추가합니다.

1. **게시물 메서드 추가**:
   `index.php`에서 게시물을 로드하는 메서드를 추가합니다:
   ```php
   Flight::map('posts', function () {
       $file = __DIR__ . '/../data/posts.json';
       return json_decode(file_get_contents($file), true);
   });
   ```

2. **라우트 업데이트**:
   `app/config/routes.php`를 게시물을 사용하도록 수정합니다:
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

## 6단계: 템플릿 만들기

게시물을 표시하도록 템플릿을 업데이트합니다.

1. **게시물 페이지 (`app/views/post.latte`)**:
   ```html
   {extends 'layout.latte'}

	{block content}
		<h2>{$post['title']}</h2>
		<div class="post-content">
			<p>{$post['content']}</p>
		</div>
	{/block}
   ```

## 7단계: 게시물 작성 기능 추가

양식 제출을 처리하여 새 게시물을 추가합니다.

1. **양식 만들기 (`app/views/create.latte`)**:
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

2. **POST 라우트 추가**:
   `app/config/routes.php`에서:
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

3. **테스트**:
   - `http://localhost:8000/create`에 접속합니다.
   - 새 게시물을 제출합니다 (예: "Second Post"와 내용).
   - 홈페이지에서 해당 게시물이 목록에 표시되는지 확인합니다.

## 8단계: 오류 처리 개선

더 나은 404 환경을 위해 `notFound` 메서드를 재정의합니다.

`index.php`에서:
```php
Flight::map('notFound', function () {
    Flight::view()->render('404.latte', ['title' => 'Page Not Found']);
});
```

`app/views/404.latte`를 만듭니다:
```html
{extends 'layout.latte'}

{block content}
    <h2>404 - {$title}</h2>
    <p>Sorry, that page doesn't exist!</p>
{/block}
```

## 다음 단계
- **스타일 추가**: 템플릿에 CSS를 사용하여 더 나은 디자인을 적용합니다.
- **데이터베이스**: [SimplePdo](/learn/simple-pdo)를 사용하여 `posts.json`을 SQLite 같은 데이터베이스로 대체합니다.
- **유효성 검사**: 중복 슬러그나 빈 입력에 대한 검사를 추가합니다.
- **미들웨어**: 게시물 작성에 인증을 구현합니다.

## 결론

Flight PHP로 간단한 블로그를 만들었습니다! 이 가이드는 라우팅, Latte를 사용한 템플릿 처리, 양식 제출 처리와 같은 핵심 기능을 가볍게 유지하면서 보여줍니다. 블로그를 더 발전시키려면 Flight의 문서에서 더 많은 고급 기능을 살펴보세요!