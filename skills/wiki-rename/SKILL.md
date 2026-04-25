---
name: wiki-rename
description: "Use when changing a registered wiki's display name. Updates the registry only; does not move files or change paths. Do NOT use for restructuring wiki content or moving articles — use wiki-reorg."
---

# wiki-rename

Rename a wiki in the registry. Updates the `name` field only -- does not move files, change paths, or modify any wiki contents.

---

## Pre-checks

1. **Two arguments provided:** The command requires `<old-name> <new-name>`. If either is missing: "Usage: `/wiki-rename <old-name> <new-name>`. Run `/wiki-list` to see registered wikis." Stop and ask the user for both.
2. **Registry exists:** Read `~/.claude/wikis.json`. If the file does not exist: "No wikis are registered yet. Nothing to rename. Run `/wiki-init` to create your first wiki." Stop.
3. **Registry parseable:** If the file exists but is unparseable JSON: "Registry at ~/.claude/wikis.json is corrupted. Back up the file and remove it to start fresh, then re-register your wikis with /wiki-init --register <path>." Stop.
4. **Old name exists:** Look up `<old-name>` in the `wikis` array. If not found: "No wiki named '<old-name>' is registered. Run `/wiki-list` to see available wikis." Stop.
5. **New name is unique:** Check that no entry in the `wikis` array has `name == <new-name>`. If collision found: "A wiki named '<new-name>' already exists. Choose a different name." Stop.
6. **New name format:** Verify `<new-name>` matches kebab-case: lowercase letters, digits, and hyphens only, no leading/trailing hyphens, no consecutive hyphens. If invalid: "Wiki names must be kebab-case (lowercase letters, digits, hyphens). '<new-name>' is invalid." Stop.

---

## Orchestration Flow

### Step 1: Read the registry

Read `~/.claude/wikis.json` into memory. Parse the JSON.

### Step 2: Update the name field

Find the entry in the `wikis` array where `name == <old-name>` and change its `name` field to `<new-name>`. Leave all other fields (`path`, `description`, `registered`, `last_used`) unchanged.

### Step 3: Atomic write

1. Serialize the modified registry to JSON (with indentation for readability, 2-space indent).
2. Use the Write tool to write the serialized JSON to `~/.claude/wikis.json.tmp`.
3. Use the Bash tool to rename: `mv ~/.claude/wikis.json.tmp ~/.claude/wikis.json`

This ensures the registry is never left in a partial state if the operation is interrupted.

### Step 4: Report

Report to the user:

    Renamed '<old-name>' to '<new-name>'.
    Path unchanged: <path>

Include the wiki's path in the report so the user can verify nothing moved.

---

## Error Handling

If the atomic write fails (Write tool error or Bash rename error), surface the error: "Rename failed: [error]. The registry is unchanged. Investigate the failure."

If the `.tmp` file was created but the rename failed, note it in the error so the user knows it may need manual cleanup: "A temporary file at ~/.claude/wikis.json.tmp may need to be removed manually."

Do not attempt automatic recovery. Do not modify the registry on error.

This skill only modifies the `name` field of a single registry entry. It never touches wiki files, paths, or any other registry fields.
