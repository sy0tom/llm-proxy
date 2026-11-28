# 機能一覧

## 目的

[要求定義](../requirements/REQUIREMENTS.md)の要求と、実現する機能の対応をまとめる。
詳細な仕様が必要になった機能は、`FC-xxx.md`または`NFC-xxx.md`を作成してリンクする。

## 機能要件

| No.                   | 要求No.      | 機能名                     | 説明                                                         | 関連資料                                      |
| --------------------- | ------------ | -------------------------- | ------------------------------------------------------------ | --------------------------------------------- |
| [FC-001](./FC-001.md) | FR-01, FR-05 | モデルルーティング         | モデルIDの完全一致・ワイルドカード一致でProviderを解決する   | [設定ファイル](../designs/data/app-config.md) |
| FC-002                | FR-04        | モデルエイリアス           | 抽象的なモデル名を実モデルに変換する                         | [設定ファイル](../designs/data/app-config.md) |
| FC-003                | FR-06        | 推論パラメータ制御         | モデルエイリアス単位で推論パラメータを上流へ指定する         | [設定ファイル](../designs/data/app-config.md) |
| FC-004                | FR-02        | Subscription認証の引き継ぎ | 既存のOAuth認証情報をAnthropicへのリクエストに引き継ぐ       | -                                             |
| FC-005                | FR-03        | Agent別のモデル利用        | Main AgentとSubagentで指定したモデルIDに応じて接続先を分ける | [FC-001](./FC-001.md)                         |
| FC-006                | FR-02, FR-03 | Anthropic互換API           | `/v1/messages`と`/v1/messages/count_tokens`を提供する        | [API一覧](../designs/README.md#api一覧)       |
| FC-007                | FR-12        | OpenAI互換API              | `/v1/models`と`/v1/chat/completions`を提供する               | [API一覧](../designs/README.md#api一覧)       |
| FC-008                | FR-07        | Streaming                  | SSEのレスポンスをバッファリングせず逐次転送する              | -                                             |
| FC-009                | FR-08        | Tool Calling               | Tool UseおよびTool Resultを含むリクエストを転送する          | -                                             |
| FC-010                | FR-09        | Timeout                    | モデル単位の実行時間上限で上流リクエストを中断する           | [設定ファイル](../designs/data/app-config.md) |
| FC-011                | FR-10        | Cancellation               | クライアントの切断を上流リクエストの中断へ伝播する           | -                                             |
| FC-012                | FR-11        | Logging                    | リクエスト単位の処理状況を構造化ログで出力する               | -                                             |
| FC-013                | NFR-06       | Healthcheck                | `/health`でGatewayの稼働状況を返却する                       | [API一覧](../designs/README.md#api一覧)       |

## 非機能要件

| No.     | 要求No.        | 非機能名           | 説明                                                       | 関連資料                                       |
| ------- | -------------- | ------------------ | ---------------------------------------------------------- | ---------------------------------------------- |
| NFC-001 | NFR-01, NFR-02 | Stateless          | 状態とDatabaseを持たない                                   | [設計](../designs/README.md#ストレージ設計)    |
| NFC-002 | NFR-03, NFR-04 | 設定管理           | YAMLで設定し、起動時にバリデーションする                   | [設定ファイル](../designs/data/app-config.md)  |
| NFC-003 | NFR-05         | 依存の最小化       | 標準機能を優先し、依存ライブラリを最小限にする             | [技術スタック](../architectures/tech-stach.md) |
| NFC-004 | NFR-06, NFR-07 | 実行環境           | macOSでのネイティブ実行とDockerでの実行に対応する          | [技術スタック](../architectures/tech-stach.md) |
| NFC-005 | NFR-08, NFR-09 | アクセス制限       | ユーザー認証を持たず、ポート公開をlocalhostに限定する      | -                                              |
| NFC-006 | NFR-10         | 認証情報の分離     | AnthropicのOAuth認証情報を他のProviderへ転送しない         | -                                              |
| NFC-007 | NFR-11         | ログの機密保護     | 認証情報とリクエスト・レスポンス本文をデフォルトで出さない | -                                              |
| NFC-008 | NFR-12, NFR-13 | Providerの拡張性   | Provider固有処理を分離し、互換APIを優先して利用する        | [ディレクトリ構成](../directory.md)            |
| NFC-009 | NFR-14         | エラーの透過       | 上流Providerのエラーレスポンスを可能な限り維持する         | -                                              |
| NFC-010 | NFR-15, NFR-16 | 未定義モデルの拒否 | 未定義モデルを拒否し、クラウドLLMへフォールバックしない    | [FC-001](./FC-001.md)                          |
