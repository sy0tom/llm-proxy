# 設計

## データ定義一覧

| データ                               | 内容                                    |
| ------------------------------------ | --------------------------------------- |
| [設定ファイル](./data/app-config.md) | `config.yaml`の構成、項目、モデル設定例 |

## ストレージ設計

当projectではストレージは使用しない。

## API一覧

| Method | Path                      | 用途                 | 優先度 |
| ------ | ------------------------- | -------------------- | ------ |
| GET    | /health                   | Gatewayの稼働確認    | 高     |
| POST   | /v1/messages              | Anthropic互換API     | 高     |
| POST   | /v1/messages/count_tokens | 入力トークン数の取得 | 高     |
| GET    | /v1/models                | モデル一覧の取得     | 中     |
| POST   | /v1/chat/completions      | OpenAI互換API        | 中     |
