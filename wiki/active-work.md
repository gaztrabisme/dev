# Active Work

## Foundation Eval (pre-Phase-0)
**Status:** Scoped, ready to run
**Started:** 2026-06-08
**Goal:** Lock the foundation that gates Phase 0 — the language/runtime strategy (A/B/C) + base reuse list +
board schema — so we don't build the trunk on the wrong substrate.

### Current State
- Research foundation complete (`research/01..08`), design v2 written, Align-gate spike validated on Pi (TS).
- 13 external references oriented (`research/09`); two are foundation-level: **oh-my-pi** (Rust-core superset
  of pi-mono) and **beads** (agent board).
- Gary set the direction: **Rust-native**, forever-personal, local-first → flips most refs INTEGRATE→STEAL;
  omp becomes a Rust *reference*, beads becomes *schema to extract*.
- Eval scoped (`research/10`): two bounded teardowns + a synthesis decision.

### Next Steps
- [ ] **DR1** — clone `can1357/oh-my-pi`; map its Rust/TS boundary (is the agent loop + gate in Rust?), test
      severability of `pi-iso`, reality-check the provider surface, port the gate sketch → output A/B/C + reuse list.
- [ ] **DR2** — clone `steveyegge/beads`; extract the full data model (IDs, edge types, status, `ready`-query,
      compaction/decay); map status model vs our 7-state spine → output a Rust-ready board schema + adopt/steal/hand-roll.
- [ ] **Synthesis** — `research/11-foundation-decision.md`: lock language strategy + base + board; update
      design-v2 §2/§10.
- [ ] Then **Phase 0** — thin trunk slice on the locked foundation.

### Breadcrumbs
- 2026-06-08: Spike proved the gate concept in TS on pi-mono (v0.78.1); the spike is a *port reference* if we go Rust, not wasted.
- 2026-06-08: omp ships ~60% of Execution+Memory (worktree `task` subagents, Hindsight memory, stream rules) and has a ~27k-LoC Rust core → the key case study for strategy B (hybrid) vs A (full Rust).
- 2026-06-08: Spike code + pi-mono clone live OUTSIDE this repo (`Skills/harness-spike/`, `Skills/pi-mono/`). When Phase 0 scaffolds real code, the harness likely gets its own repo and the wiki travels with it.

## Completed
- Research foundation (6 breadth tracks) — `research/01..06`
- Design v2 — `research/00-design-v2.md`
- Pi spike (Align gate validated) — `research/07`
- Symphony teardown — `research/08`
- Additions orientation + eval scope — `research/09`, `research/10`
