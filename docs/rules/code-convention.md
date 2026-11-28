# コーディング規約

## 方針

責務の分け方は[ディレクトリ構成の実装方針](../directory.md#実装方針)に従う。

## TypeScript

- `tsconfig.json`の`strict`を有効にしたまま実装する。
- 型は`interface`ではなく`type`で定義する。
- 設定値など外部入力の型はZodのスキーマから`z.infer`で導出し、手書きしない。
- 外部入力は`unknown`で受け取り、Zodで検証してから使う。
- `any`を使わない。
- `export default`を使わず、名前付きexportにする。
- 関数は`function`宣言で定義する。

## エラー

- クライアントへのエラーは、受け付けたAPIの形式で返す。
  `/v1/messages`ではAnthropicのエラー形式にする。
- 上流Providerのエラーレスポンスは書き換えずに返す。
- 自動リトライとフォールバックを実装しない。

## ログ

- 認証情報、リクエスト本文、レスポンス本文をログに出力しない。

## 静的解析とフォーマット

- ESLintの指摘を解消する。設定は`eslint.config.js`にある。
- Prettierの既定の設定で整形する。
- Markdownはmarkdownlintの指摘を解消する。設定は`.markdownlint-cli2.jsonc`にある。
- cspellの指摘は綴りを直して解消する。
  正しい綴りの固有名詞やコマンド名だけを、`cspell.json`の`words`に小文字でアルファベット順に追加する。
