<!-- Consolidation dedup algorithm — SSOT -->
<!-- REFERENCE, DO NOT PASTE. consolidate.py and wiki-consolidate SKILL.md should link here. -->

# Consolidation Dedup Specification

Mechanical dedup for the consolidation pipeline. Filters episodic entries before the expensive A->W->R (Analyst->Writer->Reviewer) pipeline runs.

Design principle: deterministic checks before LLM semantic checks. The Python pre-pass (`consolidate.py`) classifies entries by vector similarity; the Analyst LLM makes the semantic duplicate/update judgment.

## Two-Phase Classification

`consolidate.py scan` classifies each unconsolidated episodic entry into one of two categories:

| Classification | Condition | Action |
|---|---|---|
| `candidate` | No article exceeds cosine similarity threshold | Entry proceeds to A->W->R pipeline for fact extraction |
| `high_similarity` | At least one article exceeds threshold | Entry passed to Analyst with matched article slug + similarity score; Analyst decides duplicate vs update |

The Python script does NOT make the duplicate/update judgment — that is a semantic task for the Analyst LLM. Python only flags similarity.

## Cosine Similarity Mechanism

- **Embeddings:** nomic-embed-text via fastembed (same model used by the search index in `skills/wiki-index/indexer.py`)
- **Storage:** `articles_vec` table in `.wiki-index.db` (SQLite + sqlite-vec extension)
- **Query:** consolidate.py queries the SQLite database directly (NOT through search.py, which discards distance values)
- **Distance metric:** cosine distance from sqlite-vec; convert to similarity as `1 - distance`

The embedding for each episodic entry is computed at scan time using the full body text (below the frontmatter fence). This embedding is not persisted — it is used only for the similarity comparison.

## Threshold Semantics

- **Default:** 0.85
- **Configurable** per-wiki via `~/.claude/wikis.json` under `consolidation.dedup_cosine_threshold`

Rationale for 0.85 vs LLMpedia's 0.90: our threshold flags candidates for Analyst review rather than auto-deduplicating. A more permissive threshold catches more potential duplicates for human/LLM judgment. The threshold is domain-dependent:

| Domain | Suggested Threshold | Reason |
|---|---|---|
| Regulatory/compliance text | 0.80 | Formulaic language inflates similarity — lower threshold avoids false negatives |
| ML research | 0.90 | Distinct terminology per topic — higher threshold avoids false positives |
| General knowledge | 0.85 (default) | Balanced for mixed-topic wikis |

## .consolidated Marker Format

**File:** `<wiki_path>/episodic/.consolidated`

**Format:** Newline-delimited list of processed episodic entry filenames (basenames only, one per line).

```
2026-04-20T10-00-00-topic-a.md
2026-04-20T13-00-00-topic-b.md
```

**Purpose:** Fast-scan to avoid re-reading every episodic entry's frontmatter to check `consolidated_at`.

**Source of truth:** The episodic entry's own frontmatter `consolidated_at` field is authoritative; `.consolidated` is a performance cache. On conflict, frontmatter wins.

## Consolidation Result Enum

Per `episodic-conventions.md`, the `consolidation_result` frontmatter field uses:

| Value | Meaning |
|---|---|
| `extracted` | Facts pulled into inbox entries (covers both new articles and updates to existing articles) |
| `duplicate` | Entry fully covered by existing article, no net-new facts |
| `low-confidence` | Analyst uncertain, flagged for manual review |

## Graceful Degradation

| Condition | Behavior |
|---|---|
| No `.wiki-index.db` | Warn, classify all entries as `candidate` |
| Index exists but no vectors (fastembed unavailable at index time) | Classify all as `candidate` |
| No episodic entries | Report "nothing to consolidate", exit 0 |
| All entries already consolidated | Report "all entries already processed", exit 0 |

When degrading to all-candidate mode, the scan output sets `"vectors_available": false` so downstream consumers know similarity scores are absent.

## Scan Output Format

`consolidate.py scan` writes a JSON manifest to stdout:

```json
{
  "wiki_path": "/path/to/wiki",
  "threshold": 0.85,
  "vectors_available": true,
  "candidates": [
    {"entry": "2026-04-20T10-00-00-topic-a.md", "tags": ["tag1"]}
  ],
  "high_similarity": [
    {"entry": "2026-04-20T13-00-00-duplicate.md", "matched_slug": "existing-article", "score": 0.91, "tags": ["tag1"]}
  ],
  "already_processed": 0,
  "total_scanned": 5
}
```

| Field | Type | Description |
|---|---|---|
| `wiki_path` | string | Absolute path to wiki root |
| `threshold` | float | Cosine similarity threshold used |
| `vectors_available` | bool | Whether vector search was available |
| `candidates` | array | Entries below threshold, proceeding to A->W->R |
| `high_similarity` | array | Entries above threshold, with matched article slug and score |
| `already_processed` | int | Count of entries skipped (already in `.consolidated`) |
| `total_scanned` | int | Total episodic entries examined |

## Exit Codes

| Code | Meaning |
|---|---|
| 0 | Success (including "nothing to consolidate") |
| 1 | Error (wiki path missing, DB read failure, malformed episodic entry) |

## Cross-References

- `episodic-conventions.md` — frontmatter mutation contract and consolidation metadata fields
- `registry-schema.md` — `consolidation.dedup_cosine_threshold` configuration
- `search-spec.md` — search index schema (`articles_vec` table) this depends on
