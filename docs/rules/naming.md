# 命名規則

## 方針

ドメインの名前は[用語集](../glossary.md)の英名を使う。

## 規則

| 対象               | 規則                  | 例                              |
| ------------------ | --------------------- | ------------------------------- |
| ファイル           | kebab-case            | `tool-calling.test.ts`          |
| テストファイル     | `<対象>.test.ts`      | `router.test.ts`                |
| 関数・変数         | camelCase             | `resolveModel`、`upstreamModel` |
| 型                 | PascalCase            | `ResolvedModel`                 |
| Zodスキーマ        | PascalCase + `Schema` | `ProviderSchema`                |
| Provider固有の関数 | 動詞 + Provider名     | `requestOllama`                 |
| 設定ファイルのキー | snake_case            | `base_url`、`forward_auth`      |
| モデルエイリアス   | kebab-case            | `local-lite`                    |

- 関数名は動詞から始める。
- 設定ファイルのキーは、Providerの推論パラメータ名と同じ名前を使う。
  例: `num_ctx`、`num_predict`。
