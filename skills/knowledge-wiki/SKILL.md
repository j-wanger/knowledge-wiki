---
name: knowledge-wiki
description: "Route wiki operations and enforce wiki workflow discipline. Use when the user mentions 'wiki', 'knowledge base', or wants to manage project knowledge. Also activated by SessionStart hook when a registered wiki applies to the current directory. Do NOT invoke directly when the user names a specific wiki command — invoke that command instead (wiki-query, wiki-capture, etc.)."
---

<EXTREMELY-IMPORTANT>
When a registered wiki applies to the current project (or `./wiki/` exists and can be auto-registered), wiki skills are part of the workflow. Do not wait for the user to explicitly ask -- suggest wiki operations at natural moments.

Captured knowledge that never reaches the wiki is lost knowledge. Enforce the pipeline.
</EXTREMELY-IMPORTANT>

# knowledge-wiki

## The Registry

Wikis are tracked globally in `~/.claude/wikis.json`. Each command resolves its target wiki via the canonical Step 0 template (see `step-0-template.md`). Wiki selection happens in this order:

1. **Explicit:** User passes `--wiki <name>` flag
2. **CWD-scoped:** Exactly one registered wiki has a path matching the current directory
3. **Inference:** Multiple wikis match; dispatch a one-shot inference subagent
4. **Auto-register:** CWD has `./wiki/schema.md` but no registered wiki; register it on first use

If no registered wiki applies and no local `./wiki/` exists, suggest `/wiki-init` to create one.

## The Rule

**When a registered wiki is in scope, wiki operations are always in scope.** You do not need the user to say "wiki" -- if you see an opportunity to capture, query, or maintain the wiki, act on it. Suggest wiki operations at natural moments: after learning something new, after a research phase, after resolving a complex bug, after architectural decisions.

**Automatic triggers:** New knowledge surfaced → `/wiki-capture`. External docs to add → `/wiki-ingest`. Inbox has 3+ entries → `/wiki-absorb`. Domain question → `/wiki-query`. Wiki thin (0-2 articles) → `/wiki-bootstrap`. No registered wiki and no local `./wiki/schema.md` → `/wiki-init`.

**User-initiated:** `/wiki-lint`, `/wiki-reorg`, `/wiki-synthesize`, `/wiki-status`, `/wiki-list`, `/wiki-rename`. See the Command Reference table below for the full routing surface.

## Red Flags

These thoughts mean STOP -- you are rationalizing skipping wiki discipline:

| Thought | Reality |
|---------|---------|
| "I'll capture this later" | You won't. /wiki-capture now. |
| "The inbox only has 2 items" | Absorb small batches for quality. |
| "This insight is too small" | Atomic articles are the goal. |
| "I'll reorganize eventually" | Run /wiki-lint to see if it's needed now. |
| "The wiki already covers this" | Run /wiki-query first. Do NOT skip -- your assumption may be wrong. |

## Workflow Order

```
init (once) -> ingest/capture (ongoing) -> absorb (periodic) -> lint (health check) -> reorg (structural) -> synthesize (creative)
episodic entries: consolidate (5+ entries) -> inbox -> absorb -> articles
```

**Never skip absorb.** Raw inbox entries are invisible to query, lint, reorg, and synthesize. Until entries are absorbed into articles, they do not exist as far as the wiki is concerned.

## Skill Classification

### Subagent-driven (Analyst -> Writer -> Reviewer)

These operations dispatch 3 sequential subagents via the Agent tool:

- **/wiki-init** -- Interview, scaffold, review
- **/wiki-ingest** -- Extract, format, review
- **/wiki-capture** -- Draft, refine, review
- **/wiki-bootstrap** -- Research domain, generate foundational articles, review quality
- **/wiki-absorb** -- Analyze inbox, write articles, review quality
- **/wiki-consolidate** -- Scan episodic entries, dedup via search index, extract facts into inbox
- **/wiki-query** -- Search, synthesize answer, verify accuracy
- **/wiki-reorg** -- Analyze structure, restructure, review changes
- **/wiki-synthesize** -- Identify patterns, draft insights, review

Max 2 review rounds before escalating to the user. Do not loop indefinitely.

### Single-shot

These run directly without subagent dispatch:

- **/wiki-index** -- Build hybrid search index (FTS5 + sqlite-vec vectors) for a wiki
- **/wiki-list** -- Show all registered wikis
- **/wiki-rename** -- Rename a wiki in the registry
- **/wiki-lint** -- Read-only structural health audit
- **/wiki-stale** -- Detect and mark stale articles past their staleness threshold
- **/wiki-status** -- Read-only dashboard and stats

## Command Reference

Route user intent to the correct wiki skill:

| User intent | Route to |
|-------------|----------|
| "Set up a wiki" / "initialize" | /wiki-init |
| "Register existing wiki" / "adopt" | /wiki-init --register <path> |
| "List my wikis" / "show wikis" | /wiki-list |
| "Rename wiki" / "change wiki name" | /wiki-rename <old> <new> |
| "Seed domain knowledge" / "bootstrap" / wiki is empty | /wiki-bootstrap |
| "Add these docs" / "import" | /wiki-ingest |
| "I just learned something" / "capture this" | /wiki-capture |
| "Process the inbox" / "absorb" | /wiki-absorb |
| "What does the wiki say about X" | /wiki-query |
| "Build search index" / "index the wiki" | /wiki-index |
| "Check wiki health" / "audit" | /wiki-lint |
| "Reorganize" / "restructure" | /wiki-reorg |
| "Process episodic entries" / "consolidate" | /wiki-consolidate |
| "Mark stale articles" / "staleness check" | /wiki-stale |
| "What patterns are emerging?" | /wiki-synthesize |
| "Wiki stats" / "dashboard" | /wiki-status |

## If Unclear

When the user's wiki intent is ambiguous, ask:

> What would you like to do with the wiki?
>
> - **List** -- See all registered wikis
> - **Bootstrap** -- Seed foundational domain knowledge
> - **Capture** -- Save an insight or decision from this session
> - **Ingest** -- Import external files or docs
> - **Absorb** -- Process inbox entries into articles
> - **Query** -- Ask the wiki a question
> - **Lint** -- Check wiki health
> - **Status** -- See wiki dashboard

Do not guess. Present the options and let the user choose.
