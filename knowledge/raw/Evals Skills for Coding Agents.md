---
title: "Evals Skills for Coding Agents"
source: "https://hamel.dev/blog/posts/evals-skills/"
author:
  - "[[Hamel Husain]]"
published:
created: 2026-06-03
description: "Teach your coding agent evals."
tags:
  - "clippings"
---
![](https://hamel.dev/blog/posts/evals-skills/cover-original.png)

Today, I’m publishing [evals-skills](https://github.com/hamelsmu/evals-skills), a set of skills for AI product evals [^1]. They guard against common mistakes I’ve seen helping 50+ companies and teaching 4,000+ students in our [course](https://maven.com/parlance-labs/evals).

## Why Skills for Evals

Coding agents now instrument applications, run experiments, analyze data, and build interfaces. I’ve been pointing them at evals.

OpenAI’s Harness Engineering [article](https://openai.com/index/harness-engineering/) makes the case well. They built a product entirely with Codex agents — three engineers, five months, ~1 million lines of code — and found that **improving the infrastructure around the agent** mattered more than improving the model. The agents queried traces to verify their own work. Documentation tells the agent what to do. Telemetry tells it whether it worked. Evals tell it whether the output is good.

All major eval vendors now ship an MCP server [^2]. The tedious parts: instrumenting your app, orchestrating experiments and building annotation tools now fall to coding agents.

But an agent with an eval platform still needs to know what to do with it. Say a support bot tells a customer “your plan includes free returns” when it doesn’t. Another says “I’ve canceled your order” when nobody asked. Both are hallucinations, but one gets a fact wrong and the other makes up a user action. If you lump them together in a generic “hallucination score,” you’ll miss errors.

These skills fill the gaps. They complement the vendor MCP servers: those give your agent access to traces and experiments, these teach it what to do with them.

## The Skills

If you’re new to evals or inheriting an existing eval pipeline, start with **eval-audit**. It inspects your current setup (or lack of one), runs diagnostic checks across six areas, and produces a prioritized list of problems with next steps. Install the skills or give your agent this prompt:

> Install the eval skills plugin from https://github.com/hamelsmu/evals-skills, then run /evals-skills:eval-audit on my eval pipeline. Investigate each diagnostic area using a separate subagent in parallel, then synthesize the findings into a single report. Use other skills in the plugin as recommended by the audit.

If you’re experienced with evals, you can skip the audit and pick the skill you need:

| error-analysis | Read traces, categorize failures, build a vocabulary of what’s broken |
| --- | --- |
| generate-synthetic-data | Create diverse test inputs when real data is sparse |
| write-judge-prompt | Design binary Pass/Fail LLM-as-Judge evaluators |
| validate-evaluator | Calibrate judges against human labels using TPR/TNR and bias correction |
| evaluate-rag | Evaluate retrieval and generation quality separately |
| build-review-interface | Generate annotation interfaces for human trace review |

These skills are a starting point and only encode common mistakes that generalize across projects. Skills grounded in your stack, your domain, and your data will outperform them. Start here, then write your own.

👉 The repo is here: [github.com/hamelsmu/evals-skills](https://github.com/hamelsmu/evals-skills) 👈

If these skills help you, I’d love to hear from you! You can find me on [X](https://x.com/hamelhusain) or email me through my [newsletter](https://ai.hamel.dev/).

[^1]: Not foundation model benchmarks like MMLU or HELM that measure general LLM capabilities. Product evals measure whether *your* pipeline works on *your* task with *your* data. If you aren’t familiar with product-specific AI evals, check out my [AI Evals FAQ](https://hamel.dev/blog/posts/evals-faq/)

[^2]: [Braintrust](https://www.braintrust.dev/docs/reference/mcp), [LangSmith](https://github.com/langchain-ai/langsmith-mcp-server), [Phoenix](https://github.com/Arize-ai/phoenix/tree/main/js/packages/phoenix-mcp), [Truesight](https://truesight.goodeyelabs.com/docs/mcp-integration), and others.