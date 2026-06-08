# Wiki Log

## [2026-06-08] session | Wiki init + foundation eval scoped
- Initialized the project wiki (index/architecture/decisions/active-work/log) per the dev skill wiki-protocol.
  Wiki is the navigable memory layer over the detailed `research/` docs.
- Project state at init: pure-instruction `dev` skill → being re-platformed as a Pi-based (now possibly
  Rust-native) personal harness.
- Prior work this arc (all committed under `research/`):
  - Six breadth research tracks (memory, orchestration spine, test-hardening, self-evolution, parallel
    exploration, Pi internals) → `01..06`.
  - Design v2 — all six v1 open questions resolved → `00-design-v2.md`.
  - Pi spike — cloned pi-mono, wired oMLX, wrote a `tool_call` Align gate, **validated at runtime** (blocks
    write/bash in align phase; `/align` unlocks; phase persists across reload) → `07-pi-spike.md`.
  - Symphony Elixir teardown (coordinator mechanics) → `08-symphony-teardown.md`.
  - Oriented 13 external references; verdicts → `09-additions-orientation-and-plan.md`.
- **Direction shift this session:** Gary set the harness as **Rust-native** (forever-personal, local-first,
  KB MCP already Rust). This reframes the external refs (INTEGRATE→STEAL) and makes oh-my-pi a Rust reference
  + beads a schema to extract. Scoped a rigorous foundation eval (DR1 omp / DR2 beads) → `10-eval-scope.md`.
- Added project `CLAUDE.md` (+ `AGENTS.md` symlink) — the entry-point map + conventions (read wiki first,
  keep it updated, Align-before-execute, integrity constraints, commit discipline).
- **Next:** run DR1 + DR2, synthesize the language/runtime decision (`research/11`), then Phase 0.

## [2026-06-08] session | Foundation eval complete — Strategy A locked (signed off)
- Ran DR1 (oh-my-pi → `research/11`), DR2 (beads → `research/12`), DR2-prime (beads_rust/`br` → `research/15`);
  synthesized `research/13`; cataloged the author constellation `research/14` (Jeffrey Emanuel's Rust agent-flywheel).
- **Decision (signed off): Strategy A — full Rust-native own-core.** Lift `crates/pi-iso`; board on `rusqlite`
  (SQLite + JSONL) porting `br`'s `close_policy.rs` state-machine + gate engine (~250 lines); minimal Rust
  provider layer; Phase 0 opens with a sizing spike. B rejected (omp's agent brain is all TS), adopt-`br`
  rejected (welded to fsqlite-alpha + nightly + 180k LoC).
- Flipped `wiki/decisions.md` (OPEN→RESOLVED; rejected omp-as-base and beads/br/repowise-as-runtime-deps);
  rewrote design-v2 §2 (rows 1–3, 6) + §10 to the Rust-native foundation.
- **Next:** Gary clarifying 2 things before Phase 0 — the sizing spike is **held**, not kicked off.
