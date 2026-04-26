---
name: wiki-ingest
description: "Use when importing external files, URLs, or pasted reference material into the wiki. Lands entries in the inbox for later absorb. Do NOT use for insights from the current conversation — use wiki-capture."
---

# wiki-ingest

Ingest files, URLs, or pasted text into `<wiki_path>/inbox/` as raw entries for later processing by `/wiki-absorb`.

---

## Pre-checks

Step 0 below (in the Orchestration Flow) runs FIRST and resolves `wiki_path` — the absolute path to the target wiki. These pre-checks run after Step 0 and use that resolved path.

1. **wiki exists:** Verify `<wiki_path>` is a directory. Step 0 should have caught this, but double-check. If not found: "Wiki path does not exist: <wiki_path>. The registry may be stale. Run `/wiki-list` to see registered wikis." Stop.
2. **schema.md readable:** Read `<wiki_path>/schema.md`. If missing or unparseable: "schema.md is missing or corrupted at <wiki_path>/schema.md. The wiki may be corrupted." Stop.

---

## Input Handling

Three input modes are supported. Multiple sources in a single invocation are fine -- process each one separately.

### File paths (glob supported)

When the user provides file paths or glob patterns, expand globs and read each file's contents.

```
/wiki-ingest ./docs/*.md
/wiki-ingest src/config.ts README.md
```

### URLs

When the user provides URLs, fetch content using WebFetch and extract meaningful text.

```
/wiki-ingest https://example.com/docs/getting-started
```

### Pasted text

When the user provides inline content (no file path, no URL), treat the pasted text as the source.

```
/wiki-ingest
(user pastes text in the same message or follow-up)
```

---

## Orchestration Flow

### Step 0: Resolve Wiki

Read and follow the canonical template at `~/.claude/skills/knowledge-wiki/step-0-template.md`. It defines all 8 sub-steps (read registry, resolve by --wiki flag / CWD / auto-register / inference, validate, touch).

This skill is a **write operation** — in Sub-step 0.6, update the `last_used` field for the resolved wiki to today's date using the atomic write pattern documented in Sub-step 0.7.

After Step 0 completes, `wiki_path` is the canonical absolute path to the resolved wiki. All subsequent steps use `<wiki_path>/` wherever this skill previously used `wiki/`.

### Step 1: Gather context

Read the following and prepare a context summary:
- `<wiki_path>/schema.md` (domain, tags, hierarchy roots, conventions)
- All source material: read files, fetch URLs, or capture pasted text

For files and URLs, read full content. If a single source exceeds 200 lines, include the first 200 lines and note the truncation.

### Steps 2-6: Orchestration (Analyst -> Writer -> Reviewer pipeline)

Read `~/.claude/skills/knowledge-wiki/orchestration-template.md` for the standard Analyst -> Writer -> Reviewer pipeline. This skill follows that template with these skill-specific inputs:

- **Analyst prompt:** `~/.claude/skills/wiki-ingest/analyst-prompt.md`
- **Writer prompt:** `~/.claude/skills/wiki-ingest/writer-prompt.md`
- **Reviewer prompt:** `~/.claude/skills/wiki-ingest/reviewer-prompt.md`
- **Batching:** No
- **User approval gate:** No

**Skill-specific Runtime Context sections** (appended to the standard schema block):

```
### Source Material
{{for each source: filename/URL, content}}
```

**Skill-specific writer extras:** Append the source material content under `## Source Content` alongside the analyst plan.

**Skill-specific reviewer extras:** Append the original source material under `## Original Source` so the reviewer can verify extraction completeness.

### Step 7: Completion report

Report to user:
- What was ingested (list each inbox file created with its source)
- Total entry count
- Remind about next step

> Ingested N source(s) into `<wiki_path>/inbox/`:
> - `YYYY-MM-DD-slug.md` (from source)
>
> Run `/wiki-absorb` when ready to process inbox into wiki articles.

---

## Scope Boundaries

This command writes ONLY to `<wiki_path>/inbox/`. It does not touch anything else:

- Does NOT touch `<wiki_path>/index.md`
- Does NOT touch `<wiki_path>/log.md`
- Does NOT touch `<wiki_path>/articles/`
- Does NOT categorize, tag, or cross-link
- Does NOT modify `<wiki_path>/schema.md`

The inbox is a staging area. All structuring, linking, and index updates happen via `/wiki-absorb`.

---

## Tier Assignment

Inbox entries created by ingest carry a tier hint based on source type. See `tier-spec.md` for canonical tier definitions.

| Source Type | Default Tier | Rationale |
|------------|-------------|-----------|
| `file` (external docs, references) | `public` | External reference material is typically factual |
| `url` (web content) | `public` | Published web content is citable |
| `paste` (inline text) | `private` | Pasted notes may be personal analysis |

The user may override by specifying `--tier public` or `--tier private`. The tier hint is stored in the inbox entry frontmatter for absorb to use. Final tier assignment happens during absorb (see `tier-spec.md` for assignment rules).

---

## Error Handling

If any Agent tool call fails (timeout, error), surface the error to the user:

"[Phase] failed: [error]. You can retry with /wiki-ingest or investigate the error."

Do not retry automatically. Do not skip the failed phase.
