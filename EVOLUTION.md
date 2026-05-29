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

Not modified (foundational): Integrity Constraints, Wu Wei filter.

### Validation results (fill after next 3–5 ML/RAG builds)
- Pre-registered decisive gate present before expensive runs: [before: absent in skill] → [after: ?]
- Eval-set headroom/overlap check appears in traces: [before: none] → [after: ?]
- RAG tasks trigger `rag-heuristics.md`: [before: n/a] → [after: ?]
- Negative experiments end with clean revert + post-mortem (not silent drop): [after: ?]
- New friction/regressions introduced: [after: ?]
- Verdict: [keep/revert/refine — pending]
