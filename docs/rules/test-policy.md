# テスト方針

## 共通

- テストは`bun:test`で書き、`bun test`で実行する。
- テストは`tests/`に置き、`src/`と同じディレクトリ構成にする。
- 各Stepの完了条件を確認するテストを書く。

## 単体テスト

- 条件網羅とし、カバレッジは90%以上にする。
- カバレッジは`bun test --coverage`で確認する。
- 上流Providerへの通信はモックにし、OllamaやAnthropicに接続しない。

## 結合テスト

- `tests/integration/`に置く。
- Gatewayを起動し、HTTPリクエストでAPIを検証する。
- Streaming、Tool Calling、Cancellationを検証する。
- Claude CodeなどHarnessからの動作は、E2Eとして手動で確認する。
