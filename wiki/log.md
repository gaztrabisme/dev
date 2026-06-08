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
- **Next:** run DR1 + DR2, synthesize the language/runtime decision (`research/11`), then Phase 0.
