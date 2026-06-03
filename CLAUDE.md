# AI Native Engineering — Education Platform Wiki

## Project purpose

We are building the first real education platform for **AI Native Engineering** — the discipline of building software *with* and *on top of* LLMs (prompting, context engineering, agents, RAG, evals, AI-assisted coding, deployment, safety, etc.).

The build happens in **two phases**:

1. **Build the knowledge base.** The human (Ismet) curates and drops source articles into `knowledge/raw/`. Claude reads them and incrementally builds an interlinked wiki in `knowledge/wiki/` — summaries, concept pages, entity pages, and an evolving roadmap.
2. **Generate learning materials.** Once the wiki is substantial, we use it to produce the actual curriculum (course modules, reading paths, decks — format decided later). The wiki is the source of truth that learning materials are compiled *from*.

We are currently in **Phase 1**. Do not jump ahead to producing curriculum until the wiki has enough sourced content and Ismet says so.

This project applies the **LLM Wiki pattern** documented in `knowledge/llm_wiki.md`. Read that file for the full philosophy. This `CLAUDE.md` is the concrete instantiation of that pattern for this domain — it is the **schema** that turns Claude into a disciplined wiki maintainer rather than a generic chatbot.

## Architecture — three layers

1. **`knowledge/raw/`** — immutable source documents (articles, papers, notes). This is the source of truth. **Claude reads from here but NEVER edits or deletes anything in `raw/`.**
2. **`knowledge/wiki/`** — LLM-generated, LLM-owned markdown pages. Claude creates and maintains everything here: summaries, concept pages, entity pages, the index, the log, the overview.
3. **`CLAUDE.md`** (this file) — the schema. Conventions and workflows. Co-evolved by Claude and Ismet as we learn what works.

Two **top-level project docs** live at the repo root (deliverables, not wiki pages):
- **`ROADMAP.md`** — the living learning roadmap + topic taxonomy (the bridge to Phase 2). Keep its levels in sync with the taxonomy below.
- **`SOURCE.md`** — the curated candidate reading list, tagged by roadmap level with ingest-status checkboxes.

```
ROADMAP.md             # learning roadmap (6 levels) — repo root
SOURCE.md              # curated source candidates — repo root
knowledge/
├── llm_wiki.md        # the pattern reference (do not treat as a source or edit)
├── raw/               # immutable sources — Ismet drops articles here
└── wiki/              # Claude-owned
    ├── index.md       # catalog of every wiki page (update on every ingest)
    ├── log.md         # append-only chronological log
    ├── overview.md    # "What is AI Native Engineering?" — top-level synthesis
    ├── concepts/      # one page per concept (prompting, context-engineering, evals…)
    ├── entities/      # one page per tool/framework/org/person/model
    └── sources/       # one summary page per ingested raw source
```

## Page conventions

- **Filenames:** kebab-case, descriptive. e.g. `context-engineering.md`, `claude-code.md`, `2026-06-03-anthropic-building-effective-agents.md`.
- **Frontmatter:** every wiki page starts with YAML frontmatter:
  ```yaml
  ---
  title: Context Engineering
  type: concept            # one of: concept | entity | source | overview | roadmap | index | log
  tags: [context, prompting, foundations]
  roadmap_bucket: 3        # which roadmap topic bucket this belongs to (concepts only; omit if N/A)
  updated: 2026-06-03
  sources: [2026-06-03-anthropic-building-effective-agents]   # raw source slugs this page draws on
  ---
  ```
- **Cross-references:** link liberally with Obsidian-style `[[wikilinks]]`, e.g. `[[context-engineering]]`, `[[claude-code]]`. A link to a page that doesn't exist yet is fine — it marks something worth writing. Prefer many small interlinked pages over few giant ones.
- **Source citations:** when a claim comes from a source, link its source page, e.g. "Agents should be given tools, not just prompts ([[sources/2026-06-03-anthropic-building-effective-agents]])."
- **Page types & where they live:**
  - `sources/<slug>.md` — a summary of one raw source: key takeaways, notable claims, which concepts/entities it touches, links to the pages it updated.
  - `concepts/<slug>.md` — a durable explanation of one idea, synthesized across all sources that mention it. This is where real teaching content accumulates.
  - `entities/<slug>.md` — a tool, framework, model, company, or person (e.g. `claude-code`, `mcp`, `anthropic`).

## Workflows

### Ingest (the main loop of Phase 1)
When Ismet adds a source to `raw/` and says "ingest" (or names a file):
1. Read the source from `knowledge/raw/`.
2. Briefly discuss the key takeaways with Ismet before writing — confirm emphasis and where it fits in the roadmap.
3. Write `wiki/sources/<slug>.md` summarizing it (use the source's date or today's date as a filename prefix when useful).
4. Create or update the relevant `concepts/` and `entities/` pages — integrate the new information, add `[[wikilinks]]`, and **flag contradictions** with anything already in the wiki rather than silently overwriting.
5. Update `wiki/overview.md` and the root `ROADMAP.md` if the source changes the big picture or fills/reorders a level (mark levels that now have content as sourced).
6. Update `index.md` — add new pages, refresh one-line summaries.
7. Append an entry to `log.md`.
- A single source can touch 10–15 pages. Prefer ingesting one source at a time with Ismet involved.

### Query
When Ismet asks a question against the wiki:
1. Read `index.md` first to locate relevant pages, then drill in.
2. Synthesize an answer with citations to source/concept pages.
3. **Offer to file valuable answers back into the wiki** as a new concept page or note — explorations should compound, not vanish into chat.

### Lint
When Ismet asks for a health check:
- Scan for: contradictions between pages, stale claims superseded by newer sources, orphan pages (no inbound links), important concepts mentioned but lacking their own page, missing cross-references, thin/unsourced roadmap buckets.
- Suggest new questions to investigate and new sources to look for. Offer to web-search for gaps.

## index.md and log.md conventions

- **`index.md`** is content-oriented: organized by category (Overview, Roadmap, Concepts, Entities, Sources), each page listed with a `[[wikilink]]` and a one-line summary. It's the first thing to read when answering a query. Keep it current on every ingest.
- **`log.md`** is chronological and append-only. Every entry starts with a greppable prefix:
  `## [YYYY-MM-DD] <op> | <title>` where `<op>` is `ingest` | `query` | `lint` | `init` | `note`.
  So `grep "^## \[" knowledge/wiki/log.md | tail -5` shows recent activity.

## Domain scope — the canonical taxonomy (mirrors `ROADMAP.md`)

**Target persona:** an **AI Native Engineer** for software companies — someone who (1) uses AI coding tools excellently across the SDLC, and (2) builds multi-agent pipelines and AI features. **NOT** an ML engineer / data scientist: we do not cover model training, neural-net math, dataset curation, or training-time MLOps. The model is a black-box tool/API; we teach everything *around* it.

New concepts get filed under one of these **levels** (keep `CLAUDE.md` and the root `ROADMAP.md` in sync). Set `roadmap_bucket` in frontmatter to the level number.

0. **Foundations (just enough, no ML)** — LLM behavior (tokens, context windows, non-determinism, hallucination), the API loop, tool calling, structured output.
1. **Mastering AI coding tools** — Claude Code / Cursor / Copilot / Codex; context files (CLAUDE.md/AGENTS.md); spec-driven development; verification discipline; parallel-agent (worktree/PR-per-agent) workflows.
2. **Building agentic systems** — agent fundamentals; workflow patterns (routing, parallelization, orchestrator-workers, evaluator-optimizer); context engineering; harness engineering; MCP & integrations.
3. **Multi-agent pipelines for the SDLC** — orchestration (coordinator + specialists, task DAGs); plan→code→review→test→deploy→operate pipelines; quality gates; repository-native orchestration; RAG/knowledge for agents.
4. **Production reliability** — evals (offline/online, LLM-as-judge, trace-based); observability & tracing; security & safety (lethal trifecta, prompt injection, sandboxing); governance & code review at scale; cost & latency.
5. **Leading AI-native engineering** — org adoption; human-in-the-loop governance; measuring productivity honestly; fine-tune-vs-prompt-vs-retrieve as a decision.

## Operating principles

- **Claude owns `wiki/`, never touches `raw/`.** Ismet curates sources and asks questions; Claude does the summarizing, cross-referencing, and bookkeeping.
- **The roadmap is a living document** — it is the bridge to Phase 2. Keep it honest about what's sourced vs. still a placeholder.
- **Co-evolve this schema.** When we discover a better convention or workflow, update this `CLAUDE.md`.
