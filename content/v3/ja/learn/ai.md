# AIとFlightの開発者体験

## 概要

Flightは、AIを活用したツールとモダンな開発者ワークフローでPHPプロジェクトを強化します。LLM（大規模言語モデル）プロバイダーへの接続や、プロジェクト固有のAIコーディング指示の生成を行う組み込みコマンドにより、GitHub Copilot、Cursor、Windsurf、Antigravity（Gemini）などのAIアシスタントを最大限に活用できます。

## 理解する

AIコーディングアシスタントは、プロジェクトの文脈・規約・目標を理解しているときに最も役立ちます。FlightのAIヘルパーは以下を可能にします:
- 人気のLLMプロバイダー（OpenAI、Grok、Claudeなど）へのプロジェクト接続
- AIツール向けのプロジェクト固有指示の生成・更新により、一貫性のある関連性の高い支援を実現
- チーム全体の連携と生産性を向上させ、文脈説明にかかる時間を削減

これらの機能はFlightコアCLIおよび公式の[flightphp/skeleton](https://github.com/flightphp/skeleton)スタータープロジェクトに組み込まれています。

## 基本的な使い方

### LLM認証情報の設定

`ai:init`コマンドは、プロジェクトをLLMプロバイダーに接続する手順を案内します。

```bash
php runway ai:init
```

以下の入力を求められます:
- プロバイダーの選択（OpenAI、Grok、Claudeなど）
- APIキーの入力
- ベースURLとモデル名の設定

これにより、今後のLLMリクエストに必要な認証情報が作成されます。

**例:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### プロジェクト固有のAI指示の生成

`ai:generate-instructions`コマンドは、プロジェクトに合わせたAIコーディングアシスタント向け指示の作成または更新を支援します。

```bash
php runway ai:generate-instructions
```

プロジェクトに関するいくつかの質問（説明、データベース、テンプレートエンジン、セキュリティ、チーム規模など）に回答します。FlightはLLMプロバイダーを使用して指示を生成し、同じ内容を以下のファイルに書き込みます:
- `.github/copilot-instructions.md`（GitHub Copilot用）
- `.cursor/rules/project-overview.mdc`（Cursor用）
- `.windsurfrules`（Windsurf用）
- `.gemini/GEMINI.md`（Antigravity用）
- `AGENTS.md`（プロジェクトルートに配置し、ツール非依存のAIアシスタント用）

**例:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? latte
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

これで、AIツールはプロジェクトの実際の要件に基づいて、よりスマートで関連性の高い提案を行えるようになります。

## 高度な使い方

- コマンドオプション（各コマンドの`--help`を参照）を使用して、認証情報や指示ファイルの保存場所をカスタマイズできます。
- AIヘルパーは、OpenAI互換APIをサポートする任意のLLMプロバイダーで動作するよう設計されています。
- プロジェクトの進化に伴い指示を更新したい場合は、`ai:generate-instructions`を再度実行してプロンプトに回答してください。

## 関連項目

- [Flight Skeleton](https://github.com/flightphp/skeleton) – AI統合を含む公式スターター
- [Runway CLI](/awesome-plugins/runway) – これらのコマンドを支えるCLIツールの詳細

## トラブルシューティング

- 「Missing .runway-creds.json」と表示された場合は、まず`php runway ai:init`を実行してください。
- APIキーが有効で、選択したモデルにアクセス可能であることを確認してください。
- 指示が更新されない場合は、プロジェクトディレクトリのファイル権限を確認してください。

## 変更履歴

- v3.18.4 – `ai:generate-instructions`がプロジェクト指示をプロジェクトルートの`AGENTS.md`にも書き込むようになりました。
- v3.16.0 – AI統合用の`ai:init`および`ai:generate-instructions` CLIコマンドを追加しました。