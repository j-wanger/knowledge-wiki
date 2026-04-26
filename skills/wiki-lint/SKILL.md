---
name: wiki-lint
description: "Use when auditing wiki structural health. Reports broken links, orphans, bloat, stubs, and tag drift. Read-only. Do NOT use for dev wiki validation (use dev-check) or for fixing wiki issues (use wiki-reorg after diagnosing with lint)."
---

# wiki-lint

Audit the structural health of the project wiki. Scans every article, checks cross-references, validates schema and index, and produces a prioritized report of issues. This command is strictly read-only -- it reports problems but never modifies files.

---

## Conventions Reference

These conventions are authoritative for all checks in this skill.

### Article Format

Every article under `<wiki_path>/articles/` is a Markdown file with YAML frontmatter. Required frontmatter fields:

- `title` -- article title (string)
- `aliases` -- alternative names (list of strings, may be empty)
- `category` -- exactly one of: `concepts`, `patterns`, `decisions`, `action-plans`
- `tags` -- list of tag strings (may be empty)
- `parents` -- list of parent article slugs (empty list `[]` for hierarchy roots)
- `created` -- creation date (YYYY-MM-DD)
- `updated` -- last update date (YYYY-MM-DD)
- `source` -- origin of the content (string)
- `tier` -- `public` or `private` (see `tier-spec.md`)
- `status` -- lifecycle state (see `lifecycle-spec.md`)

### Linking Rules

Articles reference each other using `[[slug]]` wiki-links in their body text. The slug corresponds to the filename without the `.md` extension. A link is valid if and only if a matching `.md` file exists under `<wiki_path>/articles/`.

### Article Size Constraints

| Metric | Lines |
|--------|-------|
| Ideal range | 40 -- 80 |
| Hard cap (bloated) | > 120 |
| Stub threshold | < 15 |

### Valid Categories

The `category` field must be exactly one of:

- `concepts`
- `patterns`
- `decisions`
- `action-plans`

---

## Step 0: Resolve Wiki

Read and follow the canonical template at `~/.claude/skills/knowledge-wiki/step-0-template.md`. It defines all 8 sub-steps (read registry, resolve by --wiki flag / CWD / auto-register / inference, validate, touch).

This skill is **read-only** — in Sub-step 0.6, SKIP the touch step entirely. Do not update `last_used`. Sub-step 0.4 (auto-register an unregistered local wiki) still runs; that one-time setup write is acceptable even for read skills.

After Step 0 completes, `wiki_path` is the canonical absolute path to the resolved wiki. All subsequent steps use `<wiki_path>/` wherever this skill previously used `wiki/`.

---

## Pre-check

After Step 0 has resolved `wiki_path`, verify that the resolved wiki is healthy enough to lint:

- If `<wiki_path>` does not exist on disk, tell the user: "Wiki path does not exist: <wiki_path>. The registry may be stale. Run `/wiki-list` to see registered wikis." Then stop.
- If `<wiki_path>/articles/` is empty or missing, tell the user: "Wiki at <wiki_path> has no articles yet. Run `/wiki-ingest` to add content, then `/wiki-absorb` to process it." Then stop.

---

## Checks to Perform

Run ALL of the following checks on every invocation. Do not skip any.

### 1. Broken Links

Scan the body of every article for `[[link]]` references. For each link, verify that a corresponding `.md` file exists somewhere under `<wiki_path>/articles/`. A link is broken if no matching file can be found.

Severity: **ERROR**

### 2. Orphaned Articles

An orphan is an article that receives zero inbound `[[links]]` from any other article's body. Scan all article bodies and build an inbound-link count for each file. Any article with a count of zero is orphaned.

Severity: **WARNING**

### 3. Bloated Articles

Count the total lines in each article file. Any article exceeding 120 lines is bloated and should be split.

Severity: **WARNING**

### 4. Stub Articles

Any article with fewer than 15 lines is too thin to be useful. Flag it as a stub.

Severity: **WARNING**

### 5. Missing Tags

Collect all tags used across every article's frontmatter `tags` field. Any tag that appears in 3 or more articles but is NOT listed in the `## Custom Tags` section of `<wiki_path>/schema.md` is a missing tag that should be added to the schema.

Severity: **INFO**

### 6. Dead Tags

Read the `## Custom Tags` section of `<wiki_path>/schema.md`. Any tag listed there that is used by zero articles is a dead tag.

Severity: **INFO**

### 7. Frontmatter Issues

Every article must have all required frontmatter fields: `title`, `aliases`, `category`, `tags`, `parents`, `created`, `updated`, `source`, `tier`, `status`. Check each article for:

- Missing required fields (tier and status are validated in detail by checks 11 and 12; this check catches their absence alongside other fields)
- Invalid `category` values (must be one of: `concepts`, `patterns`, `decisions`, `action-plans`)

Severity: **ERROR**

### 8. Parent Consistency

For each article, compare two things:

- The `parents` array in frontmatter
- The `[[links]]` in the `## Related` section

Flag cases where:
- A parent is listed in frontmatter but has no corresponding `[[link]]` in the Related section
- A `[[link]]` in the Related section points to an article listed as a parent in frontmatter of the linked article, but that relationship is not reflected in the current article's own frontmatter

Both directions of this consistency check matter because parent relationships must be declared in both frontmatter and the Related section.

Severity: **WARNING**

### 9. Index Drift

Compare the articles listed in `<wiki_path>/index.md` against the actual `.md` files on disk under `<wiki_path>/articles/`:

- **Articles on disk but not in index** -- files exist that the index does not reference
- **Index entries pointing to missing files** -- the index references articles that do not exist on disk

Severity: **ERROR**

### 10. Duplicate Topics

Scan all article titles and aliases for potential overlaps. Flag pairs of articles where:

- Titles are identical or near-identical (case-insensitive)
- An alias of one article matches the title or alias of another

These may indicate duplicate coverage of the same topic.

Severity: **INFO**

### 11. Tier Validity

Every article under `<wiki_path>/articles/` must have a `tier` field in its YAML frontmatter with a value of exactly `public` or `private`. See `tier-spec.md` for canonical tier definitions.

- Missing `tier` field: flag the article
- Invalid value (anything other than `public` or `private`): flag the article

Episodic entries (files under `<wiki_path>/episodic/`) are NOT articles and are exempt from this check. See `episodic-conventions.md` for their separate conventions.

Severity: **ERROR**

### 12. Status Validity

Every article under `<wiki_path>/articles/` must have a `status` field in its YAML frontmatter with one of: `draft`, `reviewed`, `verified`, `stale`, `archived`. See `lifecycle-spec.md` for the canonical lifecycle state machine.

- Missing `status` field: flag the article
- Invalid value: flag the article

Severity: **ERROR**

### 13. Staleness Detection

For articles with `status: verified` or `status: reviewed`, compute `days_since_update = today - updated`. Compare against the applicable staleness threshold from `<wiki_path>/schema.md`:

1. Read `staleness_rules` from schema.md (if present)
2. For each article's tags, check for tag-specific overrides — use the shortest (most aggressive) matching threshold
3. If no tag match, use `default_days`
4. If no `staleness_rules` in schema, use a default of 180 days

Flag articles where `days_since_update > applicable_threshold`. Suggest: "N articles are past their staleness threshold. Run `/wiki-stale` to mark them, then review and re-verify or archive."

If `staleness_rules` is not configured in schema.md, note: "No staleness_rules configured. Using default 180-day threshold. Configure in schema.md for domain-specific thresholds."

Severity: **WARNING**

---

## Empirical-Anchor Density (Advisory)

Read `empirical-anchor-spec.md` and apply the advisory dimension alongside the structural checks above.

---

## Report Format

Present findings using this exact structure:

```
Wiki Lint Report -- YYYY-MM-DD

ERRORS (must fix):
- Broken link: [[nonexistent-article]] in datetime-standardization.md
- Index drift: 2 articles on disk not in index.md
- Frontmatter: missing 'category' field in quick-sort-overview.md

WARNINGS (should fix):
- Orphaned: proc-sql-basics.md (0 inbound links)
- Bloated: data-manipulation-patterns.md (145 lines, cap is 120)
- Stub: numpy-intro.md (8 lines)
- Parent inconsistency: caching-strategies.md lists parent 'performance-hub' in frontmatter but no [[performance-hub]] in Related section

INFO:
- Dead tag: 'data-step' in schema (0 articles use it)
- Missing tag: 'proc-sql' used by 4 articles, not in schema
- Possible duplicate: 'date-formatting.md' and 'datetime-format.md' share alias 'date format'

Summary: N errors, M warnings, K info
```

### Severity Levels

- **ERRORS** -- Broken links, index drift, missing required frontmatter. These break functionality and must be fixed.
- **WARNINGS** -- Orphans, bloat, stubs, parent inconsistency. These degrade quality and should be fixed.
- **INFO** -- Dead tags, missing tags, potential duplicates. These are worth being aware of but are not urgent.

### Clean Wiki

If all checks pass with no findings at any severity level, report:

```
Wiki health: All checks passed. No issues found.
```

---

## Schema Validation

As part of the checks above, validate `<wiki_path>/schema.md` against actual article usage:

- **Custom Tags**: Compare tags listed in schema against tags actually used in article frontmatter (checks 5 and 6 above).
- **Hierarchy Roots**: Compare roots listed in schema's `## Hierarchy Roots` against articles that actually have `parents: []` in their frontmatter. Flag roots listed in schema that do not exist as articles, and articles with empty parents that are not listed as roots.

---

## Index Validation

As part of check 9, validate `<wiki_path>/index.md` in both directions:

- Every `.md` file under `<wiki_path>/articles/` should have a corresponding entry in the index.
- Every `[[link]]` in the index should point to a file that exists on disk.

Check all three index sections (By Category, By Hierarchy, Recent) for completeness.

---

## Strictly Read-Only

This command does NOT modify any files. It reads and reports only. After presenting the report, suggest which commands can fix the issues found:

- **Broken links, orphans, structural issues**: "Run `/wiki-reorg` to fix structural issues."
- **Index drift from new articles**: "Run `/wiki-absorb` to process new articles into the index."
- **Index drift from restructuring**: "Run `/wiki-reorg` to rebuild the index."
- **Tag issues (dead or missing tags)**: "Review `schema.md` Proposed Changes or edit Custom Tags manually."
- **Frontmatter issues**: "Edit the affected article files directly to add missing fields or correct invalid values."
- **Bloated articles**: "Run `/wiki-reorg` to split oversized articles."
- **Stubs**: "Flesh out the article content or merge into a related article via `/wiki-reorg`."

## Size Rationale

This skill exceeds the 250-line complex cap at ~275 lines. Justified: 13 structural checks with severity levels, each requiring specific detection logic and remediation guidance. Read-only, single invocation per session. Companion-extraction rejected — splitting checks across files would fragment the reviewer's mental model of the full check suite.

