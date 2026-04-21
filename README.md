# claude-code-knowledge-base

Claude Code のアップデートを**自分の開発文脈でフィルタして**届ける skill 集。

毎日のようにリリースが出る Claude Code。公式 CHANGELOG は全部は読めない。でも「自分の今の作業に本当に関係するアップデートだけ」をサッと見たい。その問いに答える最小の道具。

## Skills

### `/whats-new`

前回叩いたとき以降の Claude Code リリースノートを取得し、あなたの `CLAUDE.md`・直近 20 コミット・インストール済み skill と突き合わせて、**今の作業に関係ある新機能・skill・ワークフロー改善だけ** を Markdown で返す。

- データソース: `https://github.com/anthropics/claude-code/releases.atom`
- 状態: `~/.claude/state/whats-new.json` にローカル保存（最終実行時刻と最新バージョン）
- 出力: ターミナルへの Markdown。関係あるカテゴリ (a) 今すぐ使える新機能 / (b) 試すべき skill / (c) ワークフロー改善 のみ表示、なければ省略

#### Install

```bash
git clone https://github.com/yasu551/claude-code-knowledge-base.git
cp -r claude-code-knowledge-base/skills/whats-new ~/.claude/skills/
```

#### Usage

Claude Code で:

```
/whats-new
```

初回は「直近 7 日間」を対象に。以降は前回実行時刻からの差分のみ。

#### Example output (想定)

```markdown
### (a) 今すぐ使える新機能
- **`--resume` の session ID 省略対応** (v1.3.0)
  あなたは `CLAUDE.md:18` で頻繁に session 切り替えをしているので、ID コピペが不要になる。
  試す: `claude --resume`

### (c) ワークフロー改善
- **MCP server の lazy loading** (v1.2.8)
  インストール済み skills が 40+ 個ある（`ls ~/.claude/skills/` 参照）ため、起動時間が短縮される。設定不要、自動。
```

関連する更新が無ければ "You're up to date — last release was v1.3.0." とだけ返る。

## Roadmap

### Phase 1 (現在): `/whats-new` MVP
公式リリースノート × 自分の文脈。差分 × 文脈 × LLM の仮説を検証する最小実装。

### Phase 2: 4 人のヘビーユーザーのコンテンツ取り込み
Boris Cherny / Vishwas Gopinath / Ashley Ha / Joey Anuff の X / YouTube / ブログを GitHub Actions cron で収集 → 要約 → 本リポに commit。`/whats-new` が公式＋この 4 人のコンテンツを横断で参照する。

### Phase 3: ブラウザ検索 UI
Astro + Pagefind で GitHub Pages に静的サイト。「あのときあの人が言ってたアレ」を素早く引ける。

## Design doc

`/office-hours` (gstack) で生成した設計書は `~/.gstack/projects/yasu551-claude-code-knowledge-base/` にあります。採用アプローチ、前提、代案、Open questions まで。

## License

MIT
