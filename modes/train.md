# Train Mode

Iterative ML training loops — finetuning, architecture experiments, hyperparameter search. Unlike Build mode (test → implement → verify), Train mode follows an experiment loop where success is measured by metrics, not binary tests.

Read `references/ml-heuristics.md` for problem reframing and architecture decision heuristics. Read `references/pushback-and-teach.md` — the teach directive applies to ML math too: when you introduce a loss function, activation, optimizer, or architecture choice, narrate the *mechanism* (what it computes, what it penalizes, why the shape is what it is), not just the API call.

---

## Phase 1: Define Convergence Criteria

Before any training, define what "done" looks like:

```markdown
## Convergence Criteria
- Primary: [metric] [operator] [threshold] (e.g., mAP@50 > 0.85)
- Secondary: [metric] [operator] [threshold] (e.g., inference latency < 50ms)
- Stop condition: [when to stop iterating] (e.g., 3 consecutive experiments with <1% improvement)
```

If you can't define convergence criteria, you're not ready to train — go to Design mode first.

### Proxy vs. Decisive Metric (pre-register the gate)

Most experiments are scored on a **cheap proxy** (a small in-memory pool, a held-out slice, a mini-harness) because the **decisive gate** — the full index, the real pipeline, the production-scale eval — is slow or expensive to run every iteration. That split is fine *as long as you respect the hierarchy*:

- **Proxies rank. The decisive gate decides.** A proxy can tell you which of several candidates to carry forward. It cannot tell you whether to *ship*. A win on the proxy is a hypothesis, not a result.
- **Pre-register the ship threshold before you see the decisive result.** Write down "ships only if [decisive metric] beats [recorded baseline]" *before* running it. Deciding the bar after seeing the number is how confirmation bias launders a regression into a ship.
- **Margins inside the proxy's noise floor are not wins.** If candidate A beats B by less than the proxy's run-to-run variance, the proxy can rank direction but cannot confirm the magnitude — defer to the decisive gate.
- **A proxy that scores near-ceiling is measuring the wrong thing** (see Phase 2 — eval-set validity). It can no longer discriminate; fix the proxy before trusting it.

> **Trace that earned this (knowledge-base project):** an 800-chunk in-memory pool ranked a "problem-symptom" enrichment prompt as the winner (+0.006, inside the noise floor). The pre-registered decisive gate — enrich all 48K chunks, re-index, re-eval — showed a **regression** (NDCG 0.7907→0.7759). The whole direction was reverted. The cheap proxy gave a false positive; only the pre-registered full gate caught it.

**Corollary — don't let the expensive gate block cheap certain wins.** If a costly optional pass (full re-enrichment, full retrain) would block a batch of cheap structural improvements, ship the structural wins first with the optional pass stubbed/empty, and run the expensive pass as a separate gated experiment. ("Index-now-enrich-later": structural fixes shipped immediately; the enrichment pass ran later and — per its gate — never shipped.)

---

## Phase 2: Validate Data

Before spending compute, verify the data is sound:

- [ ] Class distribution — are classes balanced? If not, is that intentional?
- [ ] Label quality — sample 20-50 items and manually verify labels
- [ ] Train/val/test splits — no data leakage between splits
- [ ] Data pipeline — does loading, augmentation, preprocessing produce expected output?
- [ ] Edge cases — what does the model see for ambiguous/hard examples?
- [ ] **Eval-set validity** — does the benchmark actually measure the lever you're pulling? Red flags: a near-ceiling baseline (no headroom to detect improvement), positives generated *from* the answer (queries inherit the answer's vocabulary → trivial match), distractor pools without hard negatives. Verify objectively (e.g., query↔positive lexical overlap) before trusting any score. See `ml-heuristics.md` → Eval-Set Validity.

If data pipeline engineering is needed (ingestion, cleaning, transformation), switch to **Build mode** for that work, then return to Train mode.

---

## Phase 3: Baseline

Establish baseline with minimal configuration:
- Pretrained model, default hyperparameters, no augmentation
- Record all metrics in experiment log
- This is experiment #1 — every future experiment is measured against it

---

## Phase 4: Experiment Loop

```
Read experiment-log.md → Change ONE variable → Train → Evaluate → Record → Decide
```

**Rules:**
1. **One variable per experiment** — changing lr AND augmentation AND architecture means you can't attribute the result
2. **Read the log first** — always, every time. This survives context compression.
3. **Record before deciding** — write the result before choosing the next experiment. Prevents confirmation bias.
4. **Stop when criteria met OR diminishing returns** — don't chase the last 0.1% unless the business requires it
5. **Iterate on the proxy, ship on the decisive gate** — use the cheap metric to rank candidates fast, but a candidate only graduates when it clears the pre-registered decisive gate (Phase 1). Never promote a proxy win straight to ship.

**Typical experiment order** (high-to-low impact):
1. Architecture / model selection
2. Data augmentation strategy
3. Learning rate and schedule
4. Batch size
5. Regularization (dropout, weight decay)
6. Fine-grained hyperparameters

---

## Phase 5: Evaluate & Handoff

When convergence criteria are met:
- Run final evaluation on held-out test set (not val set)
- Profile inference performance (latency, throughput, memory)
- Export model (see `references/ml-heuristics.md` Deployment section)
- Record final results in experiment log

**Handoff to Build mode** for: inference engine, API wrapper, deployment pipeline, monitoring.
**Handoff to Analyze mode** for: "why do certain classes underperform?", error analysis, failure mode investigation.

### When the experiment says DON'T ship

A pre-registered gate (Phase 1) exists precisely so it can come back negative. **"This change does not ship" is a first-class, successful outcome — not a failure to paper over.** When a candidate loses its gate:

1. **Honor the gate.** You pre-registered the threshold before seeing the result (Phase 1) specifically so you couldn't rationalize a regression into a ship now. Don't move the bar.
2. **Revert cleanly to the baseline** and confirm the baseline number is restored. Keep the rejected artifacts (e.g. `*_rejected/`, gitignored) so a future revisit doesn't start from scratch.
3. **Write the post-mortem as the deliverable.** Record *why* it lost and the mechanism — the negative result is reusable knowledge that stops you (and the next session) re-running a dead end. Put it in `wiki/decisions.md` with a "why it failed — the lesson" note, and add a `wiki/` Rejected Approaches entry if it's a whole direction.
4. **Keep reusable infra.** The harness/optimizer/pipeline you built to run the experiment is still valuable even when the experiment loses — note that it's re-runnable if the corpus/objective changes.

This realizes Integrity Constraint 6 ("Honest failure beats fabricated success") at the experiment level. A sprint that concludes "ship nothing" with a clear post-mortem is a clean win.

---

## Experiment Log

Maintain an append-only `experiment-log.md` in project root. This survives context compression and prevents retrying dead ends.

### Format

```markdown
# Experiment Log

## Goal
[One sentence: what we're optimizing for]

## Current Best
| Metric | Value | Config | Checkpoint |
|--------|-------|--------|------------|
| [e.g. mAP@50] | [value] | [key hyperparams] | [path/epoch] |

## Log

### Exp [N] — [timestamp]
- **Changed:** [what was different from previous]
- **Result:** [metrics]
- **Verdict:** [better/worse/inconclusive — why]
- **Next:** [what to try based on this result]
```

### Rules

1. **Always read the log before starting a new experiment** — this is your memory across context compressions
2. **Update "Current Best" immediately** when a new best is found
3. **Record failures** — "tried X, got worse because Y" prevents retrying dead ends
4. **Keep entries terse** — this file will be read many times

---

## Key Differences from Build Mode

| Build Mode | Train Mode |
|-----------|------------|
| Success = tests pass (binary) | Success = metrics meet criteria (continuous) |
| Test → implement → verify (linear) | Experiment → evaluate → adjust (loop) |
| Subagents for separation | Coordinator runs directly (training is sequential) |
| Contract changelog for changes | Experiment log for iterations |
| Adversarial review at end | Analyze mode for result investigation |

Train mode chains into: **Build** (inference engine, deployment pipeline), **Analyze** (why are results bad?), or more **Train** (next experiment).
