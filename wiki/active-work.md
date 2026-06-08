# Active Work

## Foundation Eval (pre-Phase-0)
**Status:** Foundation decision FINAL (`research/13`) — all teardowns in — awaiting Gary's sign-off, then Phase 0
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
- [x] **DR1** — oh-my-pi teardown → `research/11`. omp Rust core = leaf primitives; agent brain 100% TS → **Strategy A** (full Rust-native), lift `pi-iso`.
- [x] **DR2** — beads schema → `research/12`. **STEAL-schema**, hand-roll Rust+SQLite, skip Dolt.
- [x] **Synthesis** — `research/13-foundation-decision.md`: A + steal-beads-schema + lift pi-iso. **Awaiting Gary's sign-off.**
- [ ] **Phase 0** — opens with a Rust **sizing spike** (own loop + 2 provider clients + Rust `enum Phase` gate + lifted `pi-iso`, driving oMLX), then the trunk (board + state machine + workpad + land).
- [ ] On sign-off: flip `decisions.md`, rewrite design-v2 §2/§10 to the Rust-native foundation.

### Breadcrumbs
- 2026-06-08: Spike proved the gate concept in TS on pi-mono (v0.78.1); the spike is a *port reference* if we go Rust, not wasted.
- 2026-06-08: omp ships ~60% of Execution+Memory (worktree `task` subagents, Hindsight memory, stream rules) and has a ~27k-LoC Rust core → the key case study for strategy B (hybrid) vs A (full Rust).
- 2026-06-08: Spike code + pi-mono clone live OUTSIDE this repo (`Skills/harness-spike/`, `Skills/pi-mono/`). When Phase 0 scaffolds real code, the harness likely gets its own repo and the wiki travels with it.
- 2026-06-08: Launched DR1 (oh-my-pi teardown → `research/11-omp-teardown.md`) + DR2 (beads schema → `research/12-beads-schema.md`) as parallel background teardowns. Synthesis → `research/13-foundation-decision.md`. Clones land in `Skills/oh-my-pi/` + `Skills/beads/`.
- 2026-06-08: **DR1 done** (`research/11`). Decisive: omp's Rust core is **leaf systems-primitives only** — agent loop / tool-exec / providers / the `beforeToolCall` gate are 100% TS (≡ pi-mono); zero LLM/HTTP code in any Rust crate. So omp *is* strategy B, and hands us **none** of strategy A's hard parts. BUT `crates/pi-iso` (worktree/sandbox isolation) severs cleanly → liftable as our P2 crate. Providers = ~2 wire formats (~1–1.5k LoC Rust). Gate ports *cleaner* in Rust (typed `enum Phase` + board row vs TS hook+closure+JSONL). **→ recommends A (full Rust-native own-core), lifting pi-iso.** Flagged a non-blocking build-cost-sizing spike before trunk commit. Awaiting DR2 to synthesize.
- 2026-06-08: **DR2 done** (`research/12`). beads = Go + embedded Dolt; Dolt's branch/merge serves multi-clone
  reconciliation — a single-writer local harness skips it (SQLite + git-committed JSONL export for history).
  Steal the schema: `ticket`(workpad inline + `state` enum) · `edge`(deterministic id from (issue,target),
  CHECK-enforced tagged-union target — merge-safety lessons) · `ticket_snapshot` · `memory`; `ready` = recursive
  CTE; decay = tiered compaction + oMLX summarize; collapse 20 edge types → 4 ready-affecting + 5 knowledge +
  `Other`. **beads has NO gate states** → our Align/Verify/Review/Land is the genuine net-new. → **STEAL-schema, hand-roll Rust+SQLite.**
- 2026-06-08: **Synthesis** (`research/13`). Both teardowns converge → **Strategy A (full Rust-native own-core),
  lift `pi-iso`, steal the beads schema.** RECOMMENDED, awaiting Gary's sign-off before flipping `decisions.md` +
  rewriting design-v2 §2/§10 + starting Phase 0 (opens with a Rust sizing spike).
- 2026-06-08: **`beads_rust` (`br`) discovered** (Gary, pre-sign-off) — Jeffrey Emanuel's Rust port of beads,
  frozen at **SQLite+JSONL, no Dolt** = exactly DR2's call, already built (★939, agent-first, MCP, MIT+rider,
  no gate states). Likely shifts DR2 from "hand-roll" → "fork/vendor `br` + add our spine." Launched a focused
  teardown → `research/15-beads-rust-teardown.md` (adopt/fork/vendor/reference + license rider). **Foundation
  sign-off now held pending this.** Author constellation (parallel Rust agent-flywheel ecosystem) cataloged →
  `research/14-author-constellation.md`; only `br` is foundation-gating, rest are for later pillars (P3:
  frankensearch + cass_memory_system; safety: dcg).
- 2026-06-08: **DR2-prime done** (`research/15`). br = SQLite+JSONL Rust beads (confirms DR2) BUT storage is
  welded to **fsqlite** (alpha pure-Rust SQLite) + nightly + ~180k LoC → adopt/vendor = Wu-Wei loss. **Verdict:
  REFERENCE, hand-roll on `rusqlite`** + two lifts: (1) **correction — br *has* a state machine**
  (`close_policy.rs`: allowed-transition map + per-transition gate engine + `gate_results(issue,gate,provider,
  passed)` table ≈ our Align/Verify/Review/Land, already built+tested) → port ~250 lines instead of inventing;
  (2) simpler edge table `(issue,depends_on,kind)`. Bonus: real SQLite → clean recursive-CTE ready-query (br
  used BFS only because fsqlite can't). **Foundation decision now FINAL across DR1+DR2+DR2-prime — awaiting sign-off.**

## Completed
- Research foundation (6 breadth tracks) — `research/01..06`
- Design v2 — `research/00-design-v2.md`
- Pi spike (Align gate validated) — `research/07`
- Symphony teardown — `research/08`
- Additions orientation + eval scope — `research/09`, `research/10`
