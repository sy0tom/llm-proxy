# システム構成図

## 目的

LLM Gateway の構成要素と、外部システムとの接続関係を説明する。

## 構成図

```mermaid
flowchart TD
    CLIENT["LLM Client"]

    subgraph Gateway["LLM Gateway"]
        API["HTTP API / Hono"]
        CONFIG["Configuration / YAML"]
        ROUTER["Model Router"]
        AA["Anthropic Adapter"]
        OA["Ollama Adapter"]
        PROXY["HTTP Proxy"]

        API --> ROUTER
        CONFIG --> ROUTER

        ROUTER --> AA
        ROUTER --> OA

        AA --> PROXY
        OA --> PROXY
    end

    ANTHROPIC["Anthropic API"]
    OLLAMA["Ollama API"]

    CLIENT -->|"Anthropic / OpenAI互換API"| API

    PROXY --> ANTHROPIC
    PROXY --> OLLAMA
```

## コンポーネントの責務

| コンポーネント   | 責務                                                               |
| ---------------- | ------------------------------------------------------------------ |
| HTTP API         | リクエストの受付、バリデーション、レスポンスの返却                 |
| Configuration    | 設定ファイルの読み込み、型定義、バリデーション                     |
| Model Router     | モデルIDからProviderおよび実モデルを解決                           |
| Provider Adapter | Provider固有のリクエスト変換、認証ヘッダー制御、推論パラメータ制御 |
| HTTP Proxy       | HTTP転送、Streaming、Timeout、Cancellation                         |
| Logger           | リクエストの処理状況およびエラーの出力                             |

各コンポーネントの実装ファイルは [ディレクトリ構成](../directory.md) を参照する。
