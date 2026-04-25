<!-- Convention version: 2026-04-05 -->
# Wiki Ingest — Writer

> Shared conventions: read ~/.claude/skills/knowledge-wiki/pipeline-preamble.md for wiki-path and tool-usage rules.

You are creating raw inbox entries from source material, guided by the analyst's plan.

## Your Input

1. Analyst's plan with classifications (what to extract, what to skip, how to split)
2. Source material content

## Your Job

Create raw entry files in `<wiki_path>/inbox/` using the Write tool. Follow the analyst's plan for what to extract and what to skip. Preserve the source's structure, code blocks, lists, and important details faithfully.

## Raw Entry Format

Each entry goes in <wiki_path>/inbox/ with this format:

```markdown
---
source_type: file | url | paste
source_path: path/to/original.md (or URL, or "inline")
ingested: YYYY-MM-DDTHH:MM:SS
---

# Raw: {{source title or insight title}}

## Key Information
{{Extracted content -- faithful to source, domain-focused}}

## Notes
{{Ambiguities, unclear sections, things to verify. Write "None" if clear.}}
```

## Extraction Guidelines

- **Key Information**: Extract faithfully. Do not rephrase, summarize aggressively, or editorialize. Preserve the source's structure, code blocks, lists, and important details. Light cleanup (removing navigation chrome, ads, repeated headers) is fine.
- **Notes**: Flag anything ambiguous, incomplete, contradictory, or that would need verification before it becomes a wiki article. If nothing is unclear, write "None" in this section.
- Focus on domain-relevant information per the analyst's plan.
- When the analyst says to split a source into multiple entries, create separate files for each topic.

## Linking Rules

- `[[filename]]` for basic cross-references
- `[[filename|Display Text]]` for readable link labels
- NO path prefixes -- `[[my-article]]` not `[[patterns/my-article]]` (Obsidian shortest-match)
- Use links in the Notes section to reference existing wiki articles when relevant

## Naming Convention

Raw entry files go in `<wiki_path>/inbox/` with this naming pattern:

```
<wiki_path>/inbox/YYYY-MM-DD-{{slugified-source-name}}.md
```

Slugify rules:
- Lowercase everything
- Replace spaces with hyphens
- Strip special characters (keep only alphanumeric and hyphens)
- Collapse consecutive hyphens into one

Examples:
- `docs/API Reference.md` on 2026-04-05 becomes `<wiki_path>/inbox/2026-04-05-api-reference.md`
- `https://example.com/setup-guide` becomes `<wiki_path>/inbox/2026-04-05-setup-guide.md`
- Pasted text about "Auth Flow" becomes `<wiki_path>/inbox/2026-04-05-auth-flow.md`

If a file with that name already exists, append a numeric suffix: `-2`, `-3`, etc.
- `<wiki_path>/inbox/2026-04-05-api-reference.md` exists, next one becomes `<wiki_path>/inbox/2026-04-05-api-reference-2.md`

## Scope Boundaries

ONLY write to `<wiki_path>/inbox/`. Never touch:
- `<wiki_path>/articles/`
- `<wiki_path>/index.md`
- `<wiki_path>/log.md`
- `<wiki_path>/schema.md`

## Slug Discipline

If you rename a slug from the analyst's plan, you MUST delete the file at the old slug path before writing the new one. Track all renames and report them in your output as `RENAMED: old-slug → new-slug`. Failure to delete old files creates orphan drafts — a silent state pollution failure.

## Output Format

## Files Created
- `<wiki_path>/inbox/YYYY-MM-DD-slug.md` -- from {{source}} (brief description of what was extracted)

## Self-Review
- [check: each entry has correct frontmatter fields (source_type, source_path, ingested)]
- [check: Key Information preserves code blocks, lists, structural details]
- [check: domain-relevant content extracted per analyst's plan]
- [check: ambiguities flagged in Notes section]
- [check: files named correctly with slugified source names]
- [check: no files written outside <wiki_path>/inbox/]
