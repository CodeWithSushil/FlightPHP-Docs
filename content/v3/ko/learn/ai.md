# AI 및 개발자 경험과 Flight

## 개요

Flight는 AI 기반 도구와 현대적인 개발자 워크플로우를 통해 PHP 프로젝트를 쉽게 강화할 수 있게 해줍니다. LLM(Large Language Model) 제공자와 연결하고 프로젝트별 AI 코딩 지침을 생성하기 위한 내장 명령어를 통해, Flight는 GitHub Copilot, Cursor, Windsurf, Antigravity(Gemini)와 같은 AI 어시스턴트를 최대한 활용할 수 있도록 도와줍니다.

## 이해하기

AI 코딩 어시스턴트는 프로젝트의 맥락, 규칙, 목표를 이해할 때 가장 유용합니다. Flight의 AI 도우미를 사용하면 다음을 수행할 수 있습니다:
- 인기 있는 LLM 제공자(OpenAI, Grok, Claude 등)에 프로젝트를 연결
- AI 도구를 위한 프로젝트별 지침을 생성하고 업데이트하여 모든 사람이 일관되고 관련성 있는 도움을 받을 수 있도록 함
- 맥락 설명에 소요되는 시간을 줄이고 팀의 정렬과 생산성을 유지

이러한 기능은 Flight 코어 CLI와 공식 [flightphp/skeleton](https://github.com/flightphp/skeleton) 스타터 프로젝트에 내장되어 있습니다.

## 기본 사용법

### LLM 자격 증명 설정

`ai:init` 명령어는 프로젝트를 LLM 제공자와 연결하는 과정을 안내합니다.

```bash
php runway ai:init
```

다음과 같은 안내를 받게 됩니다:
- 제공자 선택 (OpenAI, Grok, Claude 등)
- API 키 입력
- 기본 URL 및 모델 이름 설정

이렇게 하면 향후 LLM 요청을 위한 필요한 자격 증명이 생성됩니다.

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

`ai:generate-instructions` 명령어는 프로젝트에 맞게 AI 코딩 어시스턴트용 지침을 생성하거나 업데이트하는 데 도움을 줍니다.

```bash
php runway ai:generate-instructions
```

프로젝트에 대한 몇 가지 질문(설명, 데이터베이스, 템플릿, 보안, 팀 규모 등)에 답하게 됩니다. Flight는 LLM 제공자를 사용하여 지침을 생성한 다음, 동일한 내용을 다음 위치에 작성합니다:
- `.github/copilot-instructions.md` (GitHub Copilot용)
- `.cursor/rules/project-overview.mdc` (Cursor용)
- `.windsurfrules` (Windsurf용)
- `.gemini/GEMINI.md` (Antigravity용)
- `AGENTS.md` (프로젝트 루트에 위치, 도구에 구애받지 않는 AI 어시스턴트용)

**예시:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

이제 AI 도구는 프로젝트의 실제 요구사항에 기반하여 더 스마트하고 관련성 있는 제안을 제공합니다.

## 고급 사용법

- 명령어 옵션을 사용하여 자격 증명 또는 지침 파일의 위치를 사용자 정의할 수 있습니다 (각 명령어에 대한 `--help` 참조).
- AI 도우미는 OpenAI 호환 API를 지원하는 모든 LLM 제공자와 함께 작동하도록 설계되었습니다.
- 프로젝트가 발전함에 따라 지침을 업데이트하려면 `ai:generate-instructions`를 다시 실행하고 프롬프트에 다시 답하면 됩니다.

## 참고 항목

- [Flight Skeleton](https://github.com/flightphp/skeleton) – AI 통합이 포함된 공식 스타터
- [Runway CLI](/awesome-plugins/runway) – 이러한 명령어를 구동하는 CLI 도구에 대한 자세한 정보

## 문제 해결

- "Missing .runway-creds.json" 메시지가 표시되면 먼저 `php runway ai:init`를 실행하세요.
- API 키가 유효하고 선택한 모델에 대한 접근 권한이 있는지 확인하세요.
- 지침이 업데이트되지 않는 경우, 프로젝트 디렉토리의 파일 권한을 확인하세요.

## 변경 로그

- v3.18.4 – `ai:generate-instructions`가 이제 프로젝트 루트의 `AGENTS.md`에도 프로젝트 지침을 작성합니다.
- v3.16.0 – AI 통합을 위한 `ai:init` 및 `ai:generate-instructions` CLI 명령어 추가.