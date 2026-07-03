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
4. **H4: Negative-result discipline** — `modes/train.md` (Phase 5), `../core/references/wiki-protocol.md` — applied (commit bbc24f2). Folds in pattern #6.
5. **H5: Knowledge Base Grounding Gate** — `SKILL.md` (canonical gate), `modes/design.md`, `modes/train.md`, `modes/build.md`, `references/subagent-briefs.md` (33→46 staleness fix) — applied. Surfaced by a follow-up architectural question, not the harvest: the KB MCP (the corpus this whole project exists to provide) was discoverable but **not integral** — "use situationally" + "research is reactive" + no mode-level gate kept it optional. The gate fires only on KB-covered domains, requires one search + a one-line cite-or-null in Key Decisions, and is skippable by stating the domain isn't covered. Integral where relevant, zero ceremony elsewhere.

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (measured in Evolution 4 harvest, 2026-06-30)
- Pre-registered decisive gate before expensive runs: **CONFIRMED** — the robotics project locks a binary gate per Rung ("LOCKED — do not move to match results"); knowledge-base v6.6 ran a pre-registered paired re-test rather than reading aggregates.
- Eval-set headroom/overlap check appears in traces: **CONFIRMED** — the KB gate-audit traced a single-label ceiling artifact (true ceiling 0.96→0.995) and a missing-archetype blind spot; eval-set-validity reasoning is now standing practice in that project.
- RAG tasks trigger `rag-heuristics.md`: **CONFIRMED** — the knowledge-base RAG-frontier work is grounded in exactly these heuristics (and fed Evolution 4 H6 back into the file).
- Negative experiments end with clean revert + post-mortem: **CONFIRMED** — the robotics project ("a clean negative is a required, documented result"); KB late-chunking rejection documented with the mechanism and strengthened to significant.
- KB Grounding Gate (H5): **CONFIRMED to stick** — the document-OCR project recorded the literal `KB: searched … → applied <finding>` line and it drove a load-bearing decision (dual *independent-lineage* OCR engines); the robotics project runs the gate faithfully and records "nothing relevant" honestly for its off-corpus domains.
- New friction/regressions: none observed.
- **Verdict: KEEP.** All five changes are exercised in real traces; Evolution 4 extends (not reverts) the decisive-gate line with a discrimination-floor corollary.

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
6. **⑤ Enforced output contract** — `SKILL.md` (Output Contract gate) + `../core/references/wiki-protocol.md` (per-mode artifact map + close-out checklist) + light close-out gate in all six modes (`design`/`build`/`sprint`/`assess`+`analyze`/`train`/`evolve`) — applied. (Pattern #9.)

Dropped on Wu Wei (skip is visible): migrate-first DB primitive (schema-specific, below skill altitude) and path-confinement security mechanics (harness-runtime, not instruction-altitude).

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (measured in Evolution 4 harvest, 2026-06-30)
- Modes terminate with their named wiki artifact present: **CONFIRMED** — the robotics project and the document-OCR project both maintain the full wiki set and close out with dated, mode-tagged `log.md` entries + named design artifacts. No friction; adopted as routine.
- Adversarial review fires on hard-to-reverse/silent and is skipped elsewhere without ceremony: **CONFIRMED** — OCR explicitly skipped Phase-4 with a "reversible + loud → skip" reason; the robotics project *re-ran* cold review when a scaffold pass had wrongly skipped it and caught a CRITICAL test gap. (Evolution 4 H1 extends the trigger to a third case — "green for the wrong reason" — which the the robotics project Rung-1 memorization leak proved verification structurally cannot catch.)
- "Done" claims name the artifact, not the activity: **CONFIRMED** — the robotics project's gate language ("binding FAIL (diagnosed)", "array resolution measured") reports artifacts/numbers, not "ran the step."
- Parallel subagent per-worktree isolation: **not exercised** this window (the robotics project used subagents mostly sequentially; no parallel-writer trace to test it).
- Composite-metric per-axis breakdown: **not exercised** this window (no composite-aggregate report appeared).
- New friction (output contract as bureaucracy on light tasks): **none** — the robotics project down-weighted light tasks to coordinator-direct cleanly.
- **Verdict: KEEP.** Output contract and the trigger rule are both load-bearing in practice; H1 extends the trigger rather than reverting it.

---

## Evolution 3 — 2026-06-16

### Harvest scope
- Source: **external**, not a project trace — a LinkedIn post (Crystal Widjaja) proposing Gary Klein's Data-Frame theory of sensemaking as a pedagogy prompt for AI exploratory data analysis.
- Why logged honestly: this is a *hypothesis source*, not a measured failure. Analyze mode works fine today; no trace shows it botching a root cause via confirmation bias. Applied anyway because the change is additive, reversible, and good epistemics on first principles — the cheap-and-warmed end of the Wu Wei spectrum, not a response to pain.

### Patterns found
1. **Analyze mode is linear/convergent — no disconfirmation step** — Impact: M, Effort: L. The pattern went Taxonomy → Causal Model → *Rank fixes* with nothing forcing competing hypotheses or a hunt for disconfirming evidence. The failure mode of root-cause work is a confident, wrong frame; the pattern had no guard against it. The post's own thesis (coding training biases toward linear, confirm-the-first-hypothesis reasoning) is correct and names exactly this gap.

### Hypotheses applied
1. **Stress-test before ranking** — `modes/assess.md` (Analyze pattern, new step 3) — applied. Distilled the Data-Frame *kernel* (competing frames, anchor + disconfirm, classify anomalies dismissible/concerning/frame-breaking, name the tripwire before committing) into ~one step. **Adopt the design, not the system**: rejected the post's full 6-phase protocol as domain-specific ceremony (built for a metrics warehouse) that violates Wu Wei — mined the epistemics, dropped the runtime.

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (fill after next 3–5 Analyze runs)
- Analyze traces show competing frames + a disconfirming check before fixes are ranked: [after: ?]
- A named tripwire/falsifier appears before commit: [after: ?]
- New friction (feels like ceremony on simple root-cause work): [after: ?]
- Verdict: [keep/revert/refine — pending — no Analyze-mode traces in the Evolution 4 window]

---

## Evolution 4 — 2026-06-30

### Harvest scope
- **Sources:** (idea) `github.com/DietrichGebert/ponytail` — a sibling minimalism skill; (measured, in-window post Evo-2 commit 2026-06-15 → 06-30, under `the local projects tree/`) **the robotics project**, **knowledge-base** (`gate-audit.md` + `rag-frontier-2026.md`), **the document-OCR project**.
- **Why these:** the robotics project is the richest dogfood since the harness — new project, full wiki protocol, ~19 disciplined commits across hardware/audio/sim/ML, gates catching real failures. The KB gate-audit is a real stress-test of the *gating philosophy*. the document-OCR project is a clean second-domain check that Evo-1's KB Grounding Gate is actually followed. Ponytail is mined for *shape*, logged as an idea-source not a measured failure (same honesty bar as Evolution 3).
- **Convergence signal:** the robotics project independently invented an *epistemic* "Rung" ladder (cheapest-decisive-experiment-first, **one variable per rung**, pre-registered binary gates, "a clean negative is a required result"); ponytail an *engineering* "Ladder" (stop at the first rung that holds: YAGNI → reuse → stdlib → native → dep → one line → minimum). Two independent sources landing on the ordered-rung shape.

### Patterns found
1. **Cold adversarial review catches "green for the wrong reason"** — a class verification structurally *cannot* — Impact: H, Effort: L. (the robotics project Rung-1 reported PASS that was memorization: 15/19 "held-out" pairs were trained on; spec-verify PASSED, cold review CAUGHT it — "a contract-checker inherits the contract's blind spots." Plus 5 more cold-review catches: geometry false-win, scaffold CRITICAL test gap, motor-safety try/finally, standardization-frame bug, physics-error test.)
2. **Discrimination-floor corollary to gate-by-artifact** — an instrument can be valid and still be a proxy *for the decision* if it can't resolve the delta — Impact: H, Effort: L. (KB gate-audit: a 0.012 NDCG rejection sat 3.6× below the gate's CI floor, never significance-tested → downgraded to *inconclusive*. the robotics project's coarse-grid "14.2°→0°" win vanished on a dense grid.)
3. **Planned-gate-skip discipline, generalized** — skip a committed gate only out loud; legitimate-skip test = reversible **and** loud — Impact: M, Effort: L. (Measured in two projects: the robotics project "planned gates are promises"; OCR "Phase-4 SKIPPED — reversible + loud.")
4. **Experiment-design cluster** — build the discriminating instrument before a two-way-fail result; calibrate the control to a *correct* model (1/B not 1/N); sim/finite-data overfitting (train- vs fresh-realization; K↑ not dropout; averaging-oracle is a low-variance ceiling not "impossible") — Impact: M-H, Effort: M. (the robotics project Rung 2a.)
5. **Minimalism guards** (ponytail) — "lazy about the solution, never about the problem" + a never-cut carve-out on the Wu Wei filter — Impact: M, Effort: L. (Idea-sourced; rhymes with the robotics project's read-heavy discipline — sim-before-buy, 7 research agents for OCR, "design the silent failure out structurally.")
6. **RAG frontier deltas** — measured negatives + the 2026 cost-dichotomy consensus — Impact: M, Effort: L. (KB rag-frontier.)
7. *(deferred)* **OCR/document-AI heuristic cluster** — the document-OCR project is pre-build; defer the new reference until it's built+measured.

### Hypotheses applied
1. **H1 — adversarial review unified + extended** — `modes/build.md` Phase 5, `modes/design.md` Phase 4, `references/subagent-briefs.md` (Adversarial Review brief + a new "could this pass for the wrong reason?" review question) — applied. Trigger gains a third clause ("green for the wrong reason"); cold protocol locked (reviewer gets file paths only; test-author ≠ implementer ≠ reviewer). Also reconciled an Evo-2 inconsistency — build.md/subagent-briefs still carried the old "medium/heavy only, skip light" wording the trigger rule was meant to replace. (Pattern 1.)
2. **H2 — discrimination-floor corollary** — `SKILL.md` (gate-by-artifact principle), `references/ml-heuristics.md` (new Metrics subsection "The Instrument Must Resolve the Delta"), `modes/train.md` Phase 1 — applied. (Pattern 2.)
3. **H3 — planned-gate-skip principle** — `SKILL.md` Principles (new bullet, generalizes Build's Gate Enforcement to every mode; reversible-AND-loud skip test) — applied. (Pattern 3.)
4. **H4 — experiment-design cluster** — `references/ml-heuristics.md` (new section "Experiment Design (controls, discriminators, sim data)") — applied. (Pattern 4.)
5. **H5 — minimalism guards** — `SKILL.md` Principles ("lazy about the solution, never about the problem") + Wu Wei filter carve-out ("never drop on Wu Wei grounds") — applied. (Pattern 5; idea-sourced, like Evo 3.)
6. **H6 — RAG frontier deltas** — `references/rag-heuristics.md` (new "Frontier (2026)" section + an Evaluation archetype-diversity bullet) — applied. (Pattern 6.)

**Deferred:** H7 (OCR/document-AI reference file) — eat the skill's own defer-until-measured rule: `rag-heuristics.md` was seeded off a *built* project; the document-OCR project isn't one yet. The 7 heuristics live in the the document-OCR project wiki; revisit next cycle once it ships.

**Also:** back-filled Evolution 1 & 2 validation sections — both **KEEP**, all changes exercised in real traces (KB Grounding Gate confirmed across a second domain; output contract + adversarial trigger load-bearing). Evolution 3 stays pending (no Analyze-mode traces this window).

Not modified (foundational): Integrity Constraints; Wu Wei filter *logic* (H5 adds a carve-out beside it, doesn't change the YES/NO/Priority core).

### Validation results (fill after next 3–5 builds across modes)
- Cold review is invoked specifically on "green-could-be-for-the-wrong-reason" metrics, and the reviewer is briefed context-free: [after: ?]
- Decisive gates are significance-tested / checked against their CI floor before a ship-or-reject call; sub-resolution deltas reported as inconclusive: [after: ?]
- Gate skips are stated out loud with the reversible-AND-loud justification: [after: ?]
- ML/sim experiments build a discriminating control first and calibrate the null to a correct model; sim work checks train- vs fresh-realization before regularizing: [after: ?]
- Minimalism guard holds — no validation/security/a11y/hw-calibration cut on Wu Wei grounds; reading not skimped: [after: ?]
- RAG builds cite the frontier consensus (hybrid+rerank, no LLM in query path) and the measured negatives: [after: ?]
- New friction/regressions introduced: [after: ?]
- Verdict: [keep/revert/refine — pending]

---

## Evolution 5 — 2026-06-30

### Harvest scope
- **Source: in-session friction + a deferred Evolution-4 pattern**, not a fresh project trace. Gary reported a recurring, *measured* friction — subagent/parallel patterns never auto-trigger; the coordinator defaults to doing everything inline, forcing manual "use appropriate subagent coordination" prompts. Plus a skill-size review ("we've added a lot — what to remove?") and a wiki-scaling concern (a client CV project's memory md grew to thousands of lines).
- **Why now:** the orchestration friction is exactly the gap Evolution 4 *deferred* — the robotics project had crystallized an enumerated pattern-selection taxonomy into its AGENTS.md (candidate P5), logged but not applied. The friction confirms the deferral was wrong; this evolution ports it. (Same honesty bar as Evo 3: part friction-measured, part idea-ported.)

### Patterns found
1. **Execution patterns stay advisory → silent inline default** — Impact: H, Effort: L. The skill *described* orchestration but `build.md` licensed the bad default ("coordinator-direct unless there's a reason to delegate") and buried the guidance below mode-entry. Advisory prose loses to the model's act-directly prior; only a *mandatory gate* fires — the KB Grounding Gate is the proof (it stuck across two projects in the Evo-4 harvest).
2. **Accumulated duplication after 4 evolutions** — Impact: M, Effort: L. `ml-heuristics.md` carried a ~100-line Training Mode Workflow + Experiment Log that `modes/train.md` now owns better (with the proxy/decisive gate + discrimination floor); `chub`/GitNexus sat prominently in the tool table but go unused.
3. **Wiki growth is a read-cost problem, not a storage problem** — Impact: M, Effort: L. Append-only pages (a client CV project's thousand-line memory) only hurt because they're read wholesale; the protocol had no read-on-entry budget or compaction discipline — and Evo 2 had *added* writing pressure (Output Contract) with no counterweight.

### Hypotheses applied
1. **Pattern Gate** — `SKILL.md` (new "Pattern Gate": mandatory `Pattern: <choice> — <reason>` one-liner with an enumerated menu — coordinator-direct / subagent chain / parallel fan-out / intrinsic-artifact gate / sim-probe-first / research — modeled on the KB gate *because that mechanism is measured to fire*) + `modes/build.md` (reframed "Default: coordinator-direct" → "declare, don't default; rule out fan-out on purpose"). Ports Evo-4's deferred P5 (the robotics project's taxonomy). **Gate-only, no keyword** — eating the defer-until-measured rule; add a keyword only if it under-fires. (Pattern 1.)
2. **Dedup** — `references/ml-heuristics.md` (collapsed Training Mode Workflow + Experiment Log → a pointer to `modes/train.md`, −~100 lines) + removed `chub`/GitNexus from `SKILL.md`, `modes/build.md`, `modes/design.md`, `modes/assess.md` (replaced with read-the-code/grep + official-docs fallbacks; Gary confirmed neither tool fires). (Pattern 2.)
3. **Wiki compaction discipline** — `../core/references/wiki-protocol.md` (read-on-entry budget: index + active-work + decisions only, grep the rest; compact current-state pages / append only journals; ~400-line size trigger in Lint; the `CLAUDE.md`/`AGENTS.md` bootstrap one-liner so re-init survives compaction — the robotics project's trick). Recorded *why not* an external memory tool (claude-mem/cass): dynamic per-turn injection busts prefix caching → token blowup; markdown read once stays cached. (Pattern 3.)

Not modified (foundational): Integrity Constraints, Wu Wei filter core.

### Validation results (fill after next 3–5 sessions across modes)
- Substantive tasks open with a `Pattern:` line; fan-out / sim-first get chosen when they fit, *without* the user prompting for them: [after: ?]
- No regressions from the dedup — `train.md` fully covers the ML workflow; zero dangling chub/GitNexus references: [after: ?]
- Read-on-entry stays bounded; current-state pages compacted; `log.md` grows but isn't read wholesale: [after: ?]
- Pattern Gate doesn't become ceremony on trivial tasks (coordinator-direct declared in one line; exemptions honored): [after: ?]
- Verdict: [keep/revert/refine — pending]

---

## Evolution 6 — 2026-07-01 — Migrate the shared spine to the `core` kernel

### Harvest scope
- **Source: a constellation-level refactor**, not a project trace. As the skill family grew (business-intelligence, solution-architect, ms-ai-discovery, skill-builder), all four reached into `../dev/references/` for *general* discipline — an oddness noted while building solution-architect. The general spine was extracted into a new `core` kernel; this entry records dev's own migration onto it.

### Patterns found
1. **General discipline was privileged inside `dev`** — Impact: M, Effort: M. `wiki-protocol.md` and `pushback-and-teach.md` are role-agnostic, but living in `dev` forced every non-dev skill to reference a *build* skill for non-build rules, and (once copied into `core`) created a two-copy drift risk.

### Hypotheses applied
1. **Repoint + retire** — deleted `dev/references/wiki-protocol.md` and `dev/references/pushback-and-teach.md`; repointed all 21 references (`SKILL.md`, `CLAUDE.md`, 6 modes, `subagent-briefs.md`, this log) to `../core/references/`. Reframed SKILL.md "Related skills" and CLAUDE.md structure notes: dev now **inherits** the spine from `core` (like its peers) and keeps only engineering-specific content (modes, ml/production/rag heuristics, the KB substrate). Single canonical home; drift risk closed.

Not modified: everything dev-specific (modes, heuristics, integrity constraints, Pattern Gate, KB Grounding Gate). This is a *relocation of shared references*, not a behavior change — dev reads the same protocol text, now from `core`.

### Validation results (fill after next 2–3 dev sessions)
- Modes still resolve the wiki-protocol / pushback references (now under `../core/`) with no dangling links: [after: ?]
- No behavior change from the relocation (same Output Contract, same pushback discipline): [after: ?]
- Verdict: PENDING — mechanical relocation verified (grep clean, files present in core); needs a real dev session to confirm the references read naturally from their new home.
