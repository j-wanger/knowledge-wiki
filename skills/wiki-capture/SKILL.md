---
name: wiki-capture
description: "Use when a new insight, decision, or lesson surfaces in conversation and should not be lost. Writes a raw entry to the wiki inbox. Do NOT use for importing external files or URLs — use wiki-ingest."
---

# wiki-capture

Fast, low-friction capture of in-session insights into the wiki inbox. Speed is the priority -- get the knowledge written down now, let wiki-absorb polish it later. ONLY writes to `wiki/inbox/`. NEVER write to `articles/`, `index.md`, `log.md`, or any path outside `<wiki_path>/inbox/`.

---

## Pre-checks

Step 0 below (in the Orchestration Flow) runs FIRST and resolves `wiki_path` — the absolute path to the target wiki. These pre-checks run after Step 0 and use that resolved path.

1. **wiki exists:** Verify `<wiki_path>` is a directory. Step 0 should have caught this, but double-check. If not found: "Wiki path does not exist: <wiki_path>. The registry may be stale. Run `/wiki-list` to see registered wikis." Stop.
2. **schema.md readable:** Read `<wiki_path>/schema.md`. If missing or unparseable: "schema.md is missing or corrupted at <wiki_path>/schema.md. The wiki may be corrupted." Stop.
3. **inbox/ exists:** If `<wiki_path>/inbox/` does not exist, create it silently and continue.

---

## Orchestration Flow

### Step 0: Resolve Wiki

Read and follow the canonical template at `~/.claude/skills/knowledge-wiki/step-0-template.md`. It defines all 8 sub-steps (read registry, resolve by --wiki flag / CWD / auto-register / inference, validate, touch).

This skill is a **write operation** — in Sub-step 0.6, update the `last_used` field for the resolved wiki to today's date using the atomic write pattern documented in Sub-step 0.7.

After Step 0 completes, `wiki_path` is the canonical absolute path to the resolved wiki. All subsequent steps use `<wiki_path>/` wherever this skill previously used `wiki/`.

### Step 1: Gather context

Read the following and prepare a context summary:
- `<wiki_path>/schema.md` (domain, tags, hierarchy roots, conventions)
- Determine which mode we are in:

**Mode A -- With argument:** `/wiki-capture <text>` was invoked with explicit insight text. The insight is known. The analyst will verify framing.

**Mode B -- Without argument:** `/wiki-capture` was invoked with no text (or user said "capture this" without specifying what). The analyst will review conversation context to propose an insight.

### Steps 2-7: Orchestration (Analyst -> Writer -> Reviewer pipeline)

Read `~/.claude/skills/knowledge-wiki/orchestration-template.md` for the standard Analyst -> Writer -> Reviewer pipeline. This skill follows that template with these skill-specific inputs:

- **Analyst prompt:** `~/.claude/skills/wiki-capture/analyst-prompt.md`
- **Writer prompt:** `~/.claude/skills/wiki-capture/writer-prompt.md`
- **Reviewer prompt:** `~/.claude/skills/wiki-capture/reviewer-prompt.md`
- **Batching:** No
- **User approval gate:** Conditional (Mode B only)

**Skill-specific Runtime Context sections** (appended to the standard schema block):

```
### Session Context
{{recent conversation context -- problems solved, decisions made, patterns discovered}}

### Mode
{{A or B}}

### Explicit Insight (Mode A only)
{{the user's argument text, if provided}}
```

**Skill-specific approval gate (Mode B only):** If running in Mode B (no argument), present the analyst's identified insight to the user: "I identified this insight from our conversation: **[insight statement]**. Want me to capture it?" Wait for confirmation. If running in Mode A (with argument), skip -- the user already told us what to capture.

### Step 8: Completion report

Report to user:
- What was captured (title and filename)
- Count of items currently in `<wiki_path>/inbox/`
- If inbox has 3 or more items: "Your inbox has N items -- consider running `/wiki-absorb` to process them into articles."

---

## Tier and Lifecycle

Inbox entries created by capture carry `tier: private` by default (session captures are personal analysis). The user may override by specifying a tier in their invocation (e.g., `/wiki-capture --tier public <text>`). See `tier-spec.md` for canonical tier definitions.

Status is not set at capture time — it is assigned during absorb (see `lifecycle-spec.md`). The inbox entry's `source_type: session` signals to absorb that `private` is the default tier.

---

## Error Handling

If any Agent tool call fails (timeout, error), surface the error to the user:

"[Phase] failed: [error]. You can retry with /wiki-capture or investigate the error."

Do not retry automatically. Do not skip the failed phase.
