# レビュールール

## 流れ

自己レビュー、AIレビュー、開発者レビューの順に行う。
前の段階の指摘をすべて解消してから次に進む。

## 自己レビュー

- 作業したStepの完了条件をすべて満たしているか確認する。
- 型チェック、静的解析、整形、テストが成功することを確認する。
  commit時にはpre-commitで、stageしたファイルにESLint、markdownlint、cspell、Prettierが実行される。

  ```bash
  bun run typecheck
  bun run lint
  bun run format:check
  bun test
  ```

- 変更に関わる`docs/`の資料を更新したか確認する。

## AIレビュー

- push前に、Critを使ってlocalで対話式にレビューする。
- 実装したAIエージェントとcontextを分けるため、レビューはsubagentで行う。
- 機能仕様の受け入れ条件との整合は、subagentの`sdd-spec-reviewer`で確認する。

## 開発者レビュー

- GitHubのPR上で開発者がレビューする。
- CIが成功していることを確認してからレビューを依頼する。
  CIは`main`と`release/*`に向くPRで`.github/workflows/ci.yaml`が実行する。
