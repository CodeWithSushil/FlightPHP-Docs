# AI 与 Flight 的开发者体验

## 概述

Flight 让你能够轻松地为 PHP 项目注入 AI 驱动的工具和现代化的开发者工作流。Flight 内置了连接 LLM（大型语言模型）提供商的命令，并能生成针对项目的 AI 编程指令，帮助你和你的团队充分利用 GitHub Copilot、Cursor、Windsurf 和 Antigravity（Gemini）等 AI 助手。

## 理解

当 AI 编程助手能够理解项目的上下文、约定和目标时，它们会发挥最大的作用。Flight 的 AI 助手允许你：
- 将项目连接到流行的 LLM 提供商（OpenAI、Grok、Claude 等）
- 为 AI 工具生成并更新项目特定的指令，确保每个人都能获得一致且相关的帮助
- 保持团队协同和高效，减少解释上下文的时间

这些功能已集成到 Flight 核心 CLI 和官方 [flightphp/skeleton](https://github.com/flightphp/skeleton) 启动项目中。

## 基本用法

### 设置 LLM 凭证

`ai:init` 命令将引导你将项目连接到 LLM 提供商。

```bash
php runway ai:init
```

系统将提示你：
- 选择提供商（OpenAI、Grok、Claude 等）
- 输入 API 密钥
- 设置基础 URL 和模型名称

这将创建必要的凭证，以便你未来进行 LLM 请求。

**示例：**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### 生成项目特定的 AI 指令

`ai:generate-instructions` 命令帮助你创建或更新针对 AI 编程助手的指令，这些指令会根据你的项目进行定制。

```bash
php runway ai:generate-instructions
```

你需要回答几个关于项目的问题（描述、数据库、模板引擎、安全性、团队规模等）。Flight 使用你的 LLM 提供商生成指令，然后将相同的内容写入：
- `.github/copilot-instructions.md`（用于 GitHub Copilot）
- `.cursor/rules/project-overview.mdc`（用于 Cursor）
- `.windsurfrules`（用于 Windsurf）
- `.gemini/GEMINI.md`（用于 Antigravity）
- `AGENTS.md`（项目根目录，用于与工具无关的 AI 助手）

**示例：**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

现在，你的 AI 工具将根据项目的实际需求提供更智能、更相关的建议。

## 高级用法

- 你可以使用命令选项自定义凭证或指令文件的位置（请参阅每个命令的 `--help`）。
- AI 助手设计为可与任何支持 OpenAI 兼容 API 的 LLM 提供商一起使用。
- 如果你希望在项目发展过程中更新指令，只需重新运行 `ai:generate-instructions` 并再次回答提示即可。

## 另请参阅

- [Flight Skeleton](https://github.com/flightphp/skeleton) – 官方带 AI 集成的启动项目
- [Runway CLI](/awesome-plugins/runway) – 关于支持这些命令的 CLI 工具的更多信息

## 故障排除

- 如果看到“Missing .runway-creds.json”，请先运行 `php runway ai:init`。
- 确保你的 API 密钥有效且具有所选模型的访问权限。
- 如果指令未更新，请检查项目目录中的文件权限。

## 更新日志

- v3.18.4 – `ai:generate-instructions` 现在还将项目指令写入项目根目录的 `AGENTS.md`。
- v3.16.0 – 新增 `ai:init` 和 `ai:generate-instructions` CLI 命令，用于 AI 集成。