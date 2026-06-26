# ШІ та Досвід Розробника з Flight

## Огляд

Flight полегшує прискорення ваших PHP-проектів за допомогою інструментів на базі ШІ та сучасних робочих процесів розробника. Завдяки вбудованим командам для підключення до постачальників LLM (Large Language Model) та генерації специфічних для проєкту інструкцій для ШІ-кодування, Flight допомагає вам і вашій команді максимально використовувати ШІ-асистенти, такі як GitHub Copilot, Cursor, Windsurf та Antigravity (Gemini).

## Розуміння

ШІ-асистенти для кодування найбільш корисні, коли вони розуміють контекст, домовленості та цілі вашого проєкту. ШІ-помічники Flight дозволяють вам:
- Підключити ваш проєкт до популярних постачальників LLM (OpenAI, Grok, Claude тощо)
- Генерувати та оновлювати специфічні для проєкту інструкції для ШІ-інструментів, щоб усі отримували послідовну та релевантну допомогу
- Підтримувати узгодженість та продуктивність команди, зменшуючи час на пояснення контексту

Ці функції вбудовані в основний CLI Flight та офіційний стартовий проєкт [flightphp/skeleton](https://github.com/flightphp/skeleton).

## Базове Використання

### Налаштування Облікових Даних LLM

Команда `ai:init` проведе вас через процес підключення вашого проєкту до постачальника LLM.

```bash
php runway ai:init
```

Вам буде запропоновано:
- Вибрати постачальника (OpenAI, Grok, Claude тощо)
- Ввести ваш API-ключ
- Встановити базову URL-адресу та назву моделі

Це створює необхідні облікові дані для майбутніх запитів до LLM.

**Приклад:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### Генерація Специфічних для Проєкту Інструкцій для ШІ

Команда `ai:generate-instructions` допомагає створювати або оновлювати інструкції для ШІ-асистентів кодування, адаптовані до вашого проєкту.

```bash
php runway ai:generate-instructions
```

Ви відповісте на кілька запитань про ваш проєкт (опис, база даних, шаблонізація, безпека, розмір команди тощо). Flight використовує вашого постачальника LLM для генерації інструкцій, а потім записує той самий вміст до:
- `.github/copilot-instructions.md` (для GitHub Copilot)
- `.cursor/rules/project-overview.mdc` (для Cursor)
- `.windsurfrules` (для Windsurf)
- `.gemini/GEMINI.md` (для Antigravity)
- `AGENTS.md` (у корені проєкту, для ШІ-асистентів, не залежних від інструментів)

**Приклад:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

Тепер ваші ШІ-інструменти даватимуть розумніші та релевантніші пропозиції, базуючись на реальних потребах вашого проєкту.

## Розширене Використання

- Ви можете налаштувати розташування файлів облікових даних або інструкцій за допомогою опцій команд (див. `--help` для кожної команди).
- ШІ-помічники розроблені для роботи з будь-яким постачальником LLM, який підтримує OpenAI-сумісні API.
- Якщо ви хочете оновити інструкції в міру розвитку вашого проєкту, просто повторно запустіть `ai:generate-instructions` та знову дайте відповіді на запити.

## Див. Також

- [Flight Skeleton](https://github.com/flightphp/skeleton) – Офіційний стартер з інтеграцією ШІ
- [Runway CLI](/awesome-plugins/runway) – Більше про CLI-інструмент, що живить ці команди

## Усунення Неполадок

- Якщо ви бачите "Missing .runway-creds.json", спочатку запустіть `php runway ai:init`.
- Переконайтеся, що ваш API-ключ дійсний та має доступ до вибраної моделі.
- Якщо інструкції не оновлюються, перевірте дозволи на файли у вашому каталозі проєкту.

## Журнал Змін

- v3.18.4 – `ai:generate-instructions` тепер також записує інструкції проєкту до `AGENTS.md` у корені проєкту.
- v3.16.0 – Додано команди CLI `ai:init` та `ai:generate-instructions` для інтеграції ШІ.