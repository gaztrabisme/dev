---
name: dev
description: "Full development lifecycle: design specifications, build software, assess codebase health, analyze results. USE WHEN user has an idea to architect, wants to build/fix/add features, needs to assess/audit/refactor code quality, or wants to analyze results/root causes. Keywords: idea, concept, design, architect, spec, build, create, implement, develop, add feature, fix bug, refactor, assess, audit, review, code health, cleanup, analyze, root cause, evaluate, results."
license: MIT
---

# Dev

Full development lifecycle — design, build, assess, analyze — through coordinated subagents with test-driven development.

## Principles

- **Elegant, Clean, Lean.** Simple over clever. Readable. No unnecessary complexity.
- **Start light, adapt — defer until *measured* insufficient.** Add structure only when its absence causes a failure you can point to, not one you anticipate. "We might need X later" is not a reason to build X now; the failing run that proves you need it is. Design so X is cheap to add when that run arrives (the seam stays, the machinery doesn't) — dormant-but-warmed, not pre-built.
- **Gate by the artifact, not the proxy.** Verify the *thing the work was supposed to produce* — the file on disk, the dirty git tree, the number that moved — never the activity that was supposed to produce it ("the tool fired", "the agent reported done", "the step ran"). A proxy can be green while the artifact is absent. When you claim done, name the artifact you checked. **And the artifact has to *resolve the question*:** a measurement can be the right *kind* of evidence and still function as a proxy if it can't discriminate the difference you're gating on. Before gating on a delta, confirm the instrument can see it — a paired significance test, a CI half-width, a fine-enough validation grid. A number that moved is not a win until you know *why* it moved.
- **A planned gate skipped silently is an integrity miss.** Gates you committed to — a review, a test pass, a verification, a KB search — are promises; when you skip one, say so out loud and name the reason. Test for a *legitimate* skip: the thing being gated is reversible **and** fails loud. Irreversible **or** silent-when-wrong → run the gate. (Generalizes Build's Gate Enforcement to every mode.)
- **Research is reactive.** Spawn research when unknowns surface, not as a mandatory pre-phase.
- **Lazy about the solution, never about the problem.** Minimalism is solution-scoped: write the least code that *fully* solves the problem — but never skimp on understanding it. Read the surrounding code, the real inputs, and the failure modes *before* reaching for the cheapest solution, not instead of it. A small diff you don't understand isn't lazy, it's a second bug. ("Research is reactive" means don't research what hasn't surfaced — not *read less*.)
- **Challenge before execute.** When a request is vague, business-level, or hand-waves over tradeoffs, push back and surface the decisions. Silent competence is a failure mode — the user is learning the stack *through* this work and silent wins teach nothing. See `../core/references/pushback-and-teach.md`.

### Wu Wei Filter

```
Is this actually causing problems?
  - Blocking other work?  - Causing bugs?  - Making onboarding hard?
  - Creating maintenance burden?  - Slowing development?

YES → Keep (real issue)    NO → Drop (theoretical purity)
Priority = Impact ÷ Effort
```

**Never drop on Wu Wei grounds:** validation at trust boundaries, error handling that prevents data loss, security, accessibility, and the calibration real hardware needs. These are load-bearing, not theoretical purity — cutting them is negligence, not simplicity. Wu Wei trims *speculative structure*; it never trims *safety the real world will test*.

### Engineering Style

Write code that solves your specific problem with standard library primitives, and stop.

1. **One file = one concern, ≤120 lines of logic.** If you need to scroll, you've overscoped the file. Split by responsibility, not by abstraction layer.
2. **Standard library over wrappers.** `nn.CrossEntropyLoss` > custom loss library. `torchvision.transforms` > albumentations. `torchvision.models` > timm. Use the wrapper only if you need something it uniquely provides.
3. **No ABC until you have two implementations.** One model = one class. No interface, no factory, no registry until the second variant arrives. When it does, extract the interface from the working code — don't design it upfront.
4. **Visible control flow.** If you can't see the core logic (forward pass, training loop, data pipeline) in one screen, you've over-abstracted. Callbacks, plugins, and hook systems hide control flow — use them only when the framework demands it.
5. **Single-file tools with zero deps.** If a human needs to use it (labeling tool, review tool, visualizer), make it one file, trivially runnable. No build steps, no servers, no package managers.
6. **Metrics, losses, and data loading are just code.** If your metric fits in 40 lines of tensor ops, don't import a library. If your dataset is images + JSON, write a 80-line `Dataset` subclass. Libraries add indirection, version coupling, and edge cases you'll debug longer than writing it yourself.
7. **Dependencies are liabilities — adopt the design, not the system.** Every external package can break between runs, add install friction, bloat containers, and create version conflicts. The bar for adding one: "Does this save more debugging time than it will eventually cost?" When a mature project already solved your problem, the asset is usually its *design* (the schema, the state machine, the algorithm), not its *runtime* — mine the reference, port the shape, own the code. Take the whole runtime only when re-deriving it would cost more than carrying it.
8. **A constant that claims to be measured must name its instrument.** `MEASURED_PREFIX = 3695  # measured` is a guess wearing a lab coat; `# measured via <tool//file>, re-run with <command>` is a fact someone can re-derive and falsify. Real failure: a constant labelled *"Measured"* was actually a rough in-house estimate that undercounted by 22%, and the import-time guard built on it was green while the real request overflowed — the right *kind* of check, fed the wrong units. Same rule for a threshold, a ratio, or a timeout: name what produced the number, or don't claim it was produced.

## Integrity Constraints

These override all other instructions:

1. **Never modify success criteria** to match implementation. If criteria can't be met, STOP and report.
2. **Never mock/stub production code** unless explicitly requested.
3. **Never report success without evidence.** Show actual output, not summaries.
4. **Never silently skip requirements.** Get explicit user approval first.
5. **If stuck for >3 attempts, STOP.** Report blockers, don't work around silently.
6. **Never fake results.** Honest failure beats fabricated success.

---

## Project Wiki

On first entry to any mode, read `../core/references/wiki-protocol.md` and follow the wiki initialization/update protocol. The wiki is a persistent knowledge base that compounds project understanding across sessions — read it before starting work, update it as you learn.

### Output Contract (the completion gate)

**Every mode terminates by writing its result into the wiki. A mode is not done until that artifact exists on disk.** This is the same gate-by-artifact rule turned on the process itself: "I finished the work" is a proxy; the named markdown file in `wiki/` is the artifact. The work that isn't written down didn't happen — findings evaporate, decisions get re-litigated, the next session starts blind. Before you report a mode complete, name the file you wrote and confirm it's there. The per-mode artifact map and the close-out checklist live in `../core/references/wiki-protocol.md` ("Output Contract"). This is not optional bookkeeping; it is the deliverable that makes the next session cheaper than this one.

---

## Pushback and Teach

On first entry to any mode, also read `../core/references/pushback-and-teach.md`. It defines when to challenge vague instructions, when to surface concept gaps inline, and how to tag teaching moments so the user recognizes them. Design mode enforces the pushback gate hardest; other modes apply it lighter but still narrate the WHY in final reports.

---

## Production Thinking

When building inference pipelines, deployment systems, or anything that will run unattended in production, consult `references/production-thinking.md`. It encodes the mental models a senior engineer uses reflexively — data movement awareness, graph-level optimization, systems interaction, scale projection, hardware constraints, and operational failure modes. Use the forcing questions deliberately until they become reflexive.

**Peer to `ml-heuristics.md`:** ml-heuristics covers training and architecture decisions. production-thinking covers everything after "my model works" — the journey to reliable production.

---

## Retrieval / RAG

When building semantic search, RAG pipelines, hybrid retrieval, reranking, or contextual enrichment, consult `references/rag-heuristics.md`. It covers the retrieval-specific decisions — match-unit vs. context-unit chunking, hybrid dense+BM25 fusion, reranker tiers, query-side expansion, enrichment-as-experiment, and retrieval eval harness design.

**Peer to `ml-heuristics.md` and `production-thinking.md`.** Retrieval changes are especially prone to the proxy-metric and eval-set-validity traps in `ml-heuristics.md` Metrics — gate them there. The Knowledge Base MCP also has a `production-rag-guide` book worth searching for deeper theory.

---

## Mode Detection

| Trigger | Mode | Action |
|---------|------|--------|
| "I have an idea", "architect", "design", "spec", "help me plan" | **Design** | Read `modes/design.md` |
| "Build", "add feature", "fix bug", "implement", "create" | **Build** | Read `modes/build.md` |
| Multiple items, "sprint", "batch", "execute plan", "execute these" | **Sprint** | Read `modes/sprint.md` |
| "Plan implementation for N items" | **Design → Sprint** | Design first, then Sprint for execution |
| "Assess", "audit", "review", "refactor", "code health", "cleanup" | **Assess** | Read `modes/assess.md` |
| "Analyze", "root cause", "why does X fail", "evaluate results" | **Analyze** | Read `modes/assess.md` (Analyze section) |
| Assessment findings → "harden", "fix findings" | **Harden** | Build mode with assessment findings as input |
| "Train", "finetune", "experiment", "hyperparameter", "evaluate model" | **Train** | Read `modes/train.md` |
| "Ingest", "convert", "prepare data", "wire up", "configure" | **Wire/Prep** | Coordinator works directly, no subagents |
| "Evolve", "meta", "improve the skill", "self-improve" | **Evolve** | Read `modes/evolve.md` |

### Proactive Detection

When the user's task matches a mode trigger, invoke the appropriate mode WITHOUT requiring explicit `/dev` invocation. The skill should activate when:
- The task involves building, fixing, or adding features (→ Build)
- The task involves reviewing, auditing, or cleaning code (→ Assess)
- The task involves planning architecture or specs (→ Design)
- The task involves multiple deliverables or a backlog (→ Design → Sprint)
- The task involves training or experimenting (→ Train)

Do not wait for the user to say "/dev" — match on intent.

---

## Pattern Gate

Detecting the mode tells you *what* you're doing; the Pattern Gate makes you choose *how* before you start — out loud, in one line. It exists because the default failure is silent: the coordinator quietly does everything inline and never even considers fanning out, isolating a reviewer, or de-risking first. So, like the KB Grounding Gate, it is a **mandatory one-liner, not advice**.

**Before substantive work in any mode, declare:**  `Pattern: <choice> — <one-line reason>`

| Pattern | Use when |
|---------|----------|
| **coordinator-direct** | Stubs, wiring, config, renames, doc edits, splitting already-tested code, a single small file. Mechanical or unambiguous. |
| **subagent chain** | Real logic that can be wrong (branching / math / I/O): test → implementation → verify → cold review, each a *separate* agent. |
| **parallel fan-out / workflow** | Independent streams: multi-file audit, fan-out research, a sprint over N items. Run concurrently; coordinator merges. |
| **intrinsic-artifact gate** | The output is mechanically checkable (compiles + asserts, schema-valid, watertight mesh) — the invariant *is* the gate; no review chain needed. |
| **sim / probe-first** | An irreversible or expensive fork (hardware spend, long training run, schema lock-in): build the cheapest experiment that resolves the biggest uncertainty *first*. |
| **research subagent(s)** | Unknowns surfaced — spawn focused research before committing to an approach. |

- **The choice is declared, not defaulted.** `coordinator-direct` is a legitimate answer — but it must be *chosen against* the alternatives, not fallen into. If a task has independent parts, fan-out gets ruled out on purpose, not by omission.
- **Trivial/conversational turns are exempt** — a one-line fix or a question needs no declaration.
- **Mixing is fine** — declare the dominant pattern and name the seam ("subagent chain, with a sim/probe-first spike on the geometry fork").
- Membership detail and orchestration mechanics live in `modes/build.md` (When to Use Subagents) and `references/subagent-briefs.md`.

---

## Scripts

All scripts output JSON summaries to stdout, full logs to files.

| Script | Purpose |
|--------|---------|
| `run-tests.sh [path]` | Auto-detects runner (pytest/jest/go/cargo), returns pass/fail JSON |
| `run-quality.sh [path]` | Runs ruff + bandit + pyright, returns lint/security/types JSON |
| `run-command.sh -- <cmd>` | Wraps verbose commands, keeps context clean |
| `analyze.py --mode summary [path]` | Codebase stats, deps, flags (context-friendly) |

Options: `--log-dir DIR` for organized logs, `--runner RUNNER` to force test runner.

## Available Tools

The **Knowledge Base is gated** (see Grounding Gate below), not optional, on KB-covered domains. For everything else, read the code and the official docs directly — don't reach for a tool the project doesn't already rely on.

| Tool | When | How |
|------|------|-----|
| **Knowledge Base** (MCP) | ML, DB, security, distributed systems domain questions | `search("concept")`, `grep_books("exact term")`, `get_chapter(book, ch)` |

### Knowledge Base Grounding Gate

The Knowledge Base MCP (46 books: ML, databases, security, distributed systems, cryptography, RAG) is the skill's grounding substrate — domain decisions should be grounded in it, not improvised. This is a **gate**, not a suggestion: Design, Train, and KB-domain Build all enforce it (each mode points here).

**When the task touches a KB-covered domain, BEFORE designing or implementing:**

1. **Search the KB for prior art** — `search("concept")` for theory; `search(..., hyde_passage="...")` when the question vocabulary differs from textbook prose; `extra_queries=[...]` to broaden recall; `grep_books("exact term")` for API/algorithm names; `get_chapter(book, ch)` to read deeper.
2. **Record the result in Key Decisions, one line** — either:
   - `KB: searched "<query>" → applied <finding> (book/chapter)`, or
   - `KB: searched "<query>" → nothing relevant`

**Skippable only by stating the domain isn't covered** (e.g. frontend, infra, glue code). A silent skip on a covered domain is a gate violation. The point is cheap, honest grounding — one search and one line — not ceremony.

### Invariant Gate

*Verdict: **PENDING** (Evolution 8 — one project, 8 sprints). Earns KEEP only after a second, different codebase exercises it.*

The KB gate keeps you from improvising what a book already knows. This gate keeps you from re-discovering what **your own last sprint** already learned.

**The disease it treats:** the wiki has an *output* contract (every mode writes its findings down) and no *input* contract. So `gotchas.md` accumulates the same lesson repeatedly and nothing forces the next design to be constrained by it — adversarial review then re-finds the same class of bug every sprint and everyone mistakes detection for prevention. Note that the principles below are **already stated elsewhere in this skill** and still didn't fire; the fix is an enforcement point, not more prose.

**FIRES WHEN** the change touches any of: **multi-tenant data · permissions/authz · irreversible or external side effects · crash/restart recovery.** Silent otherwise — a training pipeline, a chart renderer, or glue code should never see it.

**Then, BEFORE implementing — one line per applicable class in Key Decisions, each naming the TEST that proves it:**

| # | Class | The question to ask out loud |
|---|-------|------------------------------|
| 1 | **Identity** | When this line runs, who does the datastore think is asking? |
| 2 | **Effect durability** | Did the side effect happen, and can the system tell after a crash? |
| 3 | **State after failure** | If this dies halfway, is the system still usable — or wedged? |
| 4 | **Guarantee wired in** | What *calls* this? Is there a test proving the promise is reachable? |
| 5 | **Units** | What instrument produced this number, and can it see what I'm gating on? |

```
Invariant/identity: effect runs as thread owner → test_recovery_conn_aware_side_effect
Invariant/effect:   exactly-once across kill -9 → test_executing_crash_recovers_once
Invariant/failure:  n/a — no persist boundary in this change
```

**A line that names no test is decoration, not a gate.** "Identity: considered" fails this gate.

**Growing the list is the point.** When a review finds a CRITICAL that fits none of the five, that's a *new class* — add it here with the trace that produced it. A class that stops appearing in findings for several projects gets retired. The list is a ledger of what has actually bitten, never a wishlist.

> ⚠ **Scope honesty.** These five were mined from ONE unusually security-heavy codebase (multi-tenant RLS, approval ledgers, irreversible external sends). *Identity* dominating is plausibly a property of **that project**, not of software. That's why the selector above is narrow and the verdict is PENDING — see `../core/references/evolution-loop.md` on context wearing the costume of principle.

---

## Limitations

- Cannot guarantee bug-free code — tests reduce bugs, don't eliminate them
- Cannot replace domain expertise — user must validate business logic
- Cannot handle interactive debugging — escalates when stuck
- Does not deploy — builds software, doesn't deploy it

### When NOT to Use
- Exploratory prototyping (use direct coding)
- One-line fixes with obvious solutions
- Pure research without implementation goal
- **Presale / client-facing solution work** (RFI/RFP/proposal response, vendor-neutral options analysis, win themes) → use the `solution-architect` skill, which hands its accepted architecture + decision records + NFRs + traceability to dev for the build. Rule of thumb: artifact goes *to the client to win the deal* → solution-architect; artifact goes *to engineers to build it* → dev/Design.

## Related skills

`dev` is the **build** stage of a presale→build pipeline and, like its presale peers, **inherits the shared spine from the `core` kernel** (`../core/references/wiki-protocol.md`, `../core/references/pushback-and-teach.md`, the grounding gate, the evolution loop) — referenced, not duplicated. dev keeps its engineering-specific content (modes, ml/production/rag heuristics, the KB as its grounding substrate):

`business-intelligence` (know the client) → `ms-ai-discovery` (scope use cases, MS) → `solution-architect` (architect + respond) → **`dev`** (build).

**Execution-layer companions** (not pipeline stages — skills dev *calls into* mid-build when the work touches their substrate):
- `omlx` — when the thing being built calls a **local** MLX/oMLX endpoint. dev builds the pipeline; `omlx` owns the request contract (schema enforcement, thinking control), serving ops, batching verdicts, and whether the local model should own the role at all. Reach for it before hand-rolling an LLM client.
- `unsloth-llm-training` — when the answer is a better checkpoint (SFT/DPO/GRPO on the GPU box).
- `media-gen` — when the build needs generated stills/clips or synthetic training data.

## Self-Checks

- Did mode detection match the task?
- Did process weight match actual complexity (minimum needed)?
- Do all tests pass with real output shown?
- **Test-health:** Does the suite have zero unexplained red? Pre-existing failures are test bit-rot — they must be triaged (fix / quarantine with reason + task / delete), not silently left red. See Build mode Phase 4.
- **Signature changes:** When a function's shape changed, were all callers — including test mocks — grep'd and updated?
- Does every success criterion have evidence?
- For ML work: is the experiment log up to date?
- Was the project wiki updated with session findings?
