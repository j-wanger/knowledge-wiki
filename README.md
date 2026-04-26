# knowledge-wiki

A Claude Code skill suite for managing persistent knowledge bases. Provides a complete CRUD lifecycle for wiki-style knowledge stores: capture insights, ingest external sources, absorb raw entries into polished articles, query across articles, and maintain structural health.

## What it does

- **Capture** decisions, patterns, and lessons from conversations into a structured wiki
- **Ingest** external files, URLs, and reference material
- **Absorb** raw inbox entries into polished, cross-linked articles
- **Query** the wiki for answers using article navigation and synthesis
- **Bootstrap** new wikis from online research or seed existing ones with new topics
- **Maintain** wiki health with linting, reorganization, and synthesis

## Skills

| Skill | Purpose |
|-------|---------|
| `/knowledge-wiki` | Router — dispatches to the correct sub-skill based on user intent |
| `/wiki-init` | Bootstrap a new wiki with schema, directory structure, and initial articles |
| `/wiki-capture` | Save insights from the current conversation to the wiki inbox |
| `/wiki-ingest` | Import external files or URLs into the wiki inbox |
| `/wiki-absorb` | Convert raw inbox entries into polished, cross-linked articles |
| `/wiki-query` | Answer questions by navigating and synthesizing wiki articles |
| `/wiki-bootstrap` | Seed a wiki via online research (full gap analysis or focused topics) |
| `/wiki-lint` | Audit structural health — broken links, orphans, bloat, tag drift |
| `/wiki-reorg` | Restructure categories, fix hierarchy, garden the wiki |
| `/wiki-synthesize` | Generate new insights by connecting existing articles |
| `/wiki-status` | Dashboard of articles, cross-links, tags, and recent activity |
| `/wiki-consolidate` | Convert episodic entries into inbox entries via dedup + fact extraction |
| `/wiki-index` | Build hybrid search index (FTS5 + vector) for faster wiki-query |
| `/wiki-list` | List all registered wikis with metadata |
| `/wiki-rename` | Rename a registered wiki (registry only, no file moves) |

## Installation

Copy the skill directories to your Claude Code skills folder:

```bash
# Clone
git clone https://github.com/j-wanger/knowledge-wiki.git
cd knowledge-wiki

# Copy to Claude Code skills directory
cp -r skills/* ~/.claude/skills/
```

## Usage

```
# Start a new wiki for your project
/wiki-init

# Capture an insight from conversation
/wiki-capture

# Import reference material
/wiki-ingest

# Process inbox entries into articles
/wiki-absorb

# Ask the wiki a question
/wiki-query

# Check wiki health
/wiki-lint

# Research and add new content
/wiki-bootstrap
```

## Wiki structure

Each wiki follows this layout:

```
wiki/
  schema.md          # Domain identity, tags, hierarchy roots
  index.md           # Article index by category and hierarchy
  inbox/             # Raw captures awaiting absorb
  articles/
    concepts/        # Core domain concepts
    patterns/        # Reusable patterns and techniques
    decisions/       # Architectural Decision Records
    action-plans/    # Improvement plans with tracked progress
```

## Multi-wiki support

Register multiple wikis in `~/.claude/wikis.json`. Each wiki is scoped to a project directory and automatically activated when Claude Code's working directory matches.

## Package contents

- 14 skill directories (1 router + 13 sub-skills)
- 56 files, ~5,200 lines
- Hybrid search index (FTS5 + vector) via wiki-index — requires Python 3.11+, uv, fastembed, sqlite-vec, xxhash
- Consolidation pipeline via wiki-consolidate — converts episodic entries into inbox entries with dedup

## Related

- [dev-wiki](https://github.com/j-wanger/dev-wiki) — Project lifecycle management (phases, tasks, reviews)
