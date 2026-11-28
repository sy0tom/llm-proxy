# ディレクトリ構成

## 目的

リポジトリと`src`配下の構成、各ファイルの責務を説明する。

## 構成

```text
llm-proxy/
 |- src/
 |   |- server.ts
 |   |- app.ts
 |   |- config.ts
 |   |- router.ts
 |   |- proxy.ts
 |   |- logger.ts
 |   |- providers/
 |       |- types.ts
 |       |- anthropic.ts
 |       |- ollama.ts
 |
 |- tests/
 |   |- config.test.ts
 |   |- router.test.ts
 |   |- proxy.test.ts
 |   |- providers/
 |   |   |- anthropic.test.ts
 |   |   |- ollama.test.ts
 |   |- integration/
 |       |- messages.test.ts
 |       |- streaming.test.ts
 |       |- cancellation.test.ts
 |       |- tool-calling.test.ts
 |
 |- docs/
 |
 |- config.yaml
 |- config.example.yaml
 |- package.json
 |- tsconfig.json
 |- bun.lock
 |- Dockerfile
 |- compose.yaml
 |- .dockerignore
 |- .gitignore
 |- README.md
```

## ファイルの責務

| パス                   | 説明                                                      |
| ---------------------- | --------------------------------------------------------- |
| server.ts              | 設定の読み込み、HTTP Serverの起動                         |
| app.ts                 | Honoアプリケーションの生成、APIエンドポイント、Middleware |
| config.ts              | 設定ファイルの読み込み、型定義、バリデーション            |
| router.ts              | モデルIDからProviderおよび実モデルを解決                  |
| proxy.ts               | HTTP転送、Streaming、Timeout、Cancellation                |
| logger.ts              | 構造化ログの生成・出力                                    |
| providers/types.ts     | Provider Adapterの共通インターフェース                    |
| providers/anthropic.ts | Anthropic固有のリクエスト処理                             |
| providers/ollama.ts    | Ollama固有のリクエスト処理、推論パラメータ制御            |
| config.yaml            | 実行環境で利用する設定ファイル                            |
| config.example.yaml    | 設定ファイルのサンプル                                    |
| tests/                 | Unit Test、Integration Test                               |
| docs/                  | 要件書および設計資料                                      |
| docs/templates/        | 機能仕様、ADR、作業計画のテンプレート                     |

## 実装方針

- HTTP API、モデルルーティング、Provider固有処理、HTTP Proxy処理の責務を分離する。
- Provider固有の処理を`app.ts`や`proxy.ts`に含めない。
- Provider Adapterは上流へのリクエスト生成および必要なレスポンス変換を担当する。
- HTTP ProxyはProviderに依存しない共通処理を担当する。
- 初期段階では過剰な抽象化を行わない。
- 複数のProviderで共通処理が発生した段階で共通化する。
- 各Stepで必要となるファイルのみを作成する。
- 各Stepの完了条件を満たすテストを実装する。
