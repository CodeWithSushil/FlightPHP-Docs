# AI 与开发者体验（Flight）

## 概述

Flight 的设计目标是与 AI 编码工具协作，而不是对抗它们。小巧且可预测的 API、[官方骨架](https://github.com/flightphp/skeleton)中清晰的应用布局，以及针对项目的指令文件，使得 GitHub Copilot、Cursor、Windsurf、Claude Code 和 Gemini 等助手能够遵循与你手写代码相同的模式。

借助内置的 Runway 命令连接 LLM 提供商并生成项目指令，Flight 帮助你与你的团队获得一致且相关的帮助，而无需在每次对话中重复粘贴相同的上下文。

## 理解

当 AI 编码助手了解你项目的上下文、约定和目标时，它们的帮助最大。Flight 的 AI 辅助功能让你可以：

- 将你的项目连接到流行的 LLM 提供商（OpenAI、Grok、Claude 等）
- 生成和更新项目专属指令，使每个人获得相同的指导
- 让手写代码和 AI 生成的代码遵循统一布局（尤其是使用骨架时）

这些功能随 Flight 核心 CLI 一起提供（通过 [Runway](/awesome-plugins/runway)），并在官方 [flightphp/skeleton](https://github.com/flightphp/skeleton) 入门模板中预配置。

### 骨架为 AI 提供了什么

官方入门模板将 **`AGENTS.md`** 视为 AI 工具的**唯一事实来源**：

| 文件 | 作用 |
|------|------|
| **`AGENTS.md`**（项目根目录） | 全局规则、启动流程、命名空间、依赖注入、“不要做什么” |
| 位于 `app/`、`migrations/`、`tests/` 等目录下的**作用域 `AGENTS.md`** | 在对应目录中工作时提供的轻量、文件夹专属提示 |
| **`SECURITY.md`** | 密钥、请求头、XSS/SQL、报告——安全始终是有意为之且独立的部分 |

骨架中**没有**为 Copilot / Cursor / Gemini / Windsurf 准备独立的样式文件。请将你的助手指向根目录的 `AGENTS.md`（并让它跟随链接访问作用域文件）。人类可以完全忽略这些文件，直接使用 [README](https://github.com/flightphp/skeleton)；无论哪种方式，布局都是一样的。

> **文档教 API；骨架教布局。** 本文档中的简短 `Flight::` 示例非常适合学习。在骨架应用中，更倾向于使用 `App\…` 类、构造函数注入和 `$this->app`，而不是在控制器中使用静态外观。参见[安装](/install)和[自动加载](/learn/autoloading)。

## 基本用法

### 设置 LLM 凭据

`ai:init` 命令会引导你将项目连接到 LLM 提供商。

```bash
php runway ai:init
```

系统会提示你：

- 选择提供商（OpenAI、Grok、Claude 等）
- 输入你的 API 密钥
- 设置基础 URL 和模型名称

这将创建后续 LLM 请求所需的凭据（例如用于生成指令）。

**示例：**
```
欢迎使用 AI Init！
你想使用哪个 LLM API？[1] openai、[2] grok、[3] claude：1
输入 LLM API 的基础 URL [https://api.openai.com]：
输入你的 openai API 密钥：sk-...
输入你想使用的模型名称（例如 gpt-4、claude-3-opus 等）[gpt-4o]：
凭据已保存到 .runway-creds.json
```

### 生成项目专属 AI 指令

`ai:generate-instructions` 命令会创建或更新 AI 编码助手的指令，并针对*你的*项目进行定制。

```bash
php runway ai:generate-instructions
```

你需要回答几个问题（描述、数据库、模板引擎、安全性、团队规模等）。Flight 使用你的 LLM 提供商来生成指令，并将其主要写入：

- 项目根目录下的 **`AGENTS.md`**（与工具无关；官方骨架和大多数现代智能体都期望此文件）

根据 CLI 版本和选项的不同，该命令也可能会为较旧的工作流写入工具专属副本（例如 Copilot、Cursor、Windsurf 或 Gemini 规则文件）。对于**来自骨架的新项目**，请将 **`AGENTS.md`**（以及你在 `app/` 下保留的任何作用域 `AGENTS.md` 文件）视为唯一事实来源——不要手动维护五个不同的指令文件。

**示例：**
```
请描述你的项目用途是什么？我的超棒 API
你打算使用什么数据库？MySQL
你打算使用什么 HTML 模板引擎（如果有）？twig
安全性是这个项目的重要元素吗？（y/n）y
...
AI 指令已成功更新。
```

现在 AI 工具可以推荐与你的真实技术栈和布局相匹配的代码，而不是泛泛的 PHP 教程。

## 高级用法

- 使用命令选项自定义凭据或输出路径（参见每个命令的 `--help`）。
- 这些辅助功能适用于任何兼容 OpenAI API 的 LLM 提供商。
- 随着项目的发展，重新运行 `ai:generate-instructions`，让智能体保持同步。
- 在骨架中，将安全策略保留在 **`SECURITY.md`** 中，将代码布局保留在 **`AGENTS.md`** 中，这样两份文档都不会变成大杂烩。
- 当智能体需要 API 详细信息时，优先使用 [docs.flightphp.com](https://docs.flightphp.com) 和 Flight MCP 服务器；对未知方法请对照 `vendor/flightphp/core` 验证。

## 参见

- [Flight Skeleton](https://github.com/flightphp/skeleton) – 官方入门模板，预配置了 `AGENTS.md`、Twig、SimplePdo 和 Dice，适合 AI 友好的结构
- [安装](/install) – 推荐的 `create-project` 布局
- [自动加载](/learn/autoloading) – 文件夹**大小写**与命名空间匹配（`App\Controller` ↔ `app/Controller/`）
- [Runway CLI](/awesome-plugins/runway) – 驱动 `ai:*` 和脚手架命令的 CLI
- [安全](/learn/security) – 智能体（和人）不应削弱的默认安全设置

## 故障排除

- 如果你看到“缺少 .runway-creds.json”，请先运行 `php runway ai:init`。
- 确保你的 API 密钥有效并且有权访问所选模型。
- 如果指令没有更新，请检查项目目录中的文件权限。
- 如果智能体虚构了 Flight API 或使用了错误的文件夹布局，请将其指向根目录的 **`AGENTS.md`** 和本文档站点；`app/` 下的代码以骨架布局为准。

## 更新日志

- v3.18.4 – `ai:generate-instructions` 将项目指令写入项目根目录的 `AGENTS.md`。
- v3.16.0 – 添加了 `ai:init` 和 `ai:generate-instructions` CLI 命令以支持 AI 集成。