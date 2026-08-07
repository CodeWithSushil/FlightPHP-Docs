# AI & Developer Experience with Flight

## Overview

Flight is designed to work *with* AI coding tools—not fight them. A small, predictable API, a clear app layout in the [official skeleton](https://github.com/flightphp/skeleton), and project-specific instruction files mean assistants like GitHub Copilot, Cursor, Windsurf, Claude Code, and Gemini can follow the same patterns you would write by hand.

With built-in Runway commands for connecting to LLM providers and generating project instructions, Flight helps you and your team get consistent, relevant help without pasting the same context into every chat.

## Understanding

AI coding assistants are most helpful when they understand your project's context, conventions, and goals. Flight's AI helpers let you:

- Connect your project to popular LLM providers (OpenAI, Grok, Claude, etc.)
- Generate and update project-specific instructions so everyone gets the same guidance
- Keep hand-written and AI-generated code on one layout (especially with the skeleton)

These features ship with the Flight core CLI (via [Runway](/awesome-plugins/runway)) and are pre-wired in the official [flightphp/skeleton](https://github.com/flightphp/skeleton) starter.

### What the skeleton ships for AI

The official starter treats **`AGENTS.md` as the source of truth** for AI tools:

| File | Role |
|------|------|
| **`AGENTS.md`** (project root) | Global rules, boot flow, namespaces, DI, “what not to do” |
| **Scoped `AGENTS.md`** under `app/`, `migrations/`, `tests/`, etc. | Light, folder-specific tips when you work in that tree |
| **`SECURITY.md`** | Secrets, headers, XSS/SQL, reporting—security stays deliberate and separate |

There is **no** separate house style file for Copilot / Cursor / Gemini / Windsurf in the skeleton. Point your assistant at root `AGENTS.md` (and let it follow links to scoped files). Humans can ignore these files entirely and use the [README](https://github.com/flightphp/skeleton); the layout is the same either way.

> **Docs teach APIs; the skeleton teaches layout.** Short `Flight::` examples in these docs are great for learning. In a skeleton app, prefer `App\…` classes, constructor injection, and `$this->app` over the static facade inside controllers. See [Installation](/install) and [Autoloading](/learn/autoloading).

## Basic Usage

### Setting Up LLM Credentials

The `ai:init` command walks you through connecting your project to an LLM provider.

```bash
php runway ai:init
```

You'll be prompted to:

- Choose your provider (OpenAI, Grok, Claude, etc.)
- Enter your API key
- Set the base URL and model name

This creates the credentials used for later LLM requests (for example generating instructions).

**Example:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Generating Project-Specific AI Instructions

The `ai:generate-instructions` command creates or updates instructions for AI coding assistants, tailored to *your* project.

```bash
php runway ai:generate-instructions
```

You'll answer a few questions (description, database, templating, security, team size, etc.). Flight uses your LLM provider to generate instructions and writes them primarily to:

- **`AGENTS.md`** at the project root (tool-agnostic; what the official skeleton and most modern agents expect)

Depending on CLI version and options, the command may also write tool-specific copies for older workflows (for example Copilot, Cursor, Windsurf, or Gemini rule files). For **new projects from the skeleton**, treat **`AGENTS.md`** (plus any scoped `AGENTS.md` files you keep under `app/`) as the single source of truth—don't maintain five divergent instruction files by hand.

**Example:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Now AI tools can suggest code that matches your real stack and layout—not a generic PHP tutorial.

## Advanced Usage

- Customize credentials or output paths with command options (see `--help` on each command).
- The helpers work with any LLM provider that speaks an OpenAI-compatible API.
- Rerun `ai:generate-instructions` as the project evolves so agents stay in sync.
- In the skeleton, keep security policy in **`SECURITY.md`** and coding layout in **`AGENTS.md`** so neither document becomes a grab bag.
- Prefer [docs.flightphp.com](https://docs.flightphp.com) and the Flight MCP server when agents need API details; verify invented methods against `vendor/flightphp/core`.

## See Also

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Official starter with `AGENTS.md`, Twig, SimplePdo, and Dice wired for AI-friendly structure
- [Installation](/install) – Recommended `create-project` layout
- [Autoloading](/learn/autoloading) – Folder **case** matches namespaces (`App\Controller` ↔ `app/Controller/`)
- [Runway CLI](/awesome-plugins/runway) – CLI that powers `ai:*` and scaffolding commands
- [Security](/learn/security) – Secure defaults agents (and humans) should not weaken

## Troubleshooting

- If you see "Missing .runway-creds.json", run `php runway ai:init` first.
- Make sure your API key is valid and has access to the selected model.
- If instructions aren't updating, check file permissions in your project directory.
- If an agent invents Flight APIs or the wrong folder layout, point it at root **`AGENTS.md`** and this docs site; skeleton layout wins for code under `app/`.

## Changelog

- v3.18.4 – `ai:generate-instructions` writes project instructions to `AGENTS.md` at the project root.
- v3.16.0 – Added `ai:init` and `ai:generate-instructions` CLI commands for AI integration.
