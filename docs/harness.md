# ハーネス設定

## 目的

このprojectでAIエージェントが使うハーネスの構成を説明する。

## 構成

| パス                              | 説明                                                |
| --------------------------------- | --------------------------------------------------- |
| `AGENTS.md`                       | project共通のルールと、`docs/`の参照先を示す        |
| `CLAUDE.md`                       | `@AGENTS.md`で`AGENTS.md`を読み込む                 |
| `.agents/library/skills/`         | skillsの実体。全AIエージェントで共通にする          |
| `.agents/library/agents/`         | subagentsの実体。全AIエージェントで共通にする       |
| `.claude/`、`.cursor/`、`.codex/` | `skills/`と`agents/`に、実体へのsymbolic linkを置く |

## skillsとsubagentsの追加

実体を`.agents/library/`に作り、各AIエージェントのディレクトリからsymbolic linkで参照する。
各AIエージェントのディレクトリに直接ファイルを置かない。

```bash
for dir in .claude .cursor .codex; do
  ln -s ../../.agents/library/skills/<name> "$dir/skills/<name>"
done
```
