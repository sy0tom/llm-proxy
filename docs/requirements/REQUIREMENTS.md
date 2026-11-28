# 要求定義

## 背景・目的

Claude Codeを利用した開発作業において、すべてのタスクをAnthropicのモデルで実行すると、Subscriptionの利用枠を消費する。

コード探索、情報収集、要約など、高度な推論能力を必要としないタスクをLocal LLMに委譲することで、Subscriptionの利用枠の消費を抑えたい。

既存のLLM GatewayであるLiteLLMを利用することで実現可能だが、個人のローカル開発環境で利用するには以下の課題がある。

- PostgreSQLやPrismaなどの依存関係が多い。
- 管理画面、Virtual Key、コスト管理など、今回の用途では不要な機能が多い。
- セットアップおよびアップデート時のメンテナンスコストが高い。
- Local LLMの推論パラメータやタイムアウトを柔軟に制御したい。
- Subagentの応答遅延や実行中断時の挙動を制御したい。

上記を踏まえ、モデルルーティングとProxy機能に特化した軽量なLLM Gatewayを開発する。

## ゴール

### 機能要件

要求と機能の対応は[機能一覧](../specs/README.md#機能要件)を参照する。

- FR-01: リクエストのモデルIDに応じて接続先のLLM Providerを切り替えられる。
- FR-02: Claude Codeの既存Subscriptionを継続利用できる。
  - Anthropic API Keyによる従量課金への切り替えを必要としない。
  - 既存のOAuth認証情報をAnthropicへのリクエストに引き継ぐ。
- FR-03: Claude CodeのMain AgentとSubagentで異なるモデルを利用できる。
  - Main AgentはAnthropicのモデルを利用する。
  - 特定のSubagentはOllamaのLocal LLMを利用する。
  - `model: inherit`を指定したSubagentはMain Agentのモデルを利用する。
- FR-04: モデルエイリアスを定義できる。
  - 同一のLocal LLMに対して異なる推論パラメータを設定できる。
  - `local-lite`、`local-middle`、`local-heavy`などの抽象的なモデル名を利用できる。
  - 実モデルの変更時にクライアント側の設定変更を必要としない。
- FR-05: モデルIDのワイルドカード指定に対応する。
- FR-06: モデルエイリアス単位で推論パラメータを制御できる。
  - コンテキスト容量
  - 最大生成トークン数
  - Thinkingの有効・無効
  - Temperature
  - Top P
  - Top K
  - その他Providerが対応する推論パラメータ
- FR-07: Streaming（SSE）に対応する。
- FR-08: Tool Callingに対応する。
- FR-09: モデル単位でリクエストのタイムアウトを設定できる。
- FR-10: クライアントからのリクエスト中断を上流Providerへ伝播できる。
- FR-11: リクエストの処理状況をログで確認できる。
- FR-12: Claude Code以外のAIエージェントからも利用できる構成とする。
  - Cursor
  - Codex

### 非機能要件

要求と機能の対応は[機能一覧](../specs/README.md#非機能要件)を参照する。

- NFR-01: Statelessなアプリケーションとする。
- NFR-02: Databaseを必要としない。
- NFR-03: 設定はYAMLファイルで管理する。
- NFR-04: 起動時に設定ファイルのバリデーションを行う。
- NFR-05: 依存ライブラリを必要最小限とする。
- NFR-06: macOS上でのネイティブ実行およびDockerでの実行に対応する。
- NFR-07: ローカル開発環境での利用を前提とする。
- NFR-08: Gateway自体によるユーザー認証・API Key管理は行わない。
- NFR-09: ホスト側へのポート公開は原則としてlocalhostに限定する。
- NFR-10: AnthropicのOAuth認証情報を他のProviderへ転送しない。
- NFR-11: 認証情報、リクエスト本文、レスポンス本文はデフォルトでログに出力しない。
- NFR-12: Providerの追加・変更による既存Providerへの影響を最小限とする。
- NFR-13: Providerが提供する互換APIを優先的に利用し、独自のAPI変換処理を極力実装しない。
- NFR-14: 上流Providerのエラーレスポンスを可能な限り維持する。
- NFR-15: 未定義のモデルへのリクエストは拒否する。
- NFR-16: 意図しないクラウドLLMへのフォールバックを行わない。

### 対象外

以下の機能は実装対象外とする。

- 管理画面
- Database
- Virtual Keyの発行・管理
- ユーザー・チーム管理
- トークン使用量の集計・永続化
- トークン利用枠の管理
- コスト計算・予算管理
- レートリミット管理
- ロードバランシング
- 自動フォールバック
- リクエストの永続化
- エージェントの実行管理
- エージェント間のコンテキスト共有
- 開発ワークフローの制御

なお、クライアントの動作に必要なトークン数取得APIへの対応は、トークン使用量管理とは区別する。

また、Subagentの探索回数、ファイル読み込み数、利用可能なToolsなどの制御はClaude Code等のHarness側の責務とする。

## 技術スタック・設計

技術スタックと設計は以下の資料に記載する。

| 項目                        | 資料                                                                       |
| --------------------------- | -------------------------------------------------------------------------- |
| システム構成                | [システム構成図](../architectures/system-overview.md#構成図)               |
| コンポーネントの責務        | [システム構成図](../architectures/system-overview.md#コンポーネントの責務) |
| モデルルーティング          | [モデルルーティング](../specs/FC-001.md)                                   |
| モデル設定例                | [設定ファイル](../designs/data/app-config.md#モデル設定例)                 |
| APIエンドポイント           | [設計](../designs/README.md#api一覧)                                       |
| 設定ファイル                | [設定ファイル](../designs/data/app-config.md)                              |
| ライブラリ / フレームワーク | [技術スタック](../architectures/tech-stach.md#ライブラリ--フレームワーク)  |
| 技術選定方針                | [技術スタック](../architectures/tech-stach.md#技術選定方針)                |
| パッケージ構成              | [ディレクトリ構成](../directory.md#構成)                                   |
| ファイルの責務              | [ディレクトリ構成](../directory.md#ファイルの責務)                         |
| 実装方針                    | [ディレクトリ構成](../directory.md#実装方針)                               |
| 要求と機能の対応            | [機能一覧](../specs/README.md)                                             |
| 用語                        | [用語集](../glossary.md)                                                   |

## ロードマップ

### Stepの進め方

各Stepは仕様駆動開発(SDD)で、以下の段階の順に進める。
前の段階の成果物をもとに次の段階を進め、Stepの完了条件をすべて満たしてから次のStepに進む。

| 順番 | 段階     | やること                                                        | 成果物                                              |
| ---- | -------- | --------------------------------------------------------------- | --------------------------------------------------- |
| 1    | 要求     | Stepのやることと完了条件が、どの要求(FR、NFR)に当たるか確認する | 本書、[機能一覧](../specs/README.md)                |
| 2    | 機能仕様 | 要求を満たす振る舞いと、受け入れ条件を決める                    | `docs/specs/`の機能仕様(FC-xxx、NFC-xxx)            |
| 3    | 設計     | API、設定ファイル、構成を決め、判断の理由を残す                 | `docs/designs/`、`docs/architectures/`、`docs/adr/` |
| 4    | タスク   | Stepを、レビューできる大きさのタスクに分ける                    | 作業計画                                            |
| 5    | 実装     | 作業計画に沿って実装する                                        | `src/`                                              |
| 6    | テスト   | 受け入れ条件とStepの完了条件をテストで確認する                  | `tests/`                                            |

- 要求No.、機能No.、テストの対応を保つ。
- 各段階の書き方、成果物のテンプレート、作業計画の置き場所はStep0で決める。

### Step0: SDDの基盤づくり

#### やること

- Stepの進め方の段階ごとに、やること、成果物、置き場所、IDの付け方をルールとして定める。
- 機能仕様とADRを作る基準を定める。
- 要求No.、機能No.、テストの対応を記録する方法を定める。
- 機能仕様(FC-xxx、NFC-xxx)、ADR、作業計画のテンプレートを作成する。
- 既存の機能仕様をテンプレートの形式にそろえる。
- 機能仕様の作成、作業計画の作成、実装を行うskillsを作成する。
- 実装が機能仕様の受け入れ条件を満たしているか確認するsubagentを作成する。
- skillsとsubagentsを、Claude Code、Cursor、Codexから利用できるようにする。

#### 完了条件

- [ ] 段階ごとのやること、成果物、置き場所、IDの付け方が`docs/`のルールに書かれ、`AGENTS.md`から参照できる。
- [ ] 要求No.、機能No.、テストの対応を記録する場所と方法が決まっている。
- [ ] 機能仕様、ADR、作業計画のテンプレートがある。
- [ ] `FC-001.md`がテンプレートの形式になっている。
- [ ] skillsとsubagentsが`.agents/library/`にあり、Claude Code、Cursor、Codexから呼び出せる。
- [ ] Step1を対象に、skillsとテンプレートで機能仕様と作業計画の下書きを作成できることを確認している。

### Step1: プロジェクトの初期化・設定管理

#### やること

- Bunプロジェクトを初期化する。
- Hono、Zodを導入する。
- TypeScriptの設定を行う。
- `config.ts`を実装する。
  - YAMLファイルの読み込み
  - Zodによるバリデーション
  - TypeScriptの型定義
  - デフォルト値の設定
  - Providerの参照整合性チェック
  - Provider URLのバリデーション
  - モデル名およびワイルドカードのバリデーション
- `config.example.yaml`を作成する。
- 設定ファイルのUnit Testを実装する。
- `index.ts`および`app.ts`を実装する。
- Healthcheck APIを実装する。

#### 完了条件

- [ ] Bunでアプリケーションを起動できる。
- [ ] YAMLファイルから設定を読み込める。
- [ ] 設定値の型が正しく定義されている。
- [ ] 不正な設定値が存在する場合は起動に失敗する。
- [ ] 存在しないProviderを参照した場合は起動に失敗する。
- [ ] 環境変数で設定ファイルのパスを変更できる。
- [ ] `/health`にアクセスすると正常なレスポンスが返却される。
- [ ] Unit Testがすべて成功する。

### Step2: モデルルーティング・Ollamaへの接続

#### やること

- `router.ts`を実装する。
  - モデルIDの完全一致
  - ワイルドカード一致
  - Providerの解決
  - 実モデル名の解決
  - 未定義モデルのエラー処理
- `providers/ollama.ts`を実装する。
  - OllamaへのHTTPリクエスト
  - モデルエイリアスから実モデル名への変換
  - 認証情報の除去
- `/v1/messages`を実装する。
- OllamaのAnthropic互換APIを利用してGemmaへのリクエストを転送する。
- 上流Providerのレスポンスをクライアントへ返却する。
- HTTPエラーおよび接続エラーを処理する。
- Unit Testを実装する。

#### 完了条件

- [ ] モデルIDからProviderを解決できる。
- [ ] モデルエイリアスから実モデル名を解決できる。
- [ ] 完全一致がワイルドカード一致より優先される。
- [ ] 複数のワイルドカードに一致する場合、最長のプレフィックスが優先される。
- [ ] 未定義モデルへのリクエストが拒否される。
- [ ] Gateway経由でGemmaにリクエストを送信できる。
- [ ] Gemmaのレスポンスを取得できる。
- [ ] OllamaにAnthropicのOAuth認証情報が転送されない。
- [ ] Ollamaが停止している場合に適切なエラーが返却される。
- [ ] Unit Testがすべて成功する。

### Step3: Streaming・Tool Callingへの対応

#### やること

- Streamingレスポンスの転送を実装する。
- SSEのレスポンスヘッダーおよびイベント形式を維持する。
- レスポンス全体をバッファリングせず、逐次クライアントへ転送する。
- OllamaのAnthropic互換APIによるTool Callingの動作を確認する。
- Tool UseおよびTool Resultを含むリクエスト・レスポンスを検証する。
- Claude CodeからGateway経由でLocal LLMを利用する。
- Claude Codeの動作に必要な関連APIを調査する。
- `/v1/messages/count_tokens`への対応方針を決定する。

#### 完了条件

- [ ] 通常レスポンスが正常に返却される。
- [ ] Streamingレスポンスが正常に返却される。
- [ ] Streaming中にレスポンスが逐次表示される。
- [ ] Claude CodeからLocal LLMをMain Agentとして利用できる。
- [ ] Globによるファイル探索が正常に動作する。
- [ ] Grepによるファイル検索が正常に動作する。
- [ ] Readによるファイル読み込みが正常に動作する。
- [ ] Tool Resultを利用した後続の推論が正常に動作する。
- [ ] Claude Codeの動作に必要なAPIの対応方針が決定している。
- [ ] API変換処理を独自実装する必要性が明確になっている。

### Step4: 推論パラメータ制御

#### やること

- OllamaのAnthropic互換APIで制御可能な推論パラメータを調査する。
- OllamaのネイティブAPIとの仕様差異を確認する。
- `providers/ollama.ts`に推論パラメータの制御を実装する。
  - num_ctx
  - num_predict
  - think
  - temperature
  - top_p
  - top_k
- モデルエイリアス単位で推論パラメータを指定できるようにする。
- クライアント指定値とGateway設定値の優先順位を定義する。
- 最大生成トークン数はクライアント指定値とGateway設定値の小さい方を使用する。
- 対応していない推論パラメータを黙って無視しない。
- 同一モデルに対して複数のモデルエイリアスを設定する。
  - local-lite
  - local-middle
  - local-heavy
- コンテキスト容量、メモリ使用量、推論時間を計測する。
- コンテキスト容量の変更によるモデル再ロードの有無を確認する。

#### 完了条件

- [ ] モデルエイリアス単位で推論パラメータを設定できる。
- [ ] 同一のLocal LLMに異なる設定を適用できる。
- [ ] 最大生成トークン数を制御できる。
- [ ] コンテキスト容量を制御できる、または代替手段が確立されている。
- [ ] Thinkingの有効・無効を制御できる、またはモデルの制約が明確になっている。
- [ ] `ollama ps`でコンテキスト容量およびメモリ使用量を確認できる。
- [ ] 設定変更前後の推論時間を比較できる。
- [ ] 推論パラメータ変更によるモデル再ロードの影響を把握している。
- [ ] モデルの再作成なしで制御可能なパラメータが明確になっている。

#### 補足

OllamaのAnthropic互換APIで`num_ctx`や`think`を直接制御できるかは未確認である。

以下の優先順位で実現方法を検討する。

1. Anthropic互換APIによる直接指定
2. Ollama側の環境変数によるデフォルト値の設定
3. Modelfileによるモデル設定の分離
4. OllamaネイティブAPIへの変換処理の実装

API変換を実装する場合は、Tool Calling、Thinking、Streaming、エラーレスポンスの変換も必要となるため、実装コストを考慮して判断する。

### Step5: HTTP Proxyの共通化・Timeout・Cancellation

#### やること

- `proxy.ts`を実装する。
- Provider固有の処理とHTTP転送処理を分離する。
- Provider Adapterの共通インターフェースを定義する。
- HTTPリクエストおよびレスポンスヘッダーの転送を制御する。
- Hop-by-hopヘッダーを適切に除去する。
- リクエストボディ書き換え時のContent-Lengthを適切に処理する。
- 上流へのリダイレクトを制御し、意図しない接続先への認証情報の転送を防止する。
- モデル単位のタイムアウトを実装する。
- クライアント切断時のCancellationを実装する。
- Streaming中のタイムアウトおよびCancellationに対応する。
- 上流接続のエラーハンドリングを実装する。
- 自動リトライは行わない。
- Unit TestおよびIntegration Testを実装する。

#### 完了条件

- [ ] HTTP転送処理がProviderに依存していない。
- [ ] 通常レスポンスとStreamingレスポンスを共通処理で転送できる。
- [ ] モデル単位でタイムアウトを設定できる。
- [ ] タイムアウト時に上流リクエストが中断される。
- [ ] クライアント切断時に上流リクエストが中断される。
- [ ] Streaming中の切断時にも上流リクエストが中断される。
- [ ] Claude CodeのEscによる中断がGatewayおよびOllamaに伝播する。
- [ ] 中断後に意図しない再リクエストが発生しない。
- [ ] 上流Providerのエラーを適切に返却できる。
- [ ] Unit TestおよびIntegration Testがすべて成功する。

#### 補足

リクエストの中断とOllamaモデルのアンロードは別の処理として扱う。

Cancellationによって推論処理を中断することを目的とし、モデルのメモリからのアンロードはOllama側のkeep_alive設定に委ねる。

### Step6: Anthropicへの接続・Subscription連携

#### やること

- AnthropicのSubscription認証をGateway経由で利用する構成について、公式仕様および利用規約を確認する。
- `providers/anthropic.ts`を実装する。
- Anthropic向けリクエストの認証情報を維持する。
- Anthropic向けリクエストのモデルIDおよび推論パラメータを原則変更しない。
- Anthropic固有のHTTPヘッダーを維持する。
- Anthropic向けのURLパスおよびクエリパラメータを維持する。
- `claude-*`によるモデルルーティングを実装する。
- Anthropic API Keyによる認証への自動切り替えは行わない。
- Claude Codeの既存Subscriptionを利用した動作確認を行う。
- `/v1/messages/count_tokens`など、Claude Codeの動作に必要なAPIへ対応する。
- AnthropicとOllamaの認証情報が混在しないことを確認する。

#### 完了条件

- [ ] Subscription認証の転送方式について公式仕様および利用規約を確認している。
- [ ] Claude Codeの既存ログイン状態を維持できる。
- [ ] Claude CodeからGateway経由でAnthropicにリクエストを送信できる。
- [ ] Subscription認証で推論が実行されることを確認できる。
- [ ] Anthropic API Keyの発行を必要としない。
- [ ] 意図しないAPI従量課金への切り替えが発生しない。
- [ ] Claude Codeの通常応答およびStreamingが正常に動作する。
- [ ] Claude CodeのTool Callingが正常に動作する。
- [ ] AnthropicのOAuth認証情報がOllamaへ転送されない。
- [ ] Claudeモデルの追加時にGatewayの個別モデル設定を必要としない。

#### 補足

Subscription認証をGateway経由で利用できることと、Anthropicの利用規約に適合していることは別の問題として扱う。

公式に許容される認証転送方式を確認できない場合は、本Stepの実装・採用を保留し、Anthropicへ直接接続する代替構成を検討する。

### Step7: Logging・運用性の改善

#### やること

- `logger.ts`を実装する。
- 構造化JSONログを標準出力へ出力する。
- リクエストIDを生成する。
- リクエスト開始・終了時のログを出力する。
- 以下の情報をログに含める。
  - Timestamp
  - Request ID
  - HTTP Method
  - Path
  - Model Alias
  - Upstream Model
  - Provider
  - HTTP Status
  - Duration
  - Streamingの有無
  - Timeoutの有無
  - Cancellationの有無
  - Error Message
- ログレベルを設定できるようにする。
- 認証情報および機密情報をログに出力しない。
- Gatewayの起動時に設定済みモデル一覧を出力する。

#### 完了条件

- [ ] リクエスト単位で処理状況を追跡できる。
- [ ] モデルエイリアスと実モデルの対応を確認できる。
- [ ] リクエストの処理時間を確認できる。
- [ ] TimeoutおよびCancellationを識別できる。
- [ ] 上流Providerの接続エラーを識別できる。
- [ ] 認証情報がログに出力されない。
- [ ] リクエスト本文およびレスポンス本文がデフォルトで出力されない。
- [ ] ログの永続化にDatabaseを必要としない。

### Step8: Claude CodeのSubagentとの結合テスト

#### やること

- Claude CodeからGatewayに接続するための起動設定を作成する。
- 通常のClaude Code起動とGateway経由の起動を分離する。
- Local LLMを利用する検証用Subagentを作成する。
- Main AgentとSubagentで異なるモデルを利用する。
- Subagentのfrontmatterでモデルエイリアスを指定する。
- `model: inherit`の挙動を確認する。
- SubagentからのTool Callingを確認する。
- Subagentの実行中断を確認する。
- 複数のSubagentを並列実行した場合の挙動を確認する。
- Local LLMへの委譲によるSubscription利用枠の消費削減効果を確認する。

#### 完了条件

- [ ] Main AgentでAnthropicのモデルを利用できる。
- [ ] Main Agentで既存Subscription認証が維持されている。
- [ ] 特定のSubagentだけLocal LLMを利用できる。
- [ ] `model: inherit`のSubagentがMain Agentのモデルを利用する。
- [ ] Subagentごとに異なるモデルエイリアスを指定できる。
- [ ] Subagentによるファイル探索・情報収集が正常に動作する。
- [ ] Subagentの調査結果がMain Agentに返却される。
- [ ] Subagentの実行を中断できる。
- [ ] 複数のSubagentを並列実行した場合の挙動を把握している。
- [ ] Local LLMの利用によってSubscription利用枠の消費を削減できることを確認している。

### Step9: OpenAI互換APIへの対応

#### やること

- `/v1/models`を実装する。
- 設定済みのモデルエイリアス一覧を返却する。
- `/v1/chat/completions`を実装する。
- OllamaのOpenAI互換APIへのルーティングを実装する。
- StreamingおよびTool Callingに対応する。
- API形式ごとの対応Providerを明確にする。
- API仕様に基づいたIntegration Testを実装する。

#### 完了条件

- [ ] `/v1/models`から設定済みモデル一覧を取得できる。
- [ ] OpenAI互換API経由でLocal LLMを利用できる。
- [ ] OpenAI互換APIでStreamingが正常に動作する。
- [ ] OpenAI互換APIでTool Callingが正常に動作する。
- [ ] 特定のクライアントに依存しない。
- [ ] 複数のHarnessでモデル設定を共通化できる。
- [ ] 対応する、対応しないAPI形式とProviderの組み合わせが明確になっている。
- [ ] Integration Testがすべて成功する。

#### 補足

OpenAI互換APIからAnthropicのSubscription認証を利用することは、本Stepの必須要件としない。

API形式の変換や認証方式の変更が必要になる場合は、別途対応を検討する。

### Step10: Docker化・LiteLLMからの移行

#### やること

- Dockerfileを作成する。
- Docker Composeの設定を作成する。
- Dockerコンテナ内からmacOS上のOllamaへの接続を確認する。
- 設定ファイルをVolume Mountで管理する。
- 設定ファイルを読み取り専用でマウントする。
- Healthcheckを設定する。
- Dockerコンテナの再起動ポリシーを設定する。
- ホスト側へのポート公開をlocalhostに限定する。
- README.mdにセットアップ手順を記載する。
- 起動・停止・設定変更・ログ確認の手順を記載する。
- 既存のLiteLLMとGatewayを並行稼働させて動作を比較する。
- 問題がないことを確認してからLiteLLMを停止する。

#### 完了条件

- [ ] Docker ComposeでGatewayを起動できる。
- [ ] Docker ComposeでGatewayを停止できる。
- [ ] Dockerコンテナ内からmacOS上のOllamaに接続できる。
- [ ] Docker環境でもAnthropicへの接続が正常に動作する。
- [ ] Docker環境でもStreaming、Tool Calling、Cancellationが正常に動作する。
- [ ] Gatewayのポートがlocalhostに限定されている。
- [ ] 設定変更をコンテナの再起動によって反映できる。
- [ ] PostgreSQLやPrismaを必要とせずに動作する。
- [ ] README.mdの手順のみでセットアップできる。
- [ ] LiteLLMから移行できることを確認している。
- [ ] 問題発生時にLiteLLMへ戻せる。

### 要件書作成時点の未確定事項

- OllamaのAnthropic互換APIにおける推論パラメータの対応範囲
- Anthropic互換APIとOllamaネイティブAPI間の変換処理の要否
- Local LLM向けのcount_tokens APIの実現方法
- Subscription認証の転送方式および利用規約への適合性
- Local LLMのモデルエイリアス単位の最適な推論パラメータ
- Cancellation時のOllama側の推論停止挙動
- 各HarnessとのAPI互換性

上記は各Stepの技術検証を通じて確定する。

本要件書では、Bun + Honoによる軽量なGatewayの実装、Zodによる設定管理、Provider Adapterによる責務分離を採用方針としている。

関連資料は実装時に最新の公式仕様を確認すること。本要件書の作成時点では、リンクの有効性および各ライブラリの最新仕様の再確認は実施していない。
