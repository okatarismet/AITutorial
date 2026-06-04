---
title: Progress Tracker — Learner Mental Model
type: roadmap
tags: [progress, tracker, mental-model]
updated: 2026-06-04
sources: []
---

# Progress Tracker — Your Mental Model

> This is how I "see" what you're learning so the curriculum adapts instead of being a static reading
> list. It tracks **competence**, not pages read. I update it as we go; the **Open gaps / next** list at
> the bottom is always the live edge.

## Status legend
`not-started` · `reading` (consuming the material) · `practiced` (did a lab) · `built` (used it in a real
project/the capstone) · `can-teach` (could explain it to someone else cold — the real bar).

## Calibration (2026-06-04)
Self-reported: **built simple agents / RAG**, wants production depth. So L0–L1 fundamentals are
pre-marked `practiced` (assumed — we'll spot-check with labs, not re-teach), and the genuine work is
L2–L5. Anything pre-marked is a *hypothesis*; the first lab that contradicts it gets corrected here.

## Skill matrix

### Level 0 — Foundations *(fast-track; verify, don't re-teach)*
| Competency | Status | Notes |
|---|---|---|
| LLM mental model (tokens, context, planning) | practiced | assumed from prior agent work |
| Prompting fundamentals | practiced | confirm CoT / structured prompting depth |
| **Tokenizer & context-window economics** | not-started | common blind spot — verify via L0 lab |
| **The exact tool-use loop** (`stop_reason: tool_use`) | not-started | the seed of everything agentic |
| **Schema-enforced structured output** (vs "please return JSON") | not-started | likely a gap — verify |
| Non-determinism / hallucination mechanics | reading | knows *that*, maybe not *why* |

### Level 1 — Mastering AI coding tools *(fast-track, high ROI)*
| Competency | Status | Notes |
|---|---|---|
| Claude Code / agentic-tool power use | practiced | depth unknown — hooks/subagents/MCP? |
| Context files (CLAUDE.md / AGENTS.md) | reading | AGENTS.md standard likely new |
| Spec-driven development | not-started | high ROI; the L1 project proves it |
| **Verification discipline** | not-started | the senior/junior divide — emphasize |
| Parallel-agent / git-worktree workflows | not-started | likely new |

### Level 2 — Building agentic systems *(go deep)*
| Competency | Status | Notes |
|---|---|---|
| Anthropic workflow patterns (routing, parallel, orchestrator-workers, evaluator-optimizer) | not-started | core builder vocabulary |
| 12-Factor / "controlled decision points" thesis | not-started | mindset shift from "more agentic" |
| Context engineering (budget, compaction, JIT retrieval) | not-started | knows RAG; this is broader |
| Harness engineering (OPENDEV lessons) | not-started | the deep differentiator |
| MCP (protocol + build a server) | not-started | L2 project: ship one |
| Agent memory/state patterns | not-started | |

### Level 3 — Multi-agent SDLC pipelines *(go deep — the headline)*
| Competency | Status | Notes |
|---|---|---|
| Multi-agent orchestration (coordinator + specialists) | not-started | |
| SDLC pipeline design (plan→code→review→test→deploy) | not-started | the capstone spine |
| Quality gates between stages | not-started | demo→reliable divide |
| Repo-native orchestration (Squad pattern) | not-started | |
| RAG for agents (incl. agentic vs classic) | practiced | has built RAG — level it up to agentic |

### Level 4 — Production reliability *(go deep, in parallel)*
| Competency | Status | Notes |
|---|---|---|
| Evals (offline/online, error analysis) | not-started | Hamel methodology — highest value |
| LLM-as-judge | not-started | |
| Trace-based evals / silent drift | not-started | |
| Observability & tracing (OpenTelemetry) | not-started | L4 lab |
| Lethal trifecta / prompt injection | not-started | |
| Red-teaming (Promptfoo) + OWASP defenses | not-started | |
| Independent quality gates / governance | not-started | |
| Sandboxing untrusted agent actions | not-started | |
| Cost & latency (prompt caching, routing) | not-started | |

### Level 5 — Leading AI-native engineering *(build it out)*
| Competency | Status | Notes |
|---|---|---|
| Honest productivity measurement (durability/churn, not throughput) | not-started | |
| Fine-tune vs RAG vs prompt decision | not-started | a decision skill, not training |
| Org adoption & human-in-the-loop governance | not-started | |

## Capstone progress
See [`CAPSTONE.md`](CAPSTONE.md) milestones M0–M5.
- [ ] M0 — Agent primitive (L0)
- [ ] M1 — Disciplined human run (L1)
- [ ] M2 — Tools & inner loops (L2)
- [ ] M3 — The pipeline (L3)
- [ ] M4 — Made shippable (L4)
- [ ] M5 — Governed (L5, optional)

## Open gaps / next (the live edge)
1. **Start L0 mechanics labs** — confirm or correct the three `not-started` mechanics rows (tokenizer
   economics, tool-use loop, schema-enforced output). This recalibrates everything above.
2. Decide pace: how many hours/week, so I can size each level's reading + project realistically.
3. (Deferred) Whether to **ingest** these sources into `knowledge/wiki/` per `CLAUDE.md` Phase 1, so the
   wiki becomes the durable teaching substrate — currently out of scope by your choice.

> **How this updates:** after each lab/project I move the relevant rows forward (`practiced`/`built`/
> `can-teach`), rewrite this list, and if a gap surfaces that the roadmap doesn't cover, I add a source
> to [`READING-ROADMAP.md`](READING-ROADMAP.md) and a row here.
