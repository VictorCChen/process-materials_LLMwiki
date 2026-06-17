# Process Materials

A pipeline skill for the second brain wiki — takes uncommitted raw source changes through commit, ingest, discussion, wiki generation, and final commit.

## What it does

- Detects changes in `raw/` (new & modified source files)
- Commits raw changes with pre-commit validation (immutable/append-only rules)
- Ingests raw sources into wiki pages using the `AGENTS.md` schema
- Commits the generated wiki pages in a separate commit

## When to use

Say "process", "process new materials", "process my changes", or "commit and ingest" after editing raw sources.

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
