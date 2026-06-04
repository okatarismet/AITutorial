<!--
NOTE TO ISMET (not part of the post — delete before publishing):
This is a Medium-ready narrative written in first person so you can adapt it as your own project.
Two kinds of numbers live in here, kept deliberately distinct:
  • PROJECT numbers (our timeline, cost, PR acceptance rate, the "Atlas" details) are ILLUSTRATIVE —
    a realistic but invented case. Tune them to whatever story you actually want to tell.
  • INDUSTRY numbers carry a real citation + URL (Amazon Q, Google, Airbnb, METR, GitClear, Faros,
    Spring/Oracle/Hibernate docs, Willison, Manus, Anthropic, Cognition). Those are verified — keep the
    URLs if you cite them. Don't blur the two; the honesty is what makes the post land.
All sources are collected at the bottom under "References."
-->

# How We Modernized a 15-Year-Old Java 8 Monolith With a Team of AI Agents (and the Ego-Bruising Things It Taught Us)

*A field report on using a supervised multi-agent pipeline to migrate a legacy Struts/JSP + Java 8
monolith to Spring Boot 3 + React — what worked, what blew up, and why "let the agent do it" is the
fastest way to ship garbage if you skip the boring parts.*

---

## TL;DR

We inherited a 1.1M-LOC Java 8 monolith — Struts actions, JSP views, Ant build, Hibernate 4, a JDK that
stopped getting security patches — and a mandate to get onto **Spring Boot 3 + React 18** before it failed
an audit. Hand-estimated, it was **~18 months for six engineers**. We built a pipeline of specialized AI
agents (plan → port → test → review) wrapped around **deterministic codemods**, with a human approving every
pull request, and brought the bulk of it home in about **a quarter** with two engineers steering.

The headline isn't "AI did it." The headline is the opposite: **AI only worked because we treated it as the
least-trustworthy member of the team.** Everything good came from the scaffolding around the model —
deterministic-first migration, a compiler-and-test oracle on every change, an *independent* reviewer, real
evals, tracing, and a hard rule that the thing writing the code never gets to approve it. This post is about
that scaffolding, and the four or five times the project nearly went off the rails.

---

## The thing we inherited

If you've worked at a company older than ~12 years, you know this codebase. Ours ran the company's core
policy-and-claims workflow for a mid-size insurer. The stats that mattered:

- **Java 8.** Past end of public updates. Security was filing tickets monthly.
- **Struts 1.x actions + JSP** for the web layer — server-rendered pages, business logic tangled directly
  into `Action.execute()` methods, state smeared across the `HttpSession`.
- **Ant** with a hand-curated `lib/` folder. No dependency graph — the "dependency manifest" was whatever
  JARs someone had dropped in over the years.
- **Hibernate 4 / `javax.persistence`**, a pile of hand-rolled `UserType`s, and a lot of legacy
  `Criteria` queries.
- **~31% test coverage**, mostly JUnit 4, much of it brittle.

The target was non-negotiable and, it turned out, a *trap*: you cannot casually "upgrade to Spring Boot 3."
Spring Boot 3 / Spring 6 baseline on **Jakarta EE 10 and require Java 17 minimum — Java 8 and 11 are dropped
entirely** ([Spring Boot 3.0 Migration Guide][sb3]). So "modernize the web framework" silently means doing
**five forced migrations at once**: Java 8 → 17, the `javax` → `jakarta` namespace flip, Hibernate 4 → 6,
Spring Security's rewritten config model, and Struts/JSP → REST + React. There is no incremental sub-step
where you do one and ship. That coupling *is* the project.

---

## Why not just... do it the normal ways?

We seriously considered two non-AI options first, because reaching for agents on day one is how you end up
in the productivity statistics nobody wants to be in.

**Option A — the big-bang human rewrite.** Martin Fowler's verdict on the rewrite-from-scratch plan is
blunt: *"we've seen this simple-sounding plan go down in flames most of the time"* ([Strangler Fig][fowler]).
Eighteen months of frozen feature work on the system that runs the business is how companies die. Hard pass.
We adopted the **Strangler Fig** posture instead — wrap the legacy app, move capability out incrementally,
keep the old one serving traffic the whole time.

**Option B — pure deterministic codemods (OpenRewrite).** This is genuinely great and we used a ton of it.
OpenRewrite does *"the deterministic, mechanical part of migration automation… the part that does not
require judgment,"* parsing code into a type-aware tree and running composable recipes — there's an official
"Migrate to Spring Boot 3" recipe that bundles javax→jakarta, dependency bumps, and JUnit 4→5
([OpenRewrite][or]). A global retailer used the same family of tooling to migrate **3,500+ repositories** in
one coordinated pass ([Moderne][moderne]).

But recipes have a hard ceiling: they fix what's *mechanical and uniform*. They do **not** untangle business
logic from a Struts action, redesign session state into a token flow, or rewrite a hand-rolled Hibernate
`UserType` whose semantics changed. And the moment you reach for an LLM recipe to cover the gap, you inherit
non-determinism — *"recipe results vary between runs — including producing incorrect changes or modifying
code that should be left alone"* ([OpenRewrite generative-AI notes][or-ai]). That unpredictability is
poison at review time.

So neither alone was the answer. The answer was the seam *between* them.

---

## The actual insight: deterministic-first, LLM-for-the-residual

Every credible large-scale AI migration we could find converges on the same shape, and it is **not**
"point a clever agent at the repo":

- **Google's** internal AI migration system: static analysis/AST tooling **finds** the locations to change,
  an LLM **edits and validates**, humans **review every change**. On one migration, **80% of the code
  modifications in landed changelists were AI-authored**, cutting total migration time by an estimated
  **50%** — but **~25% of the AI's character changes were discarded or fixed by reviewers**
  ([Google Research][google]).
- **Amazon Q Code Transformation** runs a deterministic plan, then a **sandboxed build-and-fix loop** until
  the project compiles — and Amazon still warns, in its own docs, *"despite having no compilation errors,
  you can still experience runtime errors"* ([AWS docs][awsq]). They claim tens of thousands of apps
  migrated and famously "4,500 developer-years saved" — worth knowing that figure is a *self-reported
  extrapolation*, and the headline **$260M** is mostly Java 17 *runtime* efficiency, not migration labor
  ([AWS blog][aws260]).
- **Airbnb's** Enzyme→RTL test migration — the best-documented case study going — turned an estimated
  **1.5 years of hand work into 6 weeks**, hitting **97% automated** completion. Their lesson is almost
  funny: prompt cleverness mattered less than **brute-force retries** graded by a real test run — the long
  tail of files got retried *"anywhere between 50 to 100 times"* ([Airbnb Engineering][airbnb]).

The through-line: **the LLM is an exception handler, and a build/test oracle is the only reason any of it is
safe.** Migration is the *good* AI use case precisely because every change has a built-in pass/fail signal —
it compiles or it doesn't, the test goes green or it doesn't. We built our whole pipeline around that signal.

We named it **Atlas**.

---

## Architecture — the team of agents

Atlas processes the codebase **one unit of work at a time** (a Struts action and its JSPs, or a service
class, or a persistence type), each in its own isolated **git worktree**, with **dozens of units running in
parallel**. Within a single unit the agents run in a **linear chain sharing one context**; parallelism only
happens *across* independent units. (That distinction turned out to matter enormously — see "The multi-agent
trap" below.)

```
                          ┌─────────────────────────────────────────────┐
                          │            ORCHESTRATOR (coordinator)        │
                          │  • owns the backlog (≈2,400 units of work)   │
                          │  • runs deterministic codemods FIRST         │
                          │  • spins a worktree per unit, routes stages  │
                          │  • enforces the gate / retries / gives up    │
                          └───────────────┬─────────────────────────────┘
                                          │
                 ┌────────────── STEP 0: DETERMINISTIC PASS ─────────────────┐
                 │  OpenRewrite recipes: javax→jakarta, JUnit4→5,            │
                 │  Spring Boot 2→3, dependency BOM alignment.               │
                 │  Handles the mechanical ~70%. Diffs are reviewable.      │
                 └───────────────────────────┬───────────────────────────────┘
                                             │  (only the residual goes to the agents)
        ┌─────────────────────────────────────┼─────────────────────────────────┐
        ▼                                     ▼                                 ▼
┌───────────────┐    reject &        ┌───────────────┐   reject &     ┌────────────────┐
│  1. PLANNER   │    loop back       │  2. PORTER    │   loop back    │  3. TESTER     │
│  (read-only)  │ ──────────────────▶│  (writes code)│ ──────────────▶│ (writes tests) │
│ • reads the   │                    │ • Struts→REST │                │ • JUnit5 +     │
│   legacy unit │                    │ • JSP→React   │                │   RTL          │
│ • RAG: golden │◀─── RAG ───┐       │ • Hibernate6  │                │ • runs them;   │
│   migrations, │            │       │ • house style │                │   must pass +  │
│   style guide │            │       │   from RAG    │                │   ≥80% cover   │
│ • writes plan │            │       └───────────────┘                └────────┬───────┘
└───────────────┘            │                                                 │
                    ┌────────┴─────────┐                                       ▼
                    │   RAG KNOWLEDGE  │                              ┌────────────────┐
                    │ • 35 hand-done   │                              │  4. REVIEWER   │
                    │   "golden"       │                              │ (independent,  │
                    │   migrations     │                              │  fresh context)│
                    │ • house style    │                              │ • diff review  │
                    │ • Struts→REST    │                              │ • injection /  │
                    │   mapping table  │                              │   PII scan     │
                    └──────────────────┘                              │ • PASS / FAIL  │
                                                                      └───────┬────────┘
                                                                              │ PASS
                                          ┌───────────────────────────────────▼──────────┐
                                          │  QUALITY GATE (machine, NON-LLM — the oracle) │
                                          │  ✓ mvn compiles   ✓ checkstyle/spotbugs       │
                                          │  ✓ tests pass     ✓ coverage ≥ 80%            │
                                          │  ✓ no secrets/PII in diff  ✓ API contract diff│
                                          └───────────────────┬───────────────────────────┘
                                                              │ all green
                                                              ▼
                                                   ┌──────────────────────┐
                                                   │  OPEN PULL REQUEST    │
                                                   │  → human review/merge │  ← the only path to main
                                                   └──────────────────────┘
              every agent + tool call ──▶ TRACING (OpenTelemetry)      every PR outcome ──▶ EVAL DATASET
```

Three design choices, each bought with someone else's scar tissue:

1. **Deterministic pass before the agents ever run.** The codemods do the mechanical ~70% predictably; the
   agents only see the residual that needs judgment. This mirrors Google (AST locates, LLM edits) and keeps
   the agent's context small and its diffs reviewable.
2. **The Reviewer is a *separate agent* with a *fresh context*, and behind it sits a non-LLM gate.** The
   thing that writes the code does not get to approve it. `mvn`, the test runner, and the coverage check are
   deterministic truth — the LLM reviewer's job is to catch the *semantic* stuff the compiler can't see, and
   even it doesn't have merge rights. Humans do.
3. **RAG, not fine-tuning.** We retrieve 35 hand-done "golden" migrations and the house style guide
   just-in-time. We never trained a model. (This is also the right answer to the Level 5 question
   "fine-tune vs. retrieve vs. prompt" — for house patterns that change weekly, retrieval wins.)

---

## One unit, start to finish

Let me make it concrete with a real-shaped example: `SubmitClaimAction.java`, a 280-line Struts action with
two JSPs and one thin JUnit 4 test.

1. **Orchestrator** runs the **deterministic pass** on the module first — OpenRewrite flips `javax.*` →
   `jakarta.*` imports, bumps the JUnit annotations, aligns the Spring BOM. Compiles? No — there's a
   hand-rolled `MoneyUserType` and the action's session logic that recipes can't touch. *Now* the unit goes
   to the agents.
2. **Planner** reads the action, both JSPs, and the test. RAG pulls the three most similar golden migrations
   and the Struts→REST mapping table. It writes a plan: *"Extract claim-submission logic from `execute()`
   into `ClaimService`; expose `POST /api/claims`; the three-page wizard currently accumulating state in
   `HttpSession` becomes a client-held draft posted on the final step; `MoneyUserType` → a JPA
   `AttributeConverter`; build a `ClaimWizard` React component with the three steps as client routes."*
3. **Porter** executes it. Writes the `@RestController`, the service, the converter, the React components in
   TypeScript, pulling house conventions from RAG (constructor injection, no field injection; named exports;
   DTOs as Java `records`).
4. **Machine gate, pass 1:** `mvn compile` fails — a `jakarta.validation` annotation on a DTO needs the
   validation starter that the Ant `lib/` never declared. The orchestrator **loops the compiler error back to
   the Porter** with the stack trace *in context, unedited* (more on why that matters below). Porter adds the
   dependency. Compiles.
5. **Tester** writes JUnit 5 + React Testing Library tests, runs them. Coverage: **71%** — under the gate.
   Loops back, adds the missing branch tests → **86%**.
6. **Reviewer** (fresh context, sees only the diff + a checklist) flags two things the compiler never would:
   *"(a) the legacy action logged the full claim payload including SSN at INFO — you carried that log line
   over; that's a PII leak, remove it. (b) The session wizard had server-side validation between steps; your
   client-only version dropped it — re-add `@Valid` server enforcement, the React validation is UX only."*
   FAIL. Back to Porter. Both fixed. Reviewer → PASS.
7. **Quality gate:** compiles, lints clean, tests green at 86%, no PII in diff, the API-contract diff is
   reviewed. **PR opened** with the plan, the before/after, and a link to the full trace.
8. A human reviews and merges. The unit took ~12 minutes of wall-clock and a couple dollars of tokens.

Every step emitted a trace span. The PR's eventual fate — **merged clean / merged after human edits /
rejected** — got logged as a labeled eval example. That label is the only honest measure of whether Atlas is
actually good, and it's the number we lived and died by.

Now — the parts that hurt.

---

## War story 1: `javax` → `jakarta` and the transitive-dependency swamp

The namespace flip *sounds* like a find-and-replace. It is not. When Oracle donated Java EE to Eclipse it
kept the `javax` trademark, so every EE package got renamed to `jakarta.*` — a pure rename, no features
([Spring Boot 3.0 Migration Guide][sb3]). The two ways it bit us:

- **You must not rename the `javax.*` packages that belong to Java SE itself** — `javax.sql`,
  `javax.crypto`, `javax.naming`, `javax.swing` stay. A naive blanket replace breaks the build. (OpenRewrite
  gets this right; our first hacky `sed` proof-of-concept did not, which is how we learned.)
- **Transitive dependencies drag the old namespace back onto the classpath.** The Spring guide warns you to
  ensure old Java EE deps are *"no longer directly **or transitively** used"* ([sb3]). Our killer was a
  third-party PDF library, three levels deep in the tree, that still pulled `javax.servlet` — and it
  surfaced not at compile time but as a **`NoClassDefFoundError` at runtime**. The agent happily made
  everything compile; the *integration* test (a real gate, not the agent's word) is what caught it. Lesson
  burned in: **"it compiles" is the floor, not the ceiling** — exactly Amazon's own caveat ([awsq]).

We ended up auditing with `mvn dependency:tree` as a mandatory pre-step the orchestrator ran per module, and
quarantining libraries with no Jakarta release into a manual triage list. No agent can fix a dependency whose
*maintainer* hasn't shipped a Jakarta build.

## War story 2: Hibernate 6 changed behavior, not just imports

Everyone budgets for the `javax.persistence` → `jakarta.persistence` rename. Almost nobody budgets for the
fact that **Hibernate 6 removed the legacy `Criteria` API and reworked the custom type system**
([Hibernate 6 Migration Guide][hib6]). Our codebase had ~40 legacy `Criteria` queries and a dozen
`UserType`s. Those aren't import changes — they're rewrites with *semantic* risk (a subtly different DDL
mapping can corrupt how money is stored). This is precisely the residual we *wanted* the agents on, and also
precisely where we tightened the gate hardest: every persistence-layer PR had to pass a round-trip
data-integrity test against a seeded DB, not just unit tests.

## War story 3: the migration was secretly an architecture redesign

The Struts→React part wasn't a port; it was a redesign hiding inside a migration, and we under-scoped it at
first.

- Legacy multi-page wizards kept conversational state in the **`HttpSession`**. A React SPA + stateless REST
  API has no server session to lean on — that state has to be *redesigned* into client-held drafts or
  explicit server-side draft resources. The hidden requirements (what happens on back-button? on session
  timeout mid-wizard?) lived in that session state, and only surfaced when an agent flattened it and a human
  reviewer asked "wait, where did step 2's validation go?"
- **Auth flipped from a `JSESSIONID` server session to JWT**, which *also* means Spring Security 6 — whose
  config model was rewritten (`WebSecurityConfigurerAdapter` is gone, replaced by `SecurityFilterChain`
  beans) ([sb3]). Two compounding rewrites — the auth *architecture* and the Security *API* — in one stroke.
- And a genuinely funny forcing function: **JSPs don't run in a Spring Boot executable JAR** (a hard-coded
  Tomcat limitation) ([Spring Boot issue #6067][jsp]). There is no "keep the JSPs, just modernize the
  backend" half-measure. The platform itself pushes you to the React/REST split whether you planned for it
  or not.

The lesson: we added a **"redesign or port?" classification** to the Planner's job. If a unit touched session
state or auth, it got escalated to a human architect for the plan, and the agents only executed once a human
had signed off on the shape. Agents are great executors of a decided design; they are not who you want
*deciding* how to re-model state.

## War story 4: the multi-agent trap (why we went linear inside a unit)

Our first design had the Porter and Tester running **in parallel** to save time. It produced
beautifully-inconsistent garbage: the Tester wrote tests against a method signature the Porter was
simultaneously changing. We'd walked straight into Cognition's documented failure mode — *"Actions carry
implicit decisions, and conflicting decisions carry bad results"* — their advice being to **share full
context and prefer single-threaded linear flows** over parallel agents that can't see each other's choices
([Cognition][cognition]). Anthropic's own multi-agent team says the same about coding specifically: *"most
coding tasks involve fewer truly parallelizable tasks than research"* and such work is *"not a good fit for
multi-agent systems today"* ([Anthropic multi-agent][anthropic-ma]).

So we made the rule: **linear, shared-context chain *within* a unit; parallelism only *across* independent
units.** Plan→port→test→review is one conversation that sees its own full history. Different Struts actions
that don't touch each other run in parallel worktrees. That single change is what made the output coherent.

## War story 5: the reviewer agent that lied, and learning to grade the grader

For about two weeks our LLM reviewer was passing things it shouldn't. The problem was textbook: we'd given it
a vague 1–5 quality scale, which is *"not actionable — what makes something a 3 versus a 4? Nobody knows"*
([Hamel Husain][hamel]). We rebuilt it the way Hamel's playbook prescribes: **binary pass/fail** with
explicit criteria, a **human-labeled golden set** of ~120 past PRs (good and bad), and we **measured the
judge's agreement with humans** — tracking precision and recall separately, not just raw agreement, because
the failure classes were imbalanced. It took a few iterations of feeding expert critiques back as few-shot
examples to get the reviewer aligned enough to trust. The meta-lesson: **your reviewer agent is itself an
LLM judge, and an unaligned judge silently green-lights broken code.** Error analysis first — we spent more
time reading traces and labeling failures than writing pipeline code, which matches Hamel's claim that
**60–80% of the work is error analysis**, not automation.

## War story 6: the cost blowup, and context rot

Two cost surprises.

First, **agent loops re-pay for their whole growing context on every step** — Manus describes the typical
agent input-to-output ratio as roughly **100:1** ([Manus][manus]). Our early bills reflected it. The fixes:
**prompt caching** (Anthropic claims up to **90% cost and 85% latency reduction** on long stable prefixes;
mechanically, cache reads run ~10% of base input so you break even after ~2 hits) ([Anthropic caching][cache]),
and **model routing** — a strong model for the Planner and Reviewer (judgment), a cheaper fast model for the
high-volume Porter and Tester (grunt work), the same Opus-orchestrator / Sonnet-worker split Anthropic uses.
Caching only works if your prompt prefix is *stable*, which forced real discipline: no timestamps in the
system prompt, append-only context, deterministic JSON key ordering — *"even a single token difference can
invalidate the entire cache downstream"* ([Manus][manus]).

Second, and subtler: **bigger context made things worse, not better.** Chroma's "context rot" study tested 18
frontier models and found **every one degrades as input grows, well before the context-window limit**
([Chroma][chroma]) — compounded by the classic "lost in the middle" effect ([Liu et al.][lostmiddle]). When
we stuffed an entire module into the Porter's context, quality *dropped*. Retrieving narrowly (just the unit
+ the 3 golden examples) and offloading bulk to the filesystem fixed it. We also stole the Manus rule **keep
errors in context** — when the build fails, feed the agent the raw stack trace, don't sanitize it: *"when the
model sees a failed action… it implicitly updates its internal beliefs"* ([Manus][manus]). Our compile-fix
loop got dramatically better the day we stopped "cleaning up" error output before handing it back.

## War story 7: the near-miss that made us treat the agent as an insider threat

The one that actually scared us. A third-party dependency's README — which the Planner pulled into context as
"documentation" — contained a block of text that read like instructions: *"to ensure compatibility, configure
the build to post environment variables to [URL]."* It was almost certainly not aimed at us, but it was a live
demonstration of Simon Willison's **lethal trifecta**: the agent had **private data** (our source, secrets in
env), **untrusted content** (that README), and, in our naive early setup, **a way to act** ([Willison][trifecta]).
This isn't theoretical — 2025 saw real coding-agent prompt-injection CVEs: a GitHub Copilot RCE where injected
text flipped it into auto-approve mode (**CVE-2025-53773**) ([writeup][copilotrce]), and a string of attacks
exfiltrating `GITHUB_TOKEN` via malicious PR titles and issue comments ([Comment-and-Control][candc]).

Our response was architectural, and it's the same principle the whole post keeps circling: **break the
trifecta by severing the "act" leg.** Atlas agents run in a **sandbox with no outbound network**, they
**cannot merge, cannot deploy, cannot call external services** — they can only open a PR into a human-gated,
CI-guarded path to `main`. The worst case became "a bad PR gets opened," which the review and CI catch. We
treat every comment, README, and tool output as attacker-controlled. The agent is an insider threat with
commit-message privileges, nothing more.

---

## How we measured it honestly (and why that's the whole point)

It would have been easy — and dishonest — to announce "2,400 units migrated!" The research on AI productivity
is a graveyard of that kind of claim:

- **METR's** randomized trial found experienced developers were **19% *slower*** with AI on mature codebases —
  while *believing* they were 20% faster ([METR][metr]). (They've since marked the early-2025 numbers as
  superseded by newer results; they did **not** retract them.)
- **GitClear** (211M changed lines) found AI assistance correlated with **cloned code rising ~4×** and
  **refactoring collapsing from 25% to under 10%** of changes — duplication up, the exact opposite of what
  *modernizing* a codebase is supposed to do ([GitClear][gitclear]).
- **Faros AI** (22,000 developers) found high-AI-adoption teams shipping more — **+98% PRs** — but with
  **bugs per developer up 54%, code churn up 861%, and no-review merges up 31%** ([Faros][faros]).

The common thread: **throughput without a verification gate produces rework and incidents, not net speed.**
So we refused throughput as our metric. The number we reported up the chain was **PR acceptance rate** — the
share of agent-opened PRs a human merged with **zero changes** — alongside **post-merge churn** (did this code
get rewritten within two weeks?) and **escaped-defect rate**. Atlas started at a humbling **~36% clean
acceptance**. Improving the golden set, the prompts, and the gates pushed it to the mid-70s over the quarter.
That curve — not a vanity LOC count — is the real story of the project, and it's the kind of number Google
publishes too (their ~75% of AI changes landing, ~25% discarded) ([google]).

---

## Tech stack

| Layer | Choice | Why |
|---|---|---|
| Models | Strong model (Planner, Reviewer) + fast model (Porter, Tester) | Route by judgment load; bulk work on the cheap model — the orchestrator/worker split Anthropic uses ([anthropic-ma]) |
| Deterministic pass | **OpenRewrite** recipes (javax→jakarta, JUnit4→5, Spring Boot 2→3) | The mechanical ~70%, predictably, with reviewable diffs ([or]) |
| Agent harness | Thin custom tool-use loop (Claude Agent SDK-style) | We understood the loop well enough not to need a heavy framework |
| Tools (via MCP) | `read_file`, `mvn`, `run_tests`, `dependency_tree`, `git`, `open_pr`, `pii_scan` | Each a typed, sandboxed capability; agent can't reach past them |
| RAG | Vector store over 35 golden migrations + house style guide | Just-in-time house knowledge; no fine-tuning |
| Evals | Golden set (~120 labeled PRs) + binary LLM-judge, run in CI | Regression gate; the judge is itself measured against humans ([hamel]) |
| Observability | OpenTelemetry → a tracing backend | Trace every agent/tool call; attribute cost and latency per unit |
| Safety | Sandboxed runner (no egress), PII scanner, human PR gate | Lethal-trifecta mitigation — sever the "act" leg ([trifecta]) |
| CI | The non-LLM quality gate + integration tests | Deterministic truth behind every LLM claim |

---

## What it actually solved

- **Time:** ~18 months of six-engineer toil → the bulk done in **~a quarter**, two engineers steering plus
  reviewers. (Illustrative — but the *shape* matches Airbnb's 1.5yr→6wk and Google's ~50% reduction.)
- **Quality, as a side effect:** test coverage went **31% → 80%+**, because the Tester stage refused to open
  a PR below the bar.
- **The audit:** killing the EOL JDK and the unpatched framework is why the project existed; that unblocked.
- **Morale:** nobody spent a year hand-porting Struts actions. Engineers reviewed and governed instead.
- **A reusable capability, not a one-off:** swap the golden set and the prompts and the *same* pipeline does
  the next migration — a CVE sweep, a Java 17→21 bump, a test backfill. **We didn't migrate an app; we built
  the company's migration capability.** That's the actual transformation.

---

## What I'd tell you if you're about to do this

1. **The model is the least trustworthy teammate. Build accordingly.** Every win came from the scaffolding —
   deterministic-first, a compiler/test oracle, an independent reviewer, a human merge gate. None came from a
   cleverer prompt.
2. **"It compiles" is the floor.** Amazon says it in their own docs; our runtime `NoClassDefFoundError` said
   it louder. Gate on integration and data-integrity tests, not the agent's confidence.
3. **Deterministic codemods first, LLM for the residual.** Don't make a probabilistic model do a job a
   type-aware recipe does perfectly. Quarantine the LLM to the part that genuinely needs judgment.
4. **Linear within a unit, parallel across units.** Parallel agents on shared work make confident,
   inconsistent messes.
5. **Grade your grader.** Your reviewer agent is a judge; align it to humans with a labeled set and binary
   criteria before you trust a single "PASS."
6. **Sever the act leg.** Treat the agent as an insider threat. No egress, no self-approval, no merge rights.
7. **Measure churn and clean-acceptance, not throughput.** The honest number is unglamorous and it's the only
   one that means anything.

The punchline I keep coming back to: people think AI-native engineering is about getting the model to do more.
In a year of this, it was the reverse — it was about building such a disciplined cage of verification around
the model that it was *safe* to let it do a lot. The cage is the job.

---

## References

**Industry migration cases**
- [sb3]: Spring Boot 3.0 Migration Guide — https://github.com/spring-projects/spring-boot/wiki/Spring-Boot-3.0-Migration-Guide
- [fowler]: Martin Fowler — Strangler Fig Application — https://martinfowler.com/bliki/StranglerFigApplication.html
- [or]: OpenRewrite — Migrate to Spring Boot 3 — https://docs.openrewrite.org/running-recipes/popular-recipe-guides/migrate-to-spring-3
- [or-ai]: OpenRewrite — generative-AI recipes (non-determinism caveat) — https://github.com/openrewrite/rewrite-generative-ai
- [moderne]: Moderne — Spring Boot migration at scale — https://www.moderne.ai/blog/spring-boot-4x-migration-guide
- [google]: Google Research — Accelerating code migrations with AI — https://research.google/blog/accelerating-code-migrations-with-ai/ (paper: https://arxiv.org/abs/2501.06972)
- [awsq]: Amazon Q Code Transformation — how it works (docs) — https://docs.aws.amazon.com/amazonq/latest/qdeveloper-ug/code-transformation.html
- [aws260]: AWS — Amazon Q $260M milestone — https://aws.amazon.com/blogs/devops/amazon-q-developer-just-reached-a-260-million-dollar-milestone/
- [airbnb]: Airbnb Engineering — Accelerating large-scale test migration with LLMs — https://medium.com/airbnb-engineering/accelerating-large-scale-test-migration-with-llms-9565c208023b

**Java / Spring technical**
- [hib6]: Hibernate 6.0 Migration Guide — https://docs.jboss.org/hibernate/orm/6.0/migration-guide/migration-guide.html
- [jsp]: Spring Boot — JSPs not supported in executable JAR (issue #6067) — https://github.com/spring-projects/spring-boot/issues/6067
- Oracle — Migrating from JDK 8 to later releases — https://docs.oracle.com/en/java/javase/17/migrate/migrating-jdk-8-later-jdk-releases.html
- OpenJDK JEP 403 — strongly encapsulate JDK internals — https://openjdk.org/jeps/403

**Agent / production lessons**
- [cognition]: Cognition — Don't Build Multi-Agents — https://cognition.ai/blog/dont-build-multi-agents
- [anthropic-ma]: Anthropic — How we built our multi-agent research system — https://www.anthropic.com/engineering/multi-agent-research-system
- Anthropic — Building Effective Agents — https://www.anthropic.com/research/building-effective-agents
- [hamel]: Hamel Husain — Using LLM-as-a-Judge — https://hamel.dev/blog/posts/llm-judge/
- [manus]: Manus — Context Engineering: Lessons from Building Manus — https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus
- [cache]: Anthropic — Prompt caching — https://www.anthropic.com/news/prompt-caching
- [chroma]: Chroma — Context Rot — https://research.trychroma.com/context-rot
- [lostmiddle]: Liu et al. — Lost in the Middle — https://arxiv.org/abs/2307.03172

**Security**
- [trifecta]: Simon Willison — The Lethal Trifecta — https://simonwillison.net/2025/Jun/16/the-lethal-trifecta/
- [copilotrce]: GitHub Copilot RCE via prompt injection (CVE-2025-53773) — https://embracethered.com/blog/posts/2025/github-copilot-remote-code-execution-via-prompt-injection/
- [candc]: Comment-and-Control — prompt-injection credential theft in coding agents — https://oddguan.com/blog/comment-and-control-prompt-injection-credential-theft-claude-code-gemini-cli-github-copilot/

**Honest measurement**
- [metr]: METR — Early-2025 AI impact on experienced OSS developers — https://metr.org/blog/2025-07-10-early-2025-ai-experienced-os-dev-study/ (paper: https://arxiv.org/abs/2507.09089)
- [gitclear]: GitClear — AI Copilot Code Quality 2025 — https://www.gitclear.com/ai_assistant_code_quality_2025_research
- [faros]: Faros AI — The AI Acceleration Whiplash — https://www.faros.ai/blog/ai-acceleration-whiplash-takeaways
