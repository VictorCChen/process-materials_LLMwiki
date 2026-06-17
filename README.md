# Process Materials

An agent skill for **ingesting new materials into a Karpathy-style LLM wiki** — takes raw source files through commit, AI discussion, wiki page generation, and final commit in a single pipeline.

Inspired by Andrej Karpathy's approach of building a personal wiki as a second brain, this skill automates the workflow of transforming raw notes, articles, and bookmarks into structured, cross-linked wiki pages with provenance tracking.

## What it does

- Detects changes in `raw/` (new & modified source files)
- Commits raw changes with pre-commit validation (immutable/append-only rules)
- Ingests raw sources into wiki pages using the `AGENTS.md` schema
- Commits the generated wiki pages in a separate commit

## When to use

Say "process", "process new materials", "process my changes", or "commit and ingest" after editing raw sources. This is the standard workflow whenever you add new knowledge to the wiki.

## Installation

Copy `SKILL.md` into your agent's skills directory (e.g., `~/.agents/skills/process-materials/` or `~/.claude/skills/process-materials/`).

## Prerequisites

- A vault with `AGENTS.md` defining wiki schema
- `raw/.append-only` listing living-source filenames
- `raw/.provenance` tracking source origins
- `.git/hooks/pre-commit` enforcing raw/wiki commit separation
- `wiki/log.md` for ingest history

## License

See [LICENSE](./LICENSE) file.
