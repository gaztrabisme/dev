# Personal Dev Harness — Design v2 (research-grounded)

> Supersedes the v1 high-level design. Every major decision here is grounded in the six breadth
> findings docs (`research/01..06`). v1 named six open questions; **breadth resolved all six** — they
> are answered in the table below. What remains is *build risk*, concentrated in two repos we must
> clone and take apart (Pi, Symphony), listed at the end.

---

## 1. Context

The `dev` skill (pure instruction) hit its ceiling on the *orchestration* layer, not the *judgment*
layer. v1 proposed a Pi-based harness with three planes (Work / Execution / Memory) and five pillars
(P0–P5) over a git-shaped branch/promote model. Breadth research (six parallel tracks: memory,
orchestration spine, test-hardening, self-evolution, parallel exploration, Pi internals) confirmed the
shape and supplied the concrete mechanics. This doc is the buildable spec.

**The one-sentence architecture:** a coordinator state machine (Align → delegate → Verify → reflect)
running as a **Pi extension**, dispatching **worktree-isolated workers** against a **durable file-backed
board**, over a **two-store memory plane** (curated wiki prose + decaying embedded sidecar on local
oMLX), where **every gate is a number or an artifact, never a vibe**, and the harness's own behavior
layer evolves under a **constitution/case-law** split.

---

## 2. Resolved decisions (the six v1 open questions)

| # | Question | Resolved answer | Source |
|---|---|---|---|
| 1 | **Board substrate** | **Local file-backed board** — markdown tickets, git-tracked, Obsidian-visible (durable + human/AI co-editable; matches "deps are liabilities"). The board + workpads are **durable**; only the coordinator's claim/lease layer is ephemeral (rebuilt from the board on restart). | T2, T6 |
| 2 | **Pi adoption layer** | **EXTEND `pi-coding-agent` via its `ExtensionAPI`**; depend directly on `pi-ai` + `pi-agent-core` for low-level control; **fork only the subagent *example* extension**. Do not fork the host (TUI/providers/sessions/compaction/MCP/OAuth are well-maintained). Headless later via the coding-agent **SDK/RPC seam**, not raw `agentLoop`. | T6 |
| 3 | **Deployment** | **Live as a Pi extension in Pi's TUI.** Pi implements the Agent Skills standard and **can load `~/.claude/skills` directly** — so the entire judgment layer (this `dev` skill's references) ports with near-zero work and the harness coexists with Claude Code. | T6 |
| 4 | **Memory decay policy** | **Resolved in full** (schema, decay formula, retrieval score, tiers, reflection) — see §6. CoALA rule: decay applies to the episodic sidecar only; wiki prose never ages out. | T1 |
| 5 | **Mutation scope** | **Harden gate between Test and Implement**, scoped to the diff (incremental), ≥85% mutation score on critical code / ~70% soft elsewhere, equivalence-judge to drop un-killable mutants, ML fallback = metamorphic + property + criteria-coverage adversary — see §7. | T3 |
| 6 | **Model-per-role** | First-class in `pi-ai` (`getModel(provider,id)`, cross-provider handoff built-in). Recon→local oMLX/Haiku; build→Sonnet; coordination+adversarial→Opus; **test-author and mutator on different providers** (decorrelation). oMLX drops in via `openai-completions` + `baseUrl`. | T3, T6 |

---

## 3. Architecture — three planes, sharpened

**Work plane** (durable, file-backed). A **board** of markdown tickets is the unit of work. Each ticket
owns a **workpad** (Symphony's contract: Plan / Acceptance Criteria / Validation / Notes / Confusions)
and moves through an explicit **state machine** whose transitions are un-skippable gates. Promotion to
mainline = a **land** procedure gated on proof-of-work. Two FSMs, kept separate (Symphony's key move):
the human-facing **work FSM** (Todo…Done) and the coordinator's ephemeral **claim/lease** layer
(Unclaimed→Claimed→Running→Released) so workers never double-grab a ticket.

**Execution plane** (Pi). A **coordinator** extension dispatches **workers** that run as Pi subagents in
**git-worktree-isolated** workspaces (the one thing Pi doesn't give us — its subagents isolate *context*
but share `cwd`; we wrap the launcher). Churn lives/dies in the worktree; only the squashed result +
workpad promote. Model-per-role throughout.

**Memory plane** (local-model powered). **Wiki** = curated semantic+procedural prose (never decays,
human/AI co-edited, Obsidian). **Sidecar** = episodic observations with embeddings (decays), captured
deterministically by hooks, embedded on **oMLX** (marginal cost ≈ 0). Retrieval is **top-k with
progressive disclosure**, never whole-store injection.

---

## 4. The spine — state machine + gates (from T2)

```
Todo ─▶ Align ─▶ In Progress ─▶ Verify ─▶ Review ─▶ Land ─▶ Done
  ▲                                                            
  └──────────────── Rework (hard reset, fresh branch) ◀────────┘
```

| State | Entry gate (un-skippable condition) | Mechanism |
|---|---|---|
| **Todo** | On board; blockers terminal; not claimed | coordinator dispatch |
| **Align** | Workpad has Plan + Acceptance + Validation; ticket-authored validation mirrored as **non-downgradeable**; **human approves the plan**; scope frozen | **`tool_call` block** on all execution tools until an `aligned` flag flips (Pi-native; see §10) |
| **In Progress** | Came from approved Align; **worktree created** (cwd==workspace, path inside root); **reproduction signal recorded** before any code change | worktree wrapper + workpad Notes |
| **Verify** | Every acceptance + required validation item checked **with a captured proof artifact**; **Harden gate passed** (§7); tests green for *this* commit; lint/CI invariants are hard failures | mechanical completion-bar checklist |
| **Review** | Verify fully green; PR opened, CI green, workpad reconciled to reality; **human review** | checkpointed interrupt; outstanding review comments block |
| **Land** | Review approved; conflict-free with main; all comments acknowledged; CI green → **squash-merge** | the land procedure (§4.1) |
| **Done** | Merge complete | terminal |
| **Rework** | Reviewer/human rejects → close PR, **archive** workpad (feeds memory), fresh branch, rebuild plan, re-enter Align | hard reset, not incremental patching |

**Three sources of un-skippability** (mechanical, not honor-system): (a) human gates = `tool_call`
block / checkpointed interrupt that pauses *before* the gated action; (b) Verify = a checklist where each
box must be checked *and* backed by an artifact, re-evaluated against the current commit; (c) the board is
authoritative and the coordinator **reconciles before dispatch every tick**, so a human moving a ticket
steers/kills its worker within one tick.

**Scope discipline:** out-of-scope discovery → **new Backlog ticket** (`related`/`blockedBy` links),
never scope expansion. The plan is frozen at Align; a replanner may revise *remaining* steps but additions
route to new tickets.

### 4.1 Land procedure
Green-locally-before-push → mergeable-vs-main (else pull/resolve/push) → all review comments acknowledged
→ watch CI, fix-commit-push-loop on red → **squash-merge** (PR title/body as subject) → Done. Net: branch
churn stays in the worktree; main gets one squashed commit + a distilled PR body + green CI as proof.

---

## 5. The workpad contract (from T2, Symphony near-verbatim)

```md
## Workpad — {{ticket.id}}
```text
<host>:<abs-workspace-path>@<short-sha>
```
### Plan              # frozen at Align; replanner revises remaining steps only, never expands scope
### Acceptance Criteria   # human-approved at Align; the Verify oracle AND the P4 pruning function
### Validation        # required (non-downgradeable); mirrors any ticket-authored Test Plan; each item → captured evidence
### Notes             # reproduction signal (pre-change), pull/sync evidence, timestamped milestones
### Confusions        # structured ambiguity → re-opens an Align human gate
```
One workpad per ticket, edited in place, reconcile-first on entry. Never edit the ticket body for progress.
**Confusions → Align** is our addition: it's the structured channel that re-opens a human gate mid-flight.

---

## 6. Memory plane (from T1)

**Lens (CoALA):** sidecar = *episodic*; wiki = *semantic + procedural*; active workpad = *working*. Decay
applies to episodic only.

**Sidecar observation schema** (the 5 load-bearing fields: `type, salience, scope, entities, embedding`):
```
id, ts, type(decision|bugfix|feature|refactor|discovery|lesson|constraint),
title (index line), body (drill-down only), salience(0–1, LLM-rated),
scope(ticket|project|global → tiering), entities[], files[], project, ticket_id,
embedding(oMLX), usage_count, last_used_ts, links[](deferred)
```

**Decay / promotion:** `retention = salience·e^(−λ·Δt) + Σ 1/days_since_use`; evict sidecar rows < 0.15;
dedup near-identical at cosine ≥ 0.92; **high usage + salience → promotion candidate to wiki**; **wiki never
decays** (it's UPDATE/DELETE'd by reflection, not aged).

**Retrieval (hybrid, top-k by token budget):**
`score = 1.0·relevance + 0.5·recency + 0.8·salience` (each normalized), pre-filtered by `project`/`scope`.
Returns a **ranked index** (id+title+type); workers fetch `body` only for chosen ids (progressive
disclosure — the primary defense against context bloat), then bump usage.

**Tiers:** hot (active ticket, in-context) → warm (project sidecar+wiki, retrieved per task) → cold
(cross-project, retrieved on miss). Paging is by `scope` tag set at capture time (cheaper/more predictable
than agent self-paging).

**Reflection (episodic → generalized lesson):** triggered on salience-accumulation *or* land/abandon.
Cluster recent rows → local-LLM extracts a **generalized, conditional, imperative** lesson → classify
**ADD/UPDATE/DELETE/NOOP** against the wiki → promote only durable, transferable rules, **gated on the work
having landed** (Voyager self-verification analog). Carries `proof_count` so multi-ticket-confirmed lessons
outrank one-shot guesses. This is the same engine as P5's case-law (§9) — memory and self-evolution share it.

**The claude-mem cost lesson:** local embeddings kill *compression* cost but **not injection cost** — every
recalled token competes with task reasoning. Discipline (top-k + progressive disclosure + project scoping)
is the only fix; "free embeddings" must not tempt us into recalling more.

---

## 7. Verify / Harden gate (from T3) — killing the test theater

The adversarial-review-of-tests was theater because it's a late, gameable LLM opinion. Replace it with a
**number computed before implementation.** Insert a **Harden** stage between Test and Implement:

```
TEST   → test-author (Provider A) writes tests FIRST (examples + properties where an invariant exists)
HARDEN → (a) generate a stub/first impl to mutate
         (b) MUTATE the diff: rule-based (mutmut/Stryker, cheap) + LLM-mutator (Provider B, ACH-style, realistic faults)
         (c) equivalence-judge drops un-killable mutants (so we don't chase 100%)
         (d) run frozen tests vs survivors → mutation_score = killed/(total−equivalent)
         (e) if < threshold: feed survivor diffs back to Provider A ("this broken version passed — add an assertion") → loop
             (cap retries; repeated failure is the ONE place human review earns its keep)
IMPLEMENT → implementer writes real code against frozen, hardened tests
VERIFY    → run hardened tests; re-mutate the real impl to confirm the score held
```

**Why it works:** mechanical, runs *before* implementation (tests can't be shaped to the code), bar is a
number not a vote. **Scope:** always the diff (incremental cache); full mutation on money/security/logic,
lighter coverage-gap gate elsewhere, skip boilerplate. **Threshold:** ≥85% critical / ~70% soft, ratchet up,
never 100%. **Model diversity is mandatory:** author Provider A, mutator+judge Provider B — same-model
author/mutator only generates faults the author already tested (false confidence). **Data point:** Meta ACH
found 49% of fault-catching tests added *zero* line coverage → coverage-gating is provably insufficient.

**ML / non-deterministic fallback:** swap the oracle to **metamorphic relations** (invariance under benign
transforms, monotonicity, symmetry, self-consistency) + **property invariants**; a **criteria-coverage
adversary** perturbs inputs instead of code; gate on violation *rate* across N reruns (LLMs aren't
deterministic even at temp 0). The implementation-side cold adversarial review **stays** — that one catches
real logic bugs and is not theater; only the test-side ceremony is replaced.

---

## 8. Parallel exploration (from T5) — beam search over approaches

For research/experiment tickets: **beam search where each "thought" is a whole approach trajectory and the
verifier is the pre-committed Align criteria** (not an ad-hoc value function).

- **Fan-out by stakes** (don't fix N): low → 1–2, default → 3, high/ambiguous → 4–5. **Seed diversity by
  construction** — the planner assigns each worker a *distinct* strategy/assumption (the fix for N
  near-duplicates); temperature alone is not diversity. Workers run in **isolated worktrees** (backtracking
  is free: state = context).
- **Two-stage prune:** (1) **objective gate first** — run the experiment/test/metric; any branch violating a
  **hard** Align failure-criterion dies at zero judge cost; (2) **pairwise-knockout judge** on survivors
  against the success criteria (not noisy absolute scores). Hard criteria kill; soft criteria only demote.
- **Dead-end detection (the core pain):** kill on hard-criterion violation; cycle/no-state-change; **or
  effort-without-progress** — the counterintuitive, load-bearing signal: *dead branches run longer, not
  shorter*; a long trajectory with no checkpoint cleared is evidence of death. Plus self-assessed "can't meet
  criteria." On kill, the worker writes a **one-paragraph post-mortem** → fed to survivors + stored as
  negative-result memory.
- **Budget:** global per-ticket budget scaled to stakes + per-branch step budget (kills runaway branches);
  reallocate freed budget to survivors; cheap workers, rich judge/synthesizer.
- **Synthesis (never pick-and-discard):** one aggregation pass merges winner + best runner-up
  fragments/post-mortems; if survivors are too close, a debate/reconcile round instead of a coin-flip.
- Evidence it matters: ToT ablation — greedy no-backtrack 20% vs full search; breadth must not collapse to 1.

---

## 9. Self-evolution (from T4) — constitution vs case-law

**Boundary (code-enforced, not convention).** Litmus: *if the agent could weaken safety/integrity by editing
it, it's constitution.* 
- **Constitution (immutable, human-only):** the TS spine, integrity constraints, Wu Wei minimalism filter,
  gate-enforcement logic, **and the eval metric / definition of "done"** (keeping this out of agent hands is
  what blocks metric-gaming). Written as **explanation, not a rule-list** (Claude Jan-2026 style), so case-law
  is checkable for *derivability* from it.
- **Case-law (agent-proposes, human-approves):** heuristics, mode thresholds, subagent prompts, learned
  lessons — the markdown/config surface.

**Amendment pipeline:** reflection emits a case-law *diff* (add/supersede/merge/retire — never spine edits)
→ **self-critique gate** (auto-reject if it contradicts the constitution or touches a gate) → **grounding
gate** (cite ≥2 episodes; 1 = "candidate" only) → **regression gate** (no regression on a *frozen* baseline
of past tasks the agent didn't author; human is the metric where no number exists) → **human approval = git
commit** (`git revert` = rollback). Instrument a **gate-bypass counter** as the early gaming signal.

**Generalization filter** (accept a lesson only if all four hold): (a) cross-context transfer — would it have
helped on a *different* project? (b) no bound specifics — names no file/repo/literal value; (c) evidence floor
— ≥2 distinct episodes; (d) methodology form — phrased as mindset/heuristic, not task-instruction.
**Implementation = a two-column rewrite** ("specific observation" → "general principle"), **commit only the
general column**; the specific stays in the episodic log.

**Decay/cadence:** hard token budget on active case-law (forces competition); confidence × recency × usage
ranks lessons; uncorroborated lessons decay to candidate then retire (git retains them). Per-task reflection +
periodic **constitutional review** (prune drift, check the bypass-counter trend, decide if a
repeatedly-proposed pattern earns human-only promotion *into* the constitution).

**Watch:** the *vacuous-generalization* failure — lessons so general they pass the filter but carry no signal.
The evidence floor is the guard; may need an explicit anti-vacuity check.

---

## 10. Pi mapping (from T6) — adopt vs build

**Pi gives us (adopt as-is):** multi-provider model layer + per-role selection + cross-provider handoff +
local-endpoint (oMLX/vLLM) support; the streaming agent runtime (parallel/sequential tools, steering, abort,
thinking levels); **tool-call gating** (`tool_call` → `{block, reason}`); context/compaction hooks; SKILL.md
(loads `~/.claude/skills`); subagents (single/parallel/chain, as a forkable example extension); JSONL **tree**
sessions with branching/labels/custom-entries; a very wide `ExtensionAPI`.

**We build:** the **board/state-machine** (extension holding phase state, gating tools per phase, durable via
`appendEntry`/`custom` entries + file board); the **workpad** (session custom entries + fs convention); the
**memory plane** (Pi has no semantic cross-session index); **per-subagent worktree isolation** (wrap the
subagent launcher — Pi shares cwd); coordination beyond chain/parallel (a real board-driven team).

**The gate mechanism (verified in docs):** `pi.on("tool_call", …)` runs **first-block-wins, early-exit**, can
**block** execution tools and **mutate `event.input`** or `await ctx.ui.confirm()`. Align gate = hold an
`aligned`/phase flag in extension closure (restored from session `custom` entries on reload), block
`bash/write/edit` until a `/align` command flips it; harden with `setActiveTools([…read-only…])` during the
align phase so execution tools aren't even offered. This is the single most important confirmation: **our
core enforcement primitive exists and is exactly the right shape.**

**Adoption shape:** one installable Pi extension (or a small set) that registers (1) the phase/Align gate on
`tool_call`; (2) board/phase commands; (3) custom tools (workpad, memory query); (4) a forked subagent
extension with worktree isolation + model-per-role; (5) our oMLX/local providers.

---

## 11. Cross-cutting principles (what the whole sweep agreed on)

1. **Gate by a number or an artifact, never a vibe.** Mutation score, proof artifacts, mechanical checklists,
   pairwise knockout, gate-bypass counter. Every track landed here independently. It's the spine's organizing
   principle and the antidote to "theater."
2. **The Align criteria are the keystone.** They are the Verify oracle, the P4 pruning function, *and* the
   grounding for reflection. P1 is not one pillar of five — three others are functions of it. Invest there.
3. **One shape recurs across all three planes:** episodic-decays / semantic-persists (memory) = churn-discarded
   / output-promoted (context) = case-law / constitution (evolution). Build the "raw → distilled → promoted,
   verification-gated" pipeline once; reuse it three times.

---

## 12. Build order — dogfooded via branch/promote

1. **Trunk (the spine):** the Pi extension scaffold + **Align gate** (`tool_call` block) + **file board** +
   **workpad** + **state machine** + **land** + **worktree-isolated workers**. This is the minimum that makes
   the harness usable on itself. Memory plane v0 alongside (capture hooks → sidecar; manual wiki).
2. **Pillars as feature branches that promote in:** P0 auto-context priming → memory retrieval/reflection
   (P3/P5 engine) → Harden gate (P-Verify) → P4 parallel exploration → P5 constitution/case-law self-evolution.
3. **Validation = dogfooding:** if branch/promote can't comfortably build its own pillars without polluting
   main, the model is wrong and we feel it immediately.

---

## 13. Remaining build risk → targeted deep dives

Breadth resolved the *design*. The remaining risk is *mechanical*, concentrated in two repos to clone and
take apart, plus per-track prototyping spikes folded into the build.

| Priority | Deep dive | What to verify (by cloning) | Why it's risk |
|---|---|---|---|
| **P0 — ✅ DONE** (07-pi-spike.md) | **Clone `badlogic/pi-mono`** | RESOLVED by running it: gate blocks at runtime; batch atomicity is a non-issue (per-tool preflight); worktree injection is trivial (`cwd` already a spawn param); custom entries persist across reload. EXTEND-pi adoption layer confirmed. | ~~architecture sits on Pi behaving as documented~~ — verified, no surprises. |
| **P1 — ✅ DONE** (08-symphony-teardown.md) | **Clone `openai/symphony`** | RESOLVED: orchestrator is one GenServer (`orchestrator.ex`); claim-in-same-turn (no lock), per-dispatch revalidation, stall=`now−last>300s`+backoff, land=exit-code contract in `land_watch.py`, clean `Tracker` behaviour seam (swap in file board). Zero persistence — durability is ours to add. | de-risked; mechanics in hand. |
| **P2 — medium** | Memory spike on oMLX | embedding throughput for continuous capture; local cross-encoder reranker vs RRF; tune λ + retrieval weights on our own traffic | Defaults are fine for v1; tuning is empirical, not blocking. |
| **P2 — medium** | Self-evolution regression gate | how to hold a *frozen baseline* of past task transcripts as the held-out set; is "no regression" LLM-judgeable cheaply per reflection? | The one under-specified piece of P5. |
| **P3 — low** | Mutation/metamorphic tooling spike | mutant-set sizing for the LLM-mutator; minimal reusable MR library; equivalence-judge cost at our scale | Folds into the Harden-gate build. |

---

## 14. Open design forks for Gary (genuine, not research-resolved)

1. **Board format:** markdown files **[recommended]** (human/AI co-edit, git-native, Obsidian) vs SQLite. I
   lean: board+workpad = markdown; sidecar memory = SQLite+vectors. Same wiki/sidecar split, applied to work.
2. **First buildable slice:** the trunk in §12.1 is a lot. Do we (a) build the *whole* trunk before dogfooding,
   or (b) ship a thinner "Align gate + workpad + manual board" first and add worktree isolation / land once it's
   carrying real tickets? I lean (b) — earliest dogfood.
3. **Headless/server:** TUI-only for v1 **[recommended]**, or design the SDK/RPC seam in from the start? I lean
   TUI-only; revisit when a cron/remote use-case is real.
