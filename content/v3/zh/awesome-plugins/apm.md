# FlightPHP APM 文档

欢迎使用 FlightPHP APM——您应用的个人性能教练！本指南是您设置、使用和掌握 FlightPHP 应用性能监控（APM）的路线图。无论您是在追踪缓慢的请求，还是只是想深入研究延迟图表，我们都能满足您的需求。让我们让您的应用更快、用户更满意，调试过程更轻松！

查看 Flight Docs 网站仪表盘的 [演示](https://flightphp-docs-apm.sky-9.com/apm/dashboard)。

![FlightPHP APM](/images/apm.png)

## 为什么 APM 很重要

想象一下：您的应用就像一家繁忙的餐厅。如果没有办法追踪订单需要多长时间，或者厨房在哪里卡住了，您只能猜测客户为什么不满意地离开。APM 就像您的副厨——它监控每一步，从传入的请求到数据库查询，并标记任何拖慢速度的地方。缓慢的页面会流失用户（研究表明，如果网站加载时间超过 3 秒，53% 的用户会跳出！），APM 帮助您在问题*发生前*捕获这些问题。这是主动的安心——减少"为什么出问题了"的时刻，增加"看它运行得多流畅！"的胜利。

## 安装

使用 Composer 开始：

```bash
composer require flightphp/apm
```

您需要：
- **PHP 7.4+**：确保我们与 LTS Linux 发行版兼容，同时支持现代 PHP。
- **[FlightPHP Core](https://github.com/flightphp/core) v3.15+**：我们正在增强的轻量级框架。

## 支持的数据库

FlightPHP APM 目前支持以下数据库来存储指标：

- **SQLite3**：简单、基于文件，非常适合本地开发或小型应用。大多数设置中的默认选项。
- **MySQL/MariaDB**：适合需要强大、可扩展存储的更大项目或生产环境。

您可以在配置步骤中选择数据库类型（见下文）。确保您的 PHP 环境安装了必要的扩展（例如，`pdo_sqlite` 或 `pdo_mysql`）。

## 开始使用

以下是 APM 精彩功能的逐步指南：

### 1. 注册 APM

将以下内容放入您的 `index.php` 或 `services.php` 文件以开始追踪：

```php
use flight\apm\logger\LoggerFactory;
use flight\database\SimplePdo;
use flight\Apm;

$ApmLogger = LoggerFactory::create(__DIR__ . '/../../.runway-config.json');
$Apm = new Apm($ApmLogger);
$Apm->bindEventsToFlightInstance($app);

// 如果您要添加数据库连接
// 首选 SimplePdo（或开发环境中的 Tracy 扩展的 PdoQueryCapture）。
// 通过 options 数组启用 APM 查询追踪（第 5 个参数）。
$pdo = new SimplePdo('mysql:host=localhost;dbname=example', 'user', 'pass', null, [
	'trackApmQueries' => true, // 必需，用于捕获 APM 的查询
]);
$Apm->addPdoConnection($pdo);
```

**这里发生了什么？**
- `LoggerFactory::create()` 获取您的配置（稍后详述）并设置记录器——默认使用 SQLite。
- `Apm` 是主角——它监听 Flight 的事件（请求、路由、错误等）并收集指标。
- `bindEventsToFlightInstance($app)` 将它与您的 Flight 应用绑定。

**专业提示：采样**
如果您的应用很忙，记录*每个*请求可能会使系统过载。使用采样率（0.0 到 1.0）：

```php
$Apm = new Apm($ApmLogger, 0.1); // 记录 10% 的请求
```

这保持了性能敏捷，同时仍为您提供可靠的数据。

### 2. 配置它

运行此命令以创建您的 `.runway-config.json`：

```bash
php vendor/bin/runway apm:init
```

**这是做什么的？**
- 启动一个向导，询问原始指标来自哪里（源）和处理后的数据去哪里（目标）。
- 默认是 SQLite——例如，源为 `sqlite:/tmp/apm_metrics.sqlite`，目标为另一个。
- 您最终会得到一个配置，如：
  ```json
  {
    "apm": {
      "source_type": "sqlite",
      "source_db_dsn": "sqlite:/tmp/apm_metrics.sqlite",
      "storage_type": "sqlite",
      "dest_db_dsn": "sqlite:/tmp/apm_metrics_processed.sqlite"
    }
  }
  ```

> 此过程还将询问您是否要为此设置运行迁移。如果您是第一次设置，答案是肯定的。

**为什么需要两个位置？**
原始指标堆积很快（想想未过滤的日志）。工作进程将它们处理成仪表盘的结构化目标。保持整洁！

### 3. 使用工作进程处理指标

工作进程将原始指标转换为仪表盘就绪的数据。运行一次：

```bash
php vendor/bin/runway apm:worker
```

**它在做什么？**
- 从您的源读取（例如，`apm_metrics.sqlite`）。
- 将最多 100 个指标（默认批次大小）处理到您的目标。
- 完成后或没有指标时停止。

**保持运行**
对于实时应用，您需要持续处理。以下是您的选项：

- **守护进程模式**：
  ```bash
  php vendor/bin/runway apm:worker --daemon
  ```
  永久运行，处理传入的指标。适合开发或小型设置。

- **Crontab**：
  将此添加到您的 crontab（`crontab -e`）：
  ```bash
  * * * * * php /path/to/project/vendor/bin/runway apm:worker
  ```
  每分钟触发一次——适合生产环境。

- **Tmux/Screen**：
  启动一个可分离的会话：
  ```bash
  tmux new -s apm-worker
  php vendor/bin/runway apm:worker --daemon
  # Ctrl+B，然后 D 分离；`tmux attach -t apm-worker` 重新连接
  ```
  即使您退出登录也能保持运行。

- **自定义调整**：
  ```bash
  php vendor/bin/runway apm:worker --batch_size 50 --max_messages 1000 --timeout 300
  ```
  - `--batch_size 50`：一次处理 50 个指标。
  - `--max_messages 1000`：1000 个指标后停止。
  - `--timeout 300`：5 分钟后退出。

**为什么需要它？**
没有工作进程，您的仪表盘将是空的。它是原始日志和可操作见解之间的桥梁。

### 4. 启动仪表盘

查看应用的生命体征：

```bash
php vendor/bin/runway apm:dashboard
```

**这是什么？**
- 在 `http://localhost:8001/apm/dashboard` 启动 PHP 服务器。
- 显示请求日志、慢路由、错误率等。

**自定义它**：
```bash
php vendor/bin/runway apm:dashboard --host 0.0.0.0 --port 8080 --php-path=/usr/local/bin/php
```
- `--host 0.0.0.0`：可从任何 IP 访问（便于远程查看）。
- `--port 8080`：如果 8001 被占用，使用不同的端口。
- `--php-path`：如果 PHP 不在您的 PATH 中，指向 PHP。

在浏览器中打开 URL 并探索！

#### 生产模式

对于生产环境，您可能需要尝试一些技术来使仪表盘运行，因为可能存在防火墙和其他安全措施。以下是一些选项：

- **使用反向代理**：设置 Nginx 或 Apache 将请求转发到仪表盘。
- **SSH 隧道**：如果您可以 SSH 到服务器，使用 `ssh -L 8080:localhost:8001
youruser@yourserver` 将仪表盘隧道到本地机器。
- **VPN**：如果您的服务器在 VPN 后面，连接到它并直接访问仪表盘。
- **配置防火墙**：为您的 IP 或服务器的网络打开 8001 端口（或您设置的任何端口）。
- **配置 Apache/Nginx**：如果您在应用前面有 Web 服务器，您可以将其配置为域名或子域名。如果这样做，您将把文档根目录设置为 `/path/to/your/project/vendor/flightphp/apm/dashboard`

#### 想要不同的仪表盘？

如果您愿意，您可以构建自己的仪表盘！查看 vendor/flightphp/apm/src/apm/presenter 目录，了解如何为自己的仪表盘展示数据！

## 仪表盘功能

仪表盘是您的 APM 总部——以下是您将看到的内容：

- **请求日志**：每个请求的时间戳、URL、响应代码和总时间。点击"详情"查看中间件、查询和错误。
- **最慢的请求**：占用时间最多的前 5 个请求（例如，"/api/heavy" 耗时 2.5 秒）。
- **最慢的路由**：按平均时间排列的前 5 个路由——非常适合发现模式。
- **错误率**：请求失败的百分比（例如，2.3% 的 500 错误）。
- **延迟百分位数**：第 95 百分位（p95）和第 99 百分位（p99）响应时间——了解您最坏的情况。
- **响应代码图表**：可视化一段时间内的 200、404、500 错误。
- **长查询/中间件**：前 5 个慢数据库调用和中间件层。
- **缓存命中/未命中**：您的缓存节省时间的频率。

**额外功能**：
- 按"最近一小时"、"最近一天"或"最近一周"筛选。
- 为深夜会话切换深色模式。

**示例**：
对 `/users` 的请求可能显示：
- 总时间：150ms
- 中间件：`AuthMiddleware->handle`（50ms）
- 查询：`SELECT * FROM users`（80ms）
- 缓存：`user_list` 命中（5ms）

## 添加自定义事件

追踪任何事情——比如 API 调用或支付流程：

```php
use flight\apm\CustomEvent;

$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('api_call', [
    'endpoint' => 'https://api.example.com/users',
    'response_time' => 0.25,
    'status' => 200
]));
```

**它在哪里显示？**
在仪表盘的请求详情下的"自定义事件"中——可展开，带有漂亮的 JSON 格式。

**用例**：
```php
$start = microtime(true);
$apiResponse = file_get_contents('https://api.example.com/data');
$app->eventDispatcher()->trigger('apm.custom', new CustomEvent('external_api', [
    'url' => 'https://api.example.com/data',
    'time' => microtime(true) - $start,
    'success' => $apiResponse !== false
]));
```
现在您将看到该 API 是否拖慢了您的应用！

## 数据库监控

像这样追踪 PDO 查询：

```php
use flight\database\SimplePdo;

$pdo = new SimplePdo('sqlite:/path/to/db.sqlite', null, null, null, [
	'trackApmQueries' => true, // 必需，用于捕获 APM 的查询
]);
$Apm->addPdoConnection($pdo);
```

**您将获得**：
- 查询文本（例如，`SELECT * FROM users WHERE id = ?`）
- 执行时间（例如，0.015s）
- 行数（例如，42）

**注意事项**：
- **可选**：如果您不需要数据库追踪，可以跳过此步骤。
- **SimplePdo（首选）**：使用带有 `trackApmQueries => true` 的 `SimplePdo`。已弃用的 `PdoWrapper` 仍然有效（第 5 个构造函数参数 `true`）。原始核心 PDO 尚未挂钩——敬请期待！
- **性能警告**：在数据库繁重的网站上记录每个查询可能会减慢速度。使用采样（`$Apm = new Apm($ApmLogger, 0.1)`）来减轻负载。

**输出示例**：
- 查询：`SELECT name FROM products WHERE price > 100`
- 时间：0.023s
- 行数：15

## 工作进程选项

根据您的喜好调整工作进程：

- `--timeout 300`：5 分钟后停止——适合测试。
- `--max_messages 500`：限制在 500 个指标——保持有限。
- `--batch_size 200`：一次处理 200 个——平衡速度和内存。
- `--daemon`：不间断运行——适合实时监控。

**示例**：
```bash
php vendor/bin/runway apm:worker --daemon --batch_size 100 --timeout 3600
```
运行一小时，一次处理 100 个指标。

## 应用中的请求 ID

每个请求都有一个唯一的请求 ID 用于追踪。您可以在应用中使用此 ID 来关联日志和指标。例如，您可以将请求 ID 添加到错误页面：

```php
Flight::map('error', function($message) {
	// 从响应头 X-Flight-Request-Id 获取请求 ID
	$requestId = Flight::response()->getHeader('X-Flight-Request-Id');

	// 另外，您可以从 Flight 变量中获取它
	// 此方法在 swoole 或其他异步平台上效果不佳。
	// $requestId = Flight::get('apm.request_id');
	
	echo "错误：$message（请求 ID：$requestId）";
});
```

## 升级

如果您要升级到 APM 的新版本，可能需要运行数据库迁移。您可以通过运行以下命令来完成：

```bash
php vendor/bin/runway apm:migrate
```
这将运行任何需要的迁移，以将数据库架构更新到最新版本。

**注意：** 如果您的 APM 数据库很大，这些迁移可能需要一些时间才能运行。您可能希望在非高峰时段运行此命令。

### 从 0.4.3 升级到 0.5.0

如果您要从 0.4.3 升级到 0.5.0，您需要运行以下命令：

```bash
php vendor/bin/runway apm:config-migrate
```

这将把您的配置从使用 `.runway-config.json` 文件的旧格式迁移到将键/值存储在 `config.php` 文件中的新格式。

## 清除旧数据

为了保持数据库整洁，您可以清除旧数据。如果您运行繁忙的应用并希望保持数据库大小可管理，这特别有用。
您可以通过运行以下命令来完成：

```bash
php vendor/bin/runway apm:purge
```
这将从数据库中删除所有超过 30 天的数据。您可以通过向 `--days` 选项传递不同的值来调整天数：

```bash
php vendor/bin/runway apm:purge --days 7
```
这将从数据库中删除所有超过 7 天的数据。

## 故障排除

遇到问题？尝试以下方法：

- **没有仪表盘数据？**
  - 工作进程是否正在运行？检查 `ps aux | grep apm:worker`。
  - 配置路径匹配吗？验证 `.runway-config.json` DSN 指向真实文件。
  - 手动运行 `php vendor/bin/runway apm:worker` 来处理待处理的指标。

- **工作进程错误？**
  - 查看您的 SQLite 文件（例如，`sqlite3 /tmp/apm_metrics.sqlite "SELECT * FROM apm_metrics_log LIMIT 5"`）。
  - 检查 PHP 日志以获取堆栈跟踪。

- **仪表盘无法启动？**
  - 端口 8001 正在使用？使用 `--port 8080`。
  - 找不到 PHP？使用 `--php-path /usr/bin/php`。
  - 防火墙阻止？打开端口或使用 `--host localhost`。

- **太慢了？**
  - 降低采样率：`$Apm = new Apm($ApmLogger, 0.05)`（5%）。
  - 减小批次大小：`--batch_size 20`。

- **不追踪异常/错误？**
  - 如果您为项目启用了 [Tracy](https://tracy.nette.org/)，它将覆盖 Flight 的错误处理。您需要禁用 Tracy，然后确保设置了 `Flight::set('flight.handle_errors', true);`。

- **不追踪数据库查询？**
  - 首选在第 5 个构造函数参数（options 数组）中使用 `['trackApmQueries' => true]` 的 `SimplePdo`。
  - 如果您仍在使用已弃用的 `PdoWrapper`，请将 `true` 作为第 5 个参数传递。
  - 创建连接后调用 `$Apm->addPdoConnection($pdo)`。