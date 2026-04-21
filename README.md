# claude-code-knowledge-base

Claude Code のアップデートを、**自分の開発文脈でフィルタして** 届ける skill 集。

毎日のように出るリリースを全部追うのは現実的じゃない。でも「自分の今の作業に本当に関係するアップデートだけ」をサッと見たい。このリポジトリはその問いに答える最小の道具から始める。

## Skills

### `/whats-new`

前回この skill を叩いたとき以降の Claude Code リリースノートだけを取得し、あなたの `CLAUDE.md`・直近 20 コミット・`~/.claude/skills/` の中身と突き合わせて、**今の作業に関係する新機能・skill・ワークフロー改善だけ** を Markdown で返す。

**核は diff × context。** 汎用 CHANGELOG はノイズ、あなたの実作業でフィルタすれば signal。

#### How it works

1. `~/.claude/state/whats-new.json` から最終実行時刻を読む（初回は 7 日前）
2. `https://github.com/anthropics/claude-code/releases.atom` を取得
3. 最終実行時刻より新しい entry だけ残す
4. `./CLAUDE.md` + `git log --oneline -20` + `ls ~/.claude/skills/` を読み込む
5. LLM が「今の作業に関係するか」で判定し、(a) 今すぐ使える新機能 / (b) 試すべき skill / (c) ワークフロー改善 の 3 カテゴリで出力
6. 状態ファイルを現在時刻に更新

関連する更新が無ければ `You're up to date — last release was v2.1.x.` とだけ返って終わる。

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

## Roadmap

### Phase 1 (now): `/whats-new` MVP
公式リリースノート × 自分の文脈。差分 × 文脈 × LLM の仮説を最小コストで検証する。

### Phase 2: ヘビーユーザーのコンテンツ取り込み
Boris Cherny / Vishwas Gopinath / Ashley Ha / Joey Anuff らの X / YouTube / ブログを GitHub Actions cron で収集 → 要約 → 本リポに commit。`/whats-new` が公式 + 彼らのコンテンツを横断参照する。

### Phase 3: ブラウザ検索 UI
Astro + Pagefind で GitHub Pages に静的サイト。「あのときあの人が言ってた、あの Tips」を素早く引ける。

Phase 2/3 は Phase 1 が 1〜2 週間の実利用で価値を出してから着手する。仮説検証が先。

## Contributing

Issues / PRs welcome. 新しい skill を追加する場合は "Adding a new skill" の規約に従ってください。提案だけでも歓迎。

## License

MIT — see [LICENSE](./LICENSE).
