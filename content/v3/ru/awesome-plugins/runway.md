# Runway

Runway — это CLI-приложение, которое помогает управлять вашими приложениями Flight. Оно может генерировать контроллеры, отображать все маршруты, запускать помощники настройки ИИ, миграции (в скелете) и многое другое. Оно основано на отличной библиотеке [adhocore/php-cli](https://github.com/adhocore/php-cli).

Нажмите [здесь](https://github.com/flightphp/runway), чтобы просмотреть код.

Команды скаффолдинга намеренно согласованы с [официальным скелетом](https://github.com/flightphp/skeleton), поэтому [инструменты кодирования ИИ](/learn/ai) и люди получают одинаковые пути, пространства имён и стиль конструкторской инъекции каждый раз.

## Установка

Установите с помощью composer.

```bash
composer require flightphp/runway
```

Скелет уже зависит от Runway; используйте `php runway` из корня проекта.

## Базовая конфигурация

При первом запуске Runway попытается найти конфигурацию `runway` в `app/config/config.php` через ключ `'runway'`.

```php
<?php
// app/config/config.php
return [
    'runway' => [
        'app_root' => 'app/',
		'public_root' => 'public/',
		// необязательно; скелет также использует index_root для публичной точки входа
		'index_root' => 'public/index.php',
    ],
];
```

> **ПРИМЕЧАНИЕ** - Начиная с **v1.2.0**, `.runway-config.json` устарел в пользу `app/config/config.php`. Мигрируйте с помощью `php runway config:migrate` при обновлении старых проектов. Скелет может всё ещё создавать небольшой `.runway-config.json` при create-project для совместимости; предпочтите ключ `runway` в `config.php` в будущем.

### Определение корня проекта

Runway достаточно умен, чтобы определить корень вашего проекта, даже если вы запускаете его из подкаталога. Он ищет такие индикаторы, как `composer.json`, `.git` или `app/config/config.php`, чтобы определить, где находится корень проекта. Это означает, что вы можете запускать команды Runway из любого места вашего проекта!

## Использование

Runway имеет ряд команд, которые вы можете использовать для управления вашим приложением Flight. Есть два простых способа использовать Runway.

1. Если вы используете проект скелета, вы можете запустить `php runway [команда]` из корня вашего проекта.
1. Если вы используете Runway как пакет, установленный через composer, вы можете запустить `vendor/bin/runway [команда]` из корня вашего проекта.

### Список команд

Вы можете просмотреть список всех доступных команд, выполнив команду `php runway`.

```bash
php runway
```

Полагайтесь только на команды, которые действительно появляются в этом списке для вашей установки (основные команды Runway против специфичных для проекта, таких как `migrate` скелета).

### Справка по команде

Для любой команды вы можете передать флаг `--help`, чтобы получить больше информации о том, как использовать команду.

```bash
php runway routes --help
php runway make:controller --help
```

Вот несколько примеров:

### Генерация контроллера

`make:controller` создаёт каркас контроллера, который соответствует макету официального скелета:

| | |
|--|--|
| **Путь** | `app/Controller/{Name}.php` |
| **Пространство имён** | `App\Controller` |
| **Стиль** | Конструкторская инъекция `flight\Engine` (без `Flight::` в теле класса) |

```bash
php runway make:controller MyController
# → app/Controller/MyController.php
#   namespace App\Controller;
```

Пример ожидаемой структуры (упрощённый):

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
		// например $this->app->render('…', […]);
	}
}
```

Зарегистрируйте его с помощью callable класса, чтобы Dice мог создать контроллер:

```php
// app/config/routes.php
use App\Controller\MyController;

$router->get('/mine', [MyController::class, 'index']);
```

**Почему такая структура?** Регистр **папки** должен соответствовать пространству имён (`Controller`, а не `controllers`) для Composer PSR-4 на Linux — см. [Автозагрузку](/learn/autoloading). Тот же путь используется в корневых и scoped файлах `AGENTS.md`, которые указывают инструментам ИИ, что использовать, поэтому сгенерированные и написанные вручную контроллеры остаются идентичными.

> Старые документации и общественные проекты иногда использовали `app/controllers/` и `app\controllers`. Это остаётся допустимым, если *ваше* дерево всё ещё использует строчные папки. **Новые проекты скелета и текущий вывод `make:controller` используют `app/Controller/` + `App\Controller`.**

### Генерация модели Active Record

Сначала убедитесь, что вы установили плагин [Active Record](/awesome-plugins/active-record).

```bash
php runway make:record users
```

В официальном скелете модели находятся в **`app/Model/`** с пространством имён **`App\Model`**, а соединение с БД — **[SimplePdo](/learn/simple-pdo)** (внедрите его или передайте в конструктор ActiveRecord). Имена файлов/пространств имён сгенерированных файлов следуют текущим значениям по умолчанию Runway и вашей конфигурации `runway` — предпочтите согласование новых моделей с `App\Model`, чтобы они соответствовали [автозагрузке](/learn/autoloading) и `AGENTS.md`.

Пример модели, согласованной с демо постов скелета:

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

Если старый генератор всё ещё создаёт `app/records` / `app\records`, вы можете сохранить это соглашение в устаревших приложениях или переместить файлы в `app/Model/` и обновить пространство имён в соответствии с регистром папки.

### Миграции (скелет)

Официальный скелет поставляет проектную команду (обнаруженную из `app/commands/`), такую как:

```bash
php runway migrate
```

Миграции — это SQL-файлы в `migrations/` (например `YYYYMMDDHHMMSS_description.sql` для SQLite и `…_description.mysql.sql` для MySQL), выбранные из конфигурации драйвера базы данных / env. Точные флаги и поведение определяются этой проектной командой — запустите `php runway migrate --help` в вашем приложении.

### Помощники ИИ

Runway предоставляет команды, ориентированные на ИИ, используемые с [ИИ и опытом разработчика](/learn/ai):

```bash
php runway ai:init
php runway ai:generate-instructions
```

Они хранят учётные данные LLM и генерируют инструкции проекта (в первую очередь **`AGENTS.md`**). В скелете рассматривайте `AGENTS.md` (и scoped копии в `app/`) плюс **`SECURITY.md`** как источник истины для агентов.

### Отображение всех маршрутов

Это отобразит все маршруты, которые в настоящее время зарегистрированы в Flight.

```bash
php runway routes
```

Если вы хотите просмотреть только определённые маршруты, вы можете передать флаг для фильтрации маршрутов.

```bash
# Отобразить только GET маршруты
php runway routes --get

# Отобразить только POST маршруты
php runway routes --post

# и т.д.
```

## Добавление пользовательских команд в Runway

Если вы создаёте пакет для Flight или хотите добавить свои собственные пользовательские команды в свой проект, вы можете сделать это, создав директорию `src/commands/`, `flight/commands/`, `app/commands/` или `commands/` для вашего проекта/пакета. Если вам нужна дальнейшая кастомизация, см. раздел ниже о Конфигурации.

В скелете проектные команды находятся в **`app/commands/`** с пространством имён **`App\Command`**. Runway обнаруживает их по пути; синхронизируйте эту папку с classmap/PSR-4 Composer, как это делает ваш проект.

Чтобы создать команду, просто расширьте класс `AbstractBaseCommand` и реализуйте как минимум метод `__construct` и метод `execute`.

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
     * @param array<string,mixed> $config Конфигурация из app/config/config.php
     */
    public function __construct(array $config)
    {
        parent::__construct('make:example', 'Создать пример для документации', $config);
        $this->argument('<funny-gif>', 'Название забавного gif');
    }

	/**
     * Выполняет функцию
     *
     * @return void
     */
    public function execute()
    {
        $io = $this->app()->io();

		$io->info('Создание примера...');

		// Сделайте что-то здесь

		$io->ok('Пример создан!');
	}
}
```

См. [Документацию adhocore/php-cli](https://github.com/adhocore/php-cli) для получения дополнительной информации о том, как создавать свои собственные пользовательские команды в вашем приложении Flight!

## Управление конфигурацией

Поскольку конфигурация была перенесена в `app/config/config.php` начиная с `v1.2.0`, есть несколько вспомогательных команд для управления конфигурацией.

> **Совет по скелету:** Держите `config.php` как **литеральные** PHP-значения. Секреты должны находиться в `.env`. Избегайте выражений `$_ENV[...]` внутри `config.php` — `config:set` переписывает этот файл как статические данные и может встроить секреты в файл. См. [Конфигурацию](/learn/configuration).

### Миграция старой конфигурации

Если у вас есть старый файл `.runway-config.json`, вы можете легко перенести его в `app/config/config.php` с помощью следующей команды:

```bash
php runway config:migrate
```

### Установка значения конфигурации

Вы можете установить значение конфигурации с помощью команды `config:set`. Это полезно, если вы хотите обновить значение конфигурации без открытия файла.

```bash
php runway config:set app_root "app/"
```

### Получение значения конфигурации

Вы можете получить значение конфигурации с помощью команды `config:get`.

```bash
php runway config:get app_root
```

## Все конфигурации Runway

Если вам нужно настроить конфигурацию для Runway, вы можете установить эти значения в `app/config/config.php`. Ниже приведены некоторые дополнительные конфигурации, которые вы можете установить:

```php
<?php
// app/config/config.php
return [
    // ... другие значения конфигурации ...

    'runway' => [
        // Здесь находится директория вашего приложения
        'app_root' => 'app/',

        // Это директория, где находится ваш корневой index-файл
        'index_root' => 'public/',

        // Это пути к корням других проектов
        'root_paths' => [
            '/home/user/different-project',
            '/var/www/another-project'
        ],

        // Базовые пути, скорее всего, не нуждаются в настройке, но они здесь, если вам нужно
        'base_paths' => [
            '/includes/libs/vendor', // если у вас действительно уникальный путь к директории vendor или что-то подобное
        ],

        // Конечные пути — это места в проекте для поиска файлов команд
        'final_paths' => [
            'src/diff-path/commands',
            'app/module/admin/commands',
        ],

        // Если вы хотите просто добавить полный путь, вперёд (абсолютный или относительный к корню проекта)
        'paths' => [
            '/home/user/different-project/src/diff-path/commands',
            '/var/www/another-project/app/module/admin/commands',
            'app/my-unique-commands'
        ]
    ]
];
```

### Доступ к конфигурации

Если вам нужно эффективно получить доступ к значениям конфигурации, вы можете получить к ним доступ через метод `__construct` или метод `app()`. Также важно отметить, что если у вас есть файл `app/config/services.php`, эти сервисы также будут доступны для вашей команды.

```php
public function execute()
{
    $io = $this->app()->io();
    
    // Доступ к конфигурации
    $app_root = $this->config['runway']['app_root'];
    
    // Доступ к сервисам, например, к соединению с базой данных
    $database = $this->config['database']
    
    // ...
}
```

## Обёртки помощников ИИ

Runway имеет некоторые обёртки помощников, которые облегчают генерацию команд ИИ. Вы можете использовать `addOption` и `addArgument` способом, который похож на Symfony Console. Это полезно, если вы используете инструменты ИИ для генерации ваших команд.

```php
public function __construct(array $config)
{
    parent::__construct('make:example', 'Создать пример для документации', $config);
    
    // Аргумент mode может быть null и по умолчанию полностью необязателен
    $this->addOption('name', 'Название примера', null);
}
```

## См. также

- [Установка](/install) - Дерево скелета и значения по умолчанию create-project
- [Автозагрузка](/learn/autoloading) - `App\` и регистр папки
- [Внедрение зависимостей](/learn/dependency-injection-container) - Dice + инъекция Engine для сгенерированных контроллеров
- [ИИ и опыт разработчика](/learn/ai) - `ai:init`, `ai:generate-instructions`, `AGENTS.md`
- [Active Record](/awesome-plugins/active-record) - Модели, используемые с `make:record` / скелетом `App\Model`
- [SimplePdo](/learn/simple-pdo) - Соединение с БД, используемое миграциями и моделями скелета