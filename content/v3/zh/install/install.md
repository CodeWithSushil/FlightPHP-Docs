# 安装说明

在安装 Flight 之前，有一些基本的先决条件。即你需要：

1. [在系统上安装 PHP](#installing-php)
2. [安装 Composer](https://getcomposer.org) 以获得最佳开发者体验。

## 基本安装

如果你使用 [Composer](https://getcomposer.org)，可以运行以下命令：

```bash
composer require flightphp/core
```

这只会将 Flight 核心文件安装到你的系统上。你需要自行定义项目结构、[布局](/learn/templates)、[依赖](/learn/dependency-injection-container)、[配置](/learn/configuration)、[自动加载](/learn/autoloading) 等。这种方法确保除了 Flight 之外不会安装其他依赖。

你也可以直接[下载文件](https://github.com/flightphp/core/archive/master.zip) 并将它们解压到你的 Web 目录中。

基本安装非常适合学习、微型 API 和复制粘贴实验。要获得人类*和*[AI 编码工具](/learn/ai)都能以相同方式遵循的完整应用布局，请使用下面推荐的骨架。

## 推荐安装

强烈建议任何新项目都从 [flightphp/skeleton](https://github.com/flightphp/skeleton) 应用开始。安装非常简单。

```bash
composer create-project flightphp/skeleton my-project/
cd my-project/
composer start
# 可选的示例数据库 + 帖子演示
php runway migrate
```

这一步会设置项目结构、Composer PSR-4 自动加载、配置以及 [Tracy](/awesome-plugins/tracy)、[Tracy 扩展](/awesome-plugins/tracy-extensions) 和 [Runway](/awesome-plugins/runway) 等工具。它还会附带根目录的 **`AGENTS.md`**（以及 `app/` 下的作用域副本），以便 AI 助手与你共享同一布局——请参阅 [AI 与开发者体验](/learn/ai)。

### 骨架包含的内容

```
project-root/
├── AGENTS.md              # AI / 代理的事实来源
├── SECURITY.md            # 安全预期
├── .env.example           # 机密 / 部署覆盖（复制到 .env）
├── public/index.php       # 仅 Web 入口
├── app/
│   ├── config/            # 引导、路由、服务、config_sample.php
│   ├── Controller/        # App\Controller\*（PascalCase 文件夹！）
│   ├── Middleware/        # App\Middleware\*
│   ├── Model/             # App\Model\*（ActiveRecord）
│   ├── Utils/             # Config、Env、DatabaseFactory
│   ├── commands/          # Runway CLI 命令
│   ├── views/             # Twig 模板（*.twig）
│   ├── cache/
│   └── log/
├── migrations/            # SQL 迁移（.sql / .mysql.sql）
└── tests/                 # PHPUnit
```

**命名空间遵循文件夹大小写。** Composer 将 `"App\\": "app/"` 映射为：

| 磁盘路径 | 命名空间 |
|--------------|-----------|
| `app/Controller/HomeController.php` | `App\Controller\HomeController` |
| `app/Middleware/…` | `App\Middleware\…` |
| `app/Model/…` | `App\Model\…` |
| `app/Utils/…` | `App\Utils\…` |

在 Linux 上，`app/controller/` **不等同于** `app/Controller/`。自动加载区分大小写——请匹配骨架的 PascalCase 文件夹。详细信息：[自动加载](/learn/autoloading)。

**技术栈默认值（新项目）：** Twig 视图、SimplePdo + ActiveRecord、带 `Engine` 注入的 Dice（应用类中尽量不要使用 `Flight::`），在 `php runway migrate` 后可选择使用 SQLite。

`create-project` 通常会在存在时将 `app/config/config_sample.php` 复制为 `config.php`，并将 `.env.example` 复制为 `.env`。路由位于 `app/config/routes.php`；服务和 DI 位于 `app/config/services.php`。

> **文档 ↔ 骨架：** 这些文档教授 Flight **API**（通常带有简短的 `Flight::` 示例）。骨架则确定了**应用结构**。在 `app/` 下添加代码时，请遵循骨架树；对于方法名、选项和插件，请使用文档。

## 配置你的 Web 服务器

### 内置 PHP 开发服务器

这是目前最简单的启动方式。你可以使用内置服务器运行应用程序，甚至可以使用 SQLite 作为数据库（只要你的系统上安装了 sqlite3），几乎不需要任何额外配置！只需在安装 PHP 后运行以下命令：

```bash
php -S localhost:8000
# 或使用骨架应用
composer start
```

然后打开浏览器并访问 `http://localhost:8000`。

如果你希望将项目的文档根目录设置为其他目录（例如：你的项目是 `~/myproject`，但文档根目录是 `~/myproject/public/`），可以在进入 `~/myproject` 目录后运行以下命令：

```bash
php -S localhost:8000 -t public/
# 使用骨架应用时，这已默认配置
composer start
```

然后打开浏览器并访问 `http://localhost:8000`。

### Apache

确保你的系统上已安装 Apache。如果没有，请自行搜索如何在系统上安装 Apache。

对于 Apache，请编辑你的 `.htaccess` 文件，加入以下内容：

```apacheconf
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteCond %{REQUEST_FILENAME} !-d
RewriteRule ^(.*)$ index.php [QSA,L]
```

> **注意**：如果你需要在子目录中使用 Flight，请在 `RewriteEngine On` 之后添加一行
> `RewriteBase /subdir/`。

> **注意**：如果你希望保护所有服务器文件（例如数据库或 env 文件），
> 请在 `.htaccess` 文件中加入以下内容：

```apacheconf
RewriteEngine On
RewriteRule ^(.*)$ index.php
```

### Nginx

确保你的系统上已安装 Nginx。如果没有，请自行搜索如何在系统上安装 Nginx。

对于 Nginx，请在服务器配置中添加以下内容：

```nginx
server {
  location / {
    try_files $uri $uri/ /index.php;
  }
}
```

## 创建你的 `index.php` 文件

如果你正在进行基本安装，你需要编写一些代码来启动。

```php
<?php

// 如果你使用 Composer，请引入自动加载器。
require 'vendor/autoload.php';
// 如果你不使用 Composer，请直接加载框架
// require 'flight/Flight.php';

// 然后定义一个路由，并分配一个函数来处理请求。
Flight::route('/', function () {
  echo 'hello world!';
});

// 最后，启动框架。
Flight::start();
```

对于骨架应用，公共入口只负责启动应用。路由在 `app/config/routes.php` 中注册（通常是 `[App\Controller\…::class, 'method']`，以便 Dice 可以注入依赖）。服务、Twig、SimplePdo 和容器在 `app/config/services.php` 中配置。这种结构是有意设计的，以便 AI 工具和人类每次都能编辑相同的位置。

## 安装 PHP

如果你的系统上已经安装了 `php`，请跳过这些说明，直接转到[下载部分](#download-the-files)。

### **macOS**

#### **使用 Homebrew 安装 PHP**

1. **安装 Homebrew**（如果尚未安装）：
   - 打开终端并运行：
     ```bash
     /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
     ```

2. **安装 PHP**：
   - 安装最新版本：
     ```bash
     brew install php
     ```
   - 要安装特定版本，例如 PHP 8.1：
     ```bash
     brew tap shivammathur/php
     brew install shivammathur/php/php@8.1
     ```

3. **在 PHP 版本之间切换**：
   - 取消链接当前版本并链接所需版本：
     ```bash
     brew unlink php
     brew link --overwrite --force php@8.1
     ```
   - 验证已安装的版本：
     ```bash
     php -v
     ```

### **Windows 10/11**

#### **手动安装 PHP**

1. **下载 PHP**：
   - 访问 [PHP for Windows](https://windows.php.net/download/) 并下载最新版本或特定版本（例如 7.4、8.0），选择非线程安全的 zip 文件。

2. **解压 PHP**：
   - 将下载的 zip 文件解压到 `C:\php`。

3. **将 PHP 添加到系统 PATH**：
   - 转到**系统属性** > **环境变量**。
   - 在**系统变量**下，找到 **Path** 并点击**编辑**。
   - 添加路径 `C:\php`（或你解压 PHP 的位置）。
   - 点击**确定**关闭所有窗口。

4. **配置 PHP**：
   - 将 `php.ini-development` 复制为 `php.ini`。
   - 根据需要编辑 `php.ini` 来配置 PHP（例如设置 `extension_dir`、启用扩展）。

5. **验证 PHP 安装**：
   - 打开命令提示符并运行：
     ```cmd
     php -v
     ```

#### **安装多个 PHP 版本**

1. **针对每个版本重复上述步骤**，将每个版本放在单独的目录中（例如 `C:\php7`、`C:\php8`）。

2. **通过调整系统 PATH 变量**，使其指向所需的版本目录，即可在版本之间切换。

### **Ubuntu（20.04、22.04 等）**

#### **使用 apt 安装 PHP**

1. **更新软件包列表**：
   - 打开终端并运行：
     ```bash
     sudo apt update
     ```

2. **安装 PHP**：
   - 安装最新版本的 PHP：
     ```bash
     sudo apt install php
     ```
   - 要安装特定版本，例如 PHP 8.1：
     ```bash
     sudo apt install php8.1
     ```

3. **安装额外的模块**（可选）：
   - 例如，要安装 MySQL 支持：
     ```bash
     sudo apt install php8.1-mysql
     ```

4. **在 PHP 版本之间切换**：
   - 使用 `update-alternatives`：
     ```bash
     sudo update-alternatives --set php /usr/bin/php8.1
     ```

5. **验证已安装的版本**：
   - 运行：
     ```bash
     php -v
     ```

### **Rocky Linux**

#### **使用 yum/dnf 安装 PHP**

1. **启用 EPEL 仓库**：
   - 打开终端并运行：
     ```bash
     sudo dnf install epel-release
     ```

2. **安装 Remi 仓库**：
   - 运行：
     ```bash
     sudo dnf install https://rpms.remirepo.net/enterprise/remi-release-8.rpm
     sudo dnf module reset php
     ```

3. **安装 PHP**：
   - 要安装默认版本：
     ```bash
     sudo dnf install php
     ```
   - 要安装特定版本，例如 PHP 7.4：
     ```bash
     sudo dnf module install php:remi-7.4
     ```

4. **在 PHP 版本之间切换**：
   - 使用 `dnf` 模块命令：
     ```bash
     sudo dnf module reset php
     sudo dnf module enable php:remi-8.0
     sudo dnf install php
     ```

5. **验证已安装的版本**：
   - 运行：
     ```bash
     php -v
     ```

### **一般说明**

- 对于开发环境，根据项目需求配置 PHP 设置非常重要。
- 切换 PHP 版本时，请确保已为目标版本安装所有相关的 PHP 扩展。
- 在切换 PHP 版本或更新配置后，请重启 Web 服务器（Apache、Nginx 等）以应用更改。