# 技術スタック

## 目的

採用する言語、ライブラリ、実行環境と、その選定方針を説明する。

## ライブラリ / フレームワーク

| 分類                 | 技術                          | 用途                                 |
| -------------------- | ----------------------------- | ------------------------------------ |
| 言語                 | TypeScript                    | アプリケーション実装                 |
| Runtime              | Bun                           | アプリケーション実行、パッケージ管理 |
| Web Framework        | Hono                          | HTTP API、Middleware                 |
| HTTP Client          | Fetch API                     | 上流ProviderへのHTTPリクエスト       |
| Streaming            | Web Streams API               | Streamingレスポンスの転送            |
| Cancellation         | AbortController / AbortSignal | リクエストの中断                     |
| YAML Parser          | Bun.YAML                      | 設定ファイルの読み込み               |
| Validation           | Zod                           | 設定ファイルのバリデーション、型定義 |
| Test                 | bun:test                      | Unit Test、Integration Test          |
| Container            | Docker                        | 実行環境のコンテナ化                 |
| Container Management | Docker Compose                | コンテナの起動・停止・管理           |
| Logging              | console                       | 構造化ログの標準出力                 |
| Local LLM Runtime    | Ollama                        | Local LLMの推論実行                  |

## 開発ツール

| 分類       | 技術                      | 用途                                    |
| ---------- | ------------------------- | --------------------------------------- |
| Linter     | ESLint、typescript-eslint | TypeScriptの静的解析                    |
| Linter     | markdownlint-cli2         | Markdownの静的解析                      |
| Linter     | cspell                    | 英単語の綴りの検査                      |
| Formatter  | Prettier                  | コード、Markdown、設定ファイルの整形    |
| Git hooks  | husky、lint-staged        | commit前にstageしたファイルへlintと整形 |
| 依存の更新 | Renovate                  | 依存の更新PRの作成                      |
| 依存の更新 | Dependabot                | 依存のセキュリティ更新PRの作成          |
| CI         | GitHub Actions            | PRでの型チェック、lint、整形、テスト    |

typescript-eslintがTypeScript 7に対応していないため、TypeScriptは6系を使う。

## 技術選定方針

- Bunの標準機能を優先的に利用する。
- HTTP ServerにはHonoを利用する。
- HTTP Clientには外部ライブラリを導入せず、Fetch APIを利用する。
- YAML ParserにはBunの標準機能を利用する。
- TypeScriptの型定義と設定バリデーションはZodに集約する。
- Loggingライブラリは初期段階では導入しない。
- Nginxなどのリバースプロキシは前段に配置しない。
- OllamaはApple SiliconのGPUを利用するため、macOS上で実行する。
- Docker化する対象はLLM Gatewayのみとする。
- 利用するライブラリのバージョンはlockfileで管理する。
