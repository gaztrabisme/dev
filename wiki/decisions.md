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

## Foundation: Rust-native own-core — RESOLVED (signed off 2026-06-08)
**Context:** the language/runtime strategy gated Phase 0; pressure-tested via DR1 (oh-my-pi, `research/11`),
DR2 (beads, `research/12`), DR2-prime (beads_rust, `research/15`); synthesized in `research/13`.
**Decision:** **Strategy A — full Rust-native own-core.** Own agent loop + tool dispatch + a typed `enum Phase`
Align gate; **lift `crates/pi-iso`** (oh-my-pi) for worktree isolation; board hand-rolled on **`rusqlite`
(SQLite + JSONL)**, **porting `beads_rust`'s `close_policy.rs` state-machine + gate engine** (~250 lines);
minimal Rust provider layer (oMLX + Anthropic, ~1.5k LoC). Mine pi-mono/omp/symphony/beads for designs; depend
on none. **Phase 0 opens with a bounded sizing spike** (own loop + 2 providers + Rust gate + pi-iso → oMLX)
before the trunk; fallback if sizing balloons = C (stay TS-on-Pi).
**Why B/C lost:** B (Rust core + TS brain, the omp model) — DR1 found omp's agent loop/tools/providers/gate are
100% TS, so B hands us none of A's hard parts and couples us to a solo fork; the one severable Rust asset
(`pi-iso`) we lift anyway. C (stay TS-on-Pi) — couples us to two upstreams forever and forfeits the
compiler-as-guardrail the self-building endgame wants.

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

### oh-my-pi as the base (Strategy B) — REJECTED
**Why considered:** omp is a maintained superset of pi-mono with a ~27k-LoC Rust core; looked like it'd hand us
~60% of the Execution + Memory planes for free.
**Why rejected (DR1, `research/11`):** the Rust core is **leaf systems-primitives only** — the agent loop, tool
execution, providers, and the gate are 100% TypeScript. Building on omp buys none of the hard parts (we'd
reimplement them under a Rust core anyway) and couples us to a solo fork. We **lift only `crates/pi-iso`**
(cleanly severable) and own the rest.

### beads / beads_rust (`br`) / repowise as runtime dependencies — REJECTED
**Why considered:** beads + br = purpose-built agent boards; repowise = code-health intelligence.
**Why rejected (DR2 `research/12`, DR2-prime `research/15`):** beads is Go + Dolt; **`br` is welded to `fsqlite`**
(single-maintainer *alpha* pure-Rust SQLite) + pinned nightly + ~180k LoC we'd use ~10% of; repowise is AGPL.
Under a Rust-native core we **steal the designs** — port br's `close_policy.rs` gate engine + its simpler edge
table onto our own `rusqlite` board — and depend on none. (br's gate engine corrected DR2's assumption that
beads had no state machine: br has one, already built + tested.)
