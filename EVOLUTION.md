# Evolution Log

Periodic retrospectives that turn real project traces into skill improvements. See `modes/evolve.md`.

---

## Evolution 1 — 2026-05-30

### Harvest scope
- Projects: `knowledge-base` (production RAG/MCP server over 46 technical books)
- Builds analyzed: 13 commits, 2026-05-27 → 05-30, plus full `wiki/` (decisions, gotchas, dspy-enrichment-plan, architecture, ops-runbook)
- Why this project: it followed the dev skill's wiki protocol and ran a disciplined experiment-first ML methodology — an unusually rich trace.

### Patterns found
1. **Proxy gave a false positive; pre-registered decisive gate caught it** — Impact: H, Effort: L. (DSPy mini-harness ranked a prompt +0.006; full-index gate showed NDCG 0.7907→0.7759 regression → reverted.)
2. **Eval-set validity / measure-the-wrong-thing trap** — Impact: H, Effort: L. (Golden set generated from chunks scored near-ceiling 0.93 and couldn't measure recall; distractor pool needed hard negatives.)
3. **Retrieval/RAG was an unrepresented domain** — Impact: H, Effort: M. (Entire project is RAG; ml-heuristics is 100% supervised vision.)
4. **Negative result shipped as a first-class outcome** — Impact: M-H, Effort: L. (Whole sprint reverted, post-mortem written, artifacts preserved.)
5. **Decouple cheap wins from an expensive optional pass** — Impact: M, Effort: L. (Index-now-enrich-later — folded into #1.)
6. **The wiki produced exceeded the protocol** — Impact: M, Effort: L. (Rejected Approaches + post-mortems + gotchas/runbook earned their place; log.md/active-work.md went unused — folded into #4.)

### Hypotheses applied
1. **H1: Proxy-vs-decisive metric gate** — `modes/train.md` (Phase 1 + Phase 4), `references/ml-heuristics.md` (Metrics) — applied (commit a712a87). Folds in pattern #5.
2. **H2: Eval-set validity trap** — `references/ml-heuristics.md` (Metrics), `modes/train.md` (Phase 2) — applied (commit f903d9e).
3. **H3: rag-heuristics.md reference** — new `references/rag-heuristics.md`, `SKILL.md`, `modes/build.md` — applied (commit 65df1d9).
4. **H4: Negative-result discipline** — `modes/train.md` (Phase 5), `references/wiki-protocol.md` — applied (commit bbc24f2). Folds in pattern #6.
5. **H5: Knowledge Base Grounding Gate** — `SKILL.md` (canonical gate), `modes/design.md`, `modes/train.md`, `modes/build.md`, `references/subagent-briefs.md` (33→46 staleness fix) — applied. Surfaced by a follow-up architectural question, not the harvest: the KB MCP (the corpus this whole project exists to provide) was discoverable but **not integral** — "use situationally" + "research is reactive" + no mode-level gate kept it optional. The gate fires only on KB-covered domains, requires one search + a one-line cite-or-null in Key Decisions, and is skippable by stating the domain isn't covered. Integral where relevant, zero ceremony elsewhere.

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (fill after next 3–5 ML/RAG builds)
- Pre-registered decisive gate present before expensive runs: [before: absent in skill] → [after: ?]
- Eval-set headroom/overlap check appears in traces: [before: none] → [after: ?]
- RAG tasks trigger `rag-heuristics.md`: [before: n/a] → [after: ?]
- Negative experiments end with clean revert + post-mortem (not silent drop): [after: ?]
- New friction/regressions introduced: [after: ?]
- Verdict: [keep/revert/refine — pending]

---

## Evolution 2 — 2026-06-15

### Harvest scope
- Projects: `harness` (`../harness/`) — the Rust-native dev harness re-platformed *from this skill*.
- Trace analyzed: full `wiki/decisions.md` (1205 lines) + design-v2 + CLAUDE.md, spanning the months-long Phase 0 sizing spike → foundation lock (Strategy A).
- Why this project: it is the most rigorous, longest-running dogfood of this skill's own principles. Reverse-engineering direction: the *engineering decisions* made while building the harness are the trace; the lesson is what the harness had to make **load-bearing** because advisory prose got ignored — which points exactly at where this skill's prose still hand-waves.
- Meta-principle synthesized: **the gap between "looks done" and "is done" is where everything fails — close it with an artifact, a number, or a structural guarantee, never with prose.**

### Patterns found
1. **Gate by the artifact, not the proxy** — Impact: H, Effort: L. (Harness verifies the produced thing — `git dirty`, the file — never the activity that should produce it. "Tool fired"/"agent reported done" are proxies that go green while the artifact is absent.)
2. **Defer until *measured* insufficient** (dormant-but-warmed infra) — Impact: M-H, Effort: L. (Build the seam, not the machinery; the failing run that proves you need X is the trigger, not the anticipation of X.)
3. **Adopt the design, not the system** — Impact: M, Effort: L. (`pi-iso` lifted verbatim where re-deriving cost more; everything else mined for *shape* and re-owned. Sharpens "dependencies are liabilities.")
4. **Adversarial review needs a trigger rule, not "skip light builds"** — Impact: M, Effort: L. (Review earns its cost only when a mistake is hard-to-reverse OR silent; elsewhere it's theatre that trains the team to ignore reviews.)
5. **Terminal labels encode HOW, not just THAT** — Impact: M, Effort: L. (`stalled`/`truncated`/`looped`/`rejected` ≫ `failed`; a status that collapses failure modes destroys the info needed to branch on it.)
6. **Decompose composite metrics per-axis** — Impact: M, Effort: L. (An aggregate stays flat while axes move opposite and cancel; log the components beside the primary.)
7. **Subagent VCS-mutation safety** — Impact: H, Effort: L. (Parallel writers on a shared checkout corrupt silently; confine each to its own worktree, coordinator owns the index/commit.)
8. **Force the failure with the smallest input** — Impact: M, Effort: L. (A test that fails on a one-line input points at the cause; one needing a big fixture can't.)
9. **Enforced output contract (the keystone)** — Impact: H, Effort: M. (Harness memory-plane: a worker can't reach Done without its findings artifact; a gate checks it exists. Reverse-engineered into: every mode terminates by writing its named wiki md, and isn't done until that file exists.)

### Hypotheses applied
1. **① Adversarial-review trigger rule** — `modes/design.md` Phase 4 (replaced vague "Skip for: Light builds") — applied. (Pattern #4.)
2. **① Gate by the artifact, not the proxy** — `SKILL.md` Principles (new bullet) — applied. (Pattern #1.)
3. **② Defer-until-measured + adopt-design-not-system** — `SKILL.md` Principles ("Start light, adapt" rewritten) + Engineering Style #7 — applied. (Patterns #2, #3.)
4. **③ Signal integrity** — `references/production-thinking.md` (Operational Failure Modes, forcing Q5) + `references/ml-heuristics.md` (Metrics, "Decompose Composite Metrics") — applied. (Patterns #5, #6.)
5. **④ Multi-agent & test hygiene** — `references/subagent-briefs.md` (global isolation constraint + Test Subagent "force the failure") + `modes/build.md` (worker VCS hygiene) — applied. (Patterns #7, #8.)
6. **⑤ Enforced output contract** — `SKILL.md` (Output Contract gate) + `references/wiki-protocol.md` (per-mode artifact map + close-out checklist) + light close-out gate in all six modes (`design`/`build`/`sprint`/`assess`+`analyze`/`train`/`evolve`) — applied. (Pattern #9.)

Dropped on Wu Wei (skip is visible): migrate-first DB primitive (schema-specific, below skill altitude) and path-confinement security mechanics (harness-runtime, not instruction-altitude).

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (fill after next 3–5 builds across modes)
- Modes terminate with their named wiki artifact present (output contract honored, not skipped): [after: ?]
- Adversarial review fires on hard-to-reverse/silent specs and is skipped elsewhere without ceremony: [after: ?]
- "Done" claims name the artifact checked, not the activity run: [after: ?]
- Parallel subagent builds use per-worktree isolation; zero cross-staged churn on main: [after: ?]
- Composite-metric reports show per-axis breakdown: [after: ?]
- New friction/regressions introduced (esp. output-contract feeling like bureaucracy on light tasks): [after: ?]
- Verdict: [keep/revert/refine — pending]
