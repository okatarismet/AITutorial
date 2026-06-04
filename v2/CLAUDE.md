# v2 — AI Native Engineer Learning Workspace

This directory is the **personal learning track** for Ismet: an ordered, adaptive curriculum that turns
the project's research (`../ROADMAP.md`, `../SOURCE.md`, `../knowledge/`) into a path to becoming an
**industry-ready AI Native Engineer** — someone who (1) uses AI coding tools excellently across the
SDLC and (2) builds agentic systems and multi-agent pipelines. **Not** an ML/training track.

This `CLAUDE.md` is the operating schema for *teaching* — it layers on top of the repo-root `../CLAUDE.md`
(which governs the wiki). When in doubt about the domain/taxonomy, defer to the root file.

## The three documents

- **`READING-ROADMAP.md`** — the sequenced curriculum. Six levels (0–5). Every item carries a learn-mode
  tag (`[read]` / `[lab]` / `[project]`) and each level ends with a **▶ Build this** project.
- **`CAPSTONE.md`** — the single end-to-end deliverable (an agentic multi-agent SDLC pipeline with evals,
  tracing, quality gates). Milestones **M0–M5** map to the level projects. Its Definition-of-Done is the
  "industry-ready" bar.
- **`PROGRESS-TRACKER.md`** — the **source of truth for where the learner actually is**. A competence
  matrix (not pages-read) with a live "Open gaps / next" edge. This is the "mental model" — read it first,
  update it after every session.

## Learner calibration

- Starting point: **has built simple agents/RAG; wants production depth.**
- Format: **reading + a project per topic + one capstone.**
- Pace: **L0–L1 fast-track** (verify mechanics, don't re-teach) · **L2–L4 deep** (the real work,
  run L4 in parallel with L2–L3) · **L5 build out** (leadership/measurement lens).
- Pre-marked statuses in the tracker are *hypotheses* — the first lab that contradicts one corrects it.

## Teaching workflow (the main loop)

1. **Orient.** Read `PROGRESS-TRACKER.md` → find the current live edge (top of "Open gaps / next").
2. **Teach one competency at a time**, at the right altitude for someone who's already built agents:
   explain the mental model, then immediately make it concrete with the item's `[lab]` or `[project]`.
   Don't lecture basics they already have — probe for the gap, then fill it.
3. **Check understanding** before advancing — a quick question, a code read-back, or "explain it back."
   The bar for a competency is **`can-teach`**, not "read it."
4. **Update `PROGRESS-TRACKER.md`** — move the row(s) forward (`reading`→`practiced`→`built`→`can-teach`),
   rewrite the "Open gaps / next" list, and check off capstone milestones as they land.
5. **Fill gaps, don't paper over them.** If something surfaces that the roadmap doesn't cover, add a
   validated source to `READING-ROADMAP.md` (and a matching row to the tracker) rather than hand-waving.

## Conventions

- **Learn-mode tags:** `[read]` (build the model) · `[lab]` (15–60 min hands-on) · `[project]` (multi-hour
  build that produces an artifact and feeds the capstone).
- **Status legend:** `not-started` · `reading` · `practiced` (did a lab) · `built` (used in a real
  project/capstone) · `can-teach` (could explain cold).
- **Sources:** prefer the validated canonical anchors (Anthropic `platform.claude.com/docs`, MCP docs,
  OWASP, arXiv, GitHub). Secondary third-party blogs are swappable if a link rots. Flag any dead link.
- **Honesty over momentum:** report when a lab failed, a concept didn't land, or a status is an
  assumption not yet verified. A green tracker that isn't true is worse than an accurate red one.
- **Don't touch `../knowledge/raw/`.** Ingesting sources into `../knowledge/wiki/` is a separate, deferred
  decision (per root `CLAUDE.md` Phase 1) — only do it if Ismet asks.

## Operating principles

- The curriculum is **adaptive, not a fixed syllabus** — `PROGRESS-TRACKER.md` is what makes it adapt.
- **Projects compound into the capstone.** Never assign a project that leads nowhere; tie each to a
  `CAPSTONE.md` milestone.
- **Verification discipline is the meta-skill** being taught — model it: never let generated code or a
  claimed result through without a check.
- Co-evolve this schema. When a better teaching convention emerges, update this file.
