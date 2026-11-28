# AGENTS.md

Claude Codeなどのリクエストを、モデルIDに応じてAnthropicとOllamaに振り分ける軽量なLLM Gateway。
目的と要件は[要求定義](docs/requirements/REQUIREMENTS.md)にある。

## ルール

- 回答とドキュメントは日本語で書く。
- 作業はロードマップのStep単位で進め、Stepの完了条件を満たす。
- `docs/`のルールに従う。変更が`docs/`の記述に関わるときは、その資料も更新する。
- `config.yaml`に認証情報を書かない。`.env*`、鍵、認証情報は読まない、書かない、コミットしない。

## コマンド

```bash
bun install         # 依存のインストール、Git hooksの設定
bun run dev         # Gatewayの起動(Hot Reload)
bun test            # テスト
bun run typecheck   # 型チェック
bun run lint        # ESLint、markdownlint
bun run format      # Prettierでの整形
```

## ドキュメント

必要になったときに読む。

| いつ読むか                                  | 資料                                                                                                         |
| ------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| 要件、作業のStepと完了条件を確認する        | [docs/requirements/REQUIREMENTS.md](docs/requirements/REQUIREMENTS.md)                                       |
| Stepを進める、機能仕様、ADR、作業計画を書く | [docs/rules/sdd.md](docs/rules/sdd.md)                                                                       |
| 機能の仕様と要求の対応を確認する            | [docs/specs/README.md](docs/specs/README.md)                                                                 |
| 構成、技術スタックを確認する                | [docs/architectures/README.md](docs/architectures/README.md)                                                 |
| ファイルを追加・移動する                    | [docs/directory.md](docs/directory.md)                                                                       |
| API、設定ファイルを変更する                 | [docs/designs/README.md](docs/designs/README.md)                                                             |
| コードを書く                                | [docs/rules/code-convention.md](docs/rules/code-convention.md)、[docs/rules/naming.md](docs/rules/naming.md) |
| テストを書く                                | [docs/rules/test-policy.md](docs/rules/test-policy.md)                                                       |
| ブランチ、commit、PRを扱う                  | [docs/rules/git.md](docs/rules/git.md)                                                                       |
| レビューする、レビューを依頼する            | [docs/rules/review.md](docs/rules/review.md)                                                                 |
| skills、subagentsを追加・変更する           | [docs/harness.md](docs/harness.md)                                                                           |
| project固有の用語がわからない               | [docs/glossary.md](docs/glossary.md)                                                                         |
