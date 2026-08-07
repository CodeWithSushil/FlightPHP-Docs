# Runway

Runway — це CLI-додаток, який допомагає керувати вашими Flight додатками. Він може генерувати контролери, відображати всі маршрути, запускати AI-помічники налаштування, міграції (у скелеті) та інше. Він базується на чудовій бібліотеці [adhocore/php-cli](https://github.com/adhocore/php-cli).

Натисніть [тут](https://github.com/flightphp/runway), щоб переглянути код.

Команди скафолдингу навмисно узгоджені з [офіційним скелетом](https://github.com/flightphp/skeleton), щоб [AI-інструменти кодування](/learn/ai) та люди отримували однакові шляхи, простори імен та стиль конструктор-ін'єкції кожного разу.

## Встановлення

Встановіть за допомогою composer.

```bash
composer require flightphp/runway
```

Скелет вже залежить від Runway; використовуйте `php runway` з кореня проекту.

## Базова конфігурація

При першому запуску Runway спробує знайти конфігурацію `runway` у `app/config/config.php` через ключ `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// optional; skeleton also uses index_root for the public entry
		'index_root' => 'public/index.php',
    ],
];
```

> **ПРИМІТКА** - Станом на **v1.2.0**, `.runway-config.json` застарів на користь `app/config/config.php`. Міграціюйте за допомогою `php runway config:migrate` при оновленні старих проектів. Скелет все ще може записувати невеликий `.runway-config.json` при create-project для сумісності; віддавайте перевагу ключу `runway` у `config.php` надалі.

### Визначення кореня проекту

Runway достатньо розумний, щоб визначити корінь вашого проекту, навіть якщо ви запускаєте його з підкаталогу. Він шукає індикатори типу `composer.json`, `.git` або `app/config/config.php`, щоб визначити, де знаходиться корінь проекту. Це означає, що ви можете запускати команди Runway з будь-якого місця у вашому проекті!

## Використання

Runway має ряд команд, які ви можете використовувати для керування вашим Flight додатком. Є два простих способи використання Runway.

1. Якщо ви використовуєте проект-скелет, ви можете запустити `php runway [команда]` з кореня вашого проекту.
1. Якщо ви використовуєте Runway як пакет, встановлений через composer, ви можете запустити `vendor/bin/runway [команда]` з кореня вашого проекту.

### Список команд

Ви можете переглянути список усіх доступних команд, виконавши команду `php runway`.

```bash
php runway
```

Покладайтеся лише на команди, які дійсно з'являються у цьому списку для вашої установки (основні команди Runway проти специфічних для проекту, таких як `migrate` скелета).

### Довідка по команді

Для будь-якої команди ви можете передати прапорець `--help`, щоб отримати більше інформації про те, як використовувати команду.

```bash
php runway routes --help
php runway make:controller --help
```

Ось кілька прикладів:

### Генерація контролера

`make:controller` створює скафолдинг контролера, який відповідає макету офіційного скелета:

| | |
|--|--|
| **Шлях** | `app/Controller/{Name}.php` |
| **Простір імен** | `App\Controller` |
| **Стиль** | Ін'єкція конструктора `flight\Engine` (без `Flight::` у тілі класу) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Приклад форми, яку ви повинні очікувати (спрощено):

```php
<?php

declare(strict_types=1);

namespace App\Controller;

use flight\Engine;

class MyController
{
	protected Engine $app;

	public function __construct(Engine $app)
	{
		$this->app = $app;
	}

	public function index(): void
	{
		// e.g. $this->app->render('…', […]);
	}
}
```

Зареєструйте його з class callable, щоб Dice міг побудувати контролер:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Чому саме такий макет?** Регістр **папки** повинен відповідати простору імен (`Controller` не `controllers`) для Composer PSR-4 на Linux—див. [Автозавантаження](/learn/autoloading). Той самий шлях — це те, що кореневі та scoped файли `AGENTS.md` вказують AI-інструментам використовувати, тому згенеровані та написані вручну контролери залишаються ідентичними.

> Старіша документація та спільнотні проекти іноді використовували `app/controllers/` та `app\controllers`. Це залишається дійсним, якщо *ваше* дерево все ще використовує папки з малими літерами. **Нові проекти скелета та поточний вивід `make:controller` використовують `app/Controller/` + `App\Controller`.**

### Генерація моделі Active Record

Спочатку переконайтеся, що ви встановили плагін [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record users
```

В офіційному скелеті моделі живуть під **`app/Model/`** з простором імен **`App\Model`**, а з'єднання з БД — це **[SimplePdo](/learn/simple-pdo)** (ін'єктуйте його або передайте у конструктор ActiveRecord). Назви файлів/просторів імен генеруються відповідно до поточних налаштувань Runway та вашої конфігурації `runway`—віддавайте перевагу узгодженню нових моделей з `App\Model`, щоб вони відповідали [автозавантаженню](/learn/autoloading) та `AGENTS.md`.

Приклад моделі, узгодженої з демо постів скелета:

```php
<?php

declare(strict_types=1);

namespace App\Model;

use flight\ActiveRecord;

/**
 * @property int $id
 * @property string $title
 * // …
 */
class Post extends ActiveRecord
{
	protected array $relations = [];

	public function __construct($databaseConnection)
	{
		parent::__construct($databaseConnection, 'posts');
	}
}
```

Якщо старіший генератор все ще видає `app/records` / `app\records`, ви можете зберегти цю конвенцію у застарілих додатках або перемістити файли у `app/Model/` та оновити простір імен відповідно до регістру папки.

### Міграції (скелет)

Офіційний скелет постачає проектну команду (виявлену з `app/commands/`), таку як:

```bash
php runway migrate
```

Міграції — це SQL-файли під `migrations/` (наприклад `YYYYMMDDHHMMSS_description.sql` для SQLite та `…_description.mysql.sql` для MySQL), вибрані з конфігурації драйвера бази даних / env. Точні прапорці та поведінка визначаються цією проектною командою—запустіть `php runway migrate --help` у вашому додатку.

### AI-помічники

Runway надає AI-орієнтовані команди, що використовуються з [AI та досвідом розробника](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Вони зберігають облікові дані LLM та генерують інструкції проекту (в основному **`AGENTS.md`**). На скелеті, розглядайте `AGENTS.md` (та scoped копії під `app/`) плюс **`SECURITY.md`** як джерело правди для агентів.

### Відображення всіх маршрутів

Це відобразить усі маршрути, які зараз зареєстровані у Flight.

```bash
php runway routes
```

Якщо ви хочете переглядати лише специфічні маршрути, ви можете передати прапорець для фільтрації маршрутів.

```bash
# Відобразити лише GET маршрути
php runway routes --get

# Відобразити лише POST маршрути
php runway routes --post

# тощо.
```

## Додавання власних команд до Runway

Якщо ви створюєте пакет для Flight або хочете додати власні команди у ваш проект, ви можете зробити це, створивши директорію `src/commands/`, `flight/commands/`, `app/commands/` або `commands/` для вашого проекту/пакету. Якщо вам потрібна подальша кастомізація, дивіться розділ нижче про Конфігурацію.

У скелеті проектні команди живуть у **`app/commands/`** з простором імен **`App\Command`**. Runway виявляє їх за шляхом; тримайте цю папку синхронізованою з Composer classmap/PSR-4, як це вже робить ваш проект.

Щоб створити команду, просто розширьте клас `AbstractBaseCommand` та реалізуйте щонайменше метод `__construct` та метод `execute`.

```php
<?php

declare(strict_types=1);

namespace App\Command;

use flight\commands\AbstractBaseCommand;

class ExampleCommand extends AbstractBaseCommand
{
	/**
     * Конструктор
     *
     * @param array<string,mixed> $config Конфігурація з app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Створити приклад для документації', $config);
        $this->argument('<funny-gif>', 'Назва кумедного gif');
    }

	/**
     * Виконує функцію
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Створення прикладу...');

		// Зробіть щось тут

		$io->ok('Приклад створено!');
	}
}
```

Дивіться [Документацію adhocore/php-cli](https://github.com/adhocore/php-cli) для отримання додаткової інформації про те, як побудувати власні команди у ваш Flight додаток!

## Управління конфігурацією

Оскільки конфігурація перемістилася до `app/config/config.php` станом на `v1.2.0`, є кілька допоміжних команд для управління конфігурацією.

> **Порада скелета:** Тримайте `config.php` як **літеральні** PHP-значення. Секрети належать у `.env`. Уникайте виразів `$_ENV[...]` всередині `config.php`—`config:set` перезаписує цей файл як статичні дані і може вбудувати секрети у файл. Див. [Конфігурація](/learn/configuration).

### Міграція старої конфігурації

Якщо у вас є старий файл `.runway-config.json`, ви можете легко мігрувати його до `app/config/config.php` за допомогою наступної команды:

```bash
php runway config:migrate
```

### Встановлення значення конфігурації

Ви можете встановити значення конфігурації за допомогою команди `config:set`. Це корисно, якщо ви хочете оновити значення конфігурації без відкриття файлу.

```bash
php runway config:set app_root "app/"
```

### Отримання значення конфігурації

Ви можете отримати значення конфігурації за допомогою команди `config:get`.

```bash
php runway config:get app_root
```

## Усі конфігурації Runway

Якщо вам потрібно кастомізувати конфігурацію для Runway, ви можете встановити ці значення у `app/config/config.php`. Нижче наведено деякі додаткові конфігурації, які ви можете встановити:

```php
<?php
// app/config/config.php
return [
    // ... інші значення конфігурації ...

    'runway' => [
        // Це місце, де знаходиться директорія вашого додатку
        'app_root' => 'app/',

        // Це директорія, де знаходиться ваш кореневий index файл
        'index_root' => 'public/',

        // Це шляхи до коренів інших проектів
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Базові шляхи, ймовірно, не потребують конфігурації, але вони тут, якщо ви цього хочете
        'base_paths' => [
            '/includes/libs/vendor', // якщо у вас дійсно унікальний шлях до вашої vendor директорії чи щось подібне
        ],

        // Фінальні шляхи — це локації в проекті для пошуку файлів команд
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Якщо ви хочете просто додати повний шлях, вперед (абсолютний або відносний до кореня проекту)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Доступ до конфігурації

Якщо вам потрібно ефективно отримати доступ до значень конфігурації, ви можете отримати їх через метод `__construct` або метод `app()`. Також важливо зазначити, що якщо у вас є файл `app/config/services.php`, ці сервіси також будуть доступні для вашої команди.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Доступ до конфігурації
    $app_root = $this->config['runway']['app_root'];
    
    // Доступ до сервісів, таких як, можливо, з'єднання з базою даних
    $database = $this->config['database']
    
    // ...
}
```

## AI-помічники-обгортки

Runway має деякі допоміжні обгортки, які полегшують AI генерувати команди. Ви можете використовувати `addOption` та `addArgument` у спосіб, схожий на Symfony Console. Це корисно, якщо ви використовуєте AI-інструменти для генерації ваших команд.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Створити приклад для документації', $config);
    
    // Аргумент mode є nullable і за замовчуванням повністю опціональний
    $this->addOption('name', 'Назва прикладу', null);
}
```

## Дивіться також

- [Встановлення](/install) - Дерево скелета та налаштування create-project
- [Автозавантаження](/learn/autoloading) - `App\` та регістр папки
- [Впровадження залежностей](/learn/dependency-injection-container) - Dice + ін'єкція Engine для згенерованих контролерів
- [AI та досвід розробника](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Моделі, що використовуються з `make:record` / скелет `App\Model`
- [SimplePdo](/learn/simple-pdo) - З'єднання з БД, що використовується міграціями та моделями скелета