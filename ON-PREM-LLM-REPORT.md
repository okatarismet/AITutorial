# On-Premise LLM Serving — Feasibility & Budget Report

**Prepared:** 2026-06-04
**Question:** With a ~€50,000 GPU budget, what kind of model, context window, and serving capacity can a software company build on-premise instead of paying for the Claude API? How many agents can it serve at acceptable latency? If €50k is not enough, how much is?

> **Note on figures:** GPU prices and model specifications below are sourced and current as of June 2026. Throughput and concurrency numbers for the recommended build are **vendor- and community-modeled projections** — no fully measured public benchmark exists yet for this exact GPU + model combination. They are flagged as such.

---

## 1. Executive summary / verdict

**€50k is enough — but only for the "fast/flash" tier, not the frontier tier.**

- For **€45–48k net** you can build a **4× RTX PRO 6000 Blackwell** server (384 GB VRAM) that runs **DeepSeek V4 Flash (284B)** or **Qwen3-Coder** at full context, serving on the order of **200–400 concurrent requests** or realistically **~40–80 simultaneously-active coding agents** (≈100–200 developers day-to-day).
- To run the **frontier open models** comparable to Claude — DeepSeek V4 **Pro** (1.6T), GLM-5.1 (754B), Kimi K2.6 — you need ~**8× H200 (≈€280–350k)**, i.e. **5–7× over budget**.
- **Cost reality:** at realistic utilization, self-hosting costs **~$1.3 per 1M output tokens vs ~$0.28 on the DeepSeek API**. On pure token economics, on-prem is *more* expensive than API unless run near 24/7. The genuine justification for on-prem is **data privacy / IP control / no per-token metering / air-gap** — not raw cost. This should be stated explicitly to leadership.

---

## 2. GPU prices — Germany, June 2026

Prices are **net (excl. 19% VAT)** where noted. A German company reclaims VAT (Vorsteuerabzug), so net is the figure that matters for budgeting.

| GPU | VRAM | Mem BW | Power | Price (Germany) | Notes |
|---|---|---|---|---|---|
| **RTX PRO 6000 Blackwell** | 96 GB GDDR7 | 1.79 TB/s | 600 W | **~€8,400 net** | Best €/GB — the build block |
| RTX 5090 | 32 GB GDDR7 | ~1.8 TB/s | 575 W | ~€2,200–2,800 | Cheap, but 32 GB limits you |
| H100 80 GB (PCIe/SXM) | 80 GB HBM3 | 3.35 TB/s | 350–700 W | €31,500–36,000 incl. VAT | Has NVLink; poor €/GB now |
| H200 | 141 GB HBM3e | 4.8 TB/s | 700 W | ~$30–40k (≈€28–37k) | Frontier-model tier |
| B200 | 192 GB HBM3e | ~8 TB/s | 1000 W | $40k+ | Scarce / allocated |

**Takeaway:** the RTX PRO 6000 Blackwell is the value king — 96 GB for ~€8.4k vs an H100's 80 GB for ~€32k. Four of them give **more VRAM (384 GB) than three H100s** at roughly a third of the price. Trade-off: no NVLink (PCIe Gen5 ~128 GB/s between cards), which hurts tensor-parallel throughput but is fine for MoE models with few active parameters.

---

## 3. Recommended build — the €50k machine

| Component | Spec | Qty | Net price |
|---|---|---|---|
| GPU | RTX PRO 6000 Blackwell 96 GB | 4 | €33,600 |
| Chassis / Server | Supermicro 4U GPU MGX or AS-4125GS (4× dual-slot PCIe5) | 1 | €4,000–6,000 |
| CPU | AMD EPYC 9004/9005 (single socket, 32–64c) | 1 | €2,500–4,000 |
| RAM | 512 GB DDR5 ECC | 1 | €2,500 |
| Storage | 2× 4 TB NVMe (models + KV spill) | 2 | €1,000 |
| PSU + cooling | 2× redundant Titanium PSU (≥3.2 kW total) | – | €1,500 |
| **Total** | | | **≈ €45,000–48,000 net** |

Lands **inside €50k** and matches the community-documented "$44k all-in" reference build for this exact GPU + model combo. Add ~€1.5–3k for vendor integration/warranty from a German systems builder (Delta, Thomas-Krenn, Megware).

**Power / opex:** ~2.4 kW GPUs + ~1 kW system ≈ **3.4 kW**. Running 24/7 at German industrial rates (~€0.22/kWh) ≈ **€6,500/yr electricity**, ~€8–9k/yr all-in with cooling. This recurring cost belongs next to "no more Claude bills" in any comparison.

---

## 4. What model fits at each budget tier

| Model | Total / Active params | Context | VRAM (served) | Min hardware | In budget? |
|---|---|---|---|---|---|
| **Qwen3-Coder-Next** | 80B / 3B MoE | 256K | ~48–60 GB | 1× RTX PRO 6000 | ✅ ~€12k |
| **DeepSeek V4 Flash** | 284B / 13B MoE | 1M | ~170 GB + KV | **4× RTX PRO 6000 (384 GB)** | ✅ ~€45k *(sweet spot)* |
| GLM-5.1 | 754B / 40B MoE | 203K | ~800 GB (FP8) | 8× H200 (1,128 GB) | ❌ ~€280k |
| DeepSeek V4 **Pro** | 1.6T / 49B MoE | 1M | ~900 GB–1 TB | 8× H200 / 8× B200 | ❌ ~€300k+ |
| Kimi K2.6 | ~1T class | long | ~600–800 GB+ | 8× H200 | ❌ ~€280k+ |

The €50k decision is effectively: **Qwen3-Coder** (one card, cheap, strong at coding — 58.7% SWE-bench Verified) vs **DeepSeek V4 Flash** (four cards, frontier-adjacent, 1M context, big concurrency). For a software company wanting agentic coding plus breadth, **DeepSeek V4 Flash on the 4-card build is the recommendation.**

---

## 5. Capacity — concurrency & latency (4× RTX PRO 6000 + DeepSeek V4 Flash)

> ⚠️ Modeled projections — no measured public benchmark yet for this exact combo.

| Metric | Figure |
|---|---|
| Concurrent users @ 32K context | **200–400** (depends on prefix-cache hit rate) |
| Single-stream decode (2K prompt) | 25–35 tok/s |
| Single-stream decode (32K prompt) | 15–25 tok/s |
| Aggregate @ 64 concurrent | 400–700 tok/s |
| Aggregate @ 256 concurrent | 1,200–2,000 tok/s |
| Time-to-first-token (chat, 2K in) | target < 500 ms |
| "Feels responsive" floor | ≥ 30 tok/s per stream |

---

## 6. How many *agents* can it serve?

A coding agent is **bursty and token-heavy**: long context (often 32–128K), many sequential tool-call round-trips, then idle while the human reads/edits. So "agents" differs from "concurrent requests":

- **In-flight requests:** 200–400 light/chat; fewer for large context.
- **Active coding-agent sessions** (32–64K context, needing ≥30 tok/s to feel responsive): realistically **~40–80 simultaneously active**.
- **Named developers served day-to-day:** since each dev's agent is active ~10–20% of the working hour, the box comfortably backs **~100–200 developers**, with auto-batching absorbing bursts.

**Rule of thumb:** one box ≈ a **100–150-engineer org's** daily agentic-coding load, or **~50 power users** hammering it continuously. Beyond that, add a second node.

---

## 7. On-prem vs API — honest comparison

| | On-prem 4× RTX PRO 6000 | DeepSeek API | Claude API |
|---|---|---|---|
| Upfront | ~€45k capex | €0 | €0 |
| Per 1M output tokens | **~$1.29 @ 60% util** (~$0.68 @ 75%+) | ~$0.28 | higher, but frontier quality |
| Recurring | ~€8k/yr power + ops | usage only | usage only |
| Data leaves building? | **No** ✅ | Yes | Yes |
| Model quality | Flash tier (good, not Claude-frontier) | Flash / Pro | Frontier |

**Strategic point:** on-prem wins on *cost* only if kept busy near 24/7. Its real value is **IP stays in-house, no per-token meter, no rate limits, full control, compliance/air-gap**. If the goal is purely "stop paying for API," that is rational — but the open Flash-tier model is **a step below Claude in quality**, and true frontier parity sits at the €300k 8×H200 tier, where API would have been cheaper for years. That tension is the central decision.

---

## 8. Software stack / GitHub repos

| Layer | Tool | Repo |
|---|---|---|
| Inference engine | **vLLM** (best general throughput) | github.com/vllm-project/vllm |
| Inference engine | **SGLang** (high concurrency / prefix caching; ~460 tok/s @ batch 64 on 1×H100 for 70B FP8) | github.com/sgl-project/sglang |
| Inference engine | **TensorRT-LLM** (max NVIDIA perf, harder to operate) | github.com/NVIDIA/TensorRT-LLM |
| Quantization | Unsloth GGUF (2-bit/1-bit for the giant models) | github.com/unslothai/unsloth |
| API gateway / routing | LiteLLM | github.com/BerriAI/litellm |
| Agent-facing | Point Claude Code / Cursor / Continue at the vLLM OpenAI endpoint | — |

Recommend **SGLang or vLLM** as the serving layer — both expose an OpenAI-compatible API, so existing agent tooling re-points to your endpoint with no code change.

---

## 9. If €50k is not enough — budget tiers

| Tier | Hardware | ~Cost (net) | What you get |
|---|---|---|---|
| Entry | 1× RTX PRO 6000 | ~€12k | Qwen3-Coder, single team |
| **Recommended** | **4× RTX PRO 6000** | **~€45k** | **DeepSeek V4 Flash, 1M ctx, ~100–200 devs** |
| Scale-out | 2–3× the 4-card node | ~€90–135k | Same Flash model, 2–3× agents + redundancy |
| Frontier | 8× H200 (1,128 GB) | ~€280–350k | DeepSeek V4 Pro / GLM-5.1 / Kimi K2.6 — true Claude-class |

**Pragmatic middle path:** buy **two** €45k nodes (~€90k) for redundancy + 2× Flash-model capacity before ever considering the €300k frontier tier.

---

## 10. Recommendation

1. **Build one 4× RTX PRO 6000 Blackwell node (~€45k net).** Run **DeepSeek V4 Flash** on **vLLM/SGLang**, OpenAI-compatible endpoint.
2. **Pilot with a subset of engineers** to measure *real* utilization and per-token cost before scaling — the projected numbers above need to be validated on your workload.
3. **Frame the decision to leadership honestly:** this buys data sovereignty and no per-token metering, at Flash-tier (sub-Claude) quality. True frontier parity is a €300k+ commitment. Keep the Claude API available for the hardest tasks rather than treating on-prem as a full replacement on day one.

---

## Sources

- DeepSeek V4 release / specs — [DataCamp](https://www.datacamp.com/blog/deepseek-v4), [DeepSeek API docs](https://api-docs.deepseek.com/news/news260424)
- DeepSeek V4 Flash on 4× RTX Pro 6000 — capacity & cost — [Codersera](https://codersera.com/blog/deepseek-v4-flash-rtx-pro-6000-blackwell-benchmarks-2026/)
- DeepSeek V4 self-hosting / VRAM — [WaveSpeed](https://wavespeed.ai/blog/posts/deepseek-v4-gpu-vram-requirements/), [Lushbinary](https://lushbinary.com/blog/deepseek-v4-self-hosting-guide-vllm-hardware-deployment/)
- RTX PRO 6000 Blackwell price (Germany) — [Servershop24](https://www.servershop24.de/en/nvidia-rtx-pro-6000-blackwell-graphic-card/a-136499/), [Thunder Compute](https://www.thundercompute.com/blog/nvidia-rtx-pro-6000-pricing)
- H100 / H200 pricing — [IntuitionLabs](https://intuitionlabs.ai/articles/nvidia-ai-gpu-pricing-guide), [Jarvislabs](https://jarvislabs.ai/blog/h100-price)
- GLM-5 / Qwen3 VRAM — [GMI Cloud](https://www.gmicloud.ai/en/blog/where-to-run-glm-5-inference-in-the-cloud-gpu-requirements-deployment-options-and-scaling-considerations), [Compute Market](https://www.compute-market.com/blog/qwen-3-coder-next-local-hardware-guide-2026)
- Best open-weight coding models 2026 — [MindStudio](https://www.mindstudio.ai/blog/best-open-source-llms-agentic-coding-2026), [BenchLM](https://benchlm.ai/blog/posts/best-chinese-llm)
- Serving throughput — [vLLM blog](https://blog.vllm.ai/2024/09/05/perf-update.html), [Cerebrium benchmark](https://cerebrium.ai/blog/benchmarking-vllm-sglang-tensorrt-for-llama-3-1-api)
- Supermicro RTX PRO 6000 systems — [Supermicro](https://www.supermicro.com/en/accelerators/nvidia/supermicro-rtx-pro-bse)
