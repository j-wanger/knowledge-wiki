# Keyword Scoring Specification

Grep-based keyword-frequency scoring for wiki article pre-selection. Run by the wiki-query orchestrator in Step 1, before analyst dispatch.

## Algorithm

### Step A: Extract Keywords

From the user's question:
1. Lowercase the question.
2. Remove common stop words (a, an, the, is, are, was, were, of, in, to, for, on, at, by, with, from, as, and, or, but, not, this, that, how, what, which, who, when, where, why, do, does, did, can, could, should, would, will, have, has, had, it, its, i, we, you, they, my, your).
3. Split into individual words. Deduplicate.
4. Keep up to 5 keywords. If more than 5, take the 5 longest words.

### Step B: Grep Frontmatter

For each keyword, run a case-insensitive Grep across `<wiki_path>/articles/` with the keyword as pattern and `glob: "**/*.md"` to restrict to markdown files. From the results, keep only lines that are frontmatter fields: lines starting with `title:`, `aliases:`, or `tags:`. Record which article file each match came from.

Use parallel Grep calls for all keywords simultaneously.

### Step C: Count and Rank

For each article that matched at least one keyword:
1. Count the number of distinct keywords that matched (not total line hits).
2. Sort articles by keyword-match count descending. Break ties alphabetically by filename.
3. Take the top 10.

### Step D: Format

Produce a markdown block:

```
### Pre-Scored Candidates
Articles matching query keywords in frontmatter (title/aliases/tags):
1. article-slug.md (matched: keyword1, keyword2)
2. ...
```

If zero articles match any keyword, do not produce the section. Report to the orchestrator that no pre-scored candidates were found.
