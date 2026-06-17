---
name: process-materials
description: >
  Commit raw source changes and ingest them into the wiki in one pipeline.
  Use this skill whenever the user says "process", "process new materials",
  "process my changes", "commit and ingest", or indicates they've finished
  editing raw sources and want them filed into the wiki. Also trigger when
  the user says "ingest" after making uncommitted raw changes. This skill
  handles the full lifecycle: detect changes, commit raw, ingest into wiki
  pages, then commit wiki — all while respecting commit discipline and
  living-source conventions.
---

# Process Materials

A full pipeline skill for the second brain wiki. Takes uncommitted raw source
changes through commit, ingest, discussion, wiki generation, and final commit.

## When to use

- User says "process", "process new materials", or "commit and ingest"
- User has finished editing raw sources and wants them filed
- User says "ingest" but has uncommitted raw changes

## Prerequisites

- The vault has `AGENTS.md` with wiki schema (read it for full conventions)
- `raw/.append-only` lists living-source filenames
- `raw/.provenance` tracks source origin per raw file
- `.git/hooks/pre-commit` enforces raw/wiki commit separation
- `wiki/log.md` tracks ingest history

## Pipeline

### Phase 1: Detect and Commit Raw Changes

1. Run `git status` to identify:
   - **New untracked files** in `raw/` (first-time sources)
   - **Modified tracked files** in `raw/` (appends to living sources)
   - If nothing changed in `raw/`, inform the user and stop

2. Commit raw changes:
   ```
   git add raw/
   git commit -m "<descriptive message about what was added/updated>"
   ```
   This fires the pre-commit hook which validates immutable/append-only rules.
   If the hook rejects, explain why and stop.

3. Record the **commit hash** — you'll need it for living-source diffing.

### Phase 2: Identify What to Ingest

For each changed raw file, classify it:

**New source (untracked before):**
- Read the entire file
- This is a standard first-time ingest

**Living source (modified, listed in `raw/.append-only`):**
- Find the last ingest entry for this source in `wiki/log.md`
- Get the commit hash from that log entry (or find it via `git log`)
- Diff: `git show <last-ingest-commit>:raw/<filename>` vs current
- Extract only the NEW content (everything after the previously ingested version)

**Modified immutable source (shouldn't happen — hook blocks this):**
- If somehow reached, warn the user and skip

**Provenance (each changed raw file):**
- Read `raw/.provenance` for an existing entry
- If the file is **new** (not in manifest), ask the owner to classify using AskQuestion:
  - Standard options: `owner-input`, `human-authored`, `ai-generated`, `mixed`
  - Include **Other** so the owner can type any custom label (e.g. `ai-assisted`, `collaborative`)
- If the file is a **living source already in `.provenance`**, skip the prompt
- If provenance is obvious from context and the owner said "skip discussion" or "go ahead", infer when reasonable but prefer asking for new sources

### Phase 3: Discuss Key Takeaways

Before writing any wiki pages, present findings to the owner:

- Summarize each source's new content (2-5 bullet points per source)
- Highlight connections to existing wiki pages
- Propose what pages to create vs. update
- Ask: "Does this look right? Anything to emphasize or skip?"

**Wait for owner confirmation before proceeding.**

### Phase 4: Ingest into Wiki

Follow the AGENTS.md ingest workflow for each source:

1. **Record provenance** — add or confirm entry in `raw/.provenance` for new sources
2. Create summary page (from `wiki/templates/summary.md`) for new sources
   - For living sources being re-ingested, update the existing summary page
3. Update or create entity/concept pages as warranted
4. Add wikilinks between all touched pages
5. Set `source_origin` in frontmatter of every created/updated wiki page (inherit from raw sources; use `mixed` when sources span types)
6. Update `wiki/index.md` — add entries under correct categories
7. Update `wiki/overview.md` if the high-level picture changed
8. Append to `wiki/log.md`:
   ```
   ## [YYYY-MM-DD] ingest | <Source Title>
   - Source: raw/<filename>
   - Created: [[page-a]], [[page-b]]
   - Updated: [[page-c]], [[page-d]]
   - Notes: <brief description>
   ```

### Phase 5: Commit Wiki Changes

```
git add wiki/
git commit -m "<descriptive message about wiki updates>"
```

Report to the user:
- Number of pages created/updated
- Commit hashes for both raw and wiki commits
- Any open questions or follow-ups

## Edge Cases

- **Multiple raw files changed:** Process each in the pipeline. Commit all
  raw changes together in one commit, but discuss/ingest each source
  separately with the owner.

- **Living source with no prior ingest in log:** Treat as first-time ingest
  (read the whole file). This happens when a file was added to `.append-only`
  but never processed before.

- **Schema files also changed (AGENTS.md, .cursor/):** Commit those in a
  separate schema commit before the raw commit.

- **PDF or binary sources:** Read using the Read tool (which handles PDFs).
  Cannot do git text-diff on binaries — always treat as full ingest.

- **Owner says "skip discussion":** Respect it — proceed directly to Phase 4.
  Note this in the log entry.

## Important Conventions

- Never modify raw files
- One commit for raw, one for wiki — never mixed
- Tag pages with appropriate categories from AGENTS.md
- Set `freshness: perishable` for fast-moving domains (technology, AI)
- Set `freshness: evergreen` for stable domains (health history, family)
- Use `source_date` from the document if available; ingest date if not
- Every new raw source must have a `raw/.provenance` entry before wiki commit completes
- Wiki pages inherit `source_origin` from raw sources; owner may use custom provenance labels
- When answering queries, cite provenance when origin affects reliability (owner observation vs AI-generated vs human-authored)
