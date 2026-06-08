# Key Decisions

Context → Decision. The "why the alternative lost" is the reusable part.

## Build a harness, not stay a pure-instruction skill
**Context:** the `dev` skill's judgment layer (heuristics) is good; the *orchestration* layer is advisory
prose the model can ignore — that's the ceiling.
**Decision:** build a harness where process is load-bearing control flow. The skill's `references/*` port as
the judgment layer.

## The Align gate is the core primitive — VALIDATED
**Context:** friction #1/#5/#6 (retyping "gather context, confirm before acting"); need it enforced, not asked.
**Decision:** a gate that blocks execution tools until a plan is confirmed. Proven at runtime in the spike
(`research/07`): a `tool_call` hook returning `{block}` denied write/bash in align phase, `/align` unlocked,
phase persisted across reload. The architecture's keystone.

## Memory = curated wiki + decaying sidecar, local embeddings
**Context:** claude-mem worked but burned usage as memory grew (friction #4).
**Decision:** CoALA split — wiki (semantic+procedural, never decays) + episodic sidecar (decays), embeddings
on local oMLX (~zero cost). Top-k retrieval w/ progressive disclosure. Detail: `research/01`.

## Test quality is a number (mutation), not a review
**Context:** stopped reviewing AI-written tests → the adversarial-review-of-tests became theater (friction).
**Decision:** a **Harden gate** between Test and Implement, gated on **mutation score**, mutator on a
different provider than the author. Runs *before* implementation so tests can't be shaped to the code.
Detail: `research/03`.

## Self-evolution under constitution / case-law
**Context:** manual evolve, and lessons over-fit to the specific problem (friction #3).
**Decision:** immutable constitution (spine, integrity, gates, the eval metric) vs agent-editable case-law
(heuristics, prompts, lessons); a **generalization filter** (two-column rewrite, commit only the general
column); git-versioned + human-gated. Detail: `research/04`.

## Model-per-role with local oMLX
**Decision:** recon→local/Haiku, build→Sonnet, coordination+adversarial→Opus, test-author vs mutator on
different providers. Validated: oMLX drops into Pi via `openai-completions` + `baseUrl`.

---

## OPEN — Foundation decision (under eval, `research/10`)
**Language/runtime strategy:** A (full Rust-native, own-core) / B (Rust core + TS orchestration, the omp
model) / C (stay TS-on-Pi). Plus base reuse list (DR1 = omp teardown) and board (DR2 = beads schema).
Gary's lean: Rust-native, *to be pressure-tested*. Gates Phase 0. → `research/11-foundation-decision.md`.

---

## Rejected Approaches

### claude-mem as-is
**Why considered:** capture→compress→inject memory for coding agents.
**Why rejected:** runs an API call per observation → cost grows with memory. We move compression/embeddings
to local oMLX. (Note: local fixes *compression* cost, not *injection* cost — retrieval discipline still required.)

### Whole-store memory injection
**Why rejected:** every recalled token competes with task reasoning. Top-k + progressive disclosure + project
scoping instead.

### Coverage as the test-quality gate
**Why rejected:** 100% line coverage with zero meaningful assertions is trivial. Meta ACH data: 49% of
fault-catching tests added zero line coverage. Mutation-gating instead.

### Symphony's full-autonomy posture
**Why considered:** Symphony is our closest work-model blueprint.
**Why rejected:** `approval_policy: never` / "user-input = hard failure" is the opposite of our gated-interactive
model. We borrow its structure (board/workpad/state-machine/land) but invert the human-gate valence.

### openlumara as a memory source
**Why rejected:** flat MessagePack save/recall, no embeddings or top-k — below our P3 bar. Reference its
module-toggle ergonomics at most.

### beads / repowise as runtime dependencies (provisional, pending DR2)
**Why considered:** beads = purpose-built agent board; repowise = code-health intelligence.
**Why leaning rejected:** beads is Go, repowise AGPL — under a Rust-native core we **steal the schema/patterns**
rather than run the binaries. To be confirmed by DR2.
