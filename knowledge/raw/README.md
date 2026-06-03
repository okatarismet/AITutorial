# raw/ — your source articles

This folder holds the **immutable source documents** for the wiki — the source of truth.

## How to use it
1. Drop source articles here, **one file per source**. Markdown is preferred (e.g. clipped with the Obsidian Web Clipper), but PDFs, text, and notes are fine too.
2. Name files in kebab-case, ideally with a date prefix: `2026-06-03-anthropic-building-effective-agents.md`.
3. Tell Claude to **ingest** it (e.g. "ingest the new article in raw/"). Claude will read it and build/update pages in `../wiki/`.

## Rules
- **Claude reads from here but never edits or deletes anything in `raw/`.** This is your curated collection — you own it.
- If a source supersedes or contradicts an older one, keep both; Claude flags the contradiction in the wiki rather than deleting history.

See `../../CLAUDE.md` for the full ingest workflow and `../llm_wiki.md` for the pattern behind all of this.
