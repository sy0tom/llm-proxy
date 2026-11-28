# ハーネス設定

## 目的

このprojectでAIエージェントが使うハーネスの構成と、skills、subagentsの一覧を説明する。

## 構成

skillsとsubagentsの正本は`.agents/library/`に置き、各AIエージェントが読む場所からsymbolic linkで参照する。

| 種類       | 正本                               | Claude Code                       | Cursor                                   | Codex                         |
| ---------- | ---------------------------------- | --------------------------------- | ---------------------------------------- | ----------------------------- |
| 共通の指示 | `AGENTS.md`                        | `CLAUDE.md`から`@AGENTS.md`で読む | `AGENTS.md`を読む                        | `AGENTS.md`を読む             |
| skills     | `.agents/library/skills/<name>/`   | `.claude/skills/<name>`(link)     | `.claude/skills`と`.agents/skills`を読む | `.agents/skills/<name>`(link) |
| subagents  | `.agents/library/agents/<name>.md` | `.claude/agents/<name>.md`(link)  | `.claude/agents`を読む                   | `.codex/agents/<name>.toml`   |

- Cursorは`.claude/skills`と`.agents/skills`の両方を読むため、同じskillが2つ見える。Cursorの仕様で避けられないため、許容する。
- CodexのsubagentsはTOMLで書く必要があり、正本のMarkdownをlinkできない。
  `.codex/agents/<name>.toml`の`developer_instructions`で、正本を読んで従うよう指示する。
  Codexの`name`はsnake_caseにする(例: `sdd_spec_reviewer`)。
- Codexは、信頼済みのプロジェクトでだけ`.codex/`を読む。
  subagentsを使うには、リポジトリとworktreeのパスを、
  `~/.codex/config.toml`の`projects`で`trust_level = "trusted"`にする。
- `.cursor/skills`、`.cursor/agents`、`.codex/skills`は使わない。

## skills

| 名前            | 使うとき                                   | SDDの段階    |
| --------------- | ------------------------------------------ | ------------ |
| `sdd-spec`      | Stepや機能No.から機能仕様を作る、直す      | 機能仕様     |
| `sdd-plan`      | Stepから、親Issueとタスクのsub-issueを作る | タスク       |
| `sdd-implement` | タスクのIssueを、テストと実装で進める      | 実装、テスト |

## subagents

| 名前                | 使うとき                                                                   |
| ------------------- | -------------------------------------------------------------------------- |
| `sdd-spec-reviewer` | 実装が受け入れ条件を満たし、対応するテストがあるかを読み取り専用で確認する |

## skillsとsubagentsの追加

実体を`.agents/library/`に作り、各AIエージェントが読む場所からsymbolic linkで参照する。
各AIエージェントのディレクトリに直接ファイルを置かない(Codexのsubagentsを除く)。

- skills: `.agents/library/skills/<name>/SKILL.md`を作り、frontmatterに`name`(ディレクトリ名と同じ)と`description`を書く。

  ```bash
  ln -s ../../.agents/library/skills/<name> .claude/skills/<name>
  ln -s ../library/skills/<name> .agents/skills/<name>
  ```

- subagents: `.agents/library/agents/<name>.md`を作り、frontmatterに`name`と`description`を書く。
  Codex向けに`.codex/agents/<name>.toml`を作り、`name`、`description`、`developer_instructions`を書く。

  ```bash
  ln -s ../../.agents/library/agents/<name>.md .claude/agents/<name>.md
  ```

- skillsの中で`docs/`を参照するときは、相対リンクではなくリポジトリのルートからのパスで書く。
  symbolic link経由で読むと、相対リンクの基準がずれるため。
- 追加したら、リンク切れがないことを確かめ、上の一覧に足す。

  ```bash
  find .claude .agents -type l ! -exec test -e {} \; -print
  ```
