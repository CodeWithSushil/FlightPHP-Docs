# Flight와 AI 및 개발자 경험

## 개요

Flight는 AI 코딩 도구와 *함께* 작동하도록 설계되었습니다. 도구와 싸우지 않습니다. 작고 예측 가능한 API, [공식 스켈레톤](https://github.com/flightphp/skeleton)의 명확한 앱 레이아웃, 그리고 프로젝트별 지침 파일 덕분에 GitHub Copilot, Cursor, Windsurf, Claude Code, Gemini 같은 어시스턴트가 직접 작성하는 것과 동일한 패턴을 따를 수 있습니다.

LLM 공급자 연결 및 프로젝트 지침 생성을 위한 내장 Runway 명령을 통해 Flight는 매번 동일한 컨텍스트를 붙여넣지 않고도 일관되고 관련성 높은 도움을 얻을 수 있도록 지원합니다.

## 이해

AI 코딩 어시스턴트는 프로젝트의 컨텍스트, 규칙, 목표를 이해할 때 가장 유용합니다. Flight의 AI 도우미를 사용하면 다음을 할 수 있습니다:

- 널리 사용되는 LLM 공급자(OpenAI, Grok, Claude 등)에 프로젝트를 연결합니다.
- 프로젝트별 지침을 생성하고 업데이트하여 모든 사람이 동일한 안내를 받을 수 있습니다.
- 손으로 작성한 코드와 AI 생성 코드를 하나의 레이아웃으로 유지합니다(특히 스켈레톤을 사용할 때).

이 기능들은 Flight 핵심 CLI([Runway](/awesome-plugins/runway) 경유)와 함께 제공되며 공식 [flightphp/skeleton](https://github.com/flightphp/skeleton) 스타터에 미리 연결되어 있습니다.

### 스켈레톤이 AI를 위해 제공하는 것

공식 스타터는 **`AGENTS.md`를 AI 도구의 기준 문서(source of truth)** 로 취급합니다:

| 파일 | 역할 |
|------|------|
| **`AGENTS.md`** (프로젝트 루트) | 전역 규칙, 부트 흐름, 네임스페이스, DI, "하지 말아야 할 것" |
| `app/`, `migrations/`, `tests/` 등 아래의 **범위 지정된 `AGENTS.md`** | 해당 트리에서 작업할 때 유용한 가볍고 폴더별 팁 |
| **`SECURITY.md`** | 비밀, 헤더, XSS/SQL, 보고—보안은 의도적이고 별도로 유지됩니다. |

스켈레톤에는 Copilot / Cursor / Gemini / Windsurf용 별도 하우스 스타일 파일이 **없습니다**. 어시스턴트를 루트 `AGENTS.md`로 안내하고(범위 파일로의 링크를 따라가도록 하세요) 사람은 이 파일들을 완전히 무시하고 [README](https://github.com/flightphp/skeleton)를 사용하면 됩니다. 레이아웃은 어느 쪽이든 동일합니다.

> **문서는 API를 가르치고, 스켈레톤은 레이아웃을 가르칩니다.** 이 문서의 짧은 `Flight::` 예제는 학습에 좋습니다. 스켈레톤 앱에서는 컨트롤러 내부에서 정적 퍼사드 대신 `App\…` 클래스, 생성자 주입, `$this->app`을 선호하세요. [설치](/install) 및 [자동 로딩](/learn/autoloading)을 참조하세요.

## 기본 사용법

### LLM 자격 증명 설정

`ai:init` 명령은 프로젝트를 LLM 공급자에 연결하는 과정을 안내합니다.

```bash
php runway ai:init
```

다음 항목을 입력하라는 메시지가 표시됩니다:

- 공급자 선택(OpenAI, Grok, Claude 등)
- API 키 입력
- 기본 URL 및 모델 이름 설정

이렇게 하면 이후 LLM 요청(예: 지침 생성)에 사용할 자격 증명이 생성됩니다.

**예시:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### 프로젝트별 AI 지침 생성

`ai:generate-instructions` 명령은 *당신의* 프로젝트에 맞춰 AI 코딩 어시스턴트를 위한 지침을 생성하거나 업데이트합니다.

```bash
php runway ai:generate-instructions
```

몇 가지 질문(설명, 데이터베이스, 템플릿, 보안, 팀 규모 등)에 답하게 됩니다. Flight는 LLM 공급자를 사용하여 지침을 생성하고 주로 다음 위치에 기록합니다:

- 프로젝트 루트의 **`AGENTS.md`** (도구에 구애받지 않으며, 공식 스켈레톤과 대부분의 최신 에이전트가 기대하는 형식)

CLI 버전과 옵션에 따라 이 명령은 이전 워크플로를 위한 도구별 사본(예: Copilot, Cursor, Windsurf, Gemini 규칙 파일)을 작성할 수도 있습니다. **스켈레톤에서 새 프로젝트를 시작하는 경우** `app/` 아래에 유지하는 범위 지정 `AGENTS.md` 파일과 함께 **`AGENTS.md`** 를 단일 정보 원본으로 취급하세요. 다섯 개의 서로 다른 지침 파일을 수동으로 관리하지 마세요.

**예시:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

이제 AI 도구는 일반적인 PHP 튜토리얼이 아니라 실제 스택과 레이아웃에 맞는 코드를 제안할 수 있습니다.

## 고급 사용법

- 명령 옵션으로 자격 증명이나 출력 경로를 사용자 지정하세요(각 명령의 `--help` 참조).
- 도우미는 OpenAI 호환 API를 제공하는 모든 LLM 공급자와 함께 작동합니다.
- 프로젝트가 발전함에 따라 `ai:generate-instructions`를 다시 실행하여 에이전트가 동기화된 상태를 유지하세요.
- 스켈레톤에서는 보안 정책을 **`SECURITY.md`** 에, 코딩 레이아웃을 **`AGENTS.md`** 에 유지하여 두 문서 모두 잡동사니가 되지 않도록 하세요.
- 에이전트가 API 세부 정보를 필요로 할 때는 [docs.flightphp.com](https://docs.flightphp.com)과 Flight MCP 서버를 선호하고, 생성된 메서드는 `vendor/flightphp/core`와 대조하여 확인하세요.

## 참고 항목

- [Flight Skeleton](https://github.com/flightphp/skeleton) – `AGENTS.md`, Twig, SimplePdo, Dice가 AI 친화적 구조로 연결된 공식 스타터
- [설치](/install) – 권장 `create-project` 레이아웃
- [자동 로딩](/learn/autoloading) – 폴더 **대소문자**가 네임스페이스와 일치(`App\Controller` ↔ `app/Controller/`)
- [Runway CLI](/awesome-plugins/runway) – `ai:*` 및 스캐폴딩 명령을 지원하는 CLI
- [보안](/learn/security) – 에이전트(및 인간)가 약화시키지 말아야 할 안전한 기본값

## 문제 해결

- ".runway-creds.json이 없습니다"라는 메시지가 표시되면 먼저 `php runway ai:init`를 실행하세요.
- API 키가 유효하고 선택한 모델에 액세스할 수 있는지 확인하세요.
- 지침이 업데이트되지 않으면 프로젝트 디렉토리의 파일 권한을 확인하세요.
- 에이전트가 잘못된 Flight API나 잘못된 폴더 레이아웃을 생성하면 루트 **`AGENTS.md`** 및 이 문서 사이트를 안내하세요. `app/` 아래 코드는 스켈레톤 레이아웃이 우선입니다.

## 변경 로그

- v3.18.4 – `ai:generate-instructions`가 프로젝트 지침을 프로젝트 루트의 `AGENTS.md`에 작성합니다.
- v3.16.0 – AI 통합을 위한 `ai:init` 및 `ai:generate-instructions` CLI 명령 추가.