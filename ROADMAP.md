---
title: AI Native Engineering — Roadmap
type: roadmap
tags: [roadmap, curriculum]
updated: 2026-06-03
sources: []
---

# AI Native Engineer — Learning Roadmap

> **Status: research-grounded v2.** Rebuilt from deep research across Anthropic/engineering blogs, Hacker News, GitHub, and practitioner write-ups. Candidate sources live in `SOURCE.md`. Still a draft to react to — reorder and challenge freely.

## Who this produces
An **AI Native Engineer**: a software engineer who (1) **uses AI coding tools excellently** in their own day-to-day SDLC, and (2) **builds multi-agent pipelines and AI features** that make software companies ship better and faster. The job is part *power-user* and part *systems builder*.

## Explicitly NOT in scope
This is **not** an ML engineer / data scientist track. We do **not** teach: model training, neural-net math, dataset curation, MLOps for training, or research-grade fine-tuning. We treat the model as a **black-box tool/API** and focus on everything *around* it. (Fine-tuning appears only as a "when/why to even consider it" decision, not a how-to.)

## The thesis (what real industry signal tells us)
- **2025 was speed; 2026 is quality.** The constraint is no longer "can the agent generate it" — it's "can the team *verify* it." Evals, tracing, review, and governance are where the real work is.
- The best agents aren't the "most agentic" — they're **well-engineered software with LLMs at controlled decision points** (12-Factor Agents, Manus lessons).
- Teams run a **multi-tool coding stack** (Claude Code + Cursor + Codex/Copilot), not one tool.
- **Context engineering > prompt tricks.** Version-controlled context files, spec-driven development, and repository-grounding are the difference between useful and "context-blind" agents.

---

## Level 0 — Foundations (just enough, no ML)
The mental model for treating an LLM as an engineering component.
- How LLMs *behave*: tokens, context windows, non-determinism, why they hallucinate, why output varies.
- The API loop: request/response, streaming, system vs user messages, temperature.
- Tool calling / function calling and structured (JSON) output — the mechanism everything agentic is built on.
- Status: not yet sourced · Concepts: _(none yet)_

## Level 1 — Mastering AI coding tools (AI in *your* SDLC)
Becoming an elite power-user of agentic coding tools. This is the fastest ROI and the daily reality.
- The coding-agent stack — Claude Code, Cursor, GitHub Copilot Agent, Codex; strengths and when to use which; composing 2–3 together.
- Context files for code — `CLAUDE.md` / `AGENTS.md`, repo conventions, version-controlled project context.
- **Spec-driven development** — write the spec, let the agent implement; the disciplined alternative to "vibe coding."
- **Verification discipline** — the senior-vs-junior divide is *how you review AI output*; never blindly trust generated code.
- Parallel-agent workflows — git worktrees, branch-per-agent, PR-per-agent.
- Status: not yet sourced · Concepts: _(none yet)_

## Level 2 — Building agentic systems (core engineering skill)
Going from *using* agents to *building* them.
- Agent fundamentals — the model-in-a-loop with tools, planning, and memory; when an agent vs. a simple workflow.
- **Workflow patterns** (Anthropic) — routing, parallelization, orchestrator-workers, evaluator-optimizer.
- **Context engineering** (the discipline) — token budgeting, compaction, memory, what goes in the window and why.
- **Harness engineering** — the runtime layer: tool dispatch, context compaction, safety enforcement, session persistence, model routing, planning/execution separation.
- **MCP & integrations** — the standard way to connect models to tools, data, and services.
- Status: not yet sourced · Concepts: _(none yet)_

## Level 3 — Multi-agent pipelines for the SDLC (the specialization)
The headline deliverable: agentic pipelines that span the software lifecycle.
- Multi-agent orchestration — coordinator + specialists, task DAGs, hand-offs, parallel execution.
- **SDLC pipeline design** — specialized agents across plan → code → review → test → deploy → operate (Requirement Analyst, Architect, Developer, Reviewer, Tester, Deployer).
- **Quality gates between stages** — making the pipeline reliable, not just impressive in a demo.
- Repository-native orchestration — agents grounded in the actual codebase; context-grounding hooks.
- RAG / knowledge for agents over codebases and docs (incl. the LLM Wiki pattern — see `knowledge/llm_wiki.md`).
- Status: not yet sourced · Concepts: _(none yet)_

## Level 4 — Production reliability (the 2026 bottleneck)
Where "year of quality" lives. Arguably the most valuable part of the curriculum.
- **Evals** — offline + online; LLM-as-judge; trace-based evals; regression suites; defining a quality baseline (Hamel Husain methodology).
- **Observability & tracing** — OpenTelemetry, Langfuse/LangSmith; detecting silent drift; correlating LLM traces with the backend.
- **Security & safety** — the **lethal trifecta** (private data + untrusted content + exfiltration), prompt injection, sandboxing, CI security gates; agents writing PRs at scale multiply vulnerabilities.
- **Governance & code review at scale** — independent quality gates, automated enforcement, AI-assisted review when agents author most PRs.
- **Cost & latency** — prompt caching, model routing, batching, token economics.
- Status: not yet sourced · Concepts: _(none yet)_

## Level 5 — Leading AI-native engineering (org layer)
For taking a whole team/company AI-native.
- Adopting AI across an engineering org; humans **govern, orchestrate, and validate** while agents execute.
- Building a verification culture and human-in-the-loop governance.
- Measuring productivity honestly (beyond demo magic).
- When to fine-tune vs. prompt vs. retrieve (a *decision*, not a training course).
- Status: not yet sourced · Concepts: _(none yet)_

---

## Suggested learner path
**0 → 1** for everyone first (foundations + tool mastery = immediate value). **2 → 3** is the builder core (agentic systems → SDLC pipelines). **4** runs *in parallel* with 2–3, not after — reliability is a habit, not a final step. **5** is for leads/architects.

## Open questions to resolve as we source
- Is Level 1 (tool mastery) a prerequisite for Level 2, or a parallel track for a different audience (every dev vs. AI-platform engineers)?
- How much of Level 0 is truly needed before someone is productive?
- What is the single capstone project that proves competence — an end-to-end multi-agent SDLC pipeline with evals + tracing + quality gates?
