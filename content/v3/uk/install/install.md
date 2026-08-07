# Інструкції зі встановлення

Перш ніж встановити Flight, потрібні деякі базові передумови. Зокрема вам знадобиться:

1. [Встановити PHP на вашій системі](#installing-php)
2. [Встановити Composer](https://getcomposer.org) для найкращого досвіду розробника.

## Базове встановлення

Якщо ви користуєтеся [Composer](https://getcomposer.org), ви можете виконати наступну
команду:

```bash
composer require flightphp/core
```

Це розмістить на вашій системі лише файли ядра Flight. Вам потрібно буде визначити структуру проєкту, [макети](/learn/templates), [залежності](/learn/dependency-injection-container), [конфігурації](/learn/configuration), [автозавантаження](/learn/autoloading) тощо. Цей метод гарантує, що жодні інші залежності, окрім Flight, не будуть встановлені.

Ви також можете [завантажити файли](https://github.com/flightphp/core/archive/master.zip)
 безпосередньо та розпакувати їх у вашу вебдиректорію.

Базове встановлення чудово підходить для навчання, мікро API та експериментів із копіюванням і вставкою. Для повного макету застосунку, якого люди *і* [AI інструменти для кодування](/learn/ai) можуть дотримуватися однаково, використовуйте рекомендований скелет нижче.

## Рекомендоване встановлення

Настійно рекомендується починати з застосунку [flightphp/skeleton](https://github.com/flightphp/skeleton) для будь-яких нових проєктів. Встановлення дуже просте.

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# необов'язкова демонстрація бази даних + пости
php runway migrate
```

Цей крок налаштовує структуру проєкту, автозавантаження Composer PSR-4, конфігурацію та інструменти, як-от [Tracy](/awesome-plugins/tracy), [Tracy Extensions](/awesome-plugins/tracy-extensions) та [Runway](/awesome-plugins/runway). Він також постачає кореневий файл **`AGENTS.md`** (та копії в межах `app/`), щоб AI-асистенти мали спільний макет із вами — див. [AI та досвід розробника](/learn/ai).

### Що надає скелет

```text
project-root/
├── AGENTS.md              # Джерело істини для AI / агентів
├── SECURITY.md            # Очікування щодо безпеки
├── .env.example           # Секрети / накладення для розгортання (копіюється в .env)
├── public/index.php       # Лише вебвхід
├── app/
│   ├── config/            # bootstrap, маршрути, сервіси, config_sample.php
│   ├── Controller/        # App\Controller\*  (папка в PascalCase!)
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\* (ActiveRecord)
│   ├── Utils/             # Config, Env, DatabaseFactory
│   ├── commands/          # CLI-команди Runway
│   ├── views/             # Twig-шаблони (*.twig)
│   ├── cache/
│   └── log/
├── migrations/            # SQL-міграції (.sql / .mysql.sql)
└── tests/                 # PHPUnit
```

**Простори імен відповідають регістру папок.** Composer зіставляє `"App\\": "app/"`, отже:

| Шлях на диску | Простір імен |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

На Linux `app/controller/` **не те саме**, що `app/Controller/`. Автозавантаження чутливе до регістру — відповідайте папкам скелета в PascalCase. Деталі: [Автозавантаження](/learn/autoloading).

**Типові складові стеку (нові проєкти):** Twig-шаблони, SimplePdo + ActiveRecord, Dice з ін'єкцією `Engine` (надавайте перевагу відсутності `Flight::` усередині класів застосунку), опційно SQLite після `php runway migrate`.

`create-project` зазвичай копіює `app/config/config_sample.php` → `config.php` та `.env.example` → `.env`, якщо вони присутні. Маршрути розташовані в `app/config/routes.php`; сервіси та DI — у `app/config/services.php`.

> **Документація ↔ скелет:** Ця документація навчає **API** Flight (часто з короткими прикладами `Flight::`). Скелет визначає **форму застосунку**. Коли додаєте код у `app/`, дотримуйтеся дерева скелета; використовуйте документацію для назв методів, параметрів і плагінів.

## Налаштування вебсервера

### Вбудований PHP-сервер розробки

Це найпростіший спосіб почати роботу. Ви можете використовувати вбудований сервер для запуску застосунку і навіть використовувати SQLite як базу даних (за умови, що sqlite3 встановлено у вашій системі) без особливих додаткових вимог! Просто виконайте наступну команду після встановлення PHP:

```bash
php -S localhost:8000
# або зі скелетним застосунком
composer start
```

Потім відкрийте браузер і перейдіть на `http://localhost:8000`.

Якщо ви хочете зробити коренем документів вашого проєкту іншу директорію (Наприклад: ваш проєкт знаходиться в `~/myproject`, але ваш корінь документів — `~/myproject/public/`), ви можете виконати наступну команду, перебуваючи в директорії `~/myproject`:

```bash
php -S localhost:8000 -t public/
# зі скелетним застосунком це вже налаштовано
composer start
```

Потім відкрийте браузер і перейдіть на `http://localhost:8000`.

### Apache

Переконайтеся, що Apache вже встановлено у вашій системі. Якщо ні, загугліть, як встановити Apache у вашій системі.

Для Apache відредагуйте ваш файл `.htaccess` наступним чином:

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **Примітка**: Якщо вам потрібно використовувати flight у піддиректорії, додайте рядок
> `RewriteBase /subdir/` одразу після `RewriteEngine On`.

> **Примітка**: Якщо ви хочете захистити всі серверні файли, наприклад файл бази даних або `.env`.
> Помістіть це у ваш файл `.htaccess`:

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

Переконайтеся, що Nginx вже встановлено у вашій системі. Якщо ні, загугліть, як встановити Nginx у вашій системі.

Для Nginx додайте наступне до оголошення вашого сервера:

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## Створіть ваш файл `index.php`

Якщо ви виконуєте базове встановлення, вам знадобиться трохи коду для початку.

```php
<?php

// Якщо ви використовуєте Composer, підключіть автозавантажувач.
require 'vendor/autoload.php';
// якщо ви не використовуєте Composer, завантажте фреймворк безпосередньо
// require 'flight/Flight.php';

// Потім визначте маршрут і призначте функцію для обробки запиту.
Flight::route('/', function () {
  echo 'hello world!';
});

// Нарешті, запустіть фреймворк.
Flight::start();
```

Зі скелетним застосунком публічна точка входу лише запускає застосунок. Маршрути реєструються в `app/config/routes.php` (зазвичай `[App\Controller\…::class, 'method']`, щоб Dice міг впроваджувати залежності). Сервіси, Twig, SimplePdo та контейнер налаштовані в `app/config/services.php`. Ця структура є навмисною, щоб AI-інструменти та люди редагували одні й ті самі місця щоразу.

## Встановлення PHP

Якщо у вашій системі вже встановлено `php`, можете пропустити ці інструкції та перейти до [розділу завантаження](#download-the-files)

### **macOS**

#### **Встановлення PHP за допомогою Homebrew**

1. **Встановіть Homebrew** (якщо ще не встановлено):
   - Відкрийте Terminal і виконайте:
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **Встановіть PHP**:
   - Встановіть останню версію:
     ```bash
     brew install php
     ```
   - Щоб встановити конкретну версію, наприклад, PHP 8.1:
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **Перемикання між версіями PHP**:
   - Видаліть поточну версію зі зв'язку та зв'яжіть бажану версію:
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - Перевірте встановлену версію:
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **Ручне встановлення PHP**

1. **Завантажте PHP**:
   - Відвідайте [PHP для Windows](https://windows.php.net/download/) та завантажте останню або конкретну версію (наприклад, 7.4, 8.0) як zip-файл без потокової безпеки (non-thread-safe).

2. **Розпакуйте PHP**:
   - Розпакуйте завантажений zip-файл у `C:\php`.

3. **Додайте PHP до системного PATH**:
   - Перейдіть до **System Properties** > **Environment Variables**.
   - У розділі **System variables** знайдіть **Path** і натисніть **Edit**.
   - Додайте шлях `C:\php` (або куди ви розпакували PHP).
   - Натисніть **OK**, щоб закрити всі вікна.

4. **Налаштуйте PHP**:
   - Скопіюйте `php.ini-development` у `php.ini`.
   - Відредагуйте `php.ini`, щоб налаштувати PHP за потреби (наприклад, встановити `extension_dir`, увімкнути розширення).

5. **Перевірте встановлення PHP**:
   - Відкрийте Command Prompt і виконайте:
     ```cmd
     php -v
     ```

#### **Встановлення кількох версій PHP**

1. **Повторіть наведені вище кроки** для кожної версії, розміщуючи кожну в окремій директорії (наприклад, `C:\php7`, `C:\php8`).

2. **Перемикайтеся між версіями**, змінюючи системну змінну PATH, щоб вказувати на бажану директорію версії.

### **Ubuntu (20.04, 22.04 тощо)**

#### **Встановлення PHP за допомогою apt**

1. **Оновіть списки пакетів**:
   - Відкрийте Terminal і виконайте:
     ```bash
     sudo apt update
     ```

2. **Встановіть PHP**:
   - Встановіть останню версію PHP:
     ```bash
     sudo apt install php
     ```
   - Щоб встановити конкретну версію, наприклад, PHP 8.1:
     ```bash
     sudo apt install php8.1
     ```

3. **Встановіть додаткові модулі** (необов'язково):
   - Наприклад, щоб встановити підтримку MySQL:
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **Перемикання між версіями PHP**:
   - Використовуйте `update-alternatives`:
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **Перевірте встановлену версію**:
   - Виконайте:
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **Встановлення PHP за допомогою yum/dnf**

1. **Увімкніть сховище EPEL**:
   - Відкрийте Terminal і виконайте:
     ```bash
     sudo dnf install epel-release
     ```

2. **Встановіть сховище Remi**:
   - Виконайте:
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **Встановіть PHP**:
   - Щоб встановити версію за замовчуванням:
     ```bash
     sudo dnf install php
     ```
   - Щоб встановити конкретну версію, наприклад, PHP 7.4:
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **Перемикання між версіями PHP**:
   - Використовуйте команду модуля `dnf`:
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **Перевірте встановлену версію**:
   - Виконайте:
     ```bash
     php -v
     ```

### **Загальні примітки**

- Для середовищ розробки важливо налаштувати параметри PHP відповідно до вимог вашого проєкту.
- Під час перемикання версій PHP переконайтеся, що всі необхідні розширення PHP встановлені для конкретної версії, яку ви збираєтеся використовувати.
- Перезапустіть ваш вебсервер (Apache, Nginx тощо) після перемикання версій PHP або оновлення конфігурацій, щоб застосувати зміни.