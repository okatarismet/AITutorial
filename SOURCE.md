# Source Candidates — AI Native Engineering

A curated, industry-grounded reading list to feed the wiki (`knowledge/wiki/`). These are pieces real teams cite. Canonical/must-read items marked ⭐. HN threads listed where the practitioner debate is itself valuable.

Status legend: `[ ]` not yet ingested · `[x]` ingested into wiki · `[~]` queued

---

## Foundations & the discipline — roadmap buckets 1, overview

- [ ] ⭐ **Chip Huyen — Agents (adapted from *AI Engineering*, 2025)** — overview of agents, tools, planning, and evaluating agents. From the author of the definitive AI engineering book.
  https://huyenchip.com/2025/01/07/agents.html
- [ ] **Chip Huyen — aie-book (resources for AI engineers)** — companion repo to *AI Engineering*; curated resource list across the whole lifecycle.
  https://github.com/chiphuyen/aie-book

## Prompting & prompt engineering — roadmap bucket 2

- [ ] ⭐ **Anthropic — Interactive Prompt Engineering Tutorial** — hands-on, the best structured intro; the five-step ladder (be clear → examples → chain-of-thought → …).
  https://github.com/anthropics/prompt-eng-interactive-tutorial
- [ ] **OpenAI — Prompt Engineering Guide** — system messages, role assignment, formatting for GPT-class models; pin to model snapshots in production.
  https://developers.openai.com/api/docs/guides/prompt-engineering

## Context engineering — roadmap bucket 3

- [ ] ⭐ **Anthropic — Effective Context Engineering for AI Agents** — context engineering as the discipline beyond prompting; token budgeting, compaction, tool-cost evaluation.
  https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  HN thread: https://news.ycombinator.com/item?id=45418251

## Agents & multi-agent — roadmap buckets 5, 9

- [ ] ⭐ **Anthropic — Building Effective Agents** — the reference; production patterns (routing, parallelization, orchestrator-workers, evaluator-optimizer). Real deployments: Coinbase, Intercom, Thomson Reuters.
  https://www.anthropic.com/engineering/building-effective-agents
- [ ] ⭐ **12-Factor Agents (HumanLayer)** — analyzed 100+ production agents; best agents aren't the "most agentic," they're well-engineered software with LLMs at controlled decision points.
  https://github.com/humanlayer/12-factor-agents
  HN debate: https://news.ycombinator.com/item?id=43699271
- [ ] ⭐ **Context Engineering: Lessons from Building Manus** — battle scars from shipping a real agent: KV-cache economics, masking tools vs removing them, file system as context.
  https://news.ycombinator.com/item?id=44635141

## AI-assisted software development — roadmap bucket 8

- [ ] ⭐ **Anthropic — Claude Code: Best practices for agentic coding** — how to actually work with an agentic coding tool (CLAUDE.md, subagents, hooks, MCP).
  https://www.anthropic.com/engineering/claude-code-best-practices
- [ ] **The New Stack — Cursor, Claude Code, and Codex are merging into one AI coding stack** — the 2026 reality: composition over consolidation; teams run 2–3 tools together.
  https://thenewstack.io/ai-coding-tool-stack/
- [ ] **Coding Agents Comparison (Artificial Analysis)** — data-driven comparison of Cursor, Claude Code, Copilot, Codex, Aider.
  https://artificialanalysis.ai/agents/coding

## RAG & knowledge systems — roadmap bucket 6

- [ ] **From RAG to Context — 2025 year-end review (RAGFlow)** — where RAG actually went; honest about what fell out of favor.
  https://ragflow.io/blog/rag-review-2025-from-rag-to-context
- [ ] **Production RAG: strategies that actually work — Towards AI** — ~80% of RAG failures trace to ingestion/chunking, not the model; chunk sizes, hybrid search, reranking trade-offs.
  https://towardsai.net/p/machine-learning/production-rag-the-chunking-retrieval-and-evaluation-strategies-that-actually-work

## Evals — roadmap bucket 7 (Hamel Husain = de facto authority)

- [ ] ⭐ **Your AI Product Needs Evals — Hamel Husain** — foundational; why evals matter more than model choice.
  https://hamel.dev/blog/posts/evals/
- [ ] **A Field Guide to Rapidly Improving AI Products — Hamel Husain** — playbook from helping 30+ companies.
  https://hamel.dev/blog/posts/field-guide/
- [ ] **Using LLM-as-a-Judge for Evaluation — Hamel Husain** — practical guide to the most common eval method.
  https://hamel.dev/blog/posts/llm-judge/index.html
- [ ] **AWS — Evaluating AI Agents: lessons from building agentic systems at Amazon** — trace-based evals, silent drift, why final-outcome evals aren't enough.
  https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/
- [ ] **Anthropic — Demystifying Evals for AI Agents** — how to think about agent evaluation.
  https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents

## Tracing & observability — roadmap buckets 7, 12

- [ ] **OpenTelemetry — Observability for LLM applications** — vendor-neutral standard; correlate LLM traces with the rest of the backend.
  https://opentelemetry.io/blog/2024/llm-observability/
- [ ] **Langfuse vs LangSmith vs OpenTelemetry comparison** — real cost/lock-in trade-offs (the "$1,200/month after a traffic surge" story).
  https://explore.n1n.ai/blog/llm-observability-langfuse-langsmith-opentelemetry-2026-05-17

## MCP & integrations — roadmap bucket 10

- [ ] ⭐ **MCP — Architecture overview (official docs)** — host/client/server model, JSON-RPC, transports (STDIO vs HTTP+SSE), resources/tools/prompts primitives.
  https://modelcontextprotocol.io/docs/learn/architecture
- [ ] **Neo4j — Getting Started with MCP Servers: A Technical Deep Dive** — hands-on server build, good concrete grounding.
  https://neo4j.com/blog/developer/model-context-protocol/

## AI-Native SDLC — roadmap buckets 1, 3, 5

- [ ] ⭐ **AI-Native SDLC: How We Ship Software with AI Agents (Joyal Saji)** — a first-person account of agents across the full lifecycle; humans govern/orchestrate/validate.
  https://medium.com/@joyalsaji/ai-native-sdlc-how-we-ship-software-with-ai-agents-ce17ade0e2ee
- [ ] **CodeRabbit — Agentic SDLC: how AI agents are changing the SDLC** — agents at every stage (plan/code/review/ship/operate), not just autocomplete.
  https://www.coderabbit.ai/guides/agentic-sdlc
- [ ] **Xebia — 2026: The Year Software Engineering Will Become AI Native** — the macro shift and required engineer skills.
  https://xebia.com/news/2026-the-year-software-engineering-will-become-ai-native/

## Spec-driven development & coding workflows — roadmap bucket 1

- [ ] ⭐ **From Vibe Coding to Spec-Driven Development (Towards Data Science)** — write a clear spec, let the agent implement; the disciplined alternative to vibe coding.
  https://towardsdatascience.com/from-vibe-coding-to-spec-driven-development/
- [ ] **Spec-Driven Development: The Waterfall Strikes Back (HN debate)** — the skeptical practitioner view; essential counterweight.
  https://news.ycombinator.com/item?id=45935763
- [ ] **How Senior Engineers Actually Build With AI in 2026 (daily.dev)** — layered multi-tool workflow; verification as the senior/junior divide.
  https://app.daily.dev/posts/how-senior-engineers-actually-build-with-ai-in-2026-build-a-full-stack-systems-architecture-app-do59tbizd

## Multi-agent pipelines & harness engineering — roadmap buckets 2, 3

- [ ] **How Squad runs coordinated AI agents inside your repository (GitHub Blog)** — repository-native multi-agent orchestration without heavy infra; coordinator + specialists.
  https://github.blog/ai-and-ml/github-copilot/how-squad-runs-coordinated-ai-agents-inside-your-repository/
- [ ] **ComposioHQ/agent-orchestrator (GitHub)** — parallel coding agents, each with its own worktree/branch/PR; autonomous CI fixes, merge conflicts, reviews.
  https://github.com/ComposioHQ/agent-orchestrator
- [ ] **awesome-agent-orchestrators (GitHub)** — curated list of agent orchestration frameworks to survey the landscape.
  https://github.com/andyrewlee/awesome-agent-orchestrators
- [ ] ⭐ **Building Effective AI Coding Agents for the Terminal (arXiv, OPENDEV)** — the best deep dive on **harness engineering**: scaffolding vs harness, context compaction, safety invariants, model routing, dual-agent planning/execution, memory.
  https://arxiv.org/abs/2603.05344

## Verification, governance & quality gates — roadmap bucket 4

- [ ] ⭐ **Why Coding Agents Need Independent Quality Gates to Work at Scale (Codacy)** — verification becomes a governance problem; AI review is necessary but not a sufficient acceptance gate.
  https://blog.codacy.com/why-coding-agents-need-independent-quality-gates
- [ ] **How to Ship Secure AI-Generated Code in 2026 (Codebridge)** — governance model: reviews, sandboxing policies, CI gates.
  https://www.codebridge.tech/articles/how-to-ship-ai-generated-code-securely-a-governance-model-for-reviews-sandboxing-policies-and-ci-gates

## Security & safety — roadmap bucket 4

- [ ] ⭐ **The Lethal Trifecta for AI Agents — Simon Willison** — private data + untrusted content + exfiltration = structural prompt-injection vulnerability. The canonical security framing.
  https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- [ ] **Testing AI's "Lethal Trifecta" with Promptfoo** — how to actually test for it.
  https://www.promptfoo.dev/blog/lethal-trifecta-testing/

---

## Notes / thesis emerging from the community signal
Production teams are moving *away* from fully-autonomous multi-agent hype toward **well-engineered, mostly-deterministic systems with LLMs at controlled points**. **Evals + tracing are the real bottleneck**, not model choice. AI-assisted coding is consolidating into a **multi-tool stack** (Cursor + Claude Code + Codex), and **MCP** is becoming the standard integration layer. This is a strong organizing thesis for the curriculum.
