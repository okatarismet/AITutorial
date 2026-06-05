# AI Native Engineer — Distilled Reading List

*Levels, links, short titles, and a one-line summary each. Check each `[ ]` off as you finish.* Full version: [`READING-ROADMAP.md`](READING-ROADMAP.md).

---

## Level 0 — Foundations

> *Treat the LLM as a black-box engineering component. Fast-track if you've already built agents.*

Master **just enough mechanics** — no ML math. You learn how text becomes **tokens** and why context is a **budget**, the **request → tool_use → result** loop that underlies every agent, and **schema-enforced structured output**. The goal here is closing mechanics blind spots, *not* re-teaching basics.

- [S] **Chip Huyen — Agents** — https://huyenchip.com/2025/01/07/agents.html
  - *Best single mental map of the whole agent landscape.*
- [S] **Anthropic — Prompt Engineering Tutorial** — https://github.com/anthropics/prompt-eng-interactive-tutorial
  - *Hands-on chapters on chain-of-thought and avoiding hallucination.*
- [S] **Tokens & context windows** — https://platform.claude.com/docs/en/build-with-claude/token-counting
  - *How text becomes tokens; context as a budget.*
- [S] **Tool use / API loop** — https://platform.claude.com/docs/en/agents-and-tools/tool-use/overview
  - *The request → tool_use → result loop behind every agent.*
- [S] **Structured outputs** — https://platform.claude.com/docs/en/build-with-claude/structured-outputs
  - *Schema-enforced generation instead of hoping for valid JSON.*

---

## Level 1 — Mastering AI Coding Tools

> *Become an elite power-user of agentic coding tools. High-ROI, fast to fast-track.*

The throughline is **spec-driven development** — write a clear spec, let the agent implement, and **verify rigorously** — supported by **context files** (`CLAUDE.md` / `AGENTS.md`), **parallel agents** in isolated git worktrees, and the **writer/reviewer** pattern. *The discipline is what separates senior from junior usage.*

- [S] **Claude Code best practices** — https://www.anthropic.com/engineering/claude-code-best-practices
  - *The core playbook: CLAUDE.md, plan-implement-commit, subagents, verification.*
- [S] **Vibe coding → spec-driven dev** — https://towardsdatascience.com/from-vibe-coding-to-spec-driven-development/
  - *Spec first, agent implements; disciplined alternative to vibe coding.*
- [S] **Good spec for AI agents** — https://addyosmani.com/blog/good-spec/
  - *Five principles plus the Always/Ask/Never boundary tiers.*
- [S] **Spec-driven dev (HN debate)** — https://news.ycombinator.com/item?id=45935763
  - *Skeptical counterweight to the spec-driven development hype.*
- [S] **AGENTS.md standard** — https://agents.md/
  - *Vendor-neutral "README for agents," used by 60k+ projects.*
- [S] **Git-worktree parallel agents** — https://code.claude.com/docs/en/worktrees
  - *Isolated agent sessions; the writer/reviewer parallel pattern.*
- [S] **Coding agents comparison** — https://artificialanalysis.ai/agents/coding
  - *Data-driven benchmarks for picking your coding tool.*

---

## Level 2 — Building Agentic Systems

> *Go from **using** agents to **building** them. This is where real depth begins — go slow.*

You learn the reference **workflow patterns** (routing, parallelization, orchestrator-workers, evaluator-optimizer), the mindset that the best agents are **well-built software with LLMs at controlled decision points**, and the two disciplines that make them work: **context engineering** (budgeting, compaction, just-in-time retrieval) and **harness engineering**. **MCP** ties it together by giving agents tools through a standard protocol.

- [X] **Building Effective Agents** — https://www.anthropic.com/engineering/building-effective-agents
  - *Reference patterns: routing, parallelization, orchestrator-workers, evaluator-optimizer.*
- [ ] **12-Factor Agents** — https://github.com/humanlayer/12-factor-agents
  - *Best agents are engineered software with controlled LLM decisions.*
- [ ] **Effective Context Engineering** — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  - *Context as a discipline: budgeting, compaction, just-in-time retrieval.*
- [ ] **Context Engineering (Manus)** — https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
  - *Battle scars: KV-cache economics, filesystem-as-context, keep errors visible.*
- [ ] **Coding agents for the terminal (arXiv)** — https://arxiv.org/abs/2603.05344
  - *Deep dive on harness engineering, compaction, and safety invariants.*
- [ ] **MCP architecture** — https://modelcontextprotocol.io/docs/learn/architecture
  - *Host/client/server, JSON-RPC, transports, and primitive types.*
- [ ] **Neo4j — MCP servers** — https://neo4j.com/blog/developer/model-context-protocol/
  - *Hands-on server build to ground the MCP concepts.*

---

## Level 3 — Multi-Agent SDLC Pipelines

> *The headline level: coordinate specialized agents across **plan → code → review → test → deploy → operate**.*

You study real **repo-native systems** (coordinator + specialists with shared memory), **quality gates** between stages where a reviewer can reject and loop back, and how **RAG** feeds knowledge to agents — while heeding the warning that *adding AI to broken processes just yields faster broken processes*. **Humans govern, orchestrate, and validate; agents do the work.**

- [TRASH] **AI-Native SDLC (Joyal Saji)** — https://medium.com/@joyalsaji/ai-native-sdlc-how-we-ship-software-with-ai-agents-ce17ade0e2ee
  - *First-person account of specialized agents across the lifecycle.*
- [X] **CodeRabbit — Agentic SDLC** — https://www.coderabbit.ai/guides/agentic-sdlc
  - *Agents at every stage; review as the critical gate.*
- [ ] **GitHub — Squad coordinated agents** — https://github.blog/ai-and-ml/github-copilot/how-squad-runs-coordinated-ai-agents-inside-your-repository/
  - *Reference for repo-native coordinator plus specialist agents.*
- [ ] **Xebia — 2026 AI Native** — https://xebia.com/news/2026-the-year-software-engineering-will-become-ai-native/
  - *Macro framing; broken processes just break faster with AI.*
- [ ] **GitHub spec-kit** — https://github.com/github/spec-kit
  - *Open-source spec-driven toolkit: constitution → spec → plan → implement.*
- [ ] **Composio agent-orchestrator** — https://github.com/ComposioHQ/agent-orchestrator
  - *Parallel agents with worktree/PR-per-agent and autonomous CI fixes.*
- [ ] **Evaluator-optimizer (AWS)** — https://docs.aws.amazon.com/prescriptive-guidance/latest/agentic-ai-patterns/evaluator-optimizer.html
  - *Generation → evaluation → refinement as a control loop.*
- [ ] **RAG to Context (RAGFlow)** — https://ragflow.io/blog/rag-review-2025-from-rag-to-context
  - *Where RAG went and how it became context.*
- [ ] **Production RAG (Towards AI)** — https://towardsai.net/p/machine-learning/production-rag-the-chunking-retrieval-and-evaluation-strategies-that-actually-work
  - *Why most failures are ingestion and chunking, not the model.*
- [ ] **awesome-agent-orchestrators** — https://github.com/andyrewlee/awesome-agent-orchestrators
  - *Survey of the agent-orchestration framework landscape.*

---

## Level 4 — Production Reliability

> *The 2026 bottleneck — and a habit, not a final step. **Run it in parallel with Levels 2–3.***

Four pillars: **evals** (offline/online, LLM-as-judge, trace-based, starting from error analysis), **observability & tracing**, **security & safety** (the lethal trifecta, prompt injection, sandboxing untrusted actions), and **governance** via independent quality gates plus **cost/latency** control through prompt caching. *This is what turns an agentic pipeline from a demo into something shippable.*

**Evals**
- [ ] **Hamel — Your AI Product Needs Evals** — https://hamel.dev/blog/posts/evals/
  - *Foundational: evals matter more than your model choice.*
- [ ] **Hamel — Field Guide to Improving AI Products** — https://hamel.dev/blog/posts/field-guide/
  - *Error-analysis playbook drawn from 30+ companies.*
- [ ] **Hamel — LLM-as-a-Judge** — https://hamel.dev/blog/posts/llm-judge/index.html
  - *Binary judgments and expert alignment, done right.*
- [ ] **Anthropic — Demystifying Evals** — https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
  - *Grader types, pass@k/pass^k, start with 20–50 tasks.*
- [ ] **AWS — Evaluating AI Agents** — https://aws.amazon.com/blogs/machine-learning/evaluating-ai-agents-real-world-lessons-from-building-agentic-systems-at-amazon/
  - *Trace-based evals, silent drift, why outcome-only evals fail.*

**Observability**
- [ ] **OpenTelemetry — LLM observability** — https://opentelemetry.io/blog/2024/llm-observability/
  - *Vendor-neutral tracing correlating LLM calls with the backend.*

**Security & governance**
- [ ] **Simon Willison — Lethal Trifecta** — https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
  - *Private data + untrusted content + exfiltration = structural risk.*
- [ ] **Promptfoo — Lethal Trifecta testing** — https://www.promptfoo.dev/blog/lethal-trifecta-testing/
  - *Red-team the prompt-injection trifecta for real.*
- [ ] **OWASP — Prompt Injection Prevention** — https://cheatsheetseries.owasp.org/cheatsheets/LLM_Prompt_Injection_Prevention_Cheat_Sheet.html
  - *13 attack categories with concrete defenses and checklist.*
- [ ] **Northflank — Sandboxing AI agents** — https://northflank.com/blog/how-to-sandbox-ai-agents
  - *Why containers aren't enough; microVMs for untrusted actions.*
- [ ] **Codacy — Independent Quality Gates** — https://blog.codacy.com/why-coding-agents-need-independent-quality-gates
  - *Code generator shouldn't be its own acceptance gate.*
- [ ] **Codebridge — Ship Secure AI Code** — https://www.codebridge.tech/articles/how-to-ship-ai-generated-code-securely-a-governance-model-for-reviews-sandboxing-policies-and-ci-gates
  - *Governance model: reviews, sandboxing policy, CI gates.*

**Cost & latency**
- [ ] **Anthropic — Prompt caching** — https://platform.claude.com/docs/en/build-with-claude/prompt-caching
  - *Cached-prefix discounts, KV-cache mechanics, and model routing.*

---

## Level 5 — Leading AI-Native Engineering

> *Take a whole team or org AI-native, through a **leadership and measurement** lens.*

The core skill is **honest productivity measurement** — why throughput metrics inflate under AI while code **churn and bugs** quietly explode, so you track **durability alongside speed**. It also covers **human-in-the-loop governance** (agent-delegated vs. human-gated, escalation rules, regulatory context) and treating **fine-tune vs. RAG vs. prompt** as a deliberate decision rather than a default.

- [ ] **Faros — AI Acceleration Whiplash** — https://www.faros.ai/blog/ai-acceleration-whiplash-takeaways
  - *Throughput up 66% but churn and bugs explode.*
- [ ] **GitClear — AI Copilot Code Quality 2025** — https://www.gitclear.com/ai_assistant_code_quality_2025_research
  - *Duplication up 4×; refactoring is quietly collapsing.*
- [ ] **EY — CIO Playbook on Agentic AI** — https://www.ey.com/en_us/ey-center-for-executive-leadership/defining-a-cio-playbook-on-agentic-ai
  - *Executive framing for governing agentic AI adoption.*
- [ ] **Stanford — Enterprise AI Playbook** — https://digitaleconomy.stanford.edu/
  - *Lessons distilled from 51 real enterprise deployments.*
