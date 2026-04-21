# claude-code-knowledge-base

Claude Code のアップデートを、**自分の開発文脈でフィルタして** 届ける skill 集。

毎日のように出るリリースを全部追うのは現実的じゃない。でも「自分の今の作業に本当に関係するアップデートだけ」をサッと見たい。このリポジトリはその問いに答える最小の道具から始める。

## Skills

### `/whats-new`

前回この skill を叩いたとき以降の Claude Code リリースノートを取得し、さらに **Claude Code ヘビーユーザーのノウハウ** (`data/know-how.json` としてキュレーション) も合わせて、あなたの `CLAUDE.md`・直近 20 コミット・`~/.claude/skills/` の中身と突き合わせて、**今の作業に関係する新機能・skill・ワークフロー改善・ノウハウだけ** を Markdown で返す。

**核は diff × context × know-how。** 汎用 CHANGELOG はノイズ、あなたの実作業でフィルタすれば signal。さらに「公式 release notes では絶対に書かれない Why / How to use」の層をヘビーユーザーのノウハウで埋める。

#### How it works

1. `~/.claude/state/whats-new.json` から最終実行時刻を読む（初回は 7 日前）
2. `https://github.com/anthropics/claude-code/releases.atom` を取得
3. 最終実行時刻より新しい entry だけ残す
4. `./CLAUDE.md` + `git log --oneline -20` + `ls ~/.claude/skills/` を読み込み、さらに `data/know-how.json` を fetch
5. LLM が「今の作業に関係するか」で判定し、(a) 今すぐ使える新機能 / (b) 試すべき skill / (c) ワークフロー改善 / (d) ヘビーユーザーのノウハウ の最大 4 カテゴリで出力
6. 状態ファイルを現在時刻に更新

関連する更新が無ければ `You're up to date — last release was v2.1.x.` と返る。(d) は release 差分と独立なので、(a)(b)(c) が 0 件でも単独で出ることがある。

#### Requirements

- Claude Code CLI
- `curl`
- （あれば良いが必須ではない）プロジェクトの `CLAUDE.md`、git 履歴、`~/.claude/skills/` の中身

#### Install

```bash
git clone https://github.com/yasu551/claude-code-knowledge-base.git
cp -r claude-code-knowledge-base/skills/whats-new ~/.claude/skills/
```

#### Usage

Claude Code セッション内で:

```
/whats-new
```

初回は「直近 7 日間」を対象に。以降は前回実行時刻からの差分のみ。

#### Example output（illustrative — 実際の出力はあなたのプロジェクトに応じて変わります）

```markdown
### (a) 今すぐ使える新機能
- **`claude --resume` の session ID 省略** (v2.1.x)
  `CLAUDE.md:18` で session 切り替えの頻度が高いので、ID コピペが不要になる。
  試す: `claude --resume`

### (c) ワークフロー改善
- **MCP server の lazy loading** (v2.1.x)
  `~/.claude/skills/` に 40+ 個入っているため、起動時間が目に見えて縮む。設定不要、自動。

### (d) ヘビーユーザーのノウハウ
- **SKILL.md の description が skill 選定精度を決める** (Anthropic engineering, 2025-10-16)
  本プロジェクトの `CLAUDE.md:skill-file-conventions` でも強調している規約の一次ソース。
  なぜ英語で書くべきかの根拠がここにある。
  出典: https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
```

該当する更新が無ければ「このバッチでは日々の作業は変わらない」と素っ気なく返る。

## Development

### Adding a new skill

1. `skills/<skill-name>/SKILL.md` を作成
2. frontmatter に `name` と `description` を書く。description は「いつ使うか」を具体的に。Claude の skill 選択精度はこれで決まる
3. 本体は Claude が step-by-step で実行できる指示を書く（bash コマンド + Read/Write tool 呼び出しの指示 + LLM としての判断基準）
4. 実プロジェクトで `/skill-name` を叩いて dry-run、出力が実用的になるまでプロンプトを反復
5. `cp -r skills/<skill-name> ~/.claude/skills/` でローカルにインストールして自己利用

### Conventions

- 状態ファイルは `~/.claude/state/<skill-name>.json`
- 外部へのネットワーク fetch は `curl -fsSL`、失敗時は状態を更新しない（次回再試行できるように）
- 出力は前置き・ねぎらいなし。結論から
- 関係ないカテゴリは省略（空セクションを出さない）

## Data source & Removal policy

`data/know-how.json` は、Claude Code コミュニティで公開されているノウハウの **要約 + 出典 URL + topic tag のみ** を格納する。原文の本文は複製しない。各エントリは以下を含む:

- `id` — 安定識別子
- `author` — 著者のハンドル (例: `anthropic-engineering`, `bcherny`)
- `published_at` — 公開日 (ISO 8601 date)
- `source_url` — 原文への直リンク
- `summary_ja` — 1-2 文の日本語要約
- `topics` — 英語の topic tag 配列

**Removal リクエスト**: 自分が書いたノウハウが含まれており、削除を希望する場合は [GitHub Issue](https://github.com/yasu551/claude-code-knowledge-base/issues/new) で知らせてください。24 時間を目標に該当エントリを削除します (author ハンドル + source_url を明記してもらえると早いです)。

**Opt-in は現在 Phase 3 で検討中**。MVP 段階では公開情報の範囲で要約 + 出典掲載にとどめ、各著者への事前連絡は行っていません (公開 blog / 公開 SNS で公になっている内容のみ対象)。

## Roadmap

### Phase 1 (done): `/whats-new` MVP
公式リリースノート × 自分の文脈。差分 × 文脈 × LLM の仮説を最小コストで検証する。

### Phase 2 Sprint 0 (now): 手動キュレーション ノウハウ × CLI
`data/know-how.json` を手動でメンテし、`/whats-new` の (d) セクションで配信。この段階で「release notes では手に入らない Why / How to use」の仮説を最小コストで検証する。

### Phase 2 Sprint 1 (next): 自動 ingest
Anthropic engineering blog RSS (Tier-1) + 4 人の YouTube / 個人ブログ RSS (Tier-2) + X 手動キュレーション (Tier-3) を GitHub Actions cron で収集 → LLM 要約 → 本リポに commit。

### Phase 3: ブラウザ検索 UI
Astro + Pagefind で GitHub Pages に静的サイト。「あのときあの人が言ってた、あの Tips」を素早く引ける。release と know-how を時間軸で並べる。

Phase 2 Sprint 1 / Phase 3 は Sprint 0 の実利用で価値が確認できてから着手する。仮説検証が先。

## Contributing

Issues / PRs welcome. 新しい skill を追加する場合は "Adding a new skill" の規約に従ってください。提案だけでも歓迎。

## License

MIT — see [LICENSE](./LICENSE).
