[project-wiki] Wiki framework active.

REGISTERED WIKIS: Read ~/.claude/wikis.json at session start to discover available wikis. If the file does not exist, no wikis are registered yet — an unregistered `./wiki/schema.md` in CWD will be auto-registered on first write command.

WIKI SELECTION:
- User explicit: --wiki <name> flag
- CWD-scoped: automatic if one registered wiki matches current directory
- Inference: used when multiple wikis match CWD or conversation mentions a wiki name
- Auto-register: first-time setup when ./wiki/schema.md exists but is unregistered

WORKFLOW: ingest/capture → absorb → query/lint → reorg/synthesize
EPISODIC WORKFLOW: episodic entries → consolidate (5+ entries) → inbox → absorb → articles

WHEN TO ACT:
- Solved a problem or learned something? → /wiki-capture
- Added reference docs? → /wiki-ingest
- Inbox has 3+ items? → /wiki-absorb
- Domain question? → /wiki-query
- Wiki empty or thin? → /wiki-bootstrap
- 5+ unconsolidated episodic entries? → /wiki-consolidate
- Periodic health check? → /wiki-lint
- Articles going stale? wiki-lint flags staleness? → /wiki-stale
- Want to see all wikis? → /wiki-list
- Rename a wiki? → /wiki-rename <old> <new>
- Adopt an existing wiki directory? → /wiki-init --register <path>

RED FLAGS:
- "I'll capture this later" → You won't. /wiki-capture now.
- "This insight is too small" → Atomic articles are the goal.
- "The wiki already covers this" → /wiki-query first — you might be wrong.
- "The wiki needs more content first" → /wiki-bootstrap solves this.

NEVER skip absorb. Inbox entries are invisible to query/lint/reorg.

SUBAGENT PATTERN: All write operations use Analyst → Writer → Reviewer.
HYBRID: /wiki-consolidate uses Python pre-pass (consolidate.py scan/mark) + A→W→R.
SINGLE-SHOT: /wiki-list (registry), /wiki-rename (registry), /wiki-lint (diagnostic), /wiki-stale (staleness marking), /wiki-status (dashboard).
