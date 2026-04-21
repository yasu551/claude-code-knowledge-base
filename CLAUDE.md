# CLAUDE.md

このリポジトリで Claude Code が作業するためのガイド。

## Project summary

`claude-code-knowledge-base` は **Claude Code ヘビーユーザー向けの skill 集** を OSS で提供するリポジトリ。現在の柱は `/whats-new`（リリースノート差分 × ユーザー文脈で提案を返す skill）。将来的に 4 人のヘビーユーザーのコンテンツ取り込みとブラウザ検索 UI を追加する（詳細は README "Roadmap"）。

コード基盤ではなく **skill 定義ファイル (Markdown)** が中心の成果物。ビルド・テストフレームワークはない。

## Repository layout

```
skills/
  <skill-name>/
    SKILL.md        # frontmatter + Claude への実行指示
README.md           # ユーザー向け install / usage / roadmap
LICENSE             # MIT
CLAUDE.md           # 本ファイル（Claude への作業ガイド）
```

## Skill file conventions

- **frontmatter 必須**: `name` と `description` のみ
- `description` は「**いつ使うか**」を具体的に書く。Claude の skill 自動選択の精度はここで決まる。曖昧な description は自動発火しない。
- 本体は「Claude が step-by-step で実行できる指示書」。bash コマンドは \`\`\`bash フェンスで囲み、Read/Write tool 呼び出しが必要な箇所は自然言語で明示する
- Edge case（ファイル欠損、ネットワーク失敗、git 履歴なし等）は必ず本文内で扱いを明記する
- 関係ないカテゴリ・空セクションは出力しない。前置き・ねぎらいも書かせない

## State file conventions

- 状態ファイルは `~/.claude/state/<skill-name>.json` に保存
- skill 側で `mkdir -p ~/.claude/state` を責任持って実行
- 外部 fetch が失敗した場合、状態を **更新しない**（次回再試行させるため）
- 初回（state 無し）は 7 日前を既定値にするなど、妥当なデフォルトを持つ

## Development workflow

新しい skill を足すとき:

1. `skills/<name>/SKILL.md` を書く
2. `cp -r skills/<name> ~/.claude/skills/` でローカルにインストール
3. 新しい Claude Code セッションを開いて `/<name>` を dry-run
4. 出力が実用に耐えるまで、SKILL.md のプロンプト（判定ルールや出力フォーマット）を反復調整
5. 既存 skill 名と衝突しないか `ls ~/.claude/skills/` で確認

## Language

- ユーザー向け表示（skill の出力、README 本文、example）は **日本語** が基本
- skill frontmatter の `description` は **英語** にする（Claude の skill マッチングが英語ベースで最適化されているため）
- コード・識別子・ファイル名は英語

## Commits

- 1 コミット 1 論理変更
- メッセージは「何を・なぜ」を 1 行 + 本文 2-3 行。what ではなく why を優先
- `git add -A` ではなくファイルを明示的に指定（秘密を誤って混入させない）
- pre-commit hook (secretlint) が走る。失敗時は `--no-verify` で回避せず原因を直す

## Out of scope for MVP

以下は Phase 2 以降に持ち越し。触らない:

- 4 人 (Boris Cherny / Vishwas Gopinath / Ashley Ha / Joey Anuff) のコンテンツ収集
- GitHub Actions cron
- Astro / Pagefind のブラウザ UI
- 埋め込みベクトル検索

今は `/whats-new` を自分の日常で 1〜2 週間使い、仮説（差分 × 文脈 × LLM で実用的な提案が出る）を検証することが最優先。

## Do / Don't

- **Do**: skill が LLM の判断に任せる部分と、bash で確定的にやる部分を明確に分ける
- **Do**: edge case を本文で明記する（空フィード、git 無しリポ、CLAUDE.md 無し）
- **Don't**: skill 内で過度なエラーリカバリを書かない。失敗したら状態を汚さず停止
- **Don't**: 新機能提案を詰め込みすぎない。核が動いてから拡張
