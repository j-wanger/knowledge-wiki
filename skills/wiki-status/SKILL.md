---
name: wiki-status
description: "Use when showing a read-only dashboard summarizing wiki articles, cross-links, tags, hierarchy, and recent activity. Do NOT use for diagnosing problems (use wiki-lint) or fixing structure (use wiki-reorg)."
---

# wiki-status

Display a comprehensive read-only dashboard summarizing the state of the project wiki. This command scans articles, links, tags, hierarchy, and schema health to give a quick overview without modifying any files.

---

## Conventions

### Article Format

Every article under `<wiki_path>/articles/` is a Markdown file with YAML frontmatter containing at minimum:

```yaml
---
title: "Article Title"
category: concepts | patterns | decisions | action-plans
tags: [tag-a, tag-b]
parents: [parent-filename]   # empty array [] for root articles
created: YYYY-MM-DD
updated: YYYY-MM-DD
---
```

### Category Directories

Articles live in category subdirectories under `<wiki_path>/articles/`:

- `<wiki_path>/articles/concepts/` -- foundational domain ideas
- `<wiki_path>/articles/patterns/` -- recurring approaches or practices
- `<wiki_path>/articles/decisions/` -- architectural or strategic choices
- `<wiki_path>/articles/action-plans/` -- concrete next steps or implementation plans

### Linking Rules

Cross-links between articles use `[[filename]]` wiki-link syntax (without the `.md` extension). Display-text variants use `[[filename|Display Text]]`. Both forms count as a single cross-link.

### Schema

`<wiki_path>/schema.md` contains YAML frontmatter with a `domain` field naming the wiki's subject area, plus a `## Custom Tags` section listing sanctioned tags for the wiki.

### Log

`<wiki_path>/log.md` records all wiki operations. Each entry follows the format: `[timestamp] OPERATION -- summary`.

---

## Step 0: Resolve Wiki

Read and follow the canonical template at `~/.claude/skills/knowledge-wiki/step-0-template.md`. It defines all 8 sub-steps (read registry, resolve by --wiki flag / CWD / auto-register / inference, validate, touch).

This skill is **read-only** — in Sub-step 0.6, SKIP the touch step entirely. Do not update `last_used`. Sub-step 0.4 (auto-register an unregistered local wiki) still runs; that one-time setup write is acceptable even for read skills.

After Step 0 completes, `wiki_path` is the canonical absolute path to the resolved wiki. All subsequent steps use `<wiki_path>/` wherever this skill previously used `wiki/`.

---

## Pre-check

After Step 0 has resolved `wiki_path`, verify the resolved wiki has content to display:

- If `<wiki_path>` does not exist on disk, tell the user: "Wiki path does not exist: <wiki_path>. The registry may be stale. Run `/wiki-list` to see registered wikis." Then stop.
- If `<wiki_path>/articles/` is empty or contains only `.gitkeep` files, display a minimal dashboard with all counts at zero and suggest next steps:

```
Wiki Status -- (no domain configured)
======================================

Articles: 0 total
  concepts:     0
  patterns:     0
  decisions:    0
  action-plans: 0

Inbox: 0 entries pending (/wiki-absorb to process)

The wiki is empty. Next steps:
  1. /wiki-ingest <source> -- import existing documents
  2. /wiki-capture -- jot down knowledge from conversation
  3. /wiki-absorb -- process inbox into articles
```

Then stop.

---

## Dashboard Format

Read `<wiki_path>/schema.md` to extract the domain name from the frontmatter `domain` field. Then present the dashboard using this exact structure:

```
Wiki Status -- {{domain from schema.md}}
======================================

Articles: N total
  concepts:     X
  patterns:     X
  decisions:    X
  action-plans: X

Inbox: N entries pending (/wiki-absorb to process)

Links:
  Cross-links:    N
  Avg per article: N.N
  Orphaned:       N (run /wiki-lint)

Tags: N unique
  Top 5: tag-a (N), tag-b (N), tag-c (N), tag-d (N), tag-e (N)

Hierarchy:
  Hub articles: N (articles with 3+ children)
  Max depth:    N
  Roots:        N (list root names)

Tiers:
  public:       X
  private:      X

Lifecycle:
  draft:        X
  reviewed:     X
  verified:     X
  stale:        X
  archived:     X

Episodic: N entries (N unconsolidated)

Schema Health:
  Dead tags:    N
  Missing tags: N
  Drift score:  LOW | MEDIUM | HIGH (run /wiki-lint for details)

Recent Activity (from log.md):
  [date] OPERATION -- summary
  [date] OPERATION -- summary
  ...last 5 entries
```

Use double-line equals signs under the title for visual emphasis. Align the values vertically within each section for readability.

---

## Computed Metrics

Each metric in the dashboard is derived from the wiki's files. Here is exactly how to compute each one.

### 1. Article Count by Category

Count `.md` files (excluding `.gitkeep`) in each subdirectory under `<wiki_path>/articles/`:

- `<wiki_path>/articles/concepts/*.md` -> concepts count
- `<wiki_path>/articles/patterns/*.md` -> patterns count
- `<wiki_path>/articles/decisions/*.md` -> decisions count
- `<wiki_path>/articles/action-plans/*.md` -> action-plans count

The total is the sum of all four.

### 2. Inbox Count

Count `.md` files in `<wiki_path>/inbox/`, excluding `.gitkeep` files and any files inside a `.processed/` subdirectory. This tells the user how many raw entries are waiting to be absorbed.

### 3. Cross-link Count

Parse the body of every article under `<wiki_path>/articles/` and count all `[[link]]` occurrences (using the `[[...]]` wiki-link syntax). Each `[[link]]` counts as one cross-link, including `[[filename|Display Text]]` variants. Count the total across all articles.

### 4. Average Links per Article

Divide the total cross-link count by the total article count. Display one decimal place (e.g., `3.2`). If there are zero articles, display `0.0`.

### 5. Orphan Count

Build an inbound-link map: for each article, scan every other article's body for `[[links]]` that point to it. An article is orphaned if it receives zero inbound links from any other article. Count the total number of orphaned articles.

### 6. Tag Frequency and Top 5

Parse the `tags` array from every article's YAML frontmatter. Count how many articles use each tag. Report the total number of unique tags, and list the top 5 tags by frequency in descending order. If fewer than 5 unique tags exist, show however many there are. Format each as `tag-name (N)` where N is the number of articles using that tag.

### 7. Hub Count

An article is a hub if 3 or more other articles list it in their `parents` frontmatter field. Scan all articles' `parents` arrays and count how many times each article appears as a parent. Any article with 3 or more children qualifies as a hub.

### 8. Max Depth

Walk the parent-child hierarchy defined by `parents` frontmatter fields. Find the longest chain from a root article (one with `parents: []`) down through successive children. The depth of a root is 1. A child of a root has depth 2. Report the maximum depth found across all articles. If no hierarchy exists (no parent relationships), report 1.

### 9. Root Articles

Articles with `parents: []` (empty parents array) are roots. Count them and list their filenames in parentheses.

### 10. Dead Tags

Read the `## Custom Tags` section of `<wiki_path>/schema.md`. A tag is dead if it appears in the schema but is not used by any article's frontmatter `tags` field.

### 11. Missing Tags

Collect all tags used across article frontmatter. A tag is missing if it appears in 3 or more articles but is not listed in the `## Custom Tags` section of `<wiki_path>/schema.md`.

### 12. Drift Score

Combine the counts of dead tags, missing tags, and hierarchy root mismatches (roots in schema that do not exist on disk, or root articles on disk not listed in schema) into a single issue count:

- **LOW**: 0-1 total issues
- **MEDIUM**: 2-4 total issues
- **HIGH**: 5 or more total issues

### 13. Tier Breakdown

Parse the `tier` field from every article's YAML frontmatter. Count articles per tier value (`public`, `private`). Articles missing the `tier` field are counted separately as "unclassified." See `tier-spec.md` for canonical tier definitions.

### 14. Lifecycle Breakdown

Parse the `status` field from every article's YAML frontmatter. Count articles per lifecycle state (`draft`, `reviewed`, `verified`, `stale`, `archived`). Articles missing the `status` field are counted as "unknown." See `lifecycle-spec.md` for the canonical state machine.

### 15. Episodic Count

Count `.md` files in `<wiki_path>/episodic/`, excluding `.gitkeep`. To determine unconsolidated count: check each entry's frontmatter for `consolidated_at` field — entries lacking this field are unconsolidated. If `<wiki_path>/episodic/` does not exist, display `Episodic: 0 entries`. See `episodic-conventions.md` for the episodic entry format.

---

## Recent Activity

Read `<wiki_path>/log.md` and extract the last 5 entries. Each log entry follows the format `[timestamp] OPERATION -- summary`. Display them in reverse chronological order (most recent first). If `log.md` has fewer than 5 entries, show all of them. If `log.md` does not exist or is empty, display: "No activity recorded yet."

---

## Edge Cases

- **Empty wiki (no articles)**: Show the minimal dashboard from Pre-check and stop. All counts zero.
- **No cross-links at all**: Show `Cross-links: 0`, `Avg per article: 0.0`, `Orphaned: N` (all articles are orphans in this case). Suggest: "Consider adding `[[cross-links]]` between related articles, or run `/wiki-reorg`."
- **No tags at all**: Show `Tags: 0 unique` and omit the Top 5 line. Suggest: "Articles have no tags. Consider adding tags to frontmatter for better discoverability."
- **No hierarchy**: Show `Hub articles: 0`, `Max depth: 1`, `Roots: N` (all articles are roots). Suggest: "No parent-child relationships found. Consider using `parents` frontmatter to build hierarchy, or run `/wiki-reorg`."
- **schema.md missing or malformed**: If `<wiki_path>/schema.md` does not exist or cannot be parsed, use "Unknown Domain" as the domain name and skip Schema Health (note: "schema.md not found -- run `/wiki-init` to create it").
- **log.md missing**: Display "No activity recorded yet" in the Recent Activity section.

---

## Strictly Read-Only

This command does NOT modify any files. It reads and reports only. When issues are detected (orphans, drift, missing tags), suggest the appropriate command to address them:

- **Orphaned articles**: "Run `/wiki-lint` for details, then `/wiki-reorg` to fix."
- **Schema drift**: "Run `/wiki-lint` for a full audit."
- **Pending inbox items**: "Run `/wiki-absorb` to process."
- **No articles at all**: "Run `/wiki-ingest` or `/wiki-capture` to add content."

## Tier Sizing Rationale

This skill exceeds the diagnostic tier baseline (100 lines) at 228 lines. Justified exception per [[accept-skill-size-exceptions]]: 12 computed metrics with formatting logic, read-only dashboard, single invocation per session (~0 marginal token cost after first load). Companion-extraction rejected — adds indirection without meaningful savings.
