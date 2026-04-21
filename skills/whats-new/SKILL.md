---
name: whats-new
description: Use when the user wants to know what's changed in Claude Code since they last checked, or what hands-on know-how from Claude Code power users applies to their current work. Reads the last-run timestamp from local state, fetches Claude Code release notes and a curated community know-how feed, and cross-references the user's CLAUDE.md, recent git commits, and installed skills to surface only updates and tips that matter for what they're actively building.
---

# /whats-new

Surface Claude Code updates since the user's last check, filtered to what matters for their current project.

The value comes from **diff × context**. A generic changelog is noise. Filtered by what the user is actually doing, it becomes signal.

## Step 1: Load last-run state

Read `~/.claude/state/whats-new.json`.

Use the Read tool. If the file doesn't exist, compute `last_run` as 7 days ago:

```bash
date -u -v-7d +%Y-%m-%dT%H:%M:%SZ 2>/dev/null || date -u -d '7 days ago' +%Y-%m-%dT%H:%M:%SZ
```

(The first form is macOS/BSD, the second is GNU/Linux. One of them will succeed.)

Keep `last_run` in memory for the next steps.

## Step 2: Fetch the release feed

```bash
curl -fsSL https://github.com/anthropics/claude-code/releases.atom
```

If curl fails (network error, non-200 status), tell the user what failed and stop. **Do NOT update state** — next run should retry from the same point.

## Step 3: Parse and filter entries

The atom feed contains multiple `<entry>` blocks. Each entry has:
- `<title>` — version string (e.g. `v1.2.3`)
- `<updated>` — ISO 8601 timestamp
- `<content type="html">` — release notes body (HTML, may contain markdown-like content)

Select only entries where `<updated>` is newer than `last_run`. Keep them in chronological order (oldest first is usually easier to summarize).

Capture the first entry's version as `latest_version` for Step 6.

**If zero entries match**, tell the user "You're up to date — last release was `<latest_version>`, checked at `<last_run>`." Skip to Step 6 to refresh the timestamp.

## Step 4: Gather project context

Parallelize these reads where possible.

- **CLAUDE.md**: Read `./CLAUDE.md` if it exists. If not, note its absence but continue.
- **Recent commits**: `git log --oneline -20` (Bash). If not a git repo, skip.
- **Installed skills**: `ls -1 ~/.claude/skills/ 2>/dev/null | head -30`. If no skills dir, skip.
- **Heavy-user know-how**: `curl -fsSL https://raw.githubusercontent.com/yasu551/claude-code-knowledge-base/main/data/know-how.json`

  JSON は `[{id, author, published_at, source_url, summary_ja, topics}, ...]` の配列。
  fetch に失敗した場合 (network error、非-200、parse error、空配列) は know-how を空として Step 5 の (d) をスキップする。**この失敗では状態ファイルを汚さず Step 5 に進む**。

## Step 5: Synthesize suggestions

Output Markdown with up to three sections. **Omit any section that has no relevant items** — empty sections are filler.

### (a) 今すぐ使える新機能
New features from the diff that apply directly to this project. For each:
- One-line summary
- **Why it matters here** — reference something specific from CLAUDE.md or a recent commit
- Exact command, flag, or pattern to try

### (b) 試すべき skill
New skills or SDK patterns that would pair well with the user's installed skills or current workflow. Same format as (a).

### (c) ワークフロー改善
Bug fixes, new flags, config options, or performance changes that would affect how the user operates daily.

### (d) ヘビーユーザーのノウハウ
Step 4 で取得した `know-how.json` のエントリから、**今のプロジェクト文脈 (CLAUDE.md + 直近 commit + 導入済み skills) に具体的に関係する** know-how を最大 3 件選ぶ。各エントリ:

- 1-2 行の要約（`summary_ja` をそのまま使うか、文脈に合わせて 1 行で言い換える）
- **なぜ今のあなたに関係するか** — CLAUDE.md の具体的な記述、最近の commit、導入済み skill の名前を 1 つ引用
- 出典: `[author · published_at · URL]`

**(d) は skip 条件が多い**:

- `know-how.json` の fetch が失敗した (Step 4 参照) → `(d)` 全体を skip。代わりに 1 行 `_(know-how fetch failed — release notes only this run)_` だけ書く
- `know-how.json` が `[]` または相当 → `(d)` を skip。note 不要
- LLM 判定で関連する know-how が 0 件 → `(d)` を skip。「関係ない know-how を水増しで出さない」

**(d) は release diff の有無と独立**。(a)(b)(c) が 0 件 (You're up to date) でも、(d) は単独で出してよい — ユーザーが「今日の作業の文脈で読むべき過去のノウハウ」を拾えるのが価値。

**Rules for synthesis:**
- Only include updates tied to the user's actual work. If an update is orthogonal (a language binding they don't use, a feature for a workflow they don't have), skip it.
- Be concrete. Reference the exact file, function, or commit that this update helps with. Don't say "this helps your workflow" — say "this is faster than the `jq` you're using in `skills/foo/SKILL.md:42`."
- No preamble. No filler. Lead with the recommendation, cite the release, end.
- If the diff contains only maintenance/internal changes with no user-visible impact, say so plainly: "Nothing in this batch changes your day-to-day."

## Step 6: Update state

Write `~/.claude/state/whats-new.json`:

```json
{
  "last_run": "<current UTC ISO 8601 timestamp>",
  "last_version_seen": "<latest_version from Step 3>"
}
```

Create the parent dir first: `mkdir -p ~/.claude/state`.

Use the Write tool. Do NOT write this if Step 2 (fetch) failed.

## Edge cases
- **No CLAUDE.md**: note "(no CLAUDE.md — suggestions inferred from commits and installed skills)".
- **Empty git history / not a repo**: skip that context; rely on CLAUDE.md + installed skills.
- **Feed returns but has no entries**: treat as "You're up to date."
- **Malformed atom feed**: report the parse failure, don't update state.
- **First run (no state file)**: announce "First run — looking at the last 7 days." and proceed.
- **know-how.json fetch fail** (network / 404 / 非-200): `(d)` を skip、`_(know-how fetch failed — release notes only this run)_` と 1 行。**状態ファイルは更新してよい** (know-how は補助、release 側の差分判定には影響しない)。
- **know-how.json が malformed JSON**: parse 失敗を note、`(d)` を skip、他セクションは通常通り続行。
- **know-how.json が `[]`**: `(d)` を skip、note 不要。
- **LLM が関連 know-how 0 件と判断**: `(d)` を skip、note 不要。無理に埋めない。

## Why this design
- **Diff, not digest.** The user gets only what changed since they last cared, not a perpetual firehose.
- **Context, not category.** Filtering by their real project beats any taxonomy.
- **Local state, no server.** Works offline after fetch, no accounts, no telemetry.
- **One-file skill.** Easy to read, easy to fork, easy to improve.
