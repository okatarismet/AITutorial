---
title: AI Native Engineer — Reading Roadmap
type: roadmap
tags: [roadmap, reading-list, curriculum]
updated: 2026-06-04
sources: []
---

# AI Native Engineer — Reading Roadmap (v2, sequenced curriculum)

> **What this is.** A teachable, ordered path to becoming an industry-ready AI Native Engineer —
> someone who (1) uses AI coding tools excellently across their SDLC and (2) builds agentic systems
> and multi-agent pipelines. Not an ML track (no training/math). Every source below was audited for
> credibility, currency, and fit on **2026-06-04**; broken/gated ones are demoted with a note, and
> gap-fills are inserted where the original list was thin.

## How to use this doc

- **Learn-mode tags** on every item:
  - `[read]` — read it; build the mental model.
  - `[lab]` — a short (15–60 min) hands-on exercise to make the idea concrete.
  - `[project]` — a multi-hour build that produces a real artifact.
- Each level ends with a **▶ Build this** project. The level projects **chain into one capstone** —
  an end-to-end agentic SDLC pipeline. See [`CAPSTONE.md`](CAPSTONE.md).
- I track where you actually are (not just what's been read) in [`PROGRESS-TRACKER.md`](PROGRESS-TRACKER.md).
  That's the "mental model" — it's how the curriculum adapts and fills gaps instead of being a static list.
- `⭐` = canonical / do-not-skip.

## Calibration — your starting point

You've **built simple agents/RAG** and want **production depth**. So:
- **Levels 0–1: fast-track.** Skim the parts you know; do the `[lab]`s only to catch blind spots
  (the *mechanics* most self-taught builders are fuzzy on: tokenizer economics, the exact tool-use
  loop, grammar-enforced structured output, verification discipline). Don't linger.
- **Levels 2–4: the real work.** Go deep — agent/harness engineering, multi-agent SDLC pipelines,
  and production reliability (evals, tracing, security, governance). This is where industry-readiness
  is won.
- **Level 5: build it out.** Mostly new sources; read for the leadership/measurement lens.
- Suggested order: **0→1 (fast) → 2 → 3, running 4 in parallel from the start → 5.**

---

## Level 0 — Foundations (just enough, no ML) · *fast-track*
*Treat the LLM as a black-box engineering component. You likely know most of this — the goal here is
to close mechanics gaps, not re-teach basics.*

1. X `[read]` ⭐ **Chip Huyen — Agents** — best single mental map of the whole landscape (tools, planning, evaluation). https://huyenchip.com/2025/01/07/agents.html
2. `[lab]` ⭐ **Anthropic — Interactive Prompt Engineering Tutorial** — skim if fluent; do the chain-of-thought + hallucination-avoidance chapters. https://github.com/anthropics/prompt-eng-interactive-tutorial

**Mechanics block (gap-fill — the part the original list was missing):**
3. `[read]` **Tokens & context windows** — how text becomes tokens, why context is a budget, tokenizer differences. Anthropic token counting / context windows docs: https://platform.claude.com/docs/en/build-with-claude/token-counting
4. `[read]` **The API loop + tool use** — request → `stop_reason: tool_use` → you execute → return result → repeat. This loop *is* every agent. https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
5. `[read]` **Structured output ≠ "please return JSON"** — schema-enforced (grammar-constrained) generation vs. hoping the model complies. https://platform.claude.com/docs/en/build-with-claude/structured-outputs
6. `[read]` **Why outputs vary / why models hallucinate** — sampling/non-determinism and "lost in the middle" as context fills. Pair with the hallucination-avoidance chapter from #2.
`
*Reference (don't read linearly):* OpenAI Prompt Engineering Guide (cross-provider contrast) · Chip Huyen **aie-book** repo/book.

> **▶ Build this (L0):** A ~40-line script that calls the Messages API in a loop, defines **one tool**
> (e.g. a calculator or file-reader), handles the `tool_use` stop reason, and forces a **JSON-schema**
> response. Deliberately break the schema and watch enforcement catch it. *(This loop is the seed of
> the capstone's agents.)*

---

## Level 1 — Mastering AI coding tools · *fast-track, high ROI*
*Become an elite power-user of agentic coding tools across your own SDLC.*

1. `[read]` ⭐ **Anthropic — Claude Code: Best practices for agentic coding** — the core playbook: CLAUDE.md, explore-plan-implement-commit, subagents, hooks, MCP, verification loops, parallel sessions. https://www.anthropic.com/engineering/claude-code-best-practices
2. `[read]` ⭐ **From Vibe Coding to Spec-Driven Development** — spec first, agent implements; the disciplined alternative to vibe coding. https://towardsdatascience.com/from-vibe-coding-to-spec-driven-development/
3. `[read]` **Addy Osmani — How to write a good spec for AI agents** *(gap-fill — the deep "how")* — the five principles + the three-tier Always/Ask/Never boundary. https://addyosmani.com/blog/good-spec/
4. `[read]` **Spec-Driven Development: The Waterfall Strikes Back (HN debate)** — the skeptical counterweight; read right after #2–3. https://news.ycombinator.com/item?id=45935763
5. `[read]` **AGENTS.md — the open standard** *(gap-fill)* — the vendor-neutral "README for agents," used by 60k+ projects; how it relates to CLAUDE.md. https://agents.md/
6. `[lab]` **Git-worktree parallel agents** *(gap-fill)* — run isolated agent sessions that don't collide; the writer/reviewer pattern. https://code.claude.com/docs/en/worktrees
7. `[read]` **Verification discipline** *(gap-fill — the senior/junior divide)* — never blind-trust generated code; tests/builds/review as the feedback loop. Covered tactically in #1; reinforce with a testing checklist for AI-generated code.
8. *Reference:* **Coding Agents Comparison (Artificial Analysis)** — data-driven tool selection. https://artificialanalysis.ai/agents/coding

> **Demoted (audit):** *The New Stack — "Cursor/Claude Code/Codex merging into one stack"* (paywalled) and
> *daily.dev — "How Senior Engineers Actually Build With AI in 2026"* (needs registration). The 2026
> multi-tool-stack point survives via #8 and #1; the verification point via #7.

> **▶ Build this (L1):** Take one real feature through the full discipline: write a **spec** → set up
> **CLAUDE.md + AGENTS.md** → implement it with a **writer agent in one worktree** while a **reviewer
> agent in another** critiques → verify (tests/build) → merge. *(This is the human-driven dry-run of
> the capstone pipeline.)*

---

## Level 2 — Building agentic systems · *go deep*
*Go from using agents to building them — patterns, context engineering, harness, MCP.*

1. `[read]` ⭐ **Anthropic — Building Effective Agents** — the reference patterns: prompt chaining, routing, parallelization, orchestrator-workers, evaluator-optimizer; "tool design as an API." https://www.anthropic.com/engineering/building-effective-agents
2. `[read]` ⭐ **12-Factor Agents (HumanLayer)** — the thesis: the best agents are well-engineered software with LLMs at controlled decision points, not "most agentic." https://github.com/humanlayer/12-factor-agents
3. `[read]` ⭐ **Anthropic — Effective Context Engineering for AI Agents** — context as the discipline beyond prompting: budgeting, compaction, just-in-time retrieval, context rot. https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
4. `[read]` ⭐ **Context Engineering: Lessons from Building Manus** — battle-scars companion to #3: KV-cache economics, mask-don't-remove tools, filesystem-as-context, keep errors in context. https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
5. `[read]` ⭐ **Building Effective AI Coding Agents for the Terminal (arXiv 2603.05344, OPENDEV)** — the deep dive on **harness engineering**: scaffolding vs harness, adaptive compaction, safety invariants, model routing, dual-agent planning/execution. *(Verified real — Nghi D. Q. Bui.)* https://arxiv.org/abs/2603.05344
6. `[read]` ⭐ **MCP — Architecture overview (official docs)** — host/client/server, JSON-RPC, transports, resources/tools/prompts primitives. https://modelcontextprotocol.io/docs/learn/architecture
7. `[read]` **Neo4j — Getting Started with MCP Servers** — hands-on server build to ground #6. https://neo4j.com/blog/developer/model-context-protocol/
8. `[read]` **Agent memory & state patterns** *(gap-fill)* — episodic/semantic/procedural memory, store/retrieve/update/summarize; when filesystem-as-memory beats a vector DB. (Mem0 "State of AI Agent Memory" survey or equivalent.)

> **▶ Build this (L2):** **Build & test your own MCP server** wrapping a real tool (your repo's
> linter, build, or an internal API), and connect it to Claude Code. Then implement **one agent loop**
> that uses it, applying at least the **routing** and **evaluator-optimizer** patterns from #1.
> *(These become the capstone's "tools" and inner loops.)*

---

## Level 3 — Multi-agent pipelines for the SDLC · *go deep (the headline)*
*Agentic pipelines spanning plan → code → review → test → deploy → operate.*

1. `[read]` ⭐ **AI-Native SDLC: How We Ship Software with AI Agents (Joyal Saji)** — first-person account of specialized agents across the lifecycle; humans govern/orchestrate/validate. https://medium.com/@joyalsaji/ai-native-sdlc-how-we-ship-software-with-ai-agents-ce17ade0e2ee
2. `[read]` ⭐ **CodeRabbit — Agentic SDLC** — agents at every stage; the "confidence gap," review infrastructure as the critical gate. https://www.coderabbit.ai/guides/agentic-sdlc
3. `[read]` ⭐ **How Squad runs coordinated AI agents inside your repository (GitHub)** — *the* reference for repo-native coordinator + specialists; shared memory via charter + history. https://github.blog/ai-and-ml/github-copilot/how-squad-runs-coordinated-ai-agents-inside-your-repository/
4. `[read]` **Xebia — 2026: The Year Software Engineering Becomes AI Native** — macro framing + the "adding AI to broken processes yields faster broken processes" warning. https://xebia.com/news/2026-the-year-software-engineering-will-become-ai-native/
5. `[read/project]` **GitHub `spec-kit`** *(gap-fill — a studyable real pipeline)* — open-source spec-driven toolkit (constitution→spec→plan→tasks→implement) integrating 30+ agents. Read the flow, then run it. https://github.com/github/spec-kit
6. `[read]` **ComposioHQ/agent-orchestrator** — parallel agents with worktree/branch/PR-per-agent, autonomous CI fixes. https://github.com/ComposioHQ/agent-orchestrator
7. `[read]` **Evaluator-optimizer / reflect-refine loops** *(gap-fill)* — generation → evaluation → refinement as a control loop. AWS prescriptive guidance: https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/evaluator-optimizer.html
8. `[read]` **From RAG to Context (RAGFlow 2025 review)** + **Production RAG (Towards AI)** — where RAG went, and why ~80% of failures are ingestion/chunking, not the model. https://ragflow.io/blog/rag-review-2025-from-rag-to-context · https://towardsai.net/p/machine-learning/production-rag-the-chunking-retrieval-and-evaluation-strategies-that-actually-work
9. `[read]` **Agentic RAG vs classic RAG** *(gap-fill)* — fixed pipeline vs. retrieval inside a control loop; when each is worth it.

> *Reference (lookup, not linear read):* **awesome-agent-orchestrators** — survey the framework landscape. https://github.com/andyrewlee/awesome-agent-orchestrators

> **▶ Build this (L3):** A **3-agent repo-native pipeline** — planner → coder → reviewer — that takes
> a spec and opens a PR, with a **quality gate between stages** (the reviewer can reject and loop back).
> *(This is the spine of the capstone.)*

---

## Level 4 — Production reliability · *go deep, run in parallel with 2–3*
*The 2026 bottleneck: evals, tracing, security, governance, cost. Reliability is a habit, not a final step.*

**Evals**
1. `[read]` ⭐ **Hamel Husain — Your AI Product Needs Evals** — foundational; evals matter more than model choice. https://hamel.dev/blog/posts/evals/
2. `[read]` **Hamel Husain — A Field Guide to Rapidly Improving AI Products** — error-analysis playbook from 30+ companies. https://hamel.dev/blog/posts/field-guide/
3. `[read]` **Hamel Husain — Using LLM-as-a-Judge** — the common method, done right (binary judgments, expert alignment). https://hamel.dev/blog/posts/llm-judge/index.html
4. `[read]` **Anthropic — Demystifying Evals for AI Agents** — single vs multi-turn, grader types, pass@k / pass^k, "start with 20–50 tasks." https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
5. `[read]` **AWS — Evaluating AI Agents** — trace-based evals, silent drift, why outcome-only evals fail. https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/

**Observability**
6. `[lab]` **OpenTelemetry — Observability for LLM applications** — vendor-neutral tracing; correlate LLM traces with the backend. https://opentelemetry.io/blog/2024/llm-observability/
7. `[read]` **Langfuse vs LangSmith vs OpenTelemetry** — cost/lock-in trade-offs; pick one and instrument it.

**Security & governance**
8. `[read]` ⭐ **Simon Willison — The Lethal Trifecta** — private data + untrusted content + exfiltration = structural prompt-injection risk. https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
9. `[lab]` **Testing the Lethal Trifecta with Promptfoo** — red-team it for real. https://www.promptfoo.dev/blog/lethal-trifecta-testing/
10. `[read]` **OWASP — LLM Prompt Injection Prevention Cheat Sheet** *(gap-fill)* — 13 attack categories + concrete defenses and an ops checklist. https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html
11. `[read]` **Agent sandboxing / execution isolation** *(gap-fill)* — why containers aren't enough; microVMs/gVisor for untrusted agent actions. https://northflank.com/blog/how-to-sandbox-ai-agents
12. `[read]` ⭐ **Codacy — Why Coding Agents Need Independent Quality Gates** — verification as a governance problem; "the system that generates code shouldn't be its own acceptance gate." https://blog.codacy.com/why-coding-agents-need-independent-quality-gates
13. `[read]` **Codebridge — How to Ship Secure AI-Generated Code in 2026** — reviews, sandboxing policy, CI gates as a governance model. https://www.codebridge.tech/articles/how-to-ship-ai-generated-code-securely-a-governance-model-for-reviews-sandboxing-policies-and-ci-gates

**Cost & latency** *(gap-fill)*
14. `[read]` **Prompt caching & cost economics** — cached-prefix discounts, KV-cache mechanics, model routing. Anthropic prompt caching docs: https://platform.claude.com/docs/en/build-with-claude/prompt-caching

> **▶ Build this (L4):** Wrap the L3 pipeline with reliability: an **eval suite** (LLM-as-judge +
> a small regression set) wired into **CI as a gate**, **OpenTelemetry tracing** on every agent call,
> and a **Promptfoo lethal-trifecta red-team** run. *(This is what turns the capstone from a demo into
> something shippable.)*

---

## Level 5 — Leading AI-native engineering · *build it out*
*Take a whole team/org AI-native. Mostly new sources (the original list had none of its own here).*

1. `[read]` ⭐ **Honest productivity measurement** *(gap-fill)* — why DORA inflates under AI; track **code durability / churn / bugs**, not just throughput.
   - Faros — *The AI Acceleration Whiplash* (throughput +66% but churn +861%, bugs +54%): https://www.faros.ai/blog/ai-acceleration-whiplash-takeaways
   - GitClear — *AI Copilot Code Quality 2025* (duplication up 4×, refactoring collapsing): https://www.gitclear.com/ai_assistant_code_quality_2025_research
2. `[lab]` **Fine-tune vs RAG vs prompt — a decision, not a course** *(gap-fill)* — prompt/long-context first, RAG for freshness/scale, fine-tune only for learned behavior; the cost/latency timeline of each. Score your own features against the three.
3. `[read]` **Org adoption & human-in-the-loop governance** *(gap-fill)* — humans govern/orchestrate/validate; governance maturity, escalation rules, regulatory context (EU AI Act, Aug 2026).
   - EY — *Defining a CIO Playbook on Agentic AI*: https://www.ey.com/en_us/ey-center-for-executive-leadership/defining-a-cio-playbook-on-agentic-ai
   - Stanford Digital Economy — *The Enterprise AI Playbook (51 deployments)*: https://digitaleconomy.stanford.edu/
4. `[read]` *Re-read through a leadership lens:* **Joyal Saji — AI-Native SDLC** and **Xebia — 2026** (both in L3).

> **▶ Build this (L5):** A one-page **adoption + governance brief** for a hypothetical team: which
> work is agent-delegated vs human-gated, the escalation/approval rules, and the **honest metrics**
> you'd track (durability/churn alongside throughput). Plus your fine-tune/RAG/prompt decision table.

---

## The capstone (where it all converges)
All level projects feed one deliverable: **an agentic multi-agent SDLC pipeline that takes a spec and
ships a small service**, with evals + tracing + independent quality gates. Full spec and
definition-of-done in [`CAPSTONE.md`](CAPSTONE.md). Passing its checklist *is* the industry-ready bar.

## Coverage notes (post-audit, 2026-06-04)
- **Every level now has ≥1 validated source and a learn-mode tag; no orphan topics.**
- **Best-sourced:** Levels 2 and 4. **Most-improved:** Level 0 (mechanics block added) and Level 5
  (built from near-zero).
- **Demoted/replaced:** The New Stack (paywall), daily.dev senior-engineers (registration).
- **Link hygiene:** canonical Anthropic docs live at `platform.claude.com/docs/...`. A few secondary
  gap-fill links (memory survey, agentic-RAG, productivity research) are best-effort third-party blogs —
  swap freely if one rots; the canonical anchors (Anthropic, MCP, OWASP, arXiv, GitHub) are stable.

## Maintenance
This is a living doc. As you progress, I update [`PROGRESS-TRACKER.md`](PROGRESS-TRACKER.md) with what's
actually mastered and what gaps surfaced, and reorder/insert here accordingly. Sources still get the
full wiki treatment per `CLAUDE.md` when/if we ingest them into `knowledge/wiki/`.
