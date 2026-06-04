---
title: Capstone — Agentic SDLC Pipeline
type: roadmap
tags: [capstone, project, sdlc, multi-agent]
updated: 2026-06-04
sources: []
---

# Capstone — An Agentic Multi-Agent SDLC Pipeline

> One project that proves AI-native competence end-to-end. The per-level **▶ Build this** projects in
> [`READING-ROADMAP.md`](READING-ROADMAP.md) are pieces of this — by the time you finish Level 4 you've
> built it. Passing the **Definition of Done** below is the "industry-ready" bar.

## The thing you build
A pipeline where **specialized agents take a written spec and ship a small but real service**
(suggested: a small REST API with a database and 2–3 endpoints, or a CLI tool with persistence —
something with genuine edge cases, not a toy). The agents span **plan → code → review → test → deploy**,
with a **human-in-the-loop gate** and **automated quality gates** between stages, all **traced and
evaluated**.

This deliberately mirrors the project's own thesis: *well-engineered software with LLMs at controlled
decision points* — not a fully-autonomous swarm.

## Architecture (target)
```
spec.md ──▶ [Planner] ──▶ plan + tasks
                 │
                 ▼
            [Coder agent] ──(worktree/branch)──▶ implementation + tests
                 │
                 ▼
            [Reviewer agent] ──▶ critique  ──reject──▶ loop back to Coder   ◀── quality gate
                 │ approve
                 ▼
            CI: eval suite + lint + tests + security red-team  ◀── quality gate (independent)
                 │ pass
                 ▼
            Human approval ──▶ merge/deploy
                 │
                 ▼
            Tracing + metrics (every agent call instrumented)
```

## Milestones (mapped to levels)
| Milestone | From level | Output |
|---|---|---|
| **M0 — Agent primitive** | L0 | A working API loop with one tool + schema-enforced output. |
| **M1 — Disciplined human run** | L1 | The same feature shipped by hand via spec → CLAUDE.md/AGENTS.md → writer/reviewer worktrees → verified merge. Proves the workflow before you automate it. |
| **M2 — Tools & inner loops** | L2 | An MCP server wrapping a real tool; one agent applying routing + evaluator-optimizer patterns. |
| **M3 — The pipeline** | L3 | Planner → Coder → Reviewer agents wired together, repo-native, with a reject/retry gate between stages; opens a PR from a spec. |
| **M4 — Made shippable** | L4 | Eval suite (LLM-as-judge + regression) as a **CI gate**, OpenTelemetry tracing on all agent calls, Promptfoo lethal-trifecta red-team, sandboxed execution for agent actions. |
| **M5 — Governed (optional)** | L5 | A one-page adoption/governance brief + honest metrics (durability/churn, not just throughput) for running this with a team. |

## Definition of Done (the industry-ready bar)
- [ ] A change goes **spec → merged PR** driven by agents, with **no stage that blindly trusts the previous one**.
- [ ] The **reviewer agent can reject** and send work back — the gate is real, not cosmetic.
- [ ] **Independent quality gate**: at least one automated check (evals/lint/tests/security) runs in CI and can **block merge** — and it is *not* the same model/agent that wrote the code (per Codacy).
- [ ] An **eval suite** exists with ≥1 LLM-as-judge eval and a small **regression set**; it runs on every PR.
- [ ] **Every agent call is traced** (OpenTelemetry or a tracing platform); you can reconstruct a run.
- [ ] You ran a **lethal-trifecta red-team** (Promptfoo) and can state how the system resists prompt injection / exfiltration; untrusted agent actions are **sandboxed**.
- [ ] A **human gate** exists before deploy; the spec documents what's auto vs. human-approved.
- [ ] You can explain the **cost/latency** profile (where prompt caching / model routing applies).
- [ ] You can articulate **honest metrics** for the pipeline's value — beyond "it generated code fast."

## Stretch goals
- Parallelize multiple coder agents (worktree/branch/PR-per-agent, à la ComposioHQ) with conflict handling.
- Add a RAG/knowledge step so agents are grounded in the actual codebase + docs (tie to the
  `knowledge/llm_wiki.md` pattern this very repo uses).
- Add an evaluator-optimizer loop that auto-improves a weak agent against the eval suite.

## How we'll run it
We build it incrementally as you clear each level — I keep [`PROGRESS-TRACKER.md`](PROGRESS-TRACKER.md)
current so the next milestone is always the obvious next step, and we adjust scope to your pace.
