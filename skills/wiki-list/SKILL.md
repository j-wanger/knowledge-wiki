---
name: wiki-list
description: "Use when checking which wikis are registered. Lists name, description, path, and last-used date. Read-only. Do NOT use for detailed wiki health or metrics — use wiki-status or wiki-lint."
---

# wiki-list

Display all registered wikis in a formatted table. Read-only -- does not modify the registry.

---

## Pre-checks

1. **Registry readable:** Read `~/.claude/wikis.json`. If the file does not exist: "No wikis are registered yet. Run `/wiki-init` to create your first wiki, or `/wiki-init --register <path>` to adopt an existing one." Stop.
2. **Registry parseable:** If the file exists but is unparseable JSON: "Registry at ~/.claude/wikis.json is corrupted. Back up the file and remove it to start fresh, then re-register your wikis with /wiki-init --register <path>." Stop.

---

## Orchestration Flow

### Step 1: Read the registry

Read `~/.claude/wikis.json` into memory. Parse the JSON and extract the `wikis` array.

If the `wikis` array is empty (file exists but has no wiki entries), report:

    No wikis are registered. Run `/wiki-init` to create one.

Stop.

### Step 2: Gather article counts

For each registered wiki:

1. Verify the `path` exists on disk. If not, mark the wiki as "missing" in the report.
2. If the path exists, count article `.md` files. Articles may live under either `<path>/articles/` (the standard layout) or `<path>/wiki/` (the wiki-bootstrap layout). Try both:
   - Use the Glob tool: `<path>/articles/**/*.md`
   - Use the Glob tool: `<path>/wiki/**/*.md`
   - Sum the counts from both (a wiki will typically only use one layout).
3. Exclude any matches that are `schema.md`, `index.md`, or `log.md` (these are not articles).

### Step 3: Format and report

Present the wikis as a table. Use the following format:

    Registered wikis (N):

    | Name             | Description                                    | Path                              | Articles | Last Used  |
    |------------------|------------------------------------------------|-----------------------------------|----------|------------|
    | knowledge-wiki   | Claude Code skill framework design             | /Users/x/knowledge-wiki/wiki      | 18       | 2026-04-08 |
    | aml-compliance   | Anti-money laundering rules and case studies   | /Users/x/knowledge/aml/wiki       | 42       | 2026-04-07 |
    | astrology        | Chart interpretation methods                    | /Users/x/personal/astrology       | 0 (missing) | 2026-04-03 |

    Run `/wiki-rename <old> <new>` to rename a wiki.
    Run `/wiki-init --register <path>` to add an existing wiki.

Column widths may expand to fit content. For missing wikis, show `0 (missing)` in the Articles column.

Sort wikis by `last_used` descending (most recently used first). If two wikis have the same `last_used` date, sort by name ascending.

---

## Error Handling

If reading the registry fails for any reason other than "file does not exist," surface the error to the user: "Could not read ~/.claude/wikis.json: [error]. Investigate the file manually."

Do not attempt to repair the registry. Do not create or modify any files.

This skill is strictly read-only.
