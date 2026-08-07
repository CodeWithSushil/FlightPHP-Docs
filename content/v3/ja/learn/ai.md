# Flight と AI / 開発者エクスペリエンス

## 概要

Flight は AI コーディングツールと *連携する* ように設計されており、対抗するものではありません。シンプルで予測可能な API、[公式スケルトン](https://github.com/flightphp/skeleton) による明確なアプリ構成、そしてプロジェクト固有の指示ファイルにより、GitHub Copilot、Cursor、Windsurf、Claude Code、Gemini などのアシスタントは、あなたが手書きするのと同じパターンに従うことができます。

組み込みの Runway コマンドを使用して LLM プロバイダーに接続し、プロジェクトの指示を生成できるため、Flight はあなたとあなたのチームが、毎回同じコンテキストをチャットに貼り付けることなく、一貫性のある関連性の高い支援を受けられるようにします。

## はじめに

AI コーディングアシスタントは、プロジェクトのコンテキスト、規約、目標を理解しているときに最も役立ちます。Flight の AI ヘルパーを使用すると、次のことができます。

- プロジェクトを一般的な LLM プロバイダー（OpenAI、Grok、Claude など）に接続する
- プロジェクト固有の指示を生成・更新して、全員が同じガイダンスを受け取れるようにする
- 手書きコードと AI 生成コードを同じレイアウトに保つ（特にスケルトンを使用する場合）

これらの機能は Flight コア CLI（[Runway](/awesome-plugins/runway) 経由）に同梱されており、公式の [flightphp/skeleton](https://github.com/flightphp/skeleton) スターターに事前に組み込まれています。

### スケルトンが AI 向けに提供するもの

公式スターターは、AI ツールにとって **`AGENTS.md` を信頼できる情報源** として扱います。

| ファイル | 役割 |
|------|------|
| **`AGENTS.md`**（プロジェクトルート） | グローバルなルール、起動フロー、名前空間、DI、「やってはいけないこと」 |
| **スコープ付き `AGENTS.md`**（`app/`、`migrations/`、`tests/` など） | そのツリー内で作業する際の、軽量でフォルダ固有のヒント |
| **`SECURITY.md`** | シークレット、ヘッダー、XSS/SQL、報告——セキュリティは意図的に独立して管理されます |

スケルトンには、Copilot / Cursor / Gemini / Windsurf 用の独立したハウススタイルファイルは **ありません**。アシスタントはルートの `AGENTS.md` を参照させ（スコープ付きファイルへのリンクを辿らせます）、人間はこれらのファイルを完全に無視して [README](https://github.com/flightphp/skeleton) を使用できます。レイアウトはどちらの場合も同じです。

> **ドキュメントは API を教え、スケルトンはレイアウトを教えます。** これらのドキュメントにある短い `Flight::` の例は学習に適しています。スケルトンアプリでは、コントローラー内の静的ファサードよりも `App\…` クラス、コンストラクターインジェクション、`$this->app` を優先してください。[インストール](/install) と [オートローディング](/learn/autoloading) を参照してください。

## 基本的な使い方

### LLM 認証情報の設定

`ai:init` コマンドは、プロジェクトを LLM プロバイダーに接続する手順をガイドします。

```bash
php runway ai:init
```

次のプロンプトが表示されます。

- プロバイダーの選択（OpenAI、Grok、Claude など）
- API キーの入力
- ベース URL とモデル名の設定

これにより、後続の LLM リクエスト（指示の生成など）で使用される認証情報が作成されます。

**例:**
```
Welcome to AI Init!
Which LLM API do you want to use? [1] openai, [2] grok, [3] claude: 1
Enter the base URL for the LLM API [https://api.openai.com]:
Enter your API key for openai: sk-...
Enter the model name you want to use (e.g. gpt-4, claude-3-opus, etc) [gpt-4o]:
Credentials saved to .runway-creds.json
```

### プロジェクト固有の AI 指示の生成

`ai:generate-instructions` コマンドは、*あなたの*プロジェクトに合わせた AI コーディングアシスタント向けの指示を作成または更新します。

```bash
php runway ai:generate-instructions
```

いくつかの質問（説明、データベース、テンプレートエンジン、セキュリティ、チーム規模など）に答えます。Flight は LLM プロバイダーを使用して指示を生成し、主に次の場所に書き込みます。

- プロジェクトルートの **`AGENTS.md`**（ツールに依存しない形式。公式スケルトンと最新のエージェントのほとんどが期待する形式です）

CLI のバージョンとオプションによっては、このコマンドは古いワークフロー向けのツール固有のコピー（Copilot、Cursor、Windsurf、Gemini のルールファイルなど）も書き込む場合があります。**スケルトンからの新規プロジェクト**では、**`AGENTS.md`**（および `app/` の下に保持するスコープ付き `AGENTS.md` ファイル）を単一の情報源として扱い、5 つの異なる指示ファイルを手作業で保守しないでください。

**例:**
```
Please describe what your project is for? My awesome API
What database are you planning on using? MySQL
What HTML templating engine will you plan on using (if any)? twig
Is security an important element of this project? (y/n) y
...
AI instructions updated successfully.
```

これで、AI ツールは一般的な PHP チュートリアルではなく、実際のスタックとレイアウトに一致するコードを提案できるようになります。

## 高度な使い方

- コマンドオプションで認証情報や出力パスをカスタマイズできます（各コマンドの `--help` を参照）。
- これらのヘルパーは、OpenAI 互換 API を話す任意の LLM プロバイダーで動作します。
- プロジェクトの進化に合わせて `ai:generate-instructions` を再実行し、エージェントを最新の状態に保ってください。
- スケルトンでは、セキュリティポリシーは **`SECURITY.md`** に、コーディングレイアウトは **`AGENTS.md`** に維持し、どちらのドキュメントも雑多なものにしないでください。
- エージェントが API の詳細を必要とする場合は、[docs.flightphp.com](https://docs.flightphp.com) と Flight MCP サーバーを優先し、発明されたメソッドは `vendor/flightphp/core` に対して検証してください。

## 関連情報

- [Flight Skeleton](https://github.com/flightphp/skeleton) – `AGENTS.md`、Twig、SimplePdo、Dice を AI フレンドリーな構成で組み込んだ公式スターター
- [インストール](/install) – 推奨される `create-project` レイアウト
- [オートローディング](/learn/autoloading) – フォルダの**大文字小文字**が名前空間と一致します（`App\Controller` ↔ `app/Controller/`）
- [Runway CLI](/awesome-plugins/runway) – `ai:*` およびスキャフォールディングコマンドを提供する CLI
- [セキュリティ](/learn/security) – エージェント（そして人間も）が弱めるべきではない安全なデフォルト

## トラブルシューティング

- 「Missing .runway-creds.json」というエラーが表示された場合は、最初に `php runway ai:init` を実行してください。
- API キーが有効で、選択したモデルにアクセスできることを確認してください。
- 指示が更新されない場合は、プロジェクトディレクトリのファイル権限を確認してください。
- エージェントが Flight API をでっち上げたり、間違ったフォルダレイアウトを使用したりする場合は、ルートの **`AGENTS.md`** とこのドキュメントサイトを参照させてください。`app/` の下のコードではスケルトレイアウトが優先されます。

## 変更履歴

- v3.18.4 – `ai:generate-instructions` がプロジェクトルートの `AGENTS.md` にプロジェクト指示を書き込むようになりました。
- v3.16.0 – AI 統合用の CLI コマンド `ai:init` と `ai:generate-instructions` を追加しました。