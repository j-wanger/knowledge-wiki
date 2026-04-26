---
name: wiki-stale
description: "Use when marking stale articles in a wiki. Detects articles past their staleness threshold and updates their status. Use --dry-run for reporting only. Do NOT use for structural audits (use wiki-lint) or for fixing stale articles (use wiki-reorg or manual review after marking)."
---

# wiki-stale

Detect and mark stale wiki articles. Single-agent skill — no subagent pipeline.

Modes: `/wiki-stale` (mark), `/wiki-stale --dry-run` (report only), `/wiki-stale --wiki <name>` (target specific wiki).

---

## Step 0: Resolve Wiki

Read and follow `~/.claude/skills/knowledge-wiki/step-0-template.md`. Write operation (update `last_used` in Sub-step 0.6). For `--dry-run`: read-only (SKIP Sub-step 0.6).

---

## Step 1: Read Staleness Rules

Read `<wiki_path>/schema.md` and extract `staleness_rules` (see `lifecycle-spec.md` Staleness Thresholds section for format). If absent, use default: `default_days: 180`, no overrides. Emit: "No staleness_rules configured. Using default 180-day threshold."

---

## Step 2: Scan Eligible Articles

Scan `.md` files under `<wiki_path>/articles/` recursively. An article is eligible if `status` is `verified` or `reviewed`. Skip `draft`, `stale`, `archived`. Skip `<wiki_path>/episodic/`. Collect: file path, title, status, updated, tags. If an article has a missing or unparseable `updated` field, skip it with warning: "Cannot parse updated date for <title>, skipping."

---

## Step 3: Compute Staleness

Per `lifecycle-spec.md` algorithm:

1. Match article tags against `staleness_rules.overrides`.
2. One match: use that override's `days`. Multiple matches: **shortest wins**.
3. No match: use `default_days`.
4. Stale if `(today - updated) > applicable_threshold`.

---

## Step 4: Report (--dry-run)

Present findings without modifying files:

```
Wiki Staleness Report -- YYYY-MM-DD (<wiki_name>)
STALE: <title> (updated: YYYY-MM-DD, threshold: Nd, elapsed: Nd)
OK: <title> (updated: YYYY-MM-DD, threshold: Nd, remaining: Nd)
Summary: N stale, M ok
```

After reporting, STOP.

---

## Step 5: Mark Stale Articles

For each stale article, use the **read-modify-write** pattern:

1. Read full file with Read tool.
2. Use Edit tool to replace `status: verified` or `status: reviewed` with `status: stale`.
3. Do NOT modify other frontmatter fields or article body.

Emit per article: "Marked stale: <title> (<path>)"

---

## Step 6: Log and Complete

Append to `<wiki_path>/log.md`:

```
[YYYY-MM-DDTHH:MM:SS] STALE -- N articles marked stale, M within threshold
```

Summarize: "wiki-stale complete: N marked stale, M ok." If stale articles found, suggest re-verification or archival.
