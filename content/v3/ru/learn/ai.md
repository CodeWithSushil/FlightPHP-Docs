# ИИ и опыт разработчика с Flight

## Обзор

Flight упрощает интеграцию ИИ-инструментов и современных рабочих процессов разработчика в ваши PHP-проекты. С помощью встроенных команд для подключения к провайдерам LLM (Large Language Model) и генерации ИИ-инструкций для конкретного проекта, Flight помогает вам и вашей команде максимально эффективно использовать ИИ-ассистентов, таких как GitHub Copilot, Cursor, Windsurf и Antigravity (Gemini).

## Понимание

ИИ-ассистенты по программированию наиболее полезны, когда они понимают контекст вашего проекта, его соглашения и цели. ИИ-помощники Flight позволяют:
- Подключить ваш проект к популярным провайдерам LLM (OpenAI, Grok, Claude и др.)
- Генерировать и обновлять инструкции для ИИ-инструментов, специфичные для вашего проекта, чтобы все получали согласованную и релевантную помощь
- Поддерживать согласованность и продуктивность команды, сокращая время на объяснение контекста

Эти функции встроены в ядро CLI Flight и официальный стартовый проект [flightphp/skeleton](https://github.com/flightphp/skeleton).

## Основное использование

### Настройка учетных данных LLM

Команда `ai:init` проведет вас через процесс подключения вашего проекта к провайдеру LLM.

```bash
php runway ai:init
```

Вам будет предложено:
- Выбрать провайдера (OpenAI, Grok, Claude и др.)
- Ввести ваш API-ключ
- Указать базовый URL и название модели

Это создаст необходимые учетные данные для будущих запросов к LLM.

**Пример:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Генерация ИИ-инструкций для конкретного проекта

Команда `ai:generate-instructions` помогает создать или обновить инструкции для ИИ-ассистентов по программированию, адаптированные под ваш проект.

```bash
php runway ai:generate-instructions
```

Вы ответите на несколько вопросов о вашем проекте (описание, база данных, шаблонизация, безопасность, размер команды и т.д.). Flight использует вашего провайдера LLM для генерации инструкций, а затем записывает тот же контент в:
- `.github/copilot-instructions.md` (для GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (для Cursor)
- `.windsurfrules` (для Windsurf)
- `.gemini/GEMINI.md` (для Antigravity)
- `AGENTS.md` (в корне проекта, для ИИ-ассистентов, не зависящих от инструмента)

**Пример:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Теперь ваши ИИ-инструменты будут давать более умные и релевантные предложения на основе реальных потребностей вашего проекта.

## Продвинутое использование

- Вы можете настроить расположение файлов учетных данных или инструкций с помощью опций команд (см. `--help` для каждой команды).
- ИИ-помощники разработаны для работы с любым провайдером LLM, поддерживающим API, совместимые с OpenAI.
- Если вы хотите обновить инструкции по мере развития проекта, просто повторно запустите `ai:generate-instructions` и снова ответьте на вопросы.

## См. также

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Официальный стартер с интеграцией ИИ
- [Runway CLI](/awesome-plugins/runway) – Подробнее о CLI-инструменте, обеспечивающем эти команды

## Устранение неполадок

- Если вы видите "Missing .runway-creds.json", сначала выполните `php runway ai:init`.
- Убедитесь, что ваш API-ключ действителен и имеет доступ к выбранной модели.
- Если инструкции не обновляются, проверьте права доступа к файлам в каталоге вашего проекта.

## История изменений

- v3.18.4 – `ai:generate-instructions` теперь также записывает инструкции проекта в `AGENTS.md` в корне проекта.
- v3.16.0 – Добавлены команды CLI `ai:init` и `ai:generate-instructions` для интеграции с ИИ.